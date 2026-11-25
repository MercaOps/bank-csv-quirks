# Bank CSV quirks: how bank and payment exports are really laid out

Column names, delimiters, decimal styles, date orders, sign conventions and gotchas of real
exports from American Express, Interactive Brokers, N26, PayPal, Revolut, Stripe, Wise, Bancolombia, BBVA, CaixaBank, Mercado Pago, Banco Santander, plus field notes on writing QuickBooks **.QBO** (Web Connect) and
**.IIF** (Desktop) files and Alegra imports.

These are the rules the **[MercaOps CSV Cleaner](https://mercaops.com/csv-cleaner/)**
([en español](https://mercaops.com/es/limpiar-csv-bancario/)) applies in your browser to turn messy exports into files
QuickBooks, Xero, YNAB or Excel import on the first try.

## Exports (English pages)

| Export | Delimiter | Decimals | Dates | Signs | Main quirk | Links |
|---|---|---|---|---|---|---|
| American Express | auto-detected | 1,234.56 | month/day/year | money out positive (flipped) | Amex lists charges as positive numbers and payments as negative, the opposite of what QuickBooks expects for a card upload, so purchases land as money received. The preset inverts every sign. | [Clean](https://mercaops.com/csv-cleaner/amex-to-quickbooks/) · [JSON](presets/en/amex-to-quickbooks.json) |
| Interactive Brokers | comma | 1,234.56 | year-month-day | money out negative | Every row starts with its section name and Header or Data, so one CSV holds dozens of tables with different columns; Excel shows it as one jagged sheet. | [Clean](https://mercaops.com/csv-cleaner/interactive-brokers-to-excel/) · [JSON](presets/en/interactive-brokers-to-excel.json) |
| N26 | auto-detected | auto-detected | auto-detected | money out negative | N26 exports carry two dates, Booking Date and Value Date, and YNAB needs one. The preset uses the booking date, the day the transaction hit your account. | [Clean](https://mercaops.com/csv-cleaner/n26-to-ynab/) · [JSON](presets/en/n26-to-ynab.json) |
| PayPal | auto-detected | 1,234.56 | month/day/year | money out negative | Authorization holds and unsettled payments appear with Status «Pending»; importing them counts money that never settled. The preset drops Pending, Denied and Voided rows. | [Clean](https://mercaops.com/csv-cleaner/paypal-to-quickbooks/) · [JSON](presets/en/paypal-to-quickbooks.json) |
| PayPal | auto-detected | 1,234.56 | month/day/year | money out negative | PayPal's Net column is what reaches your PayPal balance. With the gross/fee split on, a Gross line and a Fee line replace each Net line and still add up to it to the cent. | [Clean](https://mercaops.com/csv-cleaner/paypal-to-xero/) · [JSON](presets/en/paypal-to-xero.json) |
| Revolut | auto-detected | 1,234.56 | year-month-day | money out negative | The Type column (CARD_PAYMENT, TOPUP, TRANSFER, EXCHANGE…) carries the context QuickBooks bank rules need, but in shouting case; the preset appends it to the description as readable text. | [Clean](https://mercaops.com/csv-cleaner/revolut-to-quickbooks/) · [JSON](presets/en/revolut-to-quickbooks.json) |
| Revolut | auto-detected | 1,234.56 | year-month-day | money out negative | Card payments that haven't settled are exported with an empty Completed Date and State PENDING; importing them creates transactions that later change or vanish. The preset drops PENDING, REVERTED, DECLINED and FAILED rows. | [Clean](https://mercaops.com/csv-cleaner/revolut-to-ynab/) · [JSON](presets/en/revolut-to-ynab.json) |
| Stripe | auto-detected | 1,234.56 | year-month-day | money out negative | Timestamps are in UTC (2024-04-06 21:47:13), so a late-evening sale in the Americas can land on the next day; the preset keeps the UTC date Stripe reports. | [Clean](https://mercaops.com/csv-cleaner/stripe-to-quickbooks/) · [JSON](presets/en/stripe-to-quickbooks.json) |
| Stripe | auto-detected | 1,234.56 | year-month-day | money out negative | Xero's statement import has a Payee field but Stripe has no payee column; the customer email (or name) is the closest match and helps invoice matching. | [Clean](https://mercaops.com/csv-cleaner/stripe-to-xero/) · [JSON](presets/en/stripe-to-xero.json) |
| Wise | auto-detected | 1,234.56 | day/month/year | money out negative | Every row carries conversion metadata (Exchange From, Exchange To, Exchange Rate) that QuickBooks' 3-column upload can't take; it is dropped. | [Clean](https://mercaops.com/csv-cleaner/wise-to-quickbooks/) · [JSON](presets/en/wise-to-quickbooks.json) |
| Wise | auto-detected | 1,234.56 | day/month/year | money out negative | Xero wants one Payee per line, but Wise spreads it over Payee Name (outgoing), Payer Name (incoming) and Merchant (card). The preset picks the filled one on each row. | [Clean](https://mercaops.com/csv-cleaner/wise-to-xero/) · [JSON](presets/en/wise-to-xero.json) |

## Exportaciones (páginas en español)

| Export | Delimiter | Decimals | Dates | Signs | Main quirk | Links |
|---|---|---|---|---|---|---|
| Bancolombia | semicolon | 1.234,56 | day/month/year | money out negative | Si guardas el XLS como CSV desde un Excel en español, sale con punto y coma y en ANSI (Windows-1252): «DESCRIPCIÓN» aparece como «DESCRIPCI�N» en otros programas. Con el XLS original no pasa, y con el CSV el ajuste recupera las tildes. | [Limpiar](https://mercaops.com/es/limpiar-csv-bancario/bancolombia-a-excel/) · [JSON](presets/es/bancolombia-a-excel.json) |
| BBVA | semicolon | 1.234,56 | day/month/year | money out negative | La exportación incluye dos fechas: la de operación y la F. Valor (cuando el importe se hace efectivo). Para conciliar gastos se usa la de operación. | [Limpiar](https://mercaops.com/es/limpiar-csv-bancario/bbva-a-excel/) · [JSON](presets/es/bbva-a-excel.json) |
| CaixaBank | semicolon | 1.234,56 | day/month/year | money out negative | Importes y saldos llevan el símbolo € y punto de miles («-1.250,00 €»); al abrirse como texto, Excel no los suma. El ajuste los convierte en números. | [Limpiar](https://mercaops.com/es/limpiar-csv-bancario/caixabank-a-excel/) · [JSON](presets/es/caixabank-a-excel.json) |
| Mercado Pago | auto-detected | auto-detected | year-month-day | money out negative | El reporte trae filas que no son movimientos: el saldo disponible inicial y el total del periodo (RECORD_TYPE initial_available_balance y total). Se excluyen para no duplicar importes. | [Limpiar](https://mercaops.com/es/limpiar-csv-bancario/mercadopago-a-excel/) · [JSON](presets/es/mercadopago-a-excel.json) |
| N26 | auto-detected | auto-detected | auto-detected | money out negative | Las exportaciones de N26 traen dos fechas, Booking Date y Value Date, y YNAB necesita una. Se usa la de contabilización, el día en que el movimiento entró en tu cuenta. | [Limpiar](https://mercaops.com/es/limpiar-csv-bancario/n26-a-ynab/) · [JSON](presets/es/n26-a-ynab.json) |
| PayPal | auto-detected | auto-detected | auto-detected | money out negative | En cuentas de España y Latinoamérica los importes pueden venir con coma decimal y punto de miles («1.250,00»), que QuickBooks no acepta; el ajuste los escribe como 1250.00. | [Limpiar](https://mercaops.com/es/limpiar-csv-bancario/paypal-a-quickbooks/) · [JSON](presets/es/paypal-a-quickbooks.json) |
| Revolut | auto-detected | 1,234.56 | year-month-day | money out negative | La columna Type (CARD_PAYMENT, TOPUP, TRANSFER, EXCHANGE…) aporta el contexto que necesitan las reglas bancarias de QuickBooks, pero en mayúsculas; el ajuste la añade a la descripción de forma legible. | [Limpiar](https://mercaops.com/es/limpiar-csv-bancario/revolut-a-quickbooks/) · [JSON](presets/es/revolut-a-quickbooks.json) |
| Revolut | auto-detected | 1,234.56 | year-month-day | money out negative | Los pagos con tarjeta aún no liquidados se exportan con la columna Completed Date vacía y State PENDING; si se importan, crean movimientos que luego cambian o desaparecen. El ajuste quita las filas PENDING, REVERTED, DECLINED y FAILED. | [Limpiar](https://mercaops.com/es/limpiar-csv-bancario/revolut-a-ynab/) · [JSON](presets/es/revolut-a-ynab.json) |
| Banco Santander | semicolon | 1.234,56 | day/month/year | money out negative | Importes y saldos pueden venir como texto con la moneda pegada («-25,40 EUR»), así que Excel no los suma; el ajuste quita la moneda y los convierte en números. | [Limpiar](https://mercaops.com/es/limpiar-csv-bancario/santander-a-excel/) · [JSON](presets/es/santander-a-excel.json) |
| Stripe | auto-detected | 1,234.56 | year-month-day | money out negative | Las horas están en UTC (2024-04-11 22:41:55), así que una venta nocturna en Latinoamérica puede caer al día siguiente; el ajuste mantiene la fecha UTC que informa Stripe. | [Limpiar](https://mercaops.com/es/limpiar-csv-bancario/stripe-a-quickbooks/) · [JSON](presets/es/stripe-a-quickbooks.json) |
| Wise | auto-detected | 1,234.56 | day/month/year | money out negative | Cada fila trae datos de conversión (Exchange From, Exchange To, Exchange Rate) que la carga de 3 columnas de QuickBooks no admite; se eliminan. | [Limpiar](https://mercaops.com/es/limpiar-csv-bancario/wise-a-quickbooks/) · [JSON](presets/es/wise-a-quickbooks.json) |

## Accounting file converters

- [Bank CSV, Excel or .QBO → QuickBooks Desktop .IIF](https://mercaops.com/csv-cleaner/csv-to-iif/) (EN)
- [Bank CSV or Excel → QuickBooks .QBO](https://mercaops.com/csv-cleaner/csv-to-qbo/) (EN)
- [OFX / QFX → QuickBooks .QBO](https://mercaops.com/csv-cleaner/ofx-to-qbo/) (EN)
- [Bancolombia → Alegra .xlsx](https://mercaops.com/es/limpiar-csv-bancario/bancolombia-a-alegra/) (ES)
- [CSV, Excel o .QBO → QuickBooks Desktop .IIF](https://mercaops.com/es/limpiar-csv-bancario/csv-a-iif/) (ES)
- [CSV o Excel del banco → QuickBooks .QBO](https://mercaops.com/es/limpiar-csv-bancario/csv-a-qbo/) (ES)

## Writing accounting files: field notes

- [QuickBooks Web Connect (.QBO)](formats/quickbooks-web-connect-qbo.md): header, `<INTU.BID>` placement, which bank ID QuickBooks accepts, stable FITIDs
- [QuickBooks Desktop (.IIF)](formats/quickbooks-desktop-iif.md): the layout Intuit's own samples use, transaction types, zero-sum entries
- [Alegra (.xlsx)](formats/alegra-xlsx.md): the bank-reconciliation template
- [Reading OFX, QFX and QIF](formats/reading-ofx-qfx-qif.md): SGML vs XML, character sets, dates, QIF quirks

## Data format

`presets.json` holds every export; `presets/<lang>/<id>.json` one each. `file` describes the
input (`columns` lists the header names seen for each field, in priority order), `knownQuirks`
what trips imports up, `columnMapping` where each column ends up, and `sample` points to a
**synthetic** example file in [`samples/`](samples) (made-up data, real layout and encoding;
Windows-1252 samples are stored in Windows-1252).

## Licence and credit

Data and notes: [CC BY 4.0](LICENSE). Reuse them, including commercially, with credit:

> Bank export notes: [MercaOps](https://mercaops.com) (mercaops.com)

Bank, payment and accounting product names are trademarks of their owners and are used only to
describe file formats; MercaOps isn't affiliated with any of them.

## About MercaOps

[MercaOps](https://mercaops.com) makes private, browser-based tools for online sellers: a
[bank CSV cleaner and QuickBooks converter](https://mercaops.com/csv-cleaner/), an
[import calculator](https://mercaops.com/import-calculator/) and a [shipping label cropper](https://mercaops.com/label-cropper/).
Your files never leave your browser.
