---
created: 2024.08.13 20:59:11
updated: 2024.08.13 20:59:11
tags:
  - Financial_accounting/Expense
Status: true
Budget: "5000"
Budget Category: Fixed monthly expenses
MOC: "[[Financial Accounting. Expense Categories]]"
Limit: 15
---
# General indicators
```dataviewjs
let accountName = dv.current().file.name; 
let totalExpenses = 0;
let totalExpensesThisMonth = 0;
let totalExpensesLastMonth = 0;
let totalExpensesLast3Months = 0;
let totalExpensesLastYear = 0;


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
    return isNaN(num) ? '0.00' : num.toFixed(2).replace(/\B(?=(\d{3})+(?!\d))/g, " ").replace('.', ',');
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

        const parts = line.split("|").map(part => part.trim());

        
        if (parts.length >= 5) {
            const expenseCategory = parts[2]; 
            const amountOut = parseFloat(parts[4]) || parseFloat(parts[3]); 
            const chronicleDate = new Date(page.chronicle_date); 


            if (expenseCategory === accountName) {
                totalExpenses += amountOut; 


                if (chronicleDate >= firstDayOfThisMonth && chronicleDate <= today) {
                    totalExpensesThisMonth += amountOut; 
                }


                if (chronicleDate >= firstDayOfLastMonth && chronicleDate <= lastDayOfLastMonth) {
                    totalExpensesLastMonth += amountOut; 
                }


                if (chronicleDate >= firstDayOfLast3Months && chronicleDate <= lastDayOfLast3Months) {
                    totalExpensesLast3Months += amountOut; 
                }


                if (chronicleDate >= firstDayOfLastYear && chronicleDate <= lastDayOfLastYear) {
                    totalExpensesLastYear += amountOut; 
                }
            }
        }
    }
}


await Promise.all(pages.map(processPage));


let averageExpensesLast3Months = totalExpensesLast3Months > 0 ? (totalExpensesLast3Months / 3) : 0; 
let averageExpensesLastYear = totalExpensesLastYear > 0 ? (totalExpensesLastYear / 12) : 0;


dv.paragraph(`- **Total expenses for the whole period**: ${formatNumber(totalExpenses)}`);
dv.paragraph(`- **The amount of expenses for this month:** ${formatNumber(totalExpensesThisMonth)}`);
dv.paragraph(`- **Amount of expenses for the previous month:** ${formatNumber(totalExpensesLastMonth)}`);
dv.paragraph(`- **Amount of expenses for the last 3 months:** ${formatNumber(totalExpensesLast3Months)}`);
dv.paragraph(`- **Average expenses for the last 3 months:** ${formatNumber(averageExpensesLast3Months)}`);
dv.paragraph(`- **Amount of expenses for the last year:** ${formatNumber(totalExpensesLastYear)}`);
dv.paragraph(`- **Average expenditure over the last year:** ${formatNumber(averageExpensesLastYear)}`);

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
