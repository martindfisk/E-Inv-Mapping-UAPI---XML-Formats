# Italy — e-invoicing laws (primary texts)

Index of the primary legal texts held in `Laws/`. Dates and versions are maintained in
`../regulatory_baseline.md`; this file points at the artefacts and explains how they fit together.
Statuses below were verified on **2026-10-29** against the texts in `Laws/` and the official pages cited.

> **Three documents, three version numbers — always name which one you mean.**
> 1. **FatturaPA format specification V1.4** (fatturapa.gov.it, "documentazione valida dal 1 aprile 2025",
>    held as `Specifiche_tecniche_del_formato_FatturaPA_V1.4.pdf` next to this file) — the XML format
>    (XSD FPA12/FPR12 v1.2.3, VFSM10 v1.0.2).
> 2. **Specifiche tecniche relative al Sistema di Interscambio v1.8.4** (fatturapa.gov.it, = DM 55/2013
>    Allegato B rules, valid from 2026-05-15, with *Elenco controlli* v2.0) — `Laws/21_…`, `Laws/22_…`.
> 3. **Agenzia delle entrate *specifiche tecniche* v1.9.1** (Allegato A of Provv. 433608/2022, dated
>    31-03-2026, usable from 2026-05-15) — `Laws/17_…`; B2B/B2C and cross-border data, error codes. v1.9
>    (31-01-2025) went with format V1.4 (TD29, RF20); v1.9.1 changed checks and channels only, no XSD change.
>
> "SDI specifiche tecniche 1.9.x" mixes publisher 2 with numbering 3. Never cite V1.3.2.

## Regimes (status 2026-10-29)

| Regime | What it governs | Core texts | Status 2026-10-29 |
|---|---|---|---|
| **B2G** | Invoices to public administrations only in FatturaPA via SdI; PA may not accept paper or pay before the e-invoice | L. 244/2007 art. 1 c. 209–214 → DM 55/2013 → DM 132/2020 (rejection causes); D.Lgs. 148/2018 (EN 16931 reception) | In force since 2014-06-06 (central) / 2015-03-31 (all other PA). EN 16931 invoices must be received since 2019-04-18 / 2020-04-18 (sub-central) |
| **B2B** | All supplies between parties resident or established in Italy: only e-invoices via SdI; an invoice issued otherwise is deemed not issued | D.Lgs. 127/2015 art. 1 c. 3, 6 (as rewritten by L. 205/2017 c. 909); D.L. 36/2022 art. 18; technical rules: Provv. 433608/2022 + Allegato A v1.9.1; SdI specs v1.8.4 | In force since 2019-01-01 (fuel / PA subcontractors 2018-07-01). Forfettari: > €25,000 since 2022-07-01, all since 2024-01-01. Only carve-out in the statute: invoices whose data go to the Sistema tessera sanitaria may not be e-invoiced via SdI (art. 10-bis D.L. 119/2018, not held; TU IVA art. 78 from 2027) |
| **B2C (invoices)** | Same SdI obligation when an invoice is issued to a consumer; AdE makes it available in the consumer's area, issuer hands over a copy | D.Lgs. 127/2015 art. 1 c. 3 | In force since 2019-01-01 |
| **Corrispettivi** (B2C receipts) | Retail operators (DPR 633 art. 22) store and transmit daily takings to AdE via RT / software; replaces the fiscal receipt; transmission within 12 days; POS must be linked to the RT | D.Lgs. 127/2015 art. 2; penalties D.Lgs. 471/1997 art. 6 c. 2-bis, 3 | In force: > €400,000 since 2019-07-01, all since 2020-01-01 (vending 2017-04-01, fuel 2018-07-01). POS–RT link (art. 2 c. 3) applies from 2026-01-01 (L. 207/2024 c. 77, note (24) in file 01) |
| **Cross-border reporting** ("esterometro") | Data on supplies to/from non-established parties sent to AdE via SdI in FatturaPA format (TD17/TD18/TD19 etc.); outbound by the invoicing deadline, inbound by the 15th of the following month | D.Lgs. 127/2015 art. 1 c. 3-bis; penalty D.Lgs. 471/1997 art. 11 c. 2-quater (€2/invoice, max €1,000 per quarter) | Via SdI since 2022-07-01 |
| **EU derogation** | Authorisation to require e-invoicing without recipient acceptance (VAT Directive arts. 218, 232) | Council Impl. Decision (EU) 2018/593, amended by 2021/2251 and 2024/3150 | Valid until **2027-12-31**; extension request due 2027-03-31; ends earlier if a general EU system enters into force |
| **Penalties** | Invoice / registration / corrispettivi violations | D.Lgs. 471/1997 art. 6 (to 2026-12-31) → D.Lgs. 173/2024 art. 31 (from 2027-01-01) | Art. 6: 70% of the VAT (€250–2,000 if no tax effect), 5% for exempt/non-taxable, min €300 (c. 4) |
| **Archiving** | Fiscal conservation of e-invoices | DPR 633 art. 39 c. 3 (→ DPR 600/1973 art. 22, not held); DM 17 giugno 2014 art. 3; D.Lgs. 127/2015 art. 1 c. 6-bis | E-invoices conserved electronically per DM 17-06-2014; storage abroad only where a mutual-assistance instrument exists (art. 39); conservation deemed satisfied for invoices stored by AdE via SdI (art. 1 c. 6-bis, per AdE provvedimento) |

