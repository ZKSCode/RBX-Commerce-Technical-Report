# Demonstrations and validation

[Português](https://github.com/ZKSCode/RBX-Commerce-Relatorio-Tecnico/blob/main/docs/demonstracoes.md) · [Technical report home](../README.md)

The demonstration ran from a temporary copy using PHP 8.3.28, Laravel 11.44.7, PHPUnit 11.5.18, and SQLite. The retailer, product, users, and amounts are fictional. The inventory test intercepted outgoing email locally.

| Scenario | Inputs and actions | Observed result |
|---|---|---|
| Frontend build | Installed lockfile dependencies with `npm ci --ignore-scripts` and ran `npm run build`. | Vite generated the manifest and build assets. Browserslist reported a data warning. |
| POS and register | Opened with BRL 100, scanned an EAN, made two BRL 40 sales, one cash and one card, deposited BRL 10, withdrew BRL 20, and counted BRL 128. | Search returned one variant. The receipt returned HTTP 200. Closing recorded BRL 130 expected and a BRL 2 shortage. Stock decreased from five to three across two movements. |
| Low-stock alert | The first sale reduced the variant from five units to its minimum of four. The next sale reduced it to three. | The observer queued an alert with email mocked. The second movement took place after the threshold had been crossed. |
| Same EAN in two stores | Two test stores used the same EAN for variants. A user from the first store searched that barcode at the POS. | Exact search returned only the user's store variant. The result applies to this specific lookup path. |
| Consent | Existing automated cases exercised pending consent, acceptance, and repeat sign-in. | All three cases passed against the demo database. |

## Printed closing summary

The database recorded one BRL 40 cash sale and one BRL 40 card sale. The dashboard showed BRL 80 in sales, while the printed summary showed BRL 40 total and BRL 0 in card sales. Printing counts the internal debit and credit values, while the POS persisted this payment under the generic card value. Physical cash figures agreed with the database. The register expected BRL 130, counted BRL 128, and recorded a BRL 2 shortage. [See both screens](gallery.md)

## Demo environment adjustments

The original copy could not list routes on Linux because class-name capitalization differed. SQLite migrations found a duplicate index name. Some Blade components also used paths whose capitalization did not match the Laravel loader. The demo copy received compatibility changes in those three areas and a fictional store configuration.

After those changes, all 60 migrations ran and the router listed 122 routes, including framework and dependency routes. The review found three declarations whose controller methods were missing in the examined revision. They covered temporary image upload, changing a user role from settings, and Pix status lookup at the POS.

The adjusted environment passed the five existing tests and the integration scenarios for a counter sale and exact EAN lookup across two stores. The initial in-memory SQLite run reported three missing-table errors and an HTTP failure. The demo used a pre-migrated database for tests that run inside a transaction over an existing schema.

## Reproduce the workflows

1. **Inventory.** Find the demo product, review its variants, and compare the current balance with the two outgoing movements. Enter a physical count of two against a recorded balance of three and review the discrepancy before applying it.
2. **Counter sale.** Open a register, find the variant by EAN, record both sales, and check the receipt, available stock, queued alert, and closing summary.
3. **Further validation.** Repeat on MySQL and verify concurrency and accessibility. Test applying a CSV import against a disposable database.

The [gallery](gallery.md) and [technical report](technical-report.md) provide the screenshots and context for these observations.
