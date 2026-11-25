# Reading OFX, QFX and QIF statements

Notes from the parsers behind the [MercaOps CSV Cleaner](https://mercaops.com/csv-cleaner/),
which reads these files in the browser.

## OFX / QFX / QBO

- **Two syntaxes, one reader.** OFX 1.x is SGML (`<TRNAMT>-42.50` with no closing tag); OFX 2.x
  is XML (`<TRNAMT>-42.50</TRNAMT>`). A regex for leaf values, `<TAG>([^<]*)`, plus one for
  aggregates (`<STMTTRN>…</STMTTRN>`) reads both. QFX (Quicken) and QBO (QuickBooks) are OFX with
  an Intuit bank ID.
- **Character set.** 1.x headers declare `CHARSET:1252` (Windows-1252) or `USASCII`; decode the
  bytes accordingly, not as UTF-8. 2.x files are UTF-8 XML.
- **Entities.** `AT&amp;T` → `AT&T`; also `&lt;`, `&gt;`, `&quot;`, `&apos;`, numeric entities.
- **Dates.** `DTPOSTED` looks like `20260908235959.000[-5:EST]`. Take the first 8 digits as the
  calendar day the bank wrote. Converting with the time-zone offset moves late-evening
  transactions to the next day.
- **Accounts.** `BANKACCTFROM` (BANKID, ACCTID, ACCTTYPE) for bank accounts, `CCACCTFROM` (ACCTID)
  for cards. `LEDGERBAL/BALAMT` is the statement's closing balance: use it to check
  opening + transactions = closing.
- **IDs.** Keep `FITID`: QuickBooks uses it to skip duplicates.

## QIF

- **Account type** in the first line: `!Type:Bank`, `!Type:CCard`, `!Type:Cash`.
- **Records** are lines of single-letter fields, ended by `^`: `D` date, `T` (or `U`) amount,
  `P` payee, `M` memo, `N` check number, `L` category.
- **Dates** come as `1/15/2024`, `15/01/2024` or `1/15'24` (apostrophe = 2000s). Day/month order
  isn't declared: infer it from values above 12.
- **Amounts** may carry thousands separators (`1,250.00`).
- **Split lines** (`S`, `E`, `$`) break a transaction down by category. Ignore them when totalling;
  the `T` line already holds the full amount.

---

Data and notes by [MercaOps](https://mercaops.com) · [CC BY 4.0](../LICENSE)