**From 2027-01-01 the statute book changes, the obligations do not.** D.Lgs. 10/2026 (Testo unico IVA)
applies from 2027-01-01 (art. 171) and its art. 170 repeals, from the same date, DPR 633 arts. 1–17,
17-ter–27, 38–39 (and others), L. 244/2007 art. 1 c. 209–214, D.Lgs. 127/2015 arts. 1, 2 (most commi), 4,
L. 205/2017 c. 916–919 and the whole D.Lgs. 148/2018. New homes: TU IVA art. 72 (invoicing), 73 (simplified
invoice), 74 (B2G/SdI), 76 (EN 16931 procurement), 77 (e-invoicing via SdI), 78 (health data), 80 (San
Marino), 82 (corrispettivi), 89 (simplifications). Penalties move to D.Lgs. 173/2024 art. 31 on the same
date (art. 102, postponed from 2026-01-01 by D.L. 200/2025 art. 4 c. 1). Both testi unici were already
amended by D.Lgs. 5 agosto 2026, n. 141 (TU adempimenti e accertamento + coordination; not held).

## `Laws/` — primary texts

Each text is held twice: the original (`.pdf`) and a Markdown conversion (`.md`) with a header block
(title, gazette reference, source URL, retrieval date, version). Normattiva texts are the consolidated
version *vigente al 29-10-2026*; Normattiva is not an official source (only the Gazzetta Ufficiale is
authentic) — the GU reference is in each header.

