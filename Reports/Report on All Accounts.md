
```dataviewjs
function formatNumber(num) {

    let formatted = parseFloat(num).toFixed(2).toString().replace(/\B(?=(\d{3})+(?!\d))/g, " ");
    return formatted === "-0.00" ? "0.00" : formatted; 
}

let filteredPages = dv.pages("#Financial_accounting/Accounts").filter(p => p["Status"] !== false && !p.file.name.includes("Template"));


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


const transactions = await Promise.all(dv.pages().where(p => p.chronicle_date).map(async transPage => {
    const fileContent = await dv.io.load(transPage.file.path);
    return fileContent.split("\n").map(line => line.trim()).filter(line => line.startsWith("💱 |"));
}));


for (const page of filteredPages) {
    const accountName = page.file.name;
    const currency = page["Account currency"] || 'currency not specified';
    const creditLimit = parseFloat(page["Credit limit"]) || 0;
    const isLiquid = page["Liquid"] !== false;

    let totalDeposits = 0, totalWithdrawals = 0;
    

    for (const trans of transactions.flat()) {
        const parts = trans.split("|").map(part => part.trim());
        
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


    if (finalBalance < 0 && creditLimit > 0) {
        totals.totalSpentCreditLimit += Math.abs(finalBalance);
    }
}


let totalUnusedCreditLimit = totals.totalCreditLimit - totals.totalSpentCreditLimit;

dv.header(1, "Balance");
dv.paragraph(`- **Total Actual Balance**: ${formatNumber(totals.totalFinalBalance)} USD`);
dv.paragraph(`- **Total Actual Balance (excluding illiquid assets)**: ${formatNumber(totals.totalFinalBalanceLiquid)} USD`);
dv.paragraph(`- **Total Available Balance**: ${formatNumber(totals.totalAvailableBalance)} USD`);

dv.header(1, "Assets and Liabilities");
dv.paragraph(`- **Total Assets**: ${formatNumber(totals.totalAssets)} USD`);
dv.paragraph(`- **Total Liquid Assets**: ${formatNumber(totals.totalLiquidAssets)} USD`);
dv.paragraph(`- **Total Liabilities**: ${formatNumber(totals.totalLiabilities)} USD`);

dv.header(1, "Credit Cards");
dv.paragraph(`- **Available Credit Limit**: ${formatNumber(totals.totalCreditLimit)} USD`);
dv.paragraph(`- **Used Credit Limit**: ${formatNumber(totals.totalSpentCreditLimit)} USD`);
dv.paragraph(`- **Unused Credit Limit**: ${formatNumber(totalUnusedCreditLimit)} USD`);


results.sort((a, b) => b.availableBalance - a.availableBalance);


if (results.length > 0) {
    dv.table(["Account", "Actual Balance", "Available Balance", "Credit Limit"], 
        results.map(r => [
            `[[${r.accountName}]]`, 
            `${formatNumber(r.finalBalance)} ${r.currency}`, 
            `${formatNumber(r.availableBalance)} ${r.currency}`, 
            `${formatNumber(r.creditLimit)} ${r.currency}`
        ])
    );
} else {
    dv.paragraph(`- **No accounts to display.**`);
}

```

