---
Text search: Pot
Date from: 2024-01-01
Date to: ""
---
> You can enter an account name, category, amount or comment in the "Search" property
---
```dataviewjs
const currentPage = dv.current();
const searchTerm = currentPage["Text search"]?.toLowerCase(); 
const dateFrom = currentPage["Date from"] ? new Date(currentPage["Date from"]) : null; 
const dateTo = currentPage["Date to"] ? new Date(currentPage["Date to"]) : null; 


let results = [];

async function processPage(page) {
    const fileContent = await dv.io.load(page.file.path);
    let lines = fileContent.split("\n");
    let foundFinancialSection = false;

    const chronicleDate = new Date(page.chronicle_date);
    const dateIsInRange = (!dateFrom || chronicleDate >= dateFrom) && (!dateTo || chronicleDate <= dateTo);

    if (!dateIsInRange) {
        return; 
    }

    for (let line of lines) {
        if (line.trim() === "# Financial Accounting") {
            foundFinancialSection = true;
        } else if (foundFinancialSection && line.trim().startsWith("💱 |")) {
            

            if (!searchTerm || line.toLowerCase().includes(searchTerm)) {

                let fileLink = `[[${page.file.name}]]`; 
                results.push({ line: line, fileLink: fileLink });
            }
        }
    }
}


for (let page of dv.pages().where(p => p.chronicle_date)) {
    await processPage(page);
}


results.sort((a, b) => b.fileLink.localeCompare(a.fileLink));
for (let result of results) {
    dv.paragraph(`${result.line} | ${result.fileLink}`);
}

```
---