| # | File stem | Instrument | Official reference | Why it matters |
|---|---|---|---|---|
| 01 | `DLgs_127-2015_Fatturazione_Elettronica_Corrispettivi_vigente` | D.Lgs. 5 agosto 2015, n. 127 | GU n. 190 del 18-08-2015 | Art. 1: e-invoicing via SdI (c. 3), esterometro (c. 3-bis), AdE storage (c. 5-bis), invoice outside SdI = not issued (c. 6), conservation (c. 6-bis). Art. 2: corrispettivi |
| 02 | `L_205-2017_art1_c909-928_Obbligo_FE_B2B_estratto` | L. 27 dicembre 2017, n. 205, art. 1 c. 909–928 (extract; PDF = whole act) | GU n. 302 del 29-12-2017, SO 62 | c. 909 = the 2019 B2B obligation; c. 916–917 = start dates |
| 03 | `DL_36-2022_art18_Forfettari_vigente` | D.L. 30 aprile 2022, n. 36 (conv. L. 79/2022) | GU n. 100 del 30-04-2022 | Art. 18 c. 2–3: forfettari brought in (2022-07-01 / 2024-01-01) |
| 04 | `DPR_633-1972_IVA_vigente` | DPR 26 ottobre 1972, n. 633 | GU n. 292 del 11-11-1972, SO 1 | Art. 21 invoice content, 12-day issuance (c. 4); art. 21-bis simplified invoice; art. 22 retail; art. 39 conservation |
| 05 | `DLgs_10-2026_Testo_Unico_IVA_vigente` | D.Lgs. 19 gennaio 2026, n. 10 | GU n. 24 del 30-01-2026, SO 4 | Applies 2027-01-01; arts. 72–90 recodify everything above; art. 170 repeals |
| 06 | `L_244-2007_art1_c209-214_FE_PA_estratto` | L. 24 dicembre 2007, n. 244, art. 1 c. 209–214 (extract; PDF = whole act) | GU n. 300 del 28-12-2007, SO 285 | B2G obligation; SdI established (c. 211) |
| 07 | `DM_55-2013_FatturaPA_B2G_vigente` | DM MEF 3 aprile 2013, n. 55 | GU n. 118 del 22-05-2013 | B2G regulation; art. 6 start dates; Allegati A–E (format, SdI rules, guidelines, codici ufficio) |
| 08 | `DLgs_148-2018_Direttiva_2014-55_EN16931_vigente` | D.Lgs. 27 dicembre 2018, n. 148 | GU n. 14 del 17-01-2019 | Directive 2014/55/EU: EN 16931 reception from 2019-04-18 (art. 3) / 2020-04-18 (art. 4) |
| 09 | `DM_132-2020_Rifiuto_Fatture_PA_vigente` | DM MEF 24 agosto 2020, n. 132 | GU n. 262 del 22-10-2020 | Closed list of reasons a PA may reject an e-invoice |
| 10 | `DLgs_471-1997_Sanzioni_vigente` | D.Lgs. 18 dicembre 1997, n. 471 | GU n. 5 del 08-01-1998, SO 4 | Penalties to 2026-12-31: art. 6 (invoicing, corrispettivi), art. 11 c. 2-quater (esterometro) |
| 11 | `DLgs_173-2024_Testo_Unico_Sanzioni_vigente` | D.Lgs. 5 novembre 2024, n. 173 | GU n. 279 del 28-11-2024, SO 40 | Penalties from 2027-01-01: art. 31 (ex art. 6 D.Lgs. 471/1997), art. 102 start date |
| 12 | `DL_200-2025_Proroga_Termini_vigente` | D.L. 31 dicembre 2025, n. 200 (conv. L. 26/2026) | GU n. 302 del 31-12-2025 | Art. 4 c. 1: TU sanzioni postponed to 2027-01-01 |
| 13 | `DM_17-06-2014_Conservazione_Documenti_Informatici` | DM MEF 17 giugno 2014 (GU original text; PDF = whole GU issue, pp. 5–7) | GU n. 146 del 26-06-2014 | Conservation of electronic tax documents (art. 3), stamp duty on e-invoices (art. 6, later amended — not consolidated) |
| 14 | `Provv_AdE_433608-2022_Regole_Tecniche_SdI` | Provv. Direttore AdE prot. 433608 del 24-11-2022 | agenziaentrate.gov.it | **Current SdI technical rules**; replaces Provv. 89757/2018 in full (point 15.1); spec updates by website publication (point 14.1) |
| 15 | `Provv_AdE_105669-2024_Modifiche_Regole_Tecniche_SdI` | Provv. Direttore AdE prot. 105669 dell'8-03-2024 | agenziaentrate.gov.it | Amends file 14 (delivery address for non-VAT entities, consultation service) |
| 16 | `Provv_AdE_433608-2022_Testo_Coordinato_2024` | AdE coordinated text of 14 + 15 (informative) | agenziaentrate.gov.it | The rules as they read today |
| 17 | `AdE_Specifiche_Tecniche_SdI_v1.9.1_Allegato_A` | Allegato A — Specifiche tecniche v1.9.1 (31-03-2026) | agenziaentrate.gov.it | SdI platform spec, usable from 2026-05-15; validation error codes (Appendice 1) |
| 18 | `Decisione_UE_2018-593_Deroga_Italia_FE` | Council Impl. Decision (EU) 2018/593 | OJ L 99, 19.4.2018 | EU authorisation for the mandate |
| 19 | `Decisione_UE_2021-2251_Proroga_Forfettari` | Council Impl. Decision (EU) 2021/2251 | OJ L 454, 17.12.2021 | Extension to 2024 + small-enterprise franchise holders |
| 20 | `Decisione_UE_2024-3150_Proroga_2027` | Council Impl. Decision (EU) 2024/3150 | OJ L 2024/3150, 19.12.2024 | Extension to 2027-12-31 |
| 21 | `SdI_Specifiche_Tecniche_v1.8.4_Allegato_B_DM55` | Specifiche delle regole tecniche di cui all'Allegato B del DM 55/2013, v1.8.4 (31-03-2026) | fatturapa.gov.it | SdI channel rules (PEC, SdICoop, SPCoop, SdIFtp), receipts/notifications incl. esito committente; valid from 2026-05-15 |
| 22 | `SdI_Elenco_Controlli_v2.0` | Elenco dei controlli effettuati sul file fattura del SdI, v2.0 (31-01-2025) | fatturapa.gov.it | SdI rejection checks and error codes — testable acceptance criteria |

