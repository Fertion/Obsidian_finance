---
created: 2024.08.13 20:58:59
updated: 2024.08.13 20:58:59
tags:
  - Financial_accounting/Accounts
Status: true
Liquid: true
Account currency: USD
Credit limit: 
MOC: "[[Financial Accounting. Accounts]]"
Limit: 15
---
# Balance
```dataviewjs
let accountName = dv.current().file.name; 
let totalDeposits = 0; 
let totalWithdrawals = 0;


let pages = dv.pages().where(p => p.chronicle_date);


function formatNumberWithSpaces(num) {
    return num.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ' ');
}


const page = dv.current();
const currency = page["Account currency"] || 'currency not specified'; 
const creditLimit = parseFloat(page["Credit limit"]) || 0;


async function processPages() {
    const transactionPromises = pages.map(async (page) => {
        const fileContent = await dv.io.load(page.file.path);
        const lines = fileContent.split("\n");
        let foundFinancialSection = false;

        for (let line of lines) {
            if (line.trim() === "# Financial Accounting") {
                foundFinancialSection = true;
                continue;
            }

            if (foundFinancialSection && line.trim().startsWith("💱 |")) {
                const parts = line.split("|").map(part => part.trim());
                
                
                if (parts.length >= 5) {
                    const fromAccount = parts[1];  
                    const toAccount = parts[2];     
                    const amountOut = parseFloat(parts[3]) || 0; 
                    const amountIn = parseFloat(parts[4]) || amountOut; 

                    if (fromAccount === accountName) {
                        totalWithdrawals += amountOut; 
                    } else if (toAccount === accountName) {
                        totalDeposits += amountIn; 
                    }
                }
            }
        }
    });

    await Promise.all(transactionPromises);
}


processPages().then(() => {
    const finalBalance = (totalDeposits - totalWithdrawals).toFixed(2);
    const availableBalance = (parseFloat(finalBalance) + creditLimit).toFixed(2);

    
    dv.paragraph(`- **Actual Balance**: ${formatNumberWithSpaces(finalBalance)} ${currency}`);
    dv.paragraph(`- **Available Balance (including credit limits)**: ${formatNumberWithSpaces(availableBalance)} ${currency}`);
});

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






