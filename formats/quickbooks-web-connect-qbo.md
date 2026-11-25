# Writing QuickBooks Web Connect (.QBO) files

A `.QBO` file is an OFX 1.02 bank statement in SGML with one Intuit-specific tag, `<INTU.BID>`.
These are the rules the [MercaOps CSV → QBO converter](https://mercaops.com/csv-cleaner/csv-to-qbo/)
follows, and the evidence behind each one.

## Header

Nine header lines, a blank line, then the body:

```text
OFXHEADER:100
DATA:OFXSGML
VERSION:102
SECURITY:NONE
ENCODING:USASCII
CHARSET:1252
COMPRESSION:NONE
OLDFILEUID:NONE
NEWFILEUID:NONE
```

## Body (bank account)

SGML: leaf elements have no closing tag, aggregates do.

```text
<OFX>
<SIGNONMSGSRSV1>
<SONRS>
<STATUS><CODE>0<SEVERITY>INFO</STATUS>
<DTSERVER>20261125093000[0:GMT]
<LANGUAGE>ENG
<FI><ORG>Wells Fargo<FID>3000</FI>
<INTU.BID>3000
</SONRS>
</SIGNONMSGSRSV1>
<BANKMSGSRSV1>
<STMTTRNRS>
<TRNUID>1
<STATUS><CODE>0<SEVERITY>INFO</STATUS>
<STMTRS>
<CURDEF>USD
<BANKACCTFROM><BANKID>000000000<ACCTID>1234<ACCTTYPE>CHECKING</BANKACCTFROM>
<BANKTRANLIST>
<DTSTART>20260908120000[0:GMT]
<DTEND>20260930120000[0:GMT]
<STMTTRN>
<TRNTYPE>DEBIT
<DTPOSTED>20260908120000[0:GMT]
<TRNAMT>-42.50
<FITID>2026090850E3A1C2B79F04D1
<NAME>AT&amp;T WIRELESS
<MEMO>Autopay
</STMTTRN>
</BANKTRANLIST>
<LEDGERBAL><BALAMT>3257.50<DTASOF>20260930120000[0:GMT]</LEDGERBAL>
</STMTRS>
</STMTTRNRS>
</BANKMSGSRSV1>
</OFX>
```

Credit cards use `<CREDITCARDMSGSRSV1>` → `<CCSTMTTRNRS>` → `<CCSTMTRS>` and
`<CCACCTFROM><ACCTID>…</CCACCTFROM>` (no `BANKID` or `ACCTTYPE`). `LEDGERBAL` appears in every
full sample we have seen; write the statement's closing balance when you know it.

A plain `.OFX` for Xero, Wave or GnuCash is the same file without `<INTU.BID>` (and the `<FI>`
block is optional).

## `<INTU.BID>`: where it goes and which number QuickBooks accepts

- **Placement:** after `</FI>`, still inside `<SONRS>`. Real QuickBooks Desktop files put it there;
  none put it inside `<FI>`.
- **Checking:** QuickBooks Desktop compares the ID with Intuit's list of Web Connect institutions
  and stops on an unknown bank ("unable to verify the financial institution"). QuickBooks Online
  is reported to be lenient. *(Secondary.)*
- **The list is public:** Intuit's own directory, the file QuickBooks Desktop downloads, is at
  `https://ofx-prod-filist.intuit.com/qb3400/data/fidir.txt` (QuickBooks 2024 list). It is
  tab-separated, Windows-1252; column 1 is the entry's ID, column 4 a group ID, column 5 the
  name, column 9 the status and column 11 the Web Connect services. Keep `ACTIVE` rows whose
  column 11 includes `WEB-CONNECT` (about 5,800 banks and credit unions).
- **Use column 1, without leading zeros.** Intuit community threads report files carrying the
  group ID (column 4) failing to import and working once changed to the entry's own ID, for
  example Chase 10898 → 2430 and Bank of America 6526 → 11162. Wells Fargo is 3000 in both.
  *(Secondary: community reports, not Intuit documentation.)*
- **Fallback:** `3000` (Wells Fargo) is the common choice when a bank isn't listed; QuickBooks
  then shows the account as Wells Fargo.
- Quicken (`.QFX`) and QuickBooks can use different IDs for the same bank, so a converted QFX
  needs its ID checked again.

## FITID: no duplicates, no gaps

QuickBooks skips any transaction whose `FITID` it has already imported into the account. So the
same transaction must get the same ID on every export, and two identical transactions must not
share one.

- When the input is OFX/QFX/QBO, keep the bank's own FITIDs.
- Otherwise compute one: `YYYYMMDD` + the first 16 hex characters (upper case) of
  `SHA-256("YYYY-MM-DD|amountInCents|PAYEE|MEMO|n")`, where payee and memo are accent-folded,
  upper-cased and whitespace-collapsed, and `n` counts identical (date, amount, payee, memo)
  tuples in file order, from 1. That gives 24 alphanumerics; two identical coffees on one day get
  different IDs, and re-exporting an overlapping month reproduces the same IDs.
- Split payouts (gross sale + processing fee) add `G` and `F` to the base ID.

## Text, amounts, dates, types

- Declared `ENCODING:USASCII`: fold accents to ASCII (`Nómina` → `Nomina`) and drop what can't be
  represented.
- Escape `&`, `<`, `>` as `&amp;`, `&lt;`, `&gt;` (OFX 1.0.2 §2.3.2.1). *(Secondary: the spec
  section as cited by two independent sources.)*
- `<NAME>` at most 32 characters, `<MEMO>` at most 255.
- `<TRNAMT>`: dot decimal, minus sign for money out.
- `<DTPOSTED>`: `YYYYMMDD120000[0:GMT]`. Noon GMT keeps every time zone on the same calendar day.
- `<TRNTYPE>`: `CREDIT` for money in, `DEBIT` for money out, `CHECK` (with `<CHECKNUM>`) for a
  numbered check.

## QuickBooks Desktop in 2026

QuickBooks Desktop 2023 lost its connected services, bank feeds included, on 31 May 2026; Direct
Connect closed to new enrolments on 30 April 2026 and retires on 30 October 2026. Web Connect
(`.QBO` import) continues on supported Desktop versions; older versions can still import `.IIF`
([notes](quickbooks-desktop-iif.md)). *(Secondary: consistent reports from Intuit partners.)*

## Sources

- Intuit financial-institution directory: <https://ofx-prod-filist.intuit.com/qb3400/data/fidir.txt>
- Intuit community, Chase Web Connect import: <https://quickbooks.intuit.com/learn-support/en-us/other-questions/chase-qbo-webconnect-download-problem/00/230924>
- Intuit community, Bank of America `INTU.BID`: <https://quickbooks.intuit.com/learn-support/en-us/banking/can-not-connect-to-bofa/00/1583365>
- OFX 1.0.2 is maintained by the Financial Data Exchange (FDX); its site serves the specifications to registered users.

---

Data and notes by [MercaOps](https://mercaops.com) · [CC BY 4.0](../LICENSE)
