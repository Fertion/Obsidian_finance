
```dataviewjs
let filteredPages = dv.pages("#Financial_accounting/Income").filter(p => p["Status"] !== false && !p.file.name.includes("Template"));

let results = [];


let today = new Date();


function formatNumber(num) {
    const parts = num.toFixed(2).split('.');
    const integerPart = parts[0].replace(/\B(?=(\d{3})+(?!\d))/g, ' ');
    const decimalPart = parts[1] || '00';
    return `${integerPart},${decimalPart}`;
}


const transactions = dv.pages().where(p => p.chronicle_date);
let transactionData = await Promise.all(transactions.map(transPage => dv.io.load(transPage.file.path)));


for (const page of filteredPages) {
    const accountName = page.file.name;
    let totalIncome = 0;
    let totalIncomeThisMonth = 0;
    let totalIncomeLastMonth = 0;
    let totalIncomeLast3Months = 0;
    let totalIncomeLastYear = 0;


    const firstDayOfThisMonth = new Date(today.getFullYear(), today.getMonth(), 1);
    const firstDayOfLastMonth = new Date(today.getFullYear(), today.getMonth() - 1, 1);
    const lastDayOfLastMonth = new Date(today.getFullYear(), today.getMonth(), 0);
    const firstDayOfLast3Months = new Date(today.getFullYear(), today.getMonth() - 3, 1);
    const lastDayOfLast3Months = new Date(today.getFullYear(), today.getMonth(), 0);
    const firstDayOfLastYear = new Date(today.getFullYear() - 1, today.getMonth(), 1);
    const lastDayOfLastYear = new Date(today.getFullYear(), today.getMonth(), 0);

    for (let i = 0; i < transactionData.length; i++) {
        const lines = transactionData[i].split("\n");
        let foundFinancialSection = false;

        for (const line of lines) {
            if (line.trim() === "# Financial Accounting") {
                foundFinancialSection = true;
                continue;
            }

            if (!foundFinancialSection || !line.trim().startsWith("💱 |")) {
                continue; 
            }

            const parts = line.split("|").map(part => part.trim());
            if (parts.length < 5) continue; 

            const incomeCategory = parts[1]; 
            const amountIn = parseFloat(parts[4]) || parseFloat(parts[3]); 
            const chronicleDate = new Date(transactions[i].chronicle_date); 


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

    results.push({
        accountName: accountName,
        totalIncome: totalIncome,
        incomeThisMonth: totalIncomeThisMonth,
        incomeLastMonth: totalIncomeLastMonth,
        avgIncomeLast3Months: totalIncomeLast3Months > 0 ? totalIncomeLast3Months / 3 : 0,
        avgIncomeLastYear: totalIncomeLastYear > 0 ? totalIncomeLastYear / 12 : 0
    });
}

results.sort((a, b) => b.avgIncomeLastYear - a.avgIncomeLastYear);
dv.header(2, "Overall Indicators");
dv.paragraph(`- **Total Income for This Month**: ${formatNumber(results.reduce((total, r) => total + r.incomeThisMonth, 0))}`);
dv.paragraph(`- **Total Income for Last Month**: ${formatNumber(results.reduce((total, r) => total + r.incomeLastMonth, 0))}`);
dv.paragraph(`- **Average Income for the Last 3 Months**: ${results.length > 0 ? formatNumber(results.reduce((total, r) => total + r.avgIncomeLast3Months, 0)) : 0}`);
dv.paragraph(`- **Average Income for the Year**: ${results.length > 0 ? formatNumber(results.reduce((total, r) => total + r.avgIncomeLastYear, 0)) : 0}`);

dv.header(2, "Income Category Summary");
if (results.length > 0) {
    dv.table(
        ["Account", "Total Income", "Income This Month", "Income Last Month", "Average Income for the Last 3 Months", "Average Income for the Last Year"],
        results.map(r => [
            `[[${r.accountName}]]`,
            formatNumber(r.totalIncome),
            formatNumber(r.incomeThisMonth),
            formatNumber(r.incomeLastMonth),
            formatNumber(r.avgIncomeLast3Months),
            formatNumber(r.avgIncomeLastYear)
        ])
    );
} else {
    dv.paragraph(`- **No accounts to display.**`);
}


```

 