# UAPI v6 → E-Invoice XML Format Mappings

As of 2026-11-04.

These pages show how each field of the fiskaly Unified API (UAPI) v6 e-invoicing payload ends up in five XML e-invoice formats. Each format has its own page, built like [`request2gobl.md`](https://github.com/fiskaly/meta/blob/master/doc/gobl/request2gobl.md) with an extra column for the EN 16931 business term (BT/BG).

## Layout

| Location | Contents |
| --- | --- |
| [`UAPI - Format Mappings/`](UAPI%20-%20Format%20Mappings/) | The mapping pages and the UAPI v6 field inventory |
| [`Laws and formats/`](Laws%20and%20formats/) | Laws, format specifications and format examples, one folder per country or format (`Italy FatturaPA/`) |
| Repo root | This README, `CLAUDE.md`, and the UAPI API docs `fiskaly.unified-api.BE_EI-DE_EI-IT_EI.2026-06-01 (1).yaml` |

## Pages

| Page | Format and version mapped | Syntax | Status |
| --- | --- | --- | --- |
| [XRechnung_3.md](UAPI%20-%20Format%20Mappings/XRechnung_3.md) | XRechnung 3.0.2 (CIUS of EN 16931-1:2017) | UBL 2.1 Invoice / CreditNote | draft for review |
| [ZUGFeRD_2.md](UAPI%20-%20Format%20Mappings/ZUGFeRD_2.md) | ZUGFeRD 2 / Factur-X, EN 16931 (COMFORT) profile | UN/CEFACT CII, D22B-based Factur-X XSD (backward compatible with D16B), in PDF/A-3 | draft for review |
| [FatturaPA_1.4.md](UAPI%20-%20Format%20Mappings/FatturaPA_1.4.md) | FatturaPA V1.4, ordinary invoice (FPR12) | FatturaPA XML | draft for review |
| [Peppol_BIS_Billing_3.md](UAPI%20-%20Format%20Mappings/Peppol_BIS_Billing_3.md) | Peppol BIS Billing 3.0, release 3.0.21 | UBL 2.1 Invoice / CreditNote | draft for review |
| [Peppol_BIS_Billing_4_PROVISIONAL.md](UAPI%20-%20Format%20Mappings/Peppol_BIS_Billing_4_PROVISIONAL.md) | Peppol BIS Billing 4.0, **unpublished**; mapped provisionally against EN 16931-1:2026 | UBL (version not yet published) | provisional draft |
| [UAPI_v6_Fields.md](UAPI%20-%20Format%20Mappings/UAPI_v6_Fields.md) | Source field inventory: every v6 path in scope, with constraints and the API docs' BT hint | — | done |

## Findings That Apply to Every Format

All of these were confirmed by independent checks against code and primary sources on 2026-11-03 and 2026-11-04. Details and file:line evidence are on each page.

| Finding | Affects | Effect |
| --- | --- | --- |
| `VAT_REVERSE_CHARGE` lines, and `MEAL_VOUCHER` / `DEPOSIT` payments, are rejected by the GOBL mapping | all | no reverse-charge (AE) invoices at all |
| The supplier's VAT ID always beats `tax_id_number`, so BT-32 (and the BE KBO/BCE number) is never emitted. UAPI has no Steuernummer field | XRechnung, ZUGFeRD, BIS | a DE taxpayer without a USt-IdNr cannot send; without a VAT ID, the Steuer-ID is sent as BT-31 and fails GOBL |
| `NOT_SUBJECT` → VAT category `O` while BT-31/BT-48 are still sent | XRechnung, ZUGFeRD, BIS 3 | fails BR-O-02. Not caught at sync, so it surfaces in Invopop's validation step after the record is `COMPLETED` (code reading, not executed). Does not carry over to BIS 4: BR-O-02 is not in the XRechnung 4.0 Vorversion |
| `buyer_id` and `company_id` are both emitted as buyer identifiers; `company_id` never reaches BT-47 | XRechnung, ZUGFeRD, BIS | fails UBL-SR-16 / FX-SCH-A-000163 when both are set |
| `operation_date` becomes BT-72 (delivery date), never BT-7 | XRechnung, ZUGFeRD, BIS | tax point date is never expressed |
| Fields the API docs tag with a BT are dropped: BT-22 `document.text`, `product.*`, `unit.factor`, GROUP `items[]`, and more | all | silent data loss |
| A correction is always a credit note (381 / TD04) with unnegated amounts; BG-3 is read back from Invopop | all | no upward correction (debit note, 384, TD05); an upward correction sent this way passes validation and credits the buyer |
| `invoicing.format` is kept only for a DE recipient address | XRechnung, ZUGFeRD | a DE supplier's `XRECHNUNG_V3` e-mail to a BE recipient silently becomes ZUGFeRD, contrary to the API docs |
| A sync failure returns 2xx and leaves the record `COMPLETED` with an error log; Invopop invoice faults are logged raw, untranslated | XRechnung, ZUGFeRD, BIS | the client sees success; nothing is sent. IT records instead stay `ACCEPTED` until the SdI webhook |
| What Invopop's "PPL" numbering step in the Peppol workflows does to BT-1 | BIS 3, XRechnung via Peppol | UNVERIFIED; check a sandbox silo entry or ask Invopop |

## Scope

- **Transactions:** `INVOICE` (BT-3 `380`, FatturaPA `TD01`) and `CORRECTION`, which becomes a credit note or corrective invoice that references the preceding invoice (BG-3).
- **Supplier context:** taxpayer master data (`POST/PATCH /taxpayers`) and the e-invoice system's Peppol identifier (system annotation `peppol_id`), as in `request2gobl.md` §4.
- **Out of scope:** receipts, pro-forma invoices, cancellations, enrichment, inbound invoices (`E_INVOICE::RECEPTION`), and the Belgian e-mail flow (`BE_SEND_INVOICE_EMAIL_OASIS_UBL_BUSINESS`, plain EN 16931 UBL).

## How each page is built

Each page has two layers:

1. **Spec mapping.** UAPI field → EN 16931 BT → XPath in the format, taken from the official format artefacts. This is the target design.
2. **As-built notes.** What today's pipeline actually does, traced from code: UAPI → GOBL (fiskaly `src/unified/einvoicing/gobl/`) → Invopop converter → XML. The notes column records where today's output differs from the target: the field is ignored, set to a fixed value, defaulted or derived.

**Columns:** `UAPI | EN 16931 BT | <format element> | notes`. Calculated rows add a formula column.

**Notes vocabulary:** *ignored*: the UAPI field exists but nothing is written. *fixed*: constant value. *default*: used when the UAPI field is empty. *derived*: computed from other fields. *gap*: the format requires or supports the element but UAPI has no source. *UNVERIFIED*: the claim could not be confirmed against a primary source. A direct mapping carries no tag.

## Sources and version pins

| Source | Version / pin | Used for |
| --- | --- | --- |
| UAPI v6 OpenAPI definition | `fiskaly/meta@cd19ec88cb299ee41a44302448006340dd028bf0`, `src/unified/api/v6/definition.yaml` | field paths and constraints |
| UAPI API docs | version `2026-06-01` (BE_EI, DE_EI, IT_EI), [`fiskaly.unified-api.BE_EI-DE_EI-IT_EI.2026-06-01 (1).yaml`](fiskaly.unified-api.BE_EI-DE_EI-IT_EI.2026-06-01%20(1).yaml) | field descriptions and fiskaly's intended BT |
| fiskaly UAPI → GOBL mapping | `fiskaly/meta@cd19ec8`, `src/unified/einvoicing/**`, Invopop workflows in `cmd/invopop-workflow-migration/workflows/` | as-built notes |
| GOBL | `invopop/gobl` v0.501.0 (pinned in fiskaly `go.mod`) | addons and normalisation |
| Invopop converters | Code read: `invopop/gobl.ubl` v0.99.0 (requires gobl ≥ v0.511.0), `invopop/gobl.cii` v0.81.0 (requires gobl ≥ v0.511.0), `invopop/gobl.fatturapa` v0.90.0 (module now `gobl.it.sdi`, requires gobl ≥ v0.508.0), `invopop/gobl.de.xinvoice` v0.0.9 (requires gobl ≥ v0.510.0, gobl.ubl ≥ v0.93.0, gobl.cii ≥ v0.75.0). All four need a newer gobl than fiskaly's pin. These were the latest tags on 2026-11-03. fiskaly cannot see which version Invopop runs in production | GOBL → XML |
| XRechnung | 3.0.2 bundle 2026-08-31 (CIUS model XML, UBL syntax binding), held in the formats repo | XRechnung page |
| ZUGFeRD / Factur-X | Field table 2.4 / 1.08, held locally; the current version is 2.5.2 / 1.09.2 (in force since 2026-09-01), and the deltas are checked on the page | ZUGFeRD page |
| FatturaPA | Format spec V1.4 and `RappresentazioneTabellareFattOrdinaria.xlsx` in [`Laws and formats/Italy FatturaPA/`](Laws%20and%20formats/Italy%20FatturaPA/); SdI *specifiche tecniche* v1.8.4 with *Elenco controlli* v2.0 (error codes); AdE *specifiche tecniche* v1.9.1, Allegato A (field checks) | FatturaPA page |
| Peppol BIS Billing | 3.0.21 (mandatory since 2026-08-17): schematron and syntax binding. 3.0.22 is due 2026-11-23, mandatory from 2027-02-24 (checked 2026-11-03) | BIS 3 page |
| EN 16931-1:2026 (stand-in) | KoSIT `XRechnung-4.0.0-Vorversion-2026-09-15.pdf` (not normative). EN 16931-1:2026 and its UBL binding CEN/TS 16931-3-2:2026 (valid from 2026-08-17) are published but paywalled and not held. The Vorversion is a PDF only, with no UBL binding. Carried-over BTs therefore use the XRechnung 3.0.2 UBL paths, and new BTs have no confirmed path | BIS 4 page |
| EN 16931 ↔ FatturaPA | Agenzia delle Entrate, Technical Rules under D.Lgs. 148/2018 art. 3 c.1, v2.6 of 15/05/2025, Appendix 4.1 (plus Appendices 4.2, 5.1, 5.4, 5.6) | FatturaPA BT column |

FatturaPA is not a CIUS of EN 16931. Its BT column therefore uses the official correspondence above.

Versions and dates follow the regulatory baseline in the formats repo (`regulatory_baseline.md`, rows verified 2026-10-29). Re-verify before relying on any of them after 2027-02-03.