## Key dates (date ≠ population — read both columns)

| Date | Obligation | Population | Legal basis |
|---|---|---|---|
| 2014-06-06 | PA accept only FatturaPA via SdI | Ministeri, Agenzie fiscali, national social-security bodies | L. 244/2007 c. 209–210; DM 55/2013 art. 6 c. 2 (12 months after 2013-06-06) |
| 2015-03-31 | Same | All other PA, incl. local | DM 55/2013 art. 6 c. 3; D.L. 66/2014 art. 25 (not held; date per fatturapa.gov.it "Normativa") |
| 2017-04-01 | Corrispettivi telematici | Vending machines | D.Lgs. 127/2015 art. 2 c. 2 |
| 2018-07-01 | E-invoicing via SdI | Fuel sales (road-station fuel: 2019-01-01); subcontractors in PA contracts | L. 205/2017 c. 917 |
| 2019-01-01 | E-invoicing via SdI, B2B and B2C | All taxable persons resident/established in Italy (forfettari/regime di vantaggio then exempt) | L. 205/2017 c. 909, 916; D.Lgs. 127/2015 art. 1 c. 3 |
| 2019-04-18 | Receive EN 16931 invoices | Contracting authorities (central) | D.Lgs. 148/2018 art. 3 |
| 2019-07-01 | Corrispettivi telematici | Volume d'affari > €400,000 | D.Lgs. 127/2015 art. 2 c. 1 |
| 2020-01-01 | Corrispettivi telematici | All other retailers | D.Lgs. 127/2015 art. 2 c. 1 |
| 2020-04-18 | Receive EN 16931 invoices | Sub-central contracting authorities | D.Lgs. 148/2018 art. 4 |
| 2022-07-01 | E-invoicing via SdI | Forfettari / regime di vantaggio with prior-year revenues > €25,000 | D.L. 36/2022 art. 18 c. 3 |
| 2022-07-01 | Cross-border data via SdI | All art. 1 c. 3 subjects | D.Lgs. 127/2015 art. 1 c. 3-bis |
| 2024-01-01 | E-invoicing via SdI | All remaining forfettari | D.L. 36/2022 art. 18 c. 3 |
| 2025-04-01 | FatturaPA format V1.4 / SdI specs 1.9 (TD29, RF20) | All SdI users | fatturapa.gov.it format page; AdE "Specifiche tecniche versione 1.9" page |
| 2026-01-01 | POS–RT link for corrispettivi | Corrispettivi operators | D.Lgs. 127/2015 art. 2 c. 3; L. 207/2024 c. 74–77 (note (24) in file 01) |
| 2026-05-15 | AdE specs 1.9.1 usable; fatturapa.gov.it SdI specs 1.8.4 valid (1.8.3 until 2026-05-14) | All SdI users | AdE "Specifiche tecniche versione 1.9.1" page (Provv. 433608/2022 point 14.1); fatturapa.gov.it "Documentazione Sistema d'Interscambio" |
| 2027-01-01 | TU IVA applies; source provisions repealed | All | D.Lgs. 10/2026 arts. 170–171 |
| 2027-01-01 | TU sanzioni applies (art. 31 replaces D.Lgs. 471/1997 art. 6) | All | D.Lgs. 173/2024 art. 102; D.L. 200/2025 art. 4 c. 1 |
| 2027-03-31 | Deadline for Italy's request to extend the derogation | Italy (Member State) | Decision 2018/593 art. 4 as replaced by 2024/3150 |
| 2027-12-31 | EU derogation expires unless extended | — | Same |

