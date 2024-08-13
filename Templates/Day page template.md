---
created: <% tp.file.creation_date("yyyy.MM.DD HH:mm:ss") %>
updated: <% tp.file.creation_date("yyyy.MM.DD HH:mm:ss") %>
chronicle_date: <% tp.file.title.replace(/\./g, '-') %>
tags:
  - Daily_Note
---
**<< [Previous day](<% tp.date.now("YYYY.MM.DD", -1, tp.file.title, "YYYY.MM.DD") %>) | [Next day](<% tp.date.now("YYYY.MM.DD", 1, tp.file.title, "YYYY.MM.DD") %>) >>**
___

# Financial Accounting
```button
name New transaction
type append template
action Financial Accounting Template
```
