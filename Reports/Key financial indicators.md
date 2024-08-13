
```dataviewjs
function formatNumber(num) {
    const parts = num.toFixed(2).split('.');
    const integerPart = parts[0].replace(/\B(?=(\d{3})+(?!\d))/g, ' ');
    const decimalPart = parts[1] || '00';
    return `${integerPart},${decimalPart}`;
}

let filteredPages = dv.pages("#Financial_accounting/Accounts").filter(p => p["Status"] !== false);

const transactions = dv.pages().where(p => p.chronicle_date);
const transactionData = await Promise.all(transactions.map(transPage => dv.io.load(transPage.file.path)));

let results = [];
let totals = {
    totalFinalBalance: 0,
    totalFinalBalanceLiquid: 0,
    totalAvailableBalance: 0,
    totalCreditLimit: 0,
    totalSpentCreditLimit: 0,
    totalAssets: 0,
    totalLiquidAssets: 0,
    totalLiabilities: 0
};

for (const page of filteredPages) {
    const accountName = page.file.name;
    const currency = page["Account currency"] || 'currency not specified';
    const creditLimit = parseFloat(page["Credit limit"]) || 0;
    const isLiquid = page["Liquid"] !== false;

    let totalDeposits = 0, totalWithdrawals = 0;


    transactionData.forEach((fileContent, index) => {
        const lines = fileContent.split("\n");
        let foundFinancialSection = false;

        for (const line of lines) {
            if (line.trim() === "# Financial Accounting") {
                foundFinancialSection = true;
                continue;
            }

            if (foundFinancialSection && line.trim().startsWith("💱 |")) {
                const parts = line.split("|").map(part => part.trim());
                if (parts.length < 5) continue;

                const fromAccount = parts[1];
                const toAccount = parts[2];
                const amountOut = parseFloat(parts[3]) || 0;
                const amountIn = parseFloat(parts[4]) || amountOut;


                if (fromAccount === accountName) {
                    totalWithdrawals += amountOut;
                }
                if (toAccount === accountName) {
                    totalDeposits += amountIn;
                }
            }
        }
    });


    const finalBalance = totalDeposits - totalWithdrawals;

    results.push({
        accountName: accountName,
        finalBalance: finalBalance,
        availableBalance: finalBalance + creditLimit,
        creditLimit: creditLimit,
        currency: currency
    });


    totals.totalFinalBalance += finalBalance;
    totals.totalAvailableBalance += finalBalance + creditLimit;
    totals.totalCreditLimit += creditLimit;

    if (isLiquid) {
        totals.totalFinalBalanceLiquid += finalBalance;
    }

    if (finalBalance > 0) {
        totals.totalAssets += finalBalance;
        if (isLiquid) {
            totals.totalLiquidAssets += finalBalance;
        }
    } else {
        totals.totalLiabilities += finalBalance;
    }


    if (finalBalance < 0) {
        totals.totalSpentCreditLimit += Math.abs(finalBalance);
    }
}


let totalUnusedCreditLimit = totals.totalCreditLimit - totals.totalSpentCreditLimit;


const expensePages = dv.pages("#Financial_accounting/Expense");


const today = new Date();
const firstDayOfThisMonth = new Date(today.getFullYear(), today.getMonth(), 1);
const lastDayOfThisMonth = new Date(today.getFullYear(), today.getMonth() + 1, 0);
const daysInRemainingMonth = lastDayOfThisMonth.getDate() - today.getDate() + 1;


let plannedExpenses = 0;
let totalExpensesThisMonth = 0;

const variableExpensePages = expensePages.filter(p => p["Status"] !== false && p["Budget Category"] === "Non-fixed costs");

for (const page of variableExpensePages) {
    plannedExpenses += parseFloat(page["Budget"]) || 0;


    transactionData.forEach((fileContent, index) => {
        const lines = fileContent.split("\n");
        let foundFinancialSection = false;

        for (const line of lines) {
            if (line.trim() === "# Financial Accounting") {
                foundFinancialSection = true;
                continue;
            }

            if (foundFinancialSection && line.trim().startsWith("💱 |")) {
                const parts = line.split("|").map(part => part.trim());
                if (parts.length < 5) continue;

                const expenseCategory = parts[2];
                const amountOut = parseFloat(parts[4]) || parseFloat(parts[3]);
                const chronicleDate = new Date(transactions[index].chronicle_date);

                if (expenseCategory === page.file.name && chronicleDate >= firstDayOfThisMonth && chronicleDate <= lastDayOfThisMonth) {
                    totalExpensesThisMonth += amountOut;
                }
            }
        }
    });
}

const remainingVariableExpenses = plannedExpenses - totalExpensesThisMonth;
const dailySpendingVariable = remainingVariableExpenses / daysInRemainingMonth;

dv.paragraph(`- **Balance**: ${formatNumber(totals.totalFinalBalanceLiquid)}`);
dv.paragraph(`- **Balance (with illiquid assets)**: ${formatNumber(totals.totalFinalBalance)}`);
dv.paragraph(`- **You can spend in a day**: ${formatNumber(dailySpendingVariable)}`);

```