## Not held / gaps

- **Provv. AdE prot. 89757 del 30-04-2018** — superseded in full by Provv. 433608/2022 (point 15.1); not held. History: https://www.agenziaentrate.gov.it/portale/schede/comunicazioni/fatture-e-corrispettivi/normativa-e-prassi-fatture-e-corrispettivi
- **Allegati B and C of Provv. 433608/2022** (dati fattura / dati fattura integrati stored by AdE) — data lists, not obligations on the issuer; PDFs linked from https://www.agenziaentrate.gov.it/portale/provv-24-nov-2022
- **Corrispettivi technical rules** — Provv. 182017 del 28-10-2016 and its amendments, RT and "soluzione software" specs: https://www.agenziaentrate.gov.it/portale/schede/comunicazioni/fatture-e-corrispettivi/fatture-e-corrispettivi-st
- **DM 10 maggio 2019** (GU n. 120 del 24-05-2019) raising the fattura semplificata limit of DPR 633 art. 21-bis to €400 — linked from the AdE "Normativa e prassi" page; Allegato A 1.9 notes the limit no longer applies to forfettari / RF20 issuers.
- **DM 28 dicembre 2018** (GU n. 5 del 07-01-2019) amending art. 6 (stamp duty) of DM 17 giugno 2014 — file 13 is the original GU text, not consolidated.
- **DM MEF 7 marzo 2008** (SdI gestore = AdE, GU n. 103 del 03-05-2008); **D.L. 66/2014 art. 25** (B2G start date, CIG/CUP); **D.L. 119/2018 art. 10-bis** (health-data exclusion, no end date in the current text); **DPR 600/1973 art. 22** and **codice civile art. 2220** (retention period); **D.Lgs. 82/2005 (CAD)** and AgID guidelines on electronic documents — all on normattiva.it / agid.gov.it, not held.
- **D.Lgs. 5 agosto 2026, n. 141** (GU n. 181 del 06-08-2026, SO 28) — Testo unico adempimenti e accertamento plus coordination of the testi unici; its amendments are already merged into files 05 and 11. https://www.normattiva.it/uri-res/N2Ls?urn:nir:stato:decreto.legislativo:2026-08-05;141
- **San Marino** (D.L. 34/2019 art. 12; Provv. AdE 211273 del 05-08-2021) and the **CIUS-IT / EN 16931** provvedimento of 18-04-2019 — not held.
- **SdI message schemas and samples** (MessaggiTypes v1.1 XSD, notification examples) on https://www.fatturapa.gov.it/it/norme-e-regole/DocumentazioneSDI/ — not held (files 21–22 are).
- EUR-Lex refuses scripted downloads (HTTP 202 challenge); the three Council decisions were taken from the Publications Office Cellar (same OJ files), so nothing is missing.
