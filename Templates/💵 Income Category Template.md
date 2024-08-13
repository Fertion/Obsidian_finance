---
created: <% tp.file.creation_date("yyyy.MM.DD HH:mm:ss") %>
updated: <% tp.file.creation_date("yyyy.MM.DD HH:mm:ss") %>
tags:
  - #Financial_accounting/Income
Status: true
MOC: "[[Financial Accounting. Income Categories]]"
Limit: 15
---
# General indicators
```dataviewjs
let accountName = dv.current().file.name; 
let totalIncome = 0;
let totalIncomeThisMonth = 0;
let totalIncomeLastMonth = 0;
let totalIncomeLast3Months = 0;
let totalIncomeLastYear = 0;


let today = new Date();
let firstDayOfThisMonth = new Date(today.getFullYear(), today.getMonth(), 1);
let firstDayOfLastMonth = new Date(today.getFullYear(), today.getMonth() - 1, 1);
let lastDayOfLastMonth = new Date(today.getFullYear(), today.getMonth(), 0); 
let firstDayOfLast3Months = new Date(today.getFullYear(), today.getMonth() - 3, 1);
let lastDayOfLast3Months = new Date(today.getFullYear(), today.getMonth(), 0); 
let firstDayOfLastYear = new Date(today.getFullYear() - 1, today.getMonth(), 1);
let lastDayOfLastYear = new Date(today.getFullYear(), today.getMonth(), 0); 


let pages = dv.pages().where(p => p.chronicle_date);


function formatNumber(num) {
    if (isNaN(num)) return '0.00'; 
    return num.toFixed(2).replace(/\B(?=(\d{3})+(?!\d))/g, " ").replace('.', ',');
}


async function processPage(page) {
    const fileContent = await dv.io.load(page.file.path);
    const lines = fileContent.split("\n");
    let foundFinancialSection = false;

    for (let line of lines) {

        if (line.trim() === "# Financial Accounting") {
            foundFinancialSection = true;
            continue;
        }


        if (!foundFinancialSection || !line.trim().startsWith("💱 |")) {
            continue;
        }

        let parts = line.split("|").map(part => part.trim());

        
        if (parts.length >= 5) {
            let incomeCategory = parts[1];  
            let amountIn = parseFloat(parts[4]) || parseFloat(parts[3]); 
            let chronicleDate = new Date(page.chronicle_date); 


            if (incomeCategory === accountName) {
                totalIncome += amountIn; 


                if (chronicleDate >= firstDayOfThisMonth && chronicleDate <= today) {
                    totalIncomeThisMonth += amountIn; 
                }


                if (chronicleDate >= firstDayOfLastMonth && chronicleDate <= lastDayOfLastMonth) {
                    totalIncomeLastMonth += amountIn; 
                }


                if (chronicleDate >= firstDayOfLast3Months && chronicleDate <= lastDayOfLast3Months) {
                    totalIncomeLast3Months += amountIn; 
                }


                if (chronicleDate >= firstDayOfLastYear && chronicleDate <= lastDayOfLastYear) {
                    totalIncomeLastYear += amountIn; 
                }
            }
        }
    }
}


await Promise.all(pages.map(processPage));


let averageIncomeLast3Months = totalIncomeLast3Months > 0 ? (totalIncomeLast3Months / 3) : 0; 
let averageIncomeLastYear = totalIncomeLastYear > 0 ? (totalIncomeLastYear / 12) : 0;


dv.paragraph(`- **Amount of income for the whole period**: ${formatNumber(totalIncome)}`);
dv.paragraph(`- **The amount of income for that month:** ${formatNumber(totalIncomeThisMonth)}`);
dv.paragraph(`- **Amount of income for the previous month:** ${formatNumber(totalIncomeLastMonth)}`);
dv.paragraph(`- **Amount of income for the last 3 months:** ${formatNumber(totalIncomeLast3Months)}`);
dv.paragraph(`- **Average income for the last 3 months:** ${formatNumber(averageIncomeLast3Months)}`);
dv.paragraph(`- **Amount of income for the last year:** ${formatNumber(totalIncomeLastYear)}`);
dv.paragraph(`- **Average income for the last year:** ${formatNumber(averageIncomeLastYear)}`);

```
# Operations
```dataviewjs
let accountName = dv.current().file.name; 
let limit = dv.current().Limit || 30; 
let pages = dv.pages().where(p => p.chronicle_date); 


let operations = [];


function processPage(page) {
    return dv.io.load(page.file.path).then(fileContent => {
        let lines = fileContent.split("\n");
        let foundFinancialSection = false;

        for (let line of lines) {

            if (line.trim() === "# Financial Accounting") {
                foundFinancialSection = true;
                continue;
            }


            if (!foundFinancialSection || !line.trim().startsWith("💱 |")) {
                continue;
            }

            let parts = line.split("|").map(part => part.trim());

            
            if (parts.length >= 5) {
                let fromAccount = parts[1];  
                let toAccount = parts[2];     


                let amountOut = parseFloat(parts[3]) || 0; 
                let amountIn = parseFloat(parts[4]) || amountOut; 


                if (fromAccount === accountName || toAccount === accountName) {

                    let fileLink = `[[${page.file.name}]]`; 
                    operations.push({
                        from: fromAccount,
                        to: toAccount,
                        amountOut,
                        amountIn,
                        line: `${line} ${fileLink}`, 
                        fileName: page.file.name, 
                    });
                }
            }
        }
    });
}


Promise.all(pages.map(processPage)).then(() => {

    operations.sort((a, b) => {
        return b.fileName.localeCompare(a.fileName); 
    });


    let limitedOperations = operations.slice(0, limit);


    if (limitedOperations.length > 0) {
        limitedOperations.forEach(op => {
            dv.paragraph(`  - ${op.line}`);
        });
    } else {
        dv.paragraph(`- **No transactions on the account "${accountName}".**`);
    }
});

```
