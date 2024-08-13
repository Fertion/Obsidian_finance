
```dataviewjs
let today = new Date();

let startDate = new Date(today.getFullYear(), today.getMonth() - 11, 1);
let endDate = new Date(today.getFullYear(), today.getMonth() + 1, 0);

function formatNumber(num) {
    const parts = num.toFixed(2).split('.');
    const integerPart = parts[0].replace(/\B(?=(\d{3})+(?!\d))/g, ' ');
    const decimalPart = parts[1] || '00';
    return `${integerPart},${decimalPart}`;
}

async function getFinancialData(tag, isIncome) {
    const totalsByMonth = new Array(12).fill(0);
    
    const filteredPages = dv.pages(tag).filter(p => p["Status"] !== false && !p.file.name.includes("Template"));

    const transactions = dv.pages().where(p => p.chronicle_date);
    for (const page of filteredPages) {
        const accountName = page.file.name;

        const transactionData = await Promise.all(transactions.map(transPage => dv.io.load(transPage.file.path)));
        
        for (let i = 0; i < transactionData.length; i++) {
            const lines = transactionData[i].split("\n");
            let foundFinancialSection = false;
            const chronicleDate = new Date(transactions[i].chronicle_date); 

            if (chronicleDate < startDate || chronicleDate > endDate) continue;

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

                const category = isIncome ? parts[1] : parts[2]; 
                const amount = parseFloat(parts[4]) || parseFloat(parts[3]); 

                if (category === accountName) {
                    const monthIndex = (chronicleDate.getFullYear() - startDate.getFullYear()) * 12 + (chronicleDate.getMonth() - startDate.getMonth());
                    totalsByMonth[monthIndex] += amount; 
                }
            }
        }
    }

    return totalsByMonth;
}

const incomeTotalsByMonth = await getFinancialData("#Financial_accounting/Income", true);
const expenseTotalsByMonth = await getFinancialData("#Financial_accounting/Expense", false);

const monthNames = [];
for (let i = 0; i < 12; i++) {
    const date = new Date(today.getFullYear(), today.getMonth() - i, 1);
    monthNames.push(date.toLocaleString('en-US', { month: 'long', year: 'numeric' })); 
}

const differenceByMonth = incomeTotalsByMonth.map((inc, index) => inc - expenseTotalsByMonth[index]);

const reversedIncomeTotals = incomeTotalsByMonth.reverse();
const reversedExpenseTotals = expenseTotalsByMonth.reverse();
const reversedDifferenceTotals = differenceByMonth.reverse();

dv.table(
    ["Category", ...monthNames], 
    [
        ["Income", ...reversedIncomeTotals.map(formatNumber)], 
        ["Expenses", ...reversedExpenseTotals.map(formatNumber)], 
        ["Difference", ...reversedDifferenceTotals.map(formatNumber)] 
    ]
);

```
