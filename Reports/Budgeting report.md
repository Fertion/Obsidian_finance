
```dataviewjs
const pages = dv.pages("#Financial_accounting/Expense");

function formatNumber(num) {
    const parts = num.toFixed(2).split('.');
    const integerPart = parts[0].replace(/\B(?=(\d{3})+(?!\d))/g, ' ');
    const decimalPart = parts[1] || '00';
    return `${integerPart},${decimalPart}`;
}

const transactions = dv.pages().where(p => p.chronicle_date);
const transactionData = await Promise.all(transactions.map(transPage => dv.io.load(transPage.file.path)));

const today = new Date();
const firstDayOfThisMonth = new Date(today.getFullYear(), today.getMonth(), 1);
const lastDayOfThisMonth = new Date(today.getFullYear(), today.getMonth() + 1, 0);
const daysInRemainingMonth = lastDayOfThisMonth.getDate() - today.getDate() + 1; 
const daysInCurrentMonth = lastDayOfThisMonth.getDate(); 
const daysPassed = today.getDate() - 1; 


async function calculateExpenses(categoryFilter) {
    let totalExpensesThisMonth = 0;
    let plannedExpenses = 0;


    const filteredPages = pages.filter(page => page["Status"] !== false && page["Budget Category"] === categoryFilter);

    for (const page of filteredPages) {
        const accountName = page.file.name;
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

                    if (expenseCategory === accountName && chronicleDate >= firstDayOfThisMonth && chronicleDate <= lastDayOfThisMonth) {
                        totalExpensesThisMonth += amountOut;
                    }
                }
            }
        });
    }

    return { totalExpensesThisMonth, plannedExpenses };
}


const { totalExpensesThisMonth: totalFixedExpenseThisMonth, plannedExpenses: plannedFixedExpenses } =
    await calculateExpenses("Fixed monthly expenses");

const { totalExpensesThisMonth: totalVariableExpenseThisMonth, plannedExpenses: plannedVariableExpenses } =
    await calculateExpenses("Non-fixed costs");


const remainingFixedExpenses = plannedFixedExpenses - totalFixedExpenseThisMonth; 
const remainingVariableExpenses = plannedVariableExpenses - totalVariableExpenseThisMonth; 
const dailySpendingFixed = remainingFixedExpenses / daysInRemainingMonth; 
const dailySpendingVariable = remainingVariableExpenses / daysInRemainingMonth; 
const initialDailySpendingFixed = plannedFixedExpenses / daysInCurrentMonth; 
const initialDailySpendingVariable = plannedVariableExpenses / daysInCurrentMonth; 


const overBudgetFixed = initialDailySpendingFixed * daysPassed; 
const overBudgetVariable = initialDailySpendingVariable * daysPassed; 


dv.table(
    ["Budgeting", "This month's expenses", "Planned expenses", "Remaining for this month", "Adjusted daily spending", "Initial daily spending", "Over budget today"],
    [
        ["Fixed monthly expenses", formatNumber(totalFixedExpenseThisMonth), formatNumber(plannedFixedExpenses), formatNumber(remainingFixedExpenses), formatNumber(dailySpendingFixed), formatNumber(initialDailySpendingFixed), formatNumber(totalFixedExpenseThisMonth - overBudgetFixed)],
        ["Non-fixed costs", formatNumber(totalVariableExpenseThisMonth), formatNumber(plannedVariableExpenses), formatNumber(remainingVariableExpenses), formatNumber(dailySpendingVariable), formatNumber(initialDailySpendingVariable), formatNumber(totalVariableExpenseThisMonth - overBudgetVariable)],
        ["Total", formatNumber(totalFixedExpenseThisMonth + totalVariableExpenseThisMonth), formatNumber(plannedFixedExpenses + plannedVariableExpenses), formatNumber((plannedFixedExpenses + plannedVariableExpenses) - (totalFixedExpenseThisMonth + totalVariableExpenseThisMonth)), formatNumber((dailySpendingFixed + dailySpendingVariable)), formatNumber((initialDailySpendingFixed + initialDailySpendingVariable)), formatNumber((totalFixedExpenseThisMonth + totalVariableExpenseThisMonth) - (overBudgetFixed + overBudgetVariable))]
    ]
);

```



