# Writing Alegra bank-statement imports (.xlsx)

Alegra's bank reconciliation imports statements from its own Excel template only (no CSV). The
[MercaOps Bancolombia → Alegra converter](https://mercaops.com/es/limpiar-csv-bancario/bancolombia-a-alegra/)
writes it from Bancolombia's XLS, XLSX or CSV downloads.

## Template

| Rule | Value |
|---|---|
| File type | `.xlsx` (Excel); CSV is not accepted |
| Sheet name | `INGRESO DE DATOS` |
| Columns | `Fecha` · `Descripción` · `Monto` |
| Dates | `dd/mm/aaaa` (MercaOps writes real Excel dates formatted that way, which is what typing into the template in a Spanish-locale Excel produces) |
| Amounts | numbers with a **point** decimal, at most 4 decimals; **negative = money out** |
| Descriptions | **no numbers or special characters** |
| Errors | the import stops after more than 20 errors |

## Cleaning descriptions

Bank descriptions are full of references (`TRANSFERENCIA A NEQUI 300*****21`,
`COMPRA EN EXITO #1234`). Keep letters, spaces and basic punctuation, drop digits and symbols,
collapse spaces, and fall back to a generic word (`Movimiento`) if nothing is left:
`TRANSFERENCIA A NEQUI 300*****21` → `TRANSFERENCIA A NEQUI`.

## Sources

Alegra's help centre and its public template for Mexico; the Spain and Colombia guides describe
the same flow and rules but their template downloads need an account. *(The identical layout for
Spain and Colombia is secondary.)* Siigo publishes no public bank-statement template.

---

Data and notes by [MercaOps](https://mercaops.com) · [CC BY 4.0](../LICENSE)
