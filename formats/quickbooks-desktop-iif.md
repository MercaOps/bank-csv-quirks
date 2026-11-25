# Writing QuickBooks Desktop (.IIF) files

`.IIF` (Intuit Interchange Format) imports transactions straight into QuickBooks Desktop
registers. It has almost no error checking and no undo, so the file has to be right the first
time. This layout follows **Intuit's own sample files** and is what the
[MercaOps CSV → IIF converter](https://mercaops.com/csv-cleaner/csv-to-iif/) writes.

## Layout

Plain ASCII, tab-separated, CRLF line endings. Three header rows, then one block per
transaction: a `TRNS` line, one or more `SPL` lines, and `ENDTRNS`.

```text
!TRNS	TRNSID	TRNSTYPE	DATE	ACCNT	NAME	AMOUNT	DOCNUM	MEMO	CLEAR
!SPL	SPLID	TRNSTYPE	DATE	ACCNT	NAME	AMOUNT	DOCNUM	MEMO	CLEAR
!ENDTRNS
TRNS		CHECK	7/22/2026	Checking	Vendor	-36.15	1043	Legal fees	N
SPL		CHECK	7/22/2026	Uncategorized Expenses	Vendor	36.15		Legal fees	N
ENDTRNS
TRNS		DEPOSIT	7/1/2026	Checking	Customer	10000.00		Deposit	N
SPL		DEPOSIT	7/1/2026	Uncategorized Income	Customer	-10000.00		Deposit	N
ENDTRNS
```

That column set is the one in Intuit's "transfer" sample; the other samples add `CLASS` and
item columns. Every data line must have as many fields as its header row.

## Transaction types

| Account | Money | `TRNSTYPE` | `TRNS` amount | `SPL` amount (offset account) |
|---|---|---|---|---|
| Bank | out | `CHECK` | negative | positive (expense) |
| Bank | in | `DEPOSIT` | positive | negative (income) |
| Credit card | charge | `CREDIT CARD` | negative | positive (expense) |
| Credit card | refund | `CCARD REFUND` | positive | negative |

`CHECK`, `DEPOSIT` and `CCARD REFUND` appear in Intuit's sample files. `CREDIT CARD` for a
charge comes from the import kit's header notes as quoted on Intuit's community; there is no
plain card-charge sample. *(Secondary for `CREDIT CARD`.)*

## The zero-sum rule

`TRNS` amount + all `SPL` amounts = 0 for every transaction, to the cent. Intuit's samples all
balance; check it in integer cents before writing anything, and refuse to write a file with a
single unbalanced transaction.

A payment-processor payout split into gross sale and fee is one three-line entry:

```text
TRNS		DEPOSIT	4/1/2024	Stripe Clearing	Invoice 1043	116.22		Invoice 1043 - Charge	N
SPL		DEPOSIT	4/1/2024	Sales	Invoice 1043	-120.00		Invoice 1043 - Charge	N
SPL		DEPOSIT	4/1/2024	Merchant Fees		3.78		Processing fee - Invoice 1043	N
ENDTRNS
```

## Fields

- **DATE:** `M/D/YYYY` as in Intuit's samples (US editions). For UK and Canadian company files,
  test with the 5-row file first: day-first dates are commonly used there. *(Unverified.)*
- **ACCNT:** exactly as in the Chart of Accounts. **QuickBooks creates a new account for any name
  it doesn't recognise**, so a typo becomes a stray account. Subaccounts use `Parent:Child`.
- **NAME:** optional. Unknown names are added to QuickBooks' name lists, so you may prefer to keep
  payees in `MEMO`.
- **DOCNUM:** numeric check numbers only.
- **CLEAR:** `N`.
- **Text:** no tabs or line breaks inside a field; fields containing a comma are wrapped in double
  quotes (Intuit's samples do this, e.g. `"Favor, Lynn:Sun Room"`); replace double quotes inside
  text.

## Before importing

Back up the company file (File → Back Up Company), import a 5-transaction test file first
(File → Utilities → Import → IIF Files), check the register, then import the rest.

## Sources

Intuit sample files (from Intuit's "IIF overview: import kit, sample files, and headers" page):

- Check: <https://http-download.intuit.com/http.intuit/OpenCms/sites/default/QBSupportSite/executables/IIF/check.zip>
- Deposit: <https://http-download.intuit.com/http.intuit/OpenCms/sites/default/QBSupportSite/executables/IIF/deposit.zip>
- Credit card refund: <https://http-download.intuit.com/http.intuit/OpenCms/sites/default/QBSupportSite/executables/IIF/credit_card_refund.zip>
- All samples (iif1): <https://http-download.intuit.com/http.intuit/OpenCms/sites/default/QBSupportSite/executables/IIF/iif1.zip>
- Overview: <https://quickbooks.intuit.com/learn-support/en-us/help-article/list-management/iif-overview-import-kit-sample-files-headers/L5CZIpJne_US_en_US>

---

Data and notes by [MercaOps](https://mercaops.com) · [CC BY 4.0](../LICENSE)
