<%*
const modalForm = app.plugins.plugins.modalforms.api;
const result = await modalForm.openForm('TransactionForm');

const from = result.getValue('from') || '';
const to = result.getValue('to') || '';
const amountOut = result.getValue('amount_out') ? result.getValue('amount_out') : '';
const amountIn = result.getValue('amount_in') ? result.getValue('amount_in') : '';
const comment = result.getValue('comment') || ''; 


const safeAmountIn = amountIn || '-';
const safeComment = comment || '-';


let outputString = '';

if (from) outputString += `💱 | ${from}`;
if (to) outputString += ` | ${to}`;
if (amountOut) outputString += ` | ${amountOut}`;
outputString += ` | ${safeAmountIn}`; 
outputString += ` | ${safeComment}`; 

tR += outputString;
-%>