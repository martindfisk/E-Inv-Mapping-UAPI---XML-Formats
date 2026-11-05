# UAPI v6 → XML Format Mappings — Claude Instructions

One Markdown page per XML format, mapping fiskaly UAPI v6 e-invoicing fields to the format with the EN 16931 BT in between. `README.md` holds the page index, scope and version pins. Read it first.

## Layout

- `UAPI - Format Mappings/`: the mapping pages and `UAPI_v6_Fields.md`. New format pages go here.
- `Laws and formats/<country or format>/`: laws, format specifications and format examples (today `Italy FatturaPA/`). New source material goes into a folder per country or format here.
- Repo root: `README.md`, this file, and the UAPI API docs YAML.
- Pages link to each other by file name. Links from a page to a root file use `../`. Paths to source material in Sources tables are written from the repo root, e.g. `Laws and formats/Italy FatturaPA/Laws/22_SdI_Elenco_Controlli_v2.0.md`.
- When a file moves, update `README.md` (Layout, Pages, Sources), this file and every page link in the same change.

## Pinned inputs

- **UAPI fields:** `fiskaly/meta@cd19ec88cb299ee41a44302448006340dd028bf0` `src/unified/api/v6/definition.yaml`. Fetch with `gh api "repos/fiskaly/meta/contents/<path>?ref=<sha>" -H "Accept: application/vnd.github.raw"`. The v6 descriptions are empty placeholders. Field descriptions and fiskaly's intended BT come from the API docs for version 2026-06-01 (the `fiskaly.unified-api.BE_EI-DE_EI-IT_EI.2026-06-01 (1).yaml` file at the repo root).
- **As-built code:** same commit, `src/unified/einvoicing/**` (UAPI → GOBL), plus the Invopop workflows in `cmd/invopop-workflow-migration/workflows/` (doc types `xrechnung-invoice-v2`, `zugferd-v2`, `bis-invoice-v3`, FatturaPA via SDI).
- **Converters:**
  - fiskaly pins `invopop/gobl` v0.501.0.
  - Code read: `gobl.ubl` v0.99.0, `gobl.cii` v0.81.0, `gobl.fatturapa` v0.90.0 and `gobl.de.xinvoice` v0.0.9. All four require a newer gobl than fiskaly's pin (v0.508.0 to v0.511.0); see `README.md`. Behaviour that runs inside a converter is cited at the gobl version that converter requires.
- **Format specs:** local copies in the formats repo (see the Workspace Map in the parent `Research/CLAUDE.md` and `local_specs_index.md`). FatturaPA specs and laws are also in `Laws and formats/Italy FatturaPA/` in this repo.

If the meta commit changes, regenerate `UAPI - Format Mappings/UAPI_v6_Fields.md` and re-check every page's paths. Paths that differ between v5 and v6 are listed at the top of that file.

## Page conventions

- **Structure:** short intro → 1. Invoice mapping (1.1 Header and references, 1.2 Buyer, 1.3 Lines, 1.4 Payment, 1.5 VAT breakdown and totals, 1.6 Calculated fields) → 2. Supplier data (taxpayer, system) → 3. Correction → 4. Fixed and derived values → 5. Format elements without a UAPI source (gaps) → 6. Common validation failures → Sources → Verification Log. `UAPI - Format Mappings/XRechnung_3.md` is the reference page. The BIS 4 page adds 7. Changes 2017 → 2026 and implementation gaps.
- **Intro:** at most two short paragraphs before §1. The first names the format and version, says in one sentence which UAPI settings select the flow, and names the spec artefact behind the XPath column. The second is the code-citation legend. No API endpoint list, routing table, workflow table or output summary: the mapping comes first.
- **Mapping table columns:** `UAPI | EN 16931 BT | <format element> | notes`.
  - **UAPI:** paths are relative to `content.operation` as in `UAPI_v6_Fields.md`. Use `-` when the value has no UAPI source.
  - **EN 16931 BT:** written as `BT-1 Invoice number`, or `BG-4 Seller` for groups.
  - **Format element:** a full XPath with namespace prefixes (`cbc:`/`cac:` for UBL, `ram:`/`rsm:` for CII). FatturaPA uses the field ID plus path, e.g. `2.1.1.4 DatiGeneraliDocumento/Numero`.
- **Notes vocabulary:** `ignored`, `fixed`, `default`, `derived`, `gap`, `UNVERIFIED` (defined in `README.md`). Notes name the as-built behaviour and cite `file:line` in the fiskaly or Invopop code when they say something non-obvious. A direct mapping needs no tag. Do not invent tags such as "Verbatim" or "compared only"; say it in plain words.
- Spec mapping and as-built behaviour are kept apart. The XPath column always holds the spec target. A deviation goes in the notes column; it never replaces the target.
- No conversational text in the pages. Every page starts with a heading and an "As of YYYY-MM-DD" line and ends with a Verification Log.

## Traps specific to this project

- `request2gobl.md` is Belgium-only and predates v6. Only its table layout is reused. Never use it as a source for paths.
- ZUGFeRD: the local field table is pinned at 2.4 / Factur-X 1.08. The current version is 2.5.2 / 1.09.2. Never call the pinned files current.
- FatturaPA: format spec V1.4 is not the same document as the SdI *specifiche tecniche* v1.8.4 or the AdE specs v1.9.1. Name the document you mean. FatturaPA is not an EN 16931 CIUS.
- Peppol BIS Billing 4.0 is unpublished (no OpenPEPPOL release; tags end at 3.0.21). EN 16931-1:2026 and its UBL binding CEN/TS 16931-3-2:2026 are published but paywalled and not held, so the page uses the XRechnung 4.0 pre-release as the stand-in source. Every row on it is marked provisional.
- Invopop runs the GOBL → XML conversion on its side. The converter versions named in `README.md` are the latest public tags, not a confirmed production version.
- Two code paths that were misread before: invoice faults from Invopop are logged raw (`buildErrorLogs`); only taxpayer-registration errors are translated (`translateInvoiceError`). fiskaly sets `tax.rounding: currency` on every invoice, so the converter's switch to `precise` never applies.
- Routing is decided in two places: `record_handler.go` keeps `invoicing.format` only for a DE recipient address, and `einvoicing/invopop.go` `documentTypeForInvoice` picks the document type. Check both before stating which format a recipient gets.
