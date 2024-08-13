
```dataviewjs
let filteredPages = dv.pages("#Financial_accounting/Expense").filter(p => p["Status"] !== false && !p.file.name.includes("Template"));

let results = [];


let today = new Date();


function formatNumber(num) {
    const parts = num.toFixed(2).split('.'); 
    const integerPart = parts[0].replace(/\B(?=(\d{3})+(?!\d))/g, ' '); 
    const decimalPart = parts[1] || '00'; 
    return `${integerPart},${decimalPart}`;
}


async function processAccount(page) {
    const accountName = page.file.name;
    let totalExpense = 0;


    let totalExpenseThisMonth = 0;
    let totalExpenseLastMonth = 0;
    let totalExpenseLast3Months = 0;
    let totalExpenseLastYear = 0;


    const firstDayOfThisMonth = new Date(today.getFullYear(), today.getMonth(), 1);
    const firstDayOfLastMonth = new Date(today.getFullYear(), today.getMonth() - 1, 1);
    const lastDayOfLastMonth = new Date(today.getFullYear(), today.getMonth(), 0); 
    const firstDayOfLast3Months = new Date(today.getFullYear(), today.getMonth() - 3, 1);
    const lastDayOfLast3Months = new Date(today.getFullYear(), today.getMonth(), 0); 
    const firstDayOfLastYear = new Date(today.getFullYear() - 1, today.getMonth(), 1);
    const lastDayOfLastYear = new Date(today.getFullYear(), today.getMonth(), 0); 


    const transactions = dv.pages().where(p => p.chronicle_date);

    for (const transPage of transactions) {
        const fileContent = await dv.io.load(transPage.file.path);
        const lines = fileContent.split("\n");
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

            const expenseCategory = parts[2]; 
            const amountOut = parseFloat(parts[4]) || parseFloat(parts[3]); 
            const chronicleDate = new Date(transPage.chronicle_date); 


            if (expenseCategory === accountName) {
                totalExpense += amountOut;

                if (chronicleDate >= firstDayOfThisMonth && chronicleDate <= today) {
                    totalExpenseThisMonth += amountOut;
                }
                if (chronicleDate >= firstDayOfLastMonth && chronicleDate <= lastDayOfLastMonth) {
                    totalExpenseLastMonth += amountOut;
                }
                if (chronicleDate >= firstDayOfLast3Months && chronicleDate <= lastDayOfLast3Months) {
                    totalExpenseLast3Months += amountOut;
                }
                if (chronicleDate >= firstDayOfLastYear && chronicleDate <= lastDayOfLastYear) {
                    totalExpenseLastYear += amountOut;
                }
            }
        }
    }


    results.push({
        accountName: accountName,
        totalExpense: totalExpense,
        expenseThisMonth: totalExpenseThisMonth,
        expenseLastMonth: totalExpenseLastMonth,
        avgExpenseLast3Months: totalExpenseLast3Months > 0 ? totalExpenseLast3Months / 3 : 0,
        avgExpenseLastYear: totalExpenseLastYear > 0 ? totalExpenseLastYear / 12 : 0
    });
}


await Promise.all(filteredPages.map(processAccount));


results.sort((a, b) => b.avgExpenseLastYear - a.avgExpenseLastYear);


dv.header(2, "General indicators");
dv.paragraph(`- **Total expenses for this month**: ${formatNumber(results.reduce((total, r) => total + r.expenseThisMonth, 0))}`);
dv.paragraph(`- **Total expenses for last month**: ${formatNumber(results.reduce((total, r) => total + r.expenseLastMonth, 0))}`);
dv.paragraph(`- **Average expenses for the last 3 months**: ${results.length > 0 ? formatNumber(results.reduce((total, r) => total + r.avgExpenseLast3Months, 0)) : 0}`);
dv.paragraph(`- **Average expenses for the year**: ${results.length > 0 ? formatNumber(results.reduce((total, r) => total + r.avgExpenseLastYear, 0)) : 0}`);

dv.header(2, "Summary by expense categories");
if (results.length > 0) {
    dv.table(
        ["Account", "Total expenses", "Expenses for the current month", "Expenses for the previous month", "Average expenses for the last 3 months", "Average expenditure over the last year"],
        results.map(r => [
            `[[${r.accountName}]]`, 
            formatNumber(r.totalExpense),
            formatNumber(r.expenseThisMonth),
            formatNumber(r.expenseLastMonth),
            formatNumber(r.avgExpenseLast3Months),
            formatNumber(r.avgExpenseLastYear)
        ])
    );
} else {
    dv.paragraph(`- **No accounts to display.**`);
}

```

