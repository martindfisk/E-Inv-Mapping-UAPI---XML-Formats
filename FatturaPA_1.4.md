# UAPI v6 → FatturaPA V1.4

As of 2026-11-03 · Status: draft for review

UAPI v6 invoice and correction fields mapped to FatturaPA V1.4, ordinary invoice (`FPR12`, B2B and B2C via SdI). FatturaPA is not an EN 16931 CIUS: each BT cell cites the row with the same FatturaPA ID in the AdE *Technical rules* v2.6, Appendix 4.1 (A4.1). `[A4.2]` = Appendix 4.2, `[A5.n]` = a decoding table, `EXT` = Italian extension without a BT, "not in A4.1" = no element carries that BT. The element column is the V1.4 spec target; as-built behaviour (fiskaly → GOBL → Invopop) is in the notes.

| Key | Artefact (pins in Sources) |
| --- | --- |
| `fk:` | fiskaly/meta@cd19ec8, `src/unified/` |
| `fkg:` | `fk:einvoicing/gobl/` (UAPI → GOBL mapper) |
| `EA` | `fk:domain/record/einvoicing_adaptations.go` (sync recipient and correction checks) |
| `fpa:` | invopop/gobl.fatturapa v0.90.0 (GOBL → FatturaPA converter) |
| `sdi:` | invopop/gobl v0.501.0, `addons/it/sdi/`: the it-sdi-v1 addon in fiskaly's sync build and validation. Invopop runs the converter's own copy (`fpa:addon/`; gobl ≥ v0.508.0 no longer ships `addons/it/sdi`), which matches for every behaviour cited here |
| Tab | `RappresentazioneTabellareFattOrdinaria.xlsx` (V1.4): ID, cardinality `<min.max>`, size, field checks |
| EC nnnnn | SdI error code (*Elenco dei controlli* v2.0; field checks per Tab and AdE *specifiche tecniche* v1.9.1) |

## 3. Invoice Mapping

### 3.1 Header and References

| UAPI | EN 16931 BT | FatturaPA element (ID + path) | notes |
| --- | --- | --- | --- |
| `-` | EXT; BT-3 Invoice type code via [A5.4] (TD01 = 380) | `2.1.1.1 FatturaElettronicaBody/DatiGenerali/DatiGeneraliDocumento/TipoDocumento` | fixed `TD01` (`sdi:scenarios.go:163-172`; `fpa:body.go:208`); TD04 for corrections (§5); no other TD code can be requested |
| `document.number` | BT-1 Invoice number | `2.1.1.4 FatturaElettronicaBody/DatiGenerali/DatiGeneraliDocumento/Numero` | `fkg:invoice.go:153`; `fpa:body.go:211`. UAPI pattern allows no digit → EC 00425. Same supplier/year/number → EC 00404 (TD04 may reuse a TD01 number); uniqueness not enforced by fiskaly |
| `document.series` | no BT for a series (A4.1 2.1.1.4 = BT-1) | `2.1.1.4 FatturaElettronicaBody/DatiGenerali/DatiGeneraliDocumento/Numero` | ignored: GOBL `series` never set; API docs say to prefix `document.number` |
| `document.issued_at` | BT-2 Invoice issue date | `2.1.1.3 FatturaElettronicaBody/DatiGenerali/DatiGeneraliDocumento/Data` | date part; absent → "today" at each build, so sync check and XML can differ (`fkg:invoice.go:706-722`). After SdI receipt → EC 00403 |
| `document.operation_date` | BT-7 Value added tax point date: no element [A4.2 row 9]; A4.1 rule for 2.1.1.3: use BT-72 when the operation date differs | `2.1.1.3 FatturaElettronicaBody/DatiGenerali/DatiGeneraliDocumento/Data` | ignored: GOBL `operation_date` (`fkg:invoice.go:152`, `716-719`) and `delivery.date` (`fkg:invoice.go:433-441`) are not read by fpa; `Data` is always the issue date. gap |
| `payments[i].details.currency` (i = 0) | BT-5 Invoice currency code | `2.1.1.2 FatturaElettronicaBody/DatiGenerali/DatiGeneraliDocumento/Divisa` | first payment only (`fkg:invoice.go:725-741`); default `EUR` (`fk:domain/record/transaction/receipt/payment/details.go:174`). CZK/SEK fail "invoice must be in EUR or provide exchange rate" (`sdi:bill.go:41`) |
| `document.text` | BT-22 Invoice note | `2.1.1.11 FatturaElettronicaBody/DatiGenerali/DatiGeneraliDocumento/Causale` | ignored (not read by `fkg:`). `Causale` (0..N × 200 chars) only carries the correction reason (§5); UAPI 256 chars would need a split |
| `document.payment_terms` | BT-20 Payment terms (A4.1: 2.4.1 + 2.4.2.4) | `2.4.1 FatturaElettronicaBody/DatiPagamento/CondizioniPagamento` | ignored as text: GOBL `payment.terms.notes` (`fkg:invoice.go:315-321`) is not written; `CondizioniPagamento` is derived (§3.4) |
| `document.references.purchase_order` | BT-13 Purchase order reference | `2.1.2.2 FatturaElettronicaBody/DatiGenerali/DatiOrdineAcquisto/IdDocumento` | GOBL `ordering.purchases` (`fkg:invoice.go:358-360`; `fpa:body.go:139`, `165-189`); no `Data` (2.1.2.3). UAPI 32 vs 20 ASCII: converter rule 46 (`fpa:addon/bill.go:172`), fails at Invopop only |
| `document.references.contract` | BT-12 Contract reference | `2.1.3.2 FatturaElettronicaBody/DatiGenerali/DatiContratto/IdDocumento` | GOBL `ordering.contracts` (`fpa:body.go:140`). UAPI 32 vs 20 ASCII: converter rule 46, fails at Invopop only |
| `document.references.tender` | BT-17 Tender or lot reference | `2.1.3.7 FatturaElettronicaBody/DatiGenerali/DatiContratto/CodiceCIG` | deviation: fpa writes `ordering.tender` to `2.1.4.2 DatiConvenzione/IdDocumento` (`fpa:body.go:141`), which A4.1 assigns to BT-18. UAPI 32 chars vs CIG 15; > 20 fails converter rule 46 |
| `document.references.project` | BT-11 Project reference | `2.1.3.6 FatturaElettronicaBody/DatiGenerali/DatiContratto/CodiceCUP` | ignored: GOBL `ordering.projects` (`fkg:invoice.go:337-354`) is not read by fpa. UAPI 32 chars vs CUP 15 |
| `document.references.buyer` | BT-10 Buyer reference | `2.1.2.5 FatturaElettronicaBody/DatiGenerali/DatiOrdineAcquisto/CodiceCommessaConvenzione` | ignored: GOBL `ordering.code` (`fkg:invoice.go:331-333`) is not read by fpa |
| `document.references.buyer_routing` | BT-19 Buyer accounting reference | `1.2.6 FatturaElettronicaHeader/CedentePrestatore/RiferimentoAmministrazione` | ignored: GOBL `ordering.cost` is not read by fpa |
| `document.references.despatch_advice` | BT-16 Despatch advice reference | `2.1.8.1 FatturaElettronicaBody/DatiGenerali/DatiDDT/NumeroDDT` | rejected at `POST /records` with 422: "despatch_advice requires deferred invoices, which are not yet supported" (`fk:einvoicing/fatturapa/fatturapa.go:22-26`; `fk:country/it/einvoicing/record.go:58-66`); GOBL rule 20 also needs the `deferred` tag (`sdi:bill.go:166-171`). No source for `DataDDT` (2.1.8.2, `<1.1>`) |
| `document.references.preceding_document.number` | BT-25 Preceding Invoice reference | `2.1.6.2 FatturaElettronicaBody/DatiGenerali/DatiFattureCollegate/IdDocumento` | INVOICE only (`fkg:invoice.go:191-205`, `371-383`; `fpa:body.go:137`); ignored on corrections (§5) |
| `document.references.preceding_document.issued_at` | BT-26 Preceding Invoice issue date | `2.1.6.3 FatturaElettronicaBody/DatiGenerali/DatiFattureCollegate/Data` | as above |
| `document.activity_code`, `document.offline.*`, `details.creators[i].*`, `details.training`, `details.properties` | not in A4.1 | `-` | ignored / record metadata |

### 3.2 Buyer

| UAPI | EN 16931 BT | FatturaPA element (ID + path) | notes |
| --- | --- | --- | --- |
| `recipients[i].type` | BG-7 BUYER | `1.4 FatturaElettronicaHeader/CessionarioCommittente` | selects mapper and workflow; only the invoicing recipient is mapped (`fkg:invoice.go:400-431`) |
| `recipients[i].invoicing.destination_code` (SdI code) | BT-49 Buyer electronic address | `1.1.4 FatturaElettronicaHeader/DatiTrasmissione/CodiceDestinatario` | inbox `it-sdi-code` (`fkg:inbox.go:56-63`; `fpa:transmission.go:61-76`). EC 00311 / 00312 |
| `recipients[i].invoicing.destination_code` = `0000000` | BT-49 Buyer electronic address | `1.1.4 FatturaElettronicaHeader/DatiTrasmissione/CodiceDestinatario` | no inbox (`fkg:inbox.go:57-58`); fpa default `0000000` (`fpa:transmission.go:21`, `73-75`). fiskaly requires `pec` for BUSINESS, although format spec V1.4 §1.1 also allows `0000000` when the channel is unknown |
| `recipients[i].invoicing.destination_code` = `XXXXXXX` | BT-49 Buyer electronic address | `1.1.4 FatturaElettronicaHeader/DatiTrasmissione/CodiceDestinatario` | derived: fpa writes `XXXXXXX` for any non-IT customer tax-ID country, whatever the inbox (`fpa:transmission.go:62-65`). EC 00313 if `IdFiscaleIVA` is missing or `IdPaese` = IT |
| `recipients[i].invoicing.pec` | not mapped in A4.1 | `1.1.6 FatturaElettronicaHeader/DatiTrasmissione/PECDestinatario` | inbox `it-sdi-pec` (`fkg:inbox.go:65-70`), written only without a code inbox (`fpa:transmission.go:78-97`). EC 00330 |
| `recipients[i].name` (BUSINESS) | BT-44 Buyer name | `1.4.1.3.1 FatturaElettronicaHeader/CessionarioCommittente/DatiAnagrafici/Anagrafica/Denominazione` | `fkg:recipient.go:175`; `fpa:parties.go:190-197`. UAPI 128 vs Tab 80 → EC 00200; Latin-1 (`sdi:bill.go:97-100`) |
| `recipients[i].name.forename`, `.surname` (CONSUMER) | BT-44 Buyer name | `1.4.1.3.2 FatturaElettronicaHeader/CessionarioCommittente/DatiAnagrafici/Anagrafica/Nome`, `1.4.1.3.3 .../Cognome` | deviation: joined "forename surname" into `customer.name` (`fkg:recipient.go:37`, `45`) → written as `Denominazione` (`fpa:parties.go:193-197`). Passes EC 00200, not the person form. `prefix`/`infix`/`suffix` ignored |
| `recipients[i].identification.number` (VAT, BUSINESS) | BT-48 Buyer VAT identifier | `1.4.1.1.2 FatturaElettronicaHeader/CessionarioCommittente/DatiAnagrafici/IdFiscaleIVA/IdCodice` | scheme fixed `VAT` (`fkg:identification.go:21-36`); non-EU → replaced by `OO99999999999` (`fpa:parties.go:238-243`). EC 00305 |
| `recipients[i].address.country` | BT-48 (country part) | `1.4.1.1.1 FatturaElettronicaHeader/CessionarioCommittente/DatiAnagrafici/IdFiscaleIVA/IdPaese` | derived from the address country, not the VAT prefix (`fkg:identification.go:22`; `fkg:recipient.go:38-48`). IT CONSUMER: no code → `IdFiscaleIVA` omitted (`fpa:parties.go:232-235`) |
| `recipients[i].identification.number` (TAX, PASSPORT, DOCUMENT, OTHER; BUSINESS) | BT-48 / BT-47 | `1.4.1.1.2 …/IdFiscaleIVA/IdCodice` / `1.4.1.2 …/CodiceFiscale` | **rejected** at sync for any recipient country: "business recipient identification is not a valid VAT identification" (`EA:64-67`), record `COMPLETED` + log. `FromIdentification` would fail too (`fkg:identification.go:20-38`). A business buyer therefore cannot carry a CodiceFiscale (`fkg:recipient.go:136-184` never adds `it-fiscal-code`). Independent check 2026-11-03 |
| `recipients[i].identification.number` (TAX, CONSUMER, IT) | BT-47 Buyer legal registration identifier (scheme 0210) | `1.4.1.2 FatturaElettronicaHeader/CessionarioCommittente/DatiAnagrafici/CodiceFiscale` | identity `it-fiscal-code` (`fkg:recipient.go:64-95`; `fpa:parties.go:156-158`); validated at sync (`EA:217-252`). EC 00306 / 00417 |
| `recipients[i].identification.number` (VAT or TAX, CONSUMER, non-IT) | BT-48 Buyer VAT identifier | `1.4.1.1.2 FatturaElettronicaHeader/CessionarioCommittente/DatiAnagrafici/IdFiscaleIVA/IdCodice` | `tax_id.code` (`fkg:recipient.go:103-117`); non-EU → `OO99999999999` |
| `recipients[i].identification.number` (PASSPORT, DOCUMENT, OTHER; CONSUMER, non-IT) | no BT for passport/document IDs (A4.1 1.4.1.1.2 = BT-48) | `1.4.1.1.2 FatturaElettronicaHeader/CessionarioCommittente/DatiAnagrafici/IdFiscaleIVA/IdCodice` | ignored: identity `other` (`fkg:recipient.go:118-132`) not read by fpa; `IdCodice` = `0000000` (`fpa:parties.go:233-237`). The comment at `EA:270-273` says the identifier reaches `IdCodice`; it does not |
| `recipients[i].company_id` (BUSINESS) | BT-47 Buyer legal registration identifier | `1.4.1.2 FatturaElettronicaHeader/CessionarioCommittente/DatiAnagrafici/CodiceFiscale` | ignored: identity `other`/`company` (`fkg:recipient.go:166-172`); fpa reads only `it-fiscal-code`. VAT-group buyers need it (EC 00326) |
| `recipients[i].buyer_id` (BUSINESS) | BT-46 Buyer identifier (A4.1: EORI only) | `1.4.1.3.5 FatturaElettronicaHeader/CessionarioCommittente/DatiAnagrafici/Anagrafica/CodEORI` | ignored (`fkg:recipient.go:158-165`) |
| `recipients[i].address.line.street` | BT-50 Buyer address line 1 | `1.4.2.1 FatturaElettronicaHeader/CessionarioCommittente/Sede/Indirizzo` | Tab 60 chars; Latin-1 (`sdi:org.go:14-17`) |
| `recipients[i].address.line.number` | BT-51 Buyer address line 2 | `1.4.2.2 FatturaElettronicaHeader/CessionarioCommittente/Sede/NumeroCivico` | Tab 8 chars vs UAPI 128 → EC 00200 |
| `recipients[i].address.line.suffix`, `.prefix`, `.infix` | not in A4.1 | `-` | ignored: `suffix` → GOBL `street_extra`, not written (`fpa:address.go:28-35`). Italy doc §6.2 claims street + suffix → `Indirizzo` |
| `recipients[i].address.code` | BT-53 Buyer post code | `1.4.2.3 FatturaElettronicaHeader/CessionarioCommittente/Sede/CAP` | IT: 5 digits, left-padded (`sdi:org.go:33-38`; `sdi:address.go:12-27`); non-IT: fixed `00000` (`fpa:address.go:36-41`) |
| `recipients[i].address.city` | BT-52 Buyer city | `1.4.2.4 FatturaElettronicaHeader/CessionarioCommittente/Sede/Comune` | Tab 60 chars |
| `recipients[i].address.region` | BT-54 Buyer country subdivision | `1.4.2.5 FatturaElettronicaHeader/CessionarioCommittente/Sede/Provincia` | required for IT (`fkg:address.go:42-45`); written only for IT and `^[A-Z]{2}$` (`fpa:address.go:48-55`) |
| `recipients[i].address.country` | BT-55 Buyer country code | `1.4.2.6 FatturaElettronicaHeader/CessionarioCommittente/Sede/Nazione` | — |
| `recipients[i].shipping.address.*` | BT-75 to BT-80 (A4.1 2.1.9.12.1–.6; no BG on 2.1.9.12) | `2.1.9.12.1`–`2.1.9.12.6 FatturaElettronicaBody/DatiGenerali/DatiTrasporto/IndirizzoResa/*` | ignored: GOBL `delivery.receiver` (`fkg:invoice.go:443-467`); fpa writes no `DatiTrasporto` |
| `recipients[i].shipping.date` | BT-72 Actual delivery date | `2.1.9.13 FatturaElettronicaBody/DatiGenerali/DatiTrasporto/DataOraConsegna` | ignored (not read) |
| `recipients[i].shipping.name` | not in A4.1; [A4.2 27.4.1.1] "Deliver to party name" (no BT ID): not mapped in FatturaPA | `-` | ignored |
| `recipients[i].origin`, `.invoicing.identifier`, `.invoicing.email`, `.invoicing.format` | not in A4.1 | `-` | not applicable to SDI |

### 3.3 Lines

| UAPI | EN 16931 BT | FatturaPA element (ID + path) | notes |
| --- | --- | --- | --- |
| `entries[i]` (position) | BT-126 Invoice line identifier | `2.2.1.1 FatturaElettronicaBody/DatiBeniServizi/DettaglioLinee/NumeroLinea` | derived `i + 1` (`fkg:entries.go:74-80`; `fpa:items.go:87`); `entries[i].details.number` ignored |
| `entries[i].type` | not in A4.1 | `-` | not read: RETURN is not negated; signs come from the client (`fkg:entries.go:34-85`) |
| `entries[i].data.type` = GROUP | BG-25 INVOICE LINE | `2.2.1 FatturaElettronicaBody/DatiBeniServizi/DettaglioLinee` | one line from group-level values; `entries[i].data.items[i].*` never mapped |
| `entries[i].data.text` | BT-153 Item name | `2.2.1.4 FatturaElettronicaBody/DatiBeniServizi/DettaglioLinee/Descrizione` | `fkg:entries.go:299-306`; `fpa:items.go:88`. A4.1 concatenates BT-153 + BT-154. Latin-1 (`sdi:bill.go:145-150`) |
| `entries[i].data.product.details.label` | BT-154 Item description | `2.2.1.4 FatturaElettronicaBody/DatiBeniServizi/DettaglioLinee/Descrizione` (appended) | ignored: `product.*` not read |
| `entries[i].data.product.type`, `.product.number` | BT-155, BT-157-1, BT-158-1 (2.2.1.3.1); BT-155, BT-157, BT-158 (2.2.1.3.2) | `2.2.1.3.1 FatturaElettronicaBody/DatiBeniServizi/DettaglioLinee/CodiceArticolo/CodiceTipo`, `2.2.1.3.2 .../CodiceValore` | ignored; fpa `LineDetail` has no `CodiceArticolo` (`fpa:items.go:28-43`) |
| `entries[i].data.product.code` | BT-158 Item classification identifier | `2.2.1.3.2 FatturaElettronicaBody/DatiBeniServizi/DettaglioLinee/CodiceArticolo/CodiceValore` | ignored, as above |
| `entries[i].data.unit.quantity` | BT-129 Invoiced quantity | `2.2.1.5 FatturaElettronicaBody/DatiBeniServizi/DettaglioLinee/Quantita` | always written, 2–8 dp (`fpa:items.go:89`); negative quantity → positive with negated price (`fpa:items.go:78-85`) |
| `entries[i].data.unit.measure` | BT-130 Invoiced quantity unit of measure | `2.2.1.6 FatturaElettronicaBody/DatiBeniServizi/DettaglioLinee/UnitaMisura` | GOBL unit key or UN/ECE code; default `1` or unknown → omitted (`fkg:entries.go:312-323`; `fpa:items.go:139-147`). Tab 10 chars |
| `entries[i].data.unit.factor` | BT-149: not in A4.1 | `-` | ignored |
| `entries[i].data.unit.price.inclusive` | BT-148 Item gross price / BT-146 Item net price (A4.1: BT-148 if present, else BT-146) | `2.2.1.9 FatturaElettronicaBody/DatiBeniServizi/DettaglioLinee/PrezzoUnitario` | derived net price after VAT removal at price precision + 2 dp (EUR: 4 dp), printed 2–8 dp (`fpa:fatturapa.go:105`; `fpa:items.go:91`); may be moved by cents to match the payments total (`fkg:line_rounding.go:21-42`) |
| `entries[i].data.unit.price.exclusive` | BT-146 Item net price | `2.2.1.9 FatturaElettronicaBody/DatiBeniServizi/DettaglioLinee/PrezzoUnitario` | ignored: recomputed from the inclusive price |
| `entries[i].data.value.discount.inclusive` | BT-136 Invoice line allowance amount (BG-27) | `2.2.1.10.1 FatturaElettronicaBody/DatiBeniServizi/DettaglioLinee/ScontoMaggiorazione/Tipo` = `SC`; `2.2.1.10.3 .../Importo` | only if ≠ 0; reason "Special Discount" dropped (`fkg:entries.go:45-55`); `Importo` = net amount ÷ quantity, as A4.1 states (`fpa:items.go:244-260`) |
| `entries[i].data.value.surcharge.inclusive` | BT-141 Invoice line charge amount (BG-28) | `2.2.1.10.1 FatturaElettronicaBody/DatiBeniServizi/DettaglioLinee/ScontoMaggiorazione/Tipo` = `MG`; `2.2.1.10.3 .../Importo` | as for discounts (`fkg:entries.go:57-67`; `fpa:items.go:262-272`) |
| `-` | BT-131 Invoice line net amount | `2.2.1.11 FatturaElettronicaBody/DatiBeniServizi/DettaglioLinee/PrezzoTotale` | GOBL `line.total` (`fpa:items.go:92`); §3.6; EC 00423 |
| `entries[i].data.vat.percentage` (VAT_RATE > 0) | BT-152 Invoiced item VAT rate | `2.2.1.12 FatturaElettronicaBody/DatiBeniServizi/DettaglioLinee/AliquotaIVA` | e.g. `22.00` (`fpa:items.go:107-108`); rate `code` STANDARD/REDUCED_n unused (`fkg:entries.go:110-122`). EC 00424 / 00443 |
| `entries[i].data.vat.percentage` = 0 (VAT_RATE) | BT-152; Natura per A4.1 2.2.1.14 | `2.2.1.12 .../DettaglioLinee/AliquotaIVA` = `0.00`; `2.2.1.14 FatturaElettronicaBody/DatiBeniServizi/DettaglioLinee/Natura` = `N1` | default: GOBL key `zero` → addon forces `N1` (art. 15 exclusion) (`sdi:tax_combo.go:40-41`); no other Natura possible this way |
| `entries[i].data.vat.code` (VAT_EXEMPTION, `CAUSE_n`) | BT-151 Invoiced item VAT category code (cross-border); BT-160 / BT-161 (domestic) | `2.2.1.14 FatturaElettronicaBody/DatiBeniServizi/DettaglioLinee/Natura`, with `2.2.1.12 AliquotaIVA` = `0.00` | Natura per §6.1; `0.00`: N1 gets percent 0 from the addon (`sdi:tax_combo.go:62-64`), other Natura have none (`fkg:entries.go:271-296`; `fpa:helpers.go:17-22`). `CAUSE_4`, `CAUSE_n` ≥ 23, `NOT_SUBJECT`, `NOT_TAXABLE` → 422 (`fk:einvoicing/fatturapa/vat/exemption/natura.go:81-103`, `135-157`) |
| `entries[i].data.vat.reason` | BT-120 VAT exemption reason text (A4.1: 2.2.2.2 + 2.2.2.8) | `2.2.2.8 FatturaElettronicaBody/DatiBeniServizi/DatiRiepilogo/RiferimentoNormativo` | ignored for IT suppliers (`fkg:entries.go:203-214`); fpa writes the Italian Natura label instead (`fpa:items.go:277-287`) |
| `entries[i].data.vat.type` = VAT_REVERSE_CHARGE | BT-151 (category AE [A5.1]) | `2.2.1.14 FatturaElettronicaBody/DatiBeniServizi/DettaglioLinee/Natura` (N6.x) | rejected: "unsupported VAT type … for line mapping" (`fkg:entries.go:164-169`); use `CAUSE_13`–`CAUSE_21` |
| `entries[i].data.vat.amount`, `.exclusive`, `.inclusive`, `.fiscal_location`, `.regime`, `.regime_base`, `.surcharge.*`; `entries[i].data.value.base.*`, `.discount.exclusive`, `.surcharge.exclusive` | not in A4.1 at line level | `-` | ignored; GOBL recalculates |
| `entries[i].details.description` | BT-127 Invoice line note: not in A4.1 | `-` | ignored |
| `entries[i].details.*` (other), `entries[i].data.product.details.*` (other) | not in A4.1 | `-` | ignored |

### 3.4 Payment

| UAPI | EN 16931 BT | FatturaPA element (ID + path) | notes |
| --- | --- | --- | --- |
| `payments[i].type` | BT-81 Payment means type code | `2.4.2.2 FatturaElettronicaBody/DatiPagamento/DettaglioPagamento/ModalitaPagamento` | derived (`sdi:pay.go:35-86`): CASH `MP01`, CARD `MP08`, ONLINE `MP08` ([A5.6]: UNTDID 68 → `MP05`). Vouchers and OTHER → GOBL key `other`, no MP → fault "payment advance requires 'it-sdi-payment-means'" (`sdi:pay.go:99-107`). MEAL_VOUCHER, DEPOSIT → "unsupported payment type" (`fkg:invoice.go:300-302`) |
| `-` | BT-20 Payment terms | `2.4.1 FatturaElettronicaBody/DatiPagamento/CondizioniPagamento` | derived: settled payments → `TP03` block; instruction → `TP02`, or `TP01` for several due dates or one below the amount due (`fpa:payments.go:35-73`, `201-215`). Fully paid by one CARD/CASH/ONLINE method on ≤ 1 date → rewritten to one instruction → `TP02` (`fkg:invoice.go:244-248`, `651-702`) |
| `payments[i].details.amount` (settled types) | BT-115 Amount due for payment | `2.4.2.6 FatturaElettronicaBody/DatiPagamento/DettaglioPagamento/ImportoPagamento` (TP03) | 2 dp (`fpa:payments.go:88-108`); sum of all payments is the target total for the rounding fix (`fkg:line_rounding.go:21-42`) |
| `payments[i].details.date` (settled types) | not mapped in A4.1 | `2.4.2.3 FatturaElettronicaBody/DatiPagamento/DettaglioPagamento/DataRiferimentoTerminiPagamento` | advance date (`fpa:payments.go:97-99`) |
| `payments[i].details.amount` (first OUTSTANDING) | BT-115 Amount due for payment | `2.4.2.6 FatturaElettronicaBody/DatiPagamento/DettaglioPagamento/ImportoPagamento` | due-date amount (`fkg:invoice.go:607-622`; `fpa:payments.go:126-138`) |
| `payments[i].details.date` (first OUTSTANDING) | BT-9 Payment due date | `2.4.2.5 FatturaElettronicaBody/DatiPagamento/DettaglioPagamento/DataScadenzaPagamento` | required: "payment due date not set" (`fkg:invoice.go:612`) |
| `payments[i]` (further OUTSTANDING) | BT-9, BT-115 | `2.4.2 FatturaElettronicaBody/DatiPagamento/DettaglioPagamento` | dropped (`fkg:invoice.go:277-283`), with a warning log on the async record only (`fk:einvoicing/invopop.go:709-727`); so `TP01` never comes from several OUTSTANDING payments, contrary to the Italy doc §5.2.2 |
| `payments[i].instruction.type` = CREDIT_TRANSFER | BT-81 | `2.4.2.2 FatturaElettronicaBody/DatiPagamento/DettaglioPagamento/ModalitaPagamento` = `MP05` | also the default for due dates without instruction (`fkg:fatturapa.go:38-58`) |
| `payments[i].instruction.account` | BT-84 Payment account identifier | `2.4.2.13 FatturaElettronicaBody/DatiPagamento/DettaglioPagamento/IBAN` | `fkg:invoice.go:531-536`; `fpa:payments.go:117-122` |
| `payments[i].instruction.payment_service_provider` | BT-86 Payment service provider identifier | `2.4.2.16 FatturaElettronicaBody/DatiPagamento/DettaglioPagamento/BIC` | UAPI free text 1–128 vs Tab 8–11 → EC 00200 if not a BIC |
| `payments[i].instruction.name` | BT-85: not in A4.1; [A4.2 29.4.2] not mapped. A4.1 2.4.2.12: no BT | `2.4.2.12 FatturaElettronicaBody/DatiPagamento/DettaglioPagamento/IstitutoFinanziario` (as-built) | deviation in fiskaly's mapper: account holder put into GOBL `credit_transfer.name`, which GOBL defines as the bank name (`fkg:invoice.go:531-535`); fpa writes it as `IstitutoFinanziario` (`fpa:payments.go:121`). Tab 80 chars |
| `payments[i].instruction.text` | BT-83 Remittance information | `2.4.2.21 FatturaElettronicaBody/DatiPagamento/DettaglioPagamento/CodicePagamento` | ignored: GOBL `instructions.detail` (`fkg:invoice.go:538-540`) never written |
| `payments[i].instruction.type` = DIRECT_DEBIT | BT-81 | `2.4.2.2 FatturaElettronicaBody/DatiPagamento/DettaglioPagamento/ModalitaPagamento` | as-built `MP16` (`sdi:pay.go:52`); [A5.6] maps UNTDID 49/59 to `MP19` |
| `payments[i].instruction.mandate_reference`, `.creditor_id.identifier`, `.debited_account` | BT-89, BT-91: not mapped in FatturaPA [A4.2 29.5.1, 29.5.2]; BT-90: not in A4.1 | `-` | not written (fpa reads only `credit_transfer`) |
| `payments[i].instruction.type` = UNKNOWN | BT-81 | `2.4.2.2 .../ModalitaPagamento` | rejected; message rewritten (`fkg:error_translation.go:17-19`) |
| `payments[i].number`, `.kind` (CARD), `.name` (ONLINE/OTHER), `.instruction.properties`, `.details.discount`, `.details.channel`, `.details.properties`, `.concept`, `.count`, `.document_number` | BT-87 [A4.2 29.3.1: not mapped]; BT-82 not in A4.1 | `-` | ignored |
| `-` | BT-59 Payee name | `2.4.2.1 FatturaElettronicaBody/DatiPagamento/DettaglioPagamento/Beneficiario` | not written; no UAPI source |

### 3.5 VAT Breakdown and Totals

| UAPI | EN 16931 BT | FatturaPA element (ID + path) | notes |
| --- | --- | --- | --- |
| `breakdown[i].percentage` | BT-119 VAT category rate | `2.2.2.1 FatturaElettronicaBody/DatiBeniServizi/DatiRiepilogo/AliquotaIVA` | ignored: one block per GOBL rate total (rate + Natura) (`fpa:items.go:216-242`) |
| `breakdown[i].code` (VAT_EXEMPTION) | BT-120 (A4.1); BT-121 VAT exemption reason code [A5.1] | `2.2.2.2 FatturaElettronicaBody/DatiBeniServizi/DatiRiepilogo/Natura` | ignored in the XML (Natura comes from the lines); only pre-validated (`fk:einvoicing/fatturapa/fatturapa.go:37-44`). EC 00429 / 00444 |
| `breakdown[i].exclusive` | BT-116 VAT category taxable amount | `2.2.2.5 FatturaElettronicaBody/DatiBeniServizi/DatiRiepilogo/ImponibileImporto` | ignored: derived, 2 dp. EC 00422 |
| `breakdown[i].amount` | BT-117 VAT category tax amount | `2.2.2.6 FatturaElettronicaBody/DatiBeniServizi/DatiRiepilogo/Imposta` | ignored: derived. EC 00421 |
| `breakdown[i].type`, `.inclusive`, `.code` (VAT_RATE) | not in A4.1 | `-` | ignored |
| `-` | BT-8 / BT-118 | `2.2.2.7 FatturaElettronicaBody/DatiBeniServizi/DatiRiepilogo/EsigibilitaIVA` | not written (no `it-sdi-vat-liability`); optional `<0.1>` |
| `-` | BT-120 VAT exemption reason text | `2.2.2.8 FatturaElettronicaBody/DatiBeniServizi/DatiRiepilogo/RiferimentoNormativo` | derived Natura label from the converter's addon (`fpa:items.go:277-287`); gobl v0.501.0 labels for N6.2 and N7 exceed 100 chars, the converter's copy shortens them |
| `totals.vat.inclusive` | BT-112 Invoice total amount with VAT | `2.1.1.9 FatturaElettronicaBody/DatiGenerali/DatiGeneraliDocumento/ImportoTotaleDocumento` | ignored: GOBL `total_with_tax` + rounding (`fpa:body.go:215`, `231-238`), incl. the €2.00 stamp duty; UAPI mismatch → warning log only |
| `totals.vat.amount`, `totals.vat.exclusive` | BT-110, BT-106, BT-109: not in A4.1 | `-` | ignored (warning comparison only) |
| `-` | BT-114 Rounding amount | `2.1.1.10 FatturaElettronicaBody/DatiGenerali/DatiGeneraliDocumento/Arrotondamento` | derived: fiskaly never sets it, but the converter's VAT removal fills it with the cent difference when the net recalculation shifts the total (`fpa:fatturapa.go:105` → gobl `bill/calculator.go:297-305`); 2 dp (`fpa:body.go:216`) |

### 3.6 Calculated Fields

| UAPI | EN 16931 BT | FatturaPA element (ID + path) | formula | notes |
| --- | --- | --- | --- | --- |
| `entries[i].data.unit.price.inclusive`, `.vat.percentage` | BT-146 | `2.2.1.9 .../DettaglioLinee/PrezzoUnitario` | `price.inclusive ÷ (1 + rate)` at price precision + 2 dp (EUR: 4 dp); printed 2–8 dp | `fpa:fatturapa.go:105`; gobl `bill/line.go:257` |
| `entries[i].data.value.discount.inclusive`, `.surcharge.inclusive`, `.unit.quantity` | BT-136 / BT-141 | `2.2.1.10.3 .../ScontoMaggiorazione/Importo` | net discount or charge ÷ quantity | `fpa:items.go:244-275` |
| as above | BT-131 | `2.2.1.11 .../DettaglioLinee/PrezzoTotale` | `round2(quantity × PrezzoUnitario) − discounts + charges`; holds because fiskaly sets `tax.rounding: currency` (`fkg:invoice.go:166-167`) | EC 00423 checks `(PrezzoUnitario ± sconto/maggiorazione) × Quantita` |
| lines | BT-116 | `2.2.2.5 .../DatiRiepilogo/ImponibileImporto` | `Σ PrezzoTotale` per (AliquotaIVA, Natura) | EC 00422: same sum per rate, ±€1 |
| lines | BT-117 | `2.2.2.6 .../DatiRiepilogo/Imposta` | GOBL rate amount, 2 dp | EC 00421: `ImponibileImporto × AliquotaIVA ÷ 100`, half-up, ±€0.01 |
| lines, stamp duty | BT-112 | `2.1.1.9 .../ImportoTotaleDocumento` | `Σ (ImponibileImporto + Imposta) + stamp duty + Arrotondamento` | `fpa:body.go:231-238`; not checked by SdI |
| first OUTSTANDING, totals | BT-115 | `2.4.2.6 .../ImportoPagamento` (TP01/TP02) | due-date amount; without due dates `payable − paid advances − split-payment VAT` | `fpa:payments.go:110-184` |
| exempt lines | BT-99 | `2.1.1.6.2 .../DatiBollo/ImportoBollo` | `2.00` if `Σ line.total` of 0 % lines with Natura N1, N2.1, N2.2, N3.1–N3.6, N4, N5 `> 77.47` | `fkg:fatturapa.go:15-36`, `80-97`; as-built list, wider than AdE's (§6) |
| `payments[i].details.amount` | BT-146 | `2.2.1.9 .../PrezzoUnitario` | `Σ payments − total_with_tax` moved into line prices in cents, only when all payments are in the invoice currency and the drift is explained by line rounding (±½ cent); reverted if it flips the stamp-duty decision | `fkg:line_rounding.go:16-42`, `95-104`, `164-174` |

## 4. Supplier Data (Taxpayer, System)

Taxpayer paths are relative to the `POST /taxpayers` content (`UAPI_v6_Fields.md`), read at build time (`fkg:taxpayer.go:25-81`). `seller.*` and `tax_representative.*` come from the invoice payload.

| UAPI | EN 16931 BT | FatturaPA element (ID + path) | notes |
| --- | --- | --- | --- |
| `fiscalization.vat_id_number` | BT-31 Seller VAT identifier | `1.2.1.1.2 FatturaElettronicaHeader/CedentePrestatore/DatiAnagrafici/IdFiscaleIVA/IdCodice` | preferred over `tax_id_number`; without it the codice fiscale is written as `IdCodice`; with neither, "taxpayer fiscalization does not have tax code" (`fkg:taxpayer.go:34-45`). EC 00301 (unknown or ceased); 00323 only for TD16/TD20/TD29 |
| `fiscalization.type` (= IT) | BT-31 (country part) | `1.2.1.1.1 FatturaElettronicaHeader/CedentePrestatore/DatiAnagrafici/IdFiscaleIVA/IdPaese` | `IT` (`fkg:taxpayer.go:56-70`) |
| `fiscalization.tax_id_number` | BT-30 Seller legal registration identifier (scheme 0210); API docs claim BT-32 | `1.2.1.2 FatturaElettronicaHeader/CedentePrestatore/DatiAnagrafici/CodiceFiscale` | only if both numbers are set and differ (`fkg:taxpayer.go:115-121`; `fpa:parties.go:122-124`); `tax_id_number` alone goes to `IdFiscaleIVA/IdCodice`. EC 00320 / 00322 |
| `name.legal` (+ `fiscalization.registration.legal_form`) | BT-27 Seller name; API docs claim BT-44 | `1.2.1.3.1 FatturaElettronicaHeader/CedentePrestatore/DatiAnagrafici/Anagrafica/Denominazione` | suffix `S.r.l.`, `S.p.A.`, … unless present or SOLE_PROPRIETORSHIP (`fkg:taxpayer.go:129-135`, `195-218`). 128 + suffix vs Tab 80 → EC 00200; Latin-1 (`sdi:bill.go:54-58`) |
| `name.person.forename`, `.surname` (INDIVIDUAL) | BT-27 Seller name | `1.2.1.3.2 .../Anagrafica/Nome`, `1.2.1.3.3 .../Anagrafica/Cognome` | ignored: `name.legal` always becomes `Denominazione` (`fkg:taxpayer.go:62`) |
| `fiscalization.registration.tax_regime` | EXT | `1.2.1.8 FatturaElettronicaHeader/CedentePrestatore/DatiAnagrafici/RegimeFiscale` | ORDINARY `RF01`, FLAT_RATE_SCHEME `RF19`, absent `RF01`. `tax_regime` sits in `registration`, which requires the full REA block (`fk:api/v6/definition.yaml:3402-3411`), so RF19 cannot be sent without REA data (`fkg:taxpayer.go:123-141`, `325-336`; `sdi:bill.go:20-27`; `fpa:parties.go:126-130`). Tab: RF19 = regime forfettario (the English A5.5 of the v2.6 rules lists RF18/RF19 swapped; Tab governs) |
| `address.line.street` | BT-35 Seller address line 1 | `1.2.2.1 FatturaElettronicaHeader/CedentePrestatore/Sede/Indirizzo` | taxpayer address (not location); Tab 60 chars |
| `address.line.number` | BT-36 Seller address line 2 | `1.2.2.2 FatturaElettronicaHeader/CedentePrestatore/Sede/NumeroCivico` | Tab 8 chars |
| `address.code` | BT-38 Seller post code | `1.2.2.3 FatturaElettronicaHeader/CedentePrestatore/Sede/CAP` | 5 digits for IT (`sdi:org.go:33-38`) |
| `address.city` | BT-37 Seller city | `1.2.2.4 FatturaElettronicaHeader/CedentePrestatore/Sede/Comune` | Tab 60 chars |
| `address.region` | BT-39 Seller country subdivision | `1.2.2.5 FatturaElettronicaHeader/CedentePrestatore/Sede/Provincia` | required for IT (`fkg:address.go:42-45`) |
| `address.country` | BT-40 Seller country code | `1.2.2.6 FatturaElettronicaHeader/CedentePrestatore/Sede/Nazione` | — |
| `fiscalization.registration.office` | BT-29 Seller identifier ("REA:" + Ufficio + NumeroREA) | `1.2.4.1 FatturaElettronicaHeader/CedentePrestatore/IscrizioneREA/Ufficio` | `fkg:taxpayer.go:157-159`; `fpa:parties.go:270-276`; office and entry required once the block exists (`sdi:bill.go:69-80`) |
| `fiscalization.registration.entry` | BT-29 Seller identifier | `1.2.4.2 FatturaElettronicaHeader/CedentePrestatore/IscrizioneREA/NumeroREA` | — |
| `fiscalization.registration.capital` | BT-33 Seller additional legal information | `1.2.4.3 FatturaElettronicaHeader/CedentePrestatore/IscrizioneREA/CapitaleSociale` | 2 dp (`fpa:parties.go:256-263`) |
| `fiscalization.registration.shareholder_status` | BT-33 | `1.2.4.4 FatturaElettronicaHeader/CedentePrestatore/IscrizioneREA/SocioUnico` | `SU` / `SM` (`fkg:taxpayer.go:340-349`); required with `registration`, so always written, for any legal form; spec V1.4 §1.2 limits it to S.p.A. and S.r.l. |
| `fiscalization.registration.liquidation_status` | BT-33 | `1.2.4.5 FatturaElettronicaHeader/CedentePrestatore/IscrizioneREA/StatoLiquidazione` | `LS` / `LN` (`fkg:taxpayer.go:353-362`); required with `registration` (`fk:api/v6/definition.yaml:3402-3411`), so the converter default `LN` (`fpa:parties.go:265-268`) is never used |
| `fiscalization.registration.company_id`, `name.trade`, `type`, `address.line.suffix`/`.prefix`/`.infix` | not in A4.1 | `-` | ignored |
| `seller.phone` | BT-42 Seller contact telephone number | `1.2.5.1 FatturaElettronicaHeader/CedentePrestatore/Contatti/Telefono` | `+39` stripped (`sdi:bill.go:29-35`); 5–12 chars (`sdi:bill.go:82-93`). Written in `CedentePrestatore/Contatti` (`fpa:parties.go:110`, `212-221`), not `1.1.5.1 FatturaElettronicaHeader/DatiTrasmissione/ContattiTrasmittente/Telefono` as the Italy doc §4.3 says |
| `seller.email` | BT-43 Seller contact email address | `1.2.5.3 FatturaElettronicaHeader/CedentePrestatore/Contatti/Email` | Tab 7–256 |
| `seller.name` | BT-41: not in A4.1; [A4.2 23.1.7.1] gives no FatturaPA target | `-` | ignored: GOBL `supplier.people` (`fkg:invoice.go:479-485`); fpa takes `Anagrafica` from `supplier.name` (`fpa:parties.go:190-197`) |
| `tax_representative.name` | BT-62 Seller tax representative name | `1.3.1.3.1 FatturaElettronicaHeader/RappresentanteFiscale/DatiAnagrafici/Anagrafica/Denominazione` | gap: GOBL `ordering.seller` (`fkg:invoice.go:208-213`, `387-398`); fpa has no `RappresentanteFiscale` (`fpa:header.go:9-16`) |
| `tax_representative.vat_number` | BT-63 Seller tax representative VAT identifier | `1.3.1.1.2 FatturaElettronicaHeader/RappresentanteFiscale/DatiAnagrafici/IdFiscaleIVA/IdCodice` | gap, as above |
| `tax_representative.country` | BT-63 (country part); API docs claim BT-69, not in A4.1 | `1.3.1.1.1 FatturaElettronicaHeader/RappresentanteFiscale/DatiAnagrafici/IdFiscaleIVA/IdPaese` | gap; used as tax-ID country (`fkg:invoice.go:499-519`) |
| System `registrations[i].type` = SDI; annotation `peppol_id`; location `address.*`, `name` | not in A4.1 | `-` | SDI registration is a gate (`EA:178-181`); the rest is unused for IT (`fk:worker/e_invoicing_worker.go:284-298`) |

## 5. Correction (Credit Note)

A `CORRECTION` with `data.type` = INVOICE becomes a GOBL `credit-note` (`fk:worker/e_invoicing_worker.go:196-197`) on the same workflow; all `data.*` paths map as in §3.

| UAPI | EN 16931 BT | FatturaPA element (ID + path) | notes |
| --- | --- | --- | --- |
| `data.type` = INVOICE | EXT; BT-3 via [A5.4] (TD04 = 381) | `2.1.1.1 FatturaElettronicaBody/DatiGenerali/DatiGeneraliDocumento/TipoDocumento` | fixed `TD04` (`sdi:scenarios.go:185-193`). `data.type` RECEIPT is routed to SIGN IT, not SdI (`fk:country/it/einvoicing/record.go:26-28`) |
| `record.id` | BG-3 PRECEDING INVOICE REFERENCE | `2.1.6 FatturaElettronicaBody/DatiGenerali/DatiFattureCollegate` | referenced record → `used_in` → Invopop silo entry → original code and date (`fk:worker/e_invoicing_worker.go:208-218`; `fk:einvoicing/invopop.go:1101-1129`). No `used_in` → worker error |
| `-` (original invoice) | BT-25 Preceding Invoice reference | `2.1.6.2 FatturaElettronicaBody/DatiGenerali/DatiFattureCollegate/IdDocumento` | original `Numero` (`fpa:body.go:137`, `165-172`) |
| `-` (original invoice) | BT-26 Preceding Invoice issue date | `2.1.6.3 FatturaElettronicaBody/DatiGenerali/DatiFattureCollegate/Data` | original `Data`; EC 00418 if the TD04 `Data` is earlier |
| `data.document.references.preceding_document.*` | BT-25, BT-26 | `2.1.6.2`, `2.1.6.3` | ignored on corrections (`fkg:invoice.go:191-205`), with a warning log on the async record only (`fk:einvoicing/invopop.go:690-707`) |
| `record.offset`, `data.entries[i].details.number` | not mapped in A4.1 | `2.1.6.1 FatturaElettronicaBody/DatiGenerali/DatiFattureCollegate/RiferimentoNumeroLinea` | ignored; the generic API docs text requires `details.number` on corrections (the IT_EI text calls it optional); nothing reads it |
| `reason.text` | BT-22 Invoice note | `2.1.1.11 FatturaElettronicaBody/DatiGenerali/DatiGeneraliDocumento/Causale` | GOBL note `reason` → `Causale` (`fkg:invoice.go:215-222`; `fpa:body.go:218`, `327-338`); 128 ≤ 200 chars |
| `reason.type` | BT-22 Invoice note | `2.1.1.11 FatturaElettronicaBody/DatiGenerali/DatiGeneraliDocumento/Causale` | without text: raw enum, e.g. `FACTUAL_ERROR` (`fk:service/einvoicing/correction.go:8-14`); FatturaPA has no reason code list |
| `data.recipients[i].*` | BG-7 | `1.4 FatturaElettronicaHeader/CessionarioCommittente` | must equal the original's recipients (serialized JSON, `EA:338-381`) |
| `data.entries[i].type` = RETURN, amounts | BT-131, BT-116, BT-117 | `2.2.1.11 PrezzoTotale`, `2.2.2.5 ImponibileImporto`, `2.2.2.6 Imposta` | sign convention: positive amounts (API docs); TD04 carries the reduction. Nothing negates (`fkg:line_rounding.go:112` is the only `Negate`); negative quantities are flipped (`fpa:items.go:78-85`) |
| CORRECTION that increases the amount | BT-3 = 383 [A5.4] | `2.1.1.1 .../TipoDocumento` = `TD05` (target) | gap: always TD04; DPR 633/1972 art. 26 c. 1 (from 2027-01-01: D.Lgs. 10/2026 art. 92 c. 1) requires the increase to be invoiced (TD05 or new TD01) |
| `data.payments[i].*`, stamp duty | as §3.4 | `2.4 FatturaElettronicaBody/DatiPagamento`, `2.1.1.6 .../DatiBollo` | as for invoices; the stamp-duty check ignores the document type (`fkg:fatturapa.go:64-97`) |

EC 00404 lets a TD04 share supplier, year and number with a TD01 (*Elenco controlli* v2.0, duplicate check). A decrease agreed between the parties is allowed only within one year of the operation (DPR 633/1972 art. 26 c. 3). `reason.type` INSOLVENCY corresponds to art. 26 c. 3-bis lett. a); UNCOLLECTIBLE qualifies only after a failed individual enforcement (lett. b, c. 12). fiskaly checks neither. From 2027-01-01 these provisions are D.Lgs. 10/2026 (TU IVA) art. 92 c. 3 and c. 4 (as of 2026-11-03).

## 6. Fixed and Derived Values

Values already described in §3–§5 (TipoDocumento, CodiceDestinatario, Natura, CondizioniPagamento, ModalitaPagamento, NumeroLinea, Divisa, Data defaults) are not repeated.

| UAPI | EN 16931 BT | FatturaPA element (ID + path) | notes |
| --- | --- | --- | --- |
| `-` | no BT (A4.1 1.1.3: "automatically generated by the mapper"); root `versione` not in A4.1 | root `versione`; `1.1.3 FatturaElettronicaHeader/DatiTrasmissione/FormatoTrasmissione` | fixed `FPR12` (`sdi:scenarios.go:142-161`; `fpa:transmission.go:53-59`); `FormatoTrasmissione` only with transmitter data |
| `-` | EXT / BT-31 (1.1.1.1); EXT / BT-30, BT-31 (1.1.1.2) | `1.1.1.1 FatturaElettronicaHeader/DatiTrasmissione/IdTrasmittente/IdPaese`, `1.1.1.2 .../IdCodice` | Invopop transmitter via `WithTransmitterData` (`fpa:transmission.go:41-45`; `fpa:options.go:11-33`). UNVERIFIED. Italy doc §6.2 claims taxpayer country and VAT number |
| `-` | no BT (A4.1 1.1.2: "automatically generated by the mapper") | `1.1.2 FatturaElettronicaHeader/DatiTrasmissione/ProgressivoInvio` | first 8 chars of the envelope UUID, only with transmitter data (`fpa:transmission.go:46`). UNVERIFIED |
| `-` | EXT | `1.2.1.8 FatturaElettronicaHeader/CedentePrestatore/DatiAnagrafici/RegimeFiscale` | default `RF01` (fiskaly, addon and converter) |
| `-` | BT-53 | `1.4.2.3 FatturaElettronicaHeader/CessionarioCommittente/Sede/CAP` | `00000` for non-IT addresses (`fpa:address.go:10-12`, `36-41`) |
| `-` | BT-48 | `1.4.1.1.2 FatturaElettronicaHeader/CessionarioCommittente/DatiAnagrafici/IdFiscaleIVA/IdCodice` | `0000000` (non-IT, no code), `OO99999999999` (non-EU with code) (`fpa:parties.go:223-249`) |
| `-` | BT-120 | `2.2.2.8 FatturaElettronicaBody/DatiBeniServizi/DatiRiepilogo/RiferimentoNormativo` | Italian Natura label |
| `-` | BT-105 / BT-98 | `2.1.1.6.1 FatturaElettronicaBody/DatiGenerali/DatiGeneraliDocumento/DatiBollo/BolloVirtuale` | `SI` when the stamp-duty charge exists (`fpa:body.go:253-264`) |
| `-` | BT-99 / BT-92 | `2.1.1.6.2 FatturaElettronicaBody/DatiGenerali/DatiGeneraliDocumento/DatiBollo/ImportoBollo` | `2.00`, added by fiskaly when the exempt total > €77.47 (`fkg:fatturapa.go:60-75`; `fkg:invoice.go:233-238`); €2.00 and the €77.47 threshold match DPR 642/1972 Tariffa I art. 13 c. 1 and nota 2 lett. a) (AdE Risposta 45/2024). The Natura list is wider than the law: AdE's own check (Elenco B, guide of June 2026) uses N2.1, N2.2, N3.5, N3.6, N4 only; N3.1 is exempt (Tabella B art. 15, Risposta 45/2024); the exemption of N3.2 (DL 331/1993 art. 66 c. 5) and of N5 is UNVERIFIED. No way to send the NB1–NB3 exclusion codes. As of 2026-11-03. The charge has no taxes: it raises `ImportoTotaleDocumento` and the payable amount but appears in no line or `DatiRiepilogo`. The rounding fix assumes "the payments include it" (`fkg:line_rounding.go:16-20`): clients must add €2.00 to `payments[i].details.amount`, else an amount stays due |
| `-` | no BT (A4.1 2.2.1.10.1 rule: BT-136 → SC, BT-141 → MG; BG-27/BG-28 sit on 2.2.1.10) | `2.2.1.10.1 .../ScontoMaggiorazione/Tipo` | `SC` discount, `MG` surcharge |

### 6.1 `CAUSE_n` → Natura

Sources: `fk:einvoicing/fatturapa/vat/exemption/natura.go:81-103`; `fkg:entries.go:271-296`; `sdi:tax_combo.go:57-83` (key after addon normalisation). BT-118 per [A5.1]. Bollo: counted towards the €77.47 threshold as-built (`fkg:fatturapa.go:23-36`); see §6 for the legal list.

| `entries[i].data.vat.code` | Natura (2.2.1.14 / 2.2.2.2) | BT-118 [A5.1] | GOBL key | Bollo |
| --- | --- | --- | --- | --- |
| `CAUSE_1` | N1 | Z | zero | yes |
| `CAUSE_2` / `CAUSE_3` | N4 / N5 | E | exempt | yes |
| `CAUSE_4` | none: 422 "invalid VAT exemption code 'CAUSE_4'" | — | — | — |
| `CAUSE_5` / `CAUSE_6` | N2.1 / N2.2 | E | outside-scope | yes |
| `CAUSE_7`, `_9`, `_10`, `_11` | N3.1, N3.3, N3.4, N3.5 | G | export | yes |
| `CAUSE_8`, `_12` | N3.2, N3.6 | K | intra-community | yes |
| `CAUSE_13` … `CAUSE_21` | N6.1 … N6.9 | AE | reverse-charge | no |
| `CAUSE_22` | N7 | K | outside-scope | no |
| `CAUSE_n` (n ≥ 23), `NOT_SUBJECT`, `NOT_TAXABLE` | none: 422 | — | — | — |

## 7. Format Elements Without a UAPI Source (Gaps)

Mandatory in FPR12 (cardinality 1 or required by an SdI control) and not covered:

| element | EN 16931 BT | cardinality / control | status today | proposal |
| --- | --- | --- | --- | --- |
| `1.1.1 FatturaElettronicaHeader/DatiTrasmissione/IdTrasmittente` | EXT / BT-31 (1.1.1.1); EXT / BT-30, BT-31 (1.1.1.2) | `<1.1>`; EC 00300 | Invopop-side, only with transmitter data (`fpa:transmission.go:41-48`). UNVERIFIED | Confirm the transmitter ID used by `sdi-it.generate` |
| `1.1.2 .../ProgressivoInvio`, `1.1.3 .../FormatoTrasmissione` | no BT (A4.1: generated by the mapper) | `<1.1>`; EC 00428 (1.1.3 vs root `versione` only) | as above. UNVERIFIED | as above |
| `1.2.1.8 .../RegimeFiscale` | EXT | `<1.1>` | only RF01, RF19 expressible; RF19 only together with a full REA block | Extend `tax_regime` to RF01, RF02, RF04–RF20 (RF03 withdrawn) |
| `2.1.1.1 .../TipoDocumento` | EXT; BT-3 via [A5.4] | `<1.1>` | TD01, TD04 only | Document-type selector; at least TD05 (art. 26 c. 1) and TD24 (deferred) |
| `2.1.1.3 .../Data` (operation date ≠ issue date) | BT-2 / BT-72 | `<1.1>`; EC 00403 | issue date only; `document.operation_date` dropped | Write the operation date when it differs (A4.1 rule); needs Invopop support |
| `2.1.8.2 .../DatiDDT/DataDDT` | EXT | `<1.1>` in `DatiDDT` | no source, so `despatch_advice` is rejected | Add despatch date and deferred flag |
| `2.4.2.2 .../ModalitaPagamento` | BT-81 | `<1.1>` per `DettaglioPagamento` | no MP for vouchers, OTHER, UNKNOWN; MEAL_VOUCHER, DEPOSIT rejected; direct debit MP16 vs MP19 [A5.6] | Explicit MP override per payment; DIRECT_DEBIT → MP19 |
| `1.4.1.2 .../CessionarioCommittente/DatiAnagrafici/CodiceFiscale` (BUSINESS) | BT-47 | `<0.1>`; required for VAT-group buyers (EC 00326); EC 00324 | `company_id` dropped | Map IT `company_id` to `it-fiscal-code` |
| `2.2.1.14 .../Natura` (`CAUSE_4`, `CAUSE_n` ≥ 23, 0 % VAT_RATE) | BT-151 / BT-160–161 | required when `AliquotaIVA` = 0 (EC 00400, 00429) | 422, or silent N1 | Define or retire `CAUSE_4`; reject 0 % VAT_RATE for IT suppliers |

Optional elements UAPI cannot express, or that are not written today:

| element | EN 16931 BT | cardinality / control | status today | proposal |
| --- | --- | --- | --- | --- |
| `2.1.1.5 .../DatiRitenuta` | EXT | `<0.N>`; required when a line (2.2.1.13, EC 00411) or a pension-fund block (2.1.1.7.6, EC 00415) has `Ritenuta` = SI | no source | Withholding block (RT01–RT06) + line flag; fpa supports retained taxes |
| `2.1.1.7 .../DatiCassaPrevidenziale` | BT-99, BT-100, BT-101, BT-103, BT-104 | `<0.N>`; EC 00413, 00414 | no source; fpa supports a `fund-contribution` charge (`fpa:body.go:266-301`) | Pension-fund charge (TC01–TC22) |
| `2.1.1.6 .../DatiBollo` | BT-105 / BT-98 (2.1.1.6.1); BT-99 / BT-92 (2.1.1.6.2) | `<0.1>` | automatic, with a Natura list wider than AdE's (§6) | Align the list with the law (at least drop N3.1); explicit stamp-duty flag and NB1–NB3 codes; document that payments must include €2.00 |
| `2.2.2.7 .../EsigibilitaIVA` | BT-8 / BT-118 | `<0.1>`; EC 00420 | not written | VAT liability `I` / `D` / `S` |
| `2.2.1.16 .../AltriDatiGestionali` | BG-32 (BT-160, BT-161) | `<0.N>`; EC 00477 reads `RiferimentoTesto` (declaration-of-intent protocol) | no source; N3.5 lines carry no protocol | Line attributes (`TipoDato`, `RiferimentoTesto`/`Numero`/`Data`) |
| `2.2.1.3 .../CodiceArticolo` | BG-31 (BT-155, BT-157, BT-158) | `<0.N>` | `product.*` not read; no fpa element | Map `product.*`; needs converter support |
| `2.2.1.7 .../DettaglioLinee/DataInizioPeriodo`, `2.2.1.8 .../DettaglioLinee/DataFinePeriodo` | BT-134, BT-135 | `<0.1>` | no source | Line period |
| `2.2.1.2 .../TipoCessionePrestazione` | BT-160 / BT-161 | `<0.1>` | no source | Optional |
| `1.3 FatturaElettronicaHeader/RappresentanteFiscale` | BG-11 (BT-62, BT-63) | `<0.1>` | UAPI has it, fpa drops it | Converter support |
| `2.1.2.5`, `1.2.6`, `2.1.3.6`, `2.1.3.7` (CodiceCommessaConvenzione, RiferimentoAmministrazione, CodiceCUP, CodiceCIG) | BT-10, BT-19, BT-11, BT-17 | `<0.1>` | UAPI fields exist; fiskaly maps them to GOBL `ordering.projects`/`code`/`cost` (not read) and tender to `DatiConvenzione`. fpa already writes CIG, CUP and order code from reference identities (`fpa:body.go:173-186`) | fiskaly mapping to reference identities; converter support only for 1.2.6; UAPI max 15 for CIG/CUP |
| `2.1.1.11 .../Causale` | BT-22 | `<0.N>` × 200 | `document.text` ignored | Map, split at 200 chars |
| `2.4.2.21 .../CodicePagamento` | BT-83 | `<0.1>` | `instruction.text` not written | Converter mapping |
| `2.1.9 .../DatiTrasporto` | BG-13 (BT-72, BT-75–BT-80) | `<0.1>` | `shipping.*` ignored | Converter mapping |
| `1.4.1.3.2 Nome`, `1.4.1.3.3 Cognome` (CONSUMER) | BT-44 | EC 00200 either/or | joined into `Denominazione` | Map to GOBL `people[0]` with an empty `name` (`fpa:parties.go:193-206`); possible only without a `tax_id.code` (`sdi:bill.go:109-112`), i.e. IT consumers |
| `2.1.6.1 .../RiferimentoNumeroLinea` | not mapped in A4.1 | `<0.N>` | not written | Optional (partial credit notes) |
| `2.1.1.12 Art73`, `1.5`, `1.6`, `2.1.10`, `2.3`, `2.5` | EXT (2.1.1.12, 1.5.x, 1.6, 2.1.10.x); BT-161 (2.3.1, 2.3.2, with BT-160 = "DatiVeicoli"); BG-24 (2.5) | `<0.1>` / `<0.N>` | no source | Out of scope |

## 8. Common Validation Failures

### 8.1 fiskaly side

| Stage | Message (abridged) | Trigger | Source |
| --- | --- | --- | --- |
| API 4xx | `consumer recipients only support SDI invoicing`; `consumer recipients require destination code '0000000' … or 'XXXXXXX'` | CONSUMER with other channel or code | `fk:domain/record/recipient/consumer_recipient.go:126-138` |
| API 4xx | `invoicing.pec is required when destination_code is '0000000'`; `invalid pec: …` | BUSINESS `0000000` without `pec`; bad `pec` | `fk:domain/record/recipient/business_recipient.go:156-161`; `fk:api/v6/record_handler.go:3407` |
| API 422 | `despatch_advice requires deferred invoices…`; `invalid VAT exemption code 'CAUSE_<n>'` / `… expected 'CAUSE_<N>'` | despatch advice; `CAUSE_4`, ≥ 23, `NOT_SUBJECT`, `NOT_TAXABLE` | `fk:einvoicing/fatturapa/fatturapa.go:22-44`; `fk:einvoicing/fatturapa/vat/exemption/natura.go:135-157` |
| Sync (record `COMPLETED` + log) | `recipients[0].identification.number: …`; `business recipient identification is not a valid VAT identification`; `… expected 'SDI' for Italian E_INVOICE_SERVICE` | BUSINESS identification not VAT (any country); IT partita IVA invalid; IT `invoicing.type` ≠ SDI | `EA:48-82` |
| Sync | `'XXXXXXX' is reserved for foreign business recipients…`; `foreign business recipients require 'XXXXXXX'…`; `pec: not allowed for foreign business recipients…` | routing vs recipient country | `EA:108-139` |
| Sync | `consumer.identification: TAX identification (codice fiscale) is required…`; `… expected 'TAX'…`; `an identification is required for foreign B2C…` | B2C identification | `EA:217-279` |
| Sync | `supplier is not registered for SDI` | registrations without SDI | `EA:178-181` |
| Sync, GOBL build | `region is required for Italy`; `unsupported VAT type … 'REVERSE_CHARGE'…`; `unsupported payment type: MEAL_VOUCHER` / `DEPOSIT`; `payment due date not set` | IT address without region; VAT_REVERSE_CHARGE; MEAL_VOUCHER, DEPOSIT; OUTSTANDING without date | `fkg:address.go:42-45`; `fkg:entries.go:164-169`; `fkg:invoice.go:300-302`, `612` |
| Sync, GOBL validation | `payment advance requires 'it-sdi-payment-means'…`; `Payment instruction type UNKNOWN is not supported for Italy…`; `invoice must be in EUR…` | vouchers, OTHER; UNKNOWN instruction; CZK/SEK | `sdi:pay.go:99-107`; `fkg:error_translation.go:17-19`; `sdi:bill.go:41` |
| Sync, GOBL validation | Latin-1 faults (names, street, locality); `Italian address code must be 5 digits`; telephone 5–12; REA office/entry missing | text and format | `sdi:bill.go:54-93`, `140-152`; `sdi:org.go:9-38` |
| Sync, correction | `invoice correction recipients do not match the original invoice`; `referenced record … could not be found` | changed recipients; bad `record.id` | `EA:338-381`; `fk:domain/record/service_v1.go:3645-3666` |
| Worker | `referenced record is missing used_in identifier` | original never reached Invopop | `fk:worker/e_invoicing_worker.go:213-216` |

A sync failure leaves the record `COMPLETED` with only an error log (`fk:domain/record/service_v1.go:3739-3766`); clients must read `logs` to learn that nothing was sent. GOBL faults are prefixed `invalid gobl invoice: ` and joined with `; ` (`fk:einvoicing/invopop.go:674-678`; `fkg:error_translation.go:37-46`), so a log with several faults can be cut off at 256 chars.

### 8.2 SdI side (most likely here)

| Code | Element | Trigger in this flow | Prevention |
| --- | --- | --- | --- |
| 00200 | any | length over XSD: `Denominazione` > 80 (incl. legal-form suffix), `NumeroCivico` > 8, `Indirizzo`/`Comune` > 60, `BIC` not 8–11, `IstitutoFinanziario` > 80 | align UAPI lengths with Tab for IT |
| 00305 / 00306 | 1.4.1.1.2 / 1.4.1.2 | buyer partita IVA / codice fiscale unknown to Anagrafe Tributaria (format passes at fiskaly) | client data |
| 00311 / 00312 | 1.1.4 | code not registered / not active | client data |
| 00313 | 1.1.4 | `XXXXXXX` with `IdPaese` IT or without `IdFiscaleIVA` | blocked by `EA:108-139` |
| 00326 | 1.4.1.2 | VAT-group buyer without `CodiceFiscale` (no UAPI source). Listed in the tabellare V1.4 (row 1.4.1.2) and AdE *Specifiche tecniche* v1.9.1 Allegato A, not in Elenco Controlli v2.0. Related: 00325 (CF not a group member), 00327 (new in 1.9.1, Allegato A only: group CF without IdFiscaleIVA) | gap §7 |
| 00403 | 2.1.1.3 | `document.issued_at` after SdI receipt | client clock / time zone |
| 00404 | 2.1.1.4 | supplier/year/`Numero` already accepted (TD01 vs TD04 allowed) | uniqueness not enforced (Italy doc) |
| 00418 | 2.1.1.3 / 2.1.6.3 | TD04 dated before the original | none |
| 00421 / 00422 / 00423 | 2.2.2.6 / 2.2.2.5 / 2.2.1.11 | rounding of gross prices; GOBL computes, cent nudges stay small | tolerances ±0.01 / ±€1 per rate / ±0.01 |
| 00425 | 2.1.1.4 | `document.number` without a digit | add an IT digit check |
| 00400 / 00429 / 00444 | 2.2.1.14 / 2.2.2.2 | Natura missing or inconsistent; prevented by §6.1 and GOBL | — |
| 00471 | 2.1.1.1 | supplier = buyer on TD01 (Allegato A v1.9.1 App. 1; Elenco v2.0 omits TD01) | none |
| 00477 | 2.2.1.16.2 | N3.5 with an invalidated declaration of intent; protocol cannot be sent | gap §7 |
| 00330 | 1.1.6 | `pec` is an SdI PEC box | none |

Invopop-side faults return through the webhook; the record goes `FAILED` with the message in `logs` (Italy doc §1.4, §2.1).

## Sources

| Source | Version / pin | Location | Used for | Retrieved |
| --- | --- | --- | --- | --- |
| AdE, Technical rules relating to the management of invoices pursuant to art. 3 c. 1 D.Lgs. 148/2018 (English; Italian "Specifiche tecniche fatturazione europea") | v2.6 of 15/05/2025; App. 4.1 identical to v2.4 (whitespace only) | https://www.fatturapa.gov.it/export/documenti/Technical-Rules-for-European-Invoicing-v2.6.pdf (sha256 `4a7f6a34…6c60`), `…/Specifiche-Tecniche-Fatturazione-Europea-v2.6.pdf` | BT column (A4.1, A4.2, A5.1, A5.4, A5.6) | 2026-11-03 |
| Same, earlier versions | v2.4 of 15/05/2024; v2.5 of 22/11/2024 | agenziaentrate.gov.it `…/Technical+Rules+for+European+Invoicing+v2.4.pdf`; fatturapa.gov.it `…-v2.5.pdf` | A4.1 diff | 2026-11-03 |
| FatturaPA format specification | V1.4 of 31/01/2025 (adds TD29, RF20, new TD20 description); valid from 2025-04-01 per fatturapa.gov.it; XSD v1.2.3 (FPA12/FPR12) per Allegato A v1.9.1 App. 2 | `Italy FatturaPA/Specifiche_tecniche_del_formato_FatturaPA_V1.4.pdf` | element semantics, `CodiceDestinatario`, `SocioUnico` | user-supplied |
| Rappresentazione tabellare, fattura ordinaria | unversioned file; content matches V1.4 (TD29, RF20) | `Italy FatturaPA/RappresentazioneTabellareFattOrdinaria.xlsx` | IDs, paths, cardinality, sizes, field checks, code lists | user-supplied |
| SdI *Elenco dei controlli* | v2.0 of 31/01/2025 | `Italy FatturaPA/Laws/22_SdI_Elenco_Controlli_v2.0.md` | EC codes, calculation and duplicate rules | 2026-10-29 |
| AdE *Specifiche tecniche* (Allegato A, Provv. 433608/2022) | v1.9.1 of 31/03/2026, usable from 2026-05-15 | `Italy FatturaPA/Laws/17_AdE_Specifiche_Tecniche_SdI_v1.9.1_Allegato_A.md` | EC 00313, 00325–00327, 00330, 00471 (TD01 scope) | 2026-10-29 |
| SdI technical rules, Allegato B DM 55/2013 | v1.8.4 of 31/03/2026 | `Italy FatturaPA/Laws/21_SdI_Specifiche_Tecniche_v1.8.4_Allegato_B_DM55.md` | channel context | 2026-10-29 |
| DPR 633/1972 art. 26; D.Lgs. 148/2018; D.Lgs. 10/2026 (TU IVA) art. 92, applies from 2027-01-01 | consolidated text as of 2026-10-29; GU n. 14 of 17-01-2019 | `Italy FatturaPA/Laws/04_…`, `08_…`, `05_…` | variations; legal basis of the technical rules | 2026-10-29 |
| DPR 642/1972, Tariffa I art. 13 and nota 2; Tabella B art. 15 | consolidated text on Normattiva, as of 2026-11-03 | https://www.normattiva.it/uri-res/N2Ls?urn:nir:presidente.repubblica:decreto:1972-10-26;642!vig=2026-11-03 | stamp duty amount, threshold, export exemption | 2026-11-03 |
| AdE Risposta n. 45/2024 | 2024 | https://www.agenziaentrate.gov.it/portale/documents/20143/5866231/Risposta+n.+45_2024.pdf/42d7fe7c-9cd8-baec-772c-365eac09e3c8 | €77.47 threshold; art. 15 covers exports only | 2026-11-03 |
| AdE guide *L'imposta di bollo sulle fatture elettroniche* | June 2026 | https://www.agenziaentrate.gov.it/portale/documents/d/guest/l-imposta_di_bollo_sulle_fatture_elettronichegiugno2026 | Elenco B criteria (Natura list, NB1–NB3) | 2026-11-03 |
| UAPI v6 definition and field inventory | `fiskaly/meta@cd19ec88cb299ee41a44302448006340dd028bf0` | `UAPI_v6_Fields.md` | paths, constraints, docs' BT claims | 2026-11-03 |
| fiskaly as-built code and Invopop workflows | same commit: `src/unified/{einvoicing,country/it,domain/record,service/einvoicing,api/v6,worker}/**`, `cmd/invopop-workflow-migration/workflows/IT_SEND_INVOICE_SDI_FATTURA_PA_*.json` | local copy of the commit | as-built notes, §8.1 | 2026-11-03 |
| invopop/gobl | v0.501.0 (commit 225e39d), pinned in fiskaly `go.mod` | `addons/it/sdi/` | addon defaults, rules, MP and Natura normalisation in fiskaly's sync build; diffed against the converter's `addon/` copy, identical for every cited behaviour | 2026-11-03 |
| invopop/gobl.fatturapa | v0.90.0 (commit ab6bb84, 2026-10-27). Its `go.mod` declares module `github.com/invopop/gobl.it.sdi`, requires gobl v0.508.0 and ships its own `addon/`; the version Invopop runs is not visible | GitHub `invopop/gobl.fatturapa` | GOBL → XML behaviour | 2026-11-03 |
| fiskaly Italy e-invoicing doc | same commit | `doc/unified/e-invoice/italy.md` | product behaviour, SdI outcome mapping; four statements contradicted by code (§3.2, §3.4, §4, §6) | 2026-11-03 |

## Verification Log

| # | Claim | How checked | Result |
| --- | --- | --- | --- |
| 1 | An official EN 16931 ↔ FatturaPA correspondence exists; A4.1 v2.6 = v2.4 | Downloaded v2.4, v2.5, v2.6; extracted App. 4.1 and diffed v2.4 vs v2.6 (whitespace only) | CONFIRMED |
| 2 | Output is `FPR12`, TD01 for invoices, TD04 for corrections | `sdi:scenarios.go:142-193`; `fpa:fatturapa.go:127`; `fpa:body.go:208`; `fk:worker/e_invoicing_worker.go:192-197` | CONFIRMED |
| 3 | `RappresentanteFiscale` is never emitted although UAPI sends `tax_representative.*` | `fpa:header.go:9-16` (no field); no fpa reference to `Ordering.Seller` | CONFIRMED |
| 4 | `vat.reason` ignored for IT suppliers; `RiferimentoNormativo` = Natura label | `fkg:entries.go:203-214`; `fpa:items.go:277-287` | CONFIRMED |
| 5 | `CAUSE_4` and `CAUSE_n` ≥ 23 have no Natura and are rejected | `fk:einvoicing/fatturapa/vat/exemption/natura.go:81-103`, `150-157`; `fk:country/it/einvoicing/record.go:58-66` | CONFIRMED |
| 6 | €2.00 stamp duty added automatically, raising total and payable; clients must include it in payments | `fkg:fatturapa.go:60-97`; `fkg:line_rounding.go:16-20`; `fpa:body.go:213`, `231-238`, `253-264` | CONFIRMED in code. €2.00 and €77.47 CONFIRMED (DPR 642/1972 Tariffa I art. 13, nota 2 lett. a; AdE Risposta 45/2024). Natura list WIDER than AdE Elenco B (guide June 2026); N3.1 exempt (Tabella B art. 15); N3.2 and N5 exemption UNVERIFIED |
| 7 | IT records stay `ACCEPTED` until the SdI webhook | `fk:domain/record/service_v1.go:3703-3713` | CONFIRMED |
| 8 | VAT_REVERSE_CHARGE, MEAL_VOUCHER, DEPOSIT rejected; vouchers and OTHER fail on the missing MP code | `fkg:entries.go:164-169`; `fkg:invoice.go:300-302`; `sdi:pay.go:35-60` (no plain `other`), `99-107` | CONFIRMED |
| 9 | REA block and RegimeFiscale from `fiscalization.registration.*`; RF19 = forfettario | `fkg:taxpayer.go:123-192`, `325-362`; `fpa:parties.go:126-130`, `251-277`; Tab codifiche | CONFIRMED |
| 10 | CONSUMER names are written as `Denominazione`, not `Nome`/`Cognome` | `fkg:recipient.go:37`, `45`; `fpa:parties.go:190-197` | CONFIRMED |
| 11 | `IdTrasmittente`, `ProgressivoInvio`, `FormatoTrasmissione` are filled by Invopop in production | `fpa:transmission.go:41-48` requires `WithTransmitterData`; Invopop's `sdi-it.generate` configuration is not visible | UNVERIFIED |
| 12 | Invopop runs gobl.fatturapa v0.90.0 | not visible to fiskaly; v0.90.0 also differs in module path and gobl version from fiskaly's gobl v0.501.0 | UNVERIFIED |


### Independent verification pass (2026-11-03)

Two separate checkers re-tested the decision-critical claims against code (fiskaly `cd19ec8`, gobl v0.501.0, the vendored converters) and against primary sources. Neither was given the page.

| Claim | Verdict | Evidence / correction |
| --- | --- | --- |
| `VAT_REVERSE_CHARGE`, `MEAL_VOUCHER`, `DEPOSIT` rejected | CONFIRMED | `entries.go:164-169`; `invoice.go:300-301` |
| Business buyer's codice fiscale silently dropped | **REFUTED → corrected** | A BUSINESS recipient with TAX/PASSPORT/DOCUMENT/OTHER identification is **rejected** at sync, in any country (`einvoicing_adaptations.go:64-67`: "business recipient identification is not a valid VAT identification"); `identification.go:20-38` would fail too. `company_id` is still dropped, so a business buyer can never carry a CodiceFiscale. Row added in §3.2 |
| DIRECT_DEBIT → MP16; official mapping MP19 | CONFIRMED | `invoice.go:555-556` → `addons/it/sdi/pay.go:52` (`MP16`); AdE Technical Rules v2.6 Appendix 5.6: "59 SEPA direct debit → MP19" |
| `references.tender` → DatiConvenzione instead of CodiceCIG | CONFIRMED | `invoice.go:364-366`; `gobl.fatturapa/body.go:46,141`; AdE Technical Rules v2.6 Appendix 4.1, element 2.1.3.7 `DatiContratto/CodiceCIG` ← BT-17 (max 15 characters, BR-IT-070) |
| `CAUSE_4` has no Natura mapping and errors | CONFIRMED | `natura.go:81-103`, `150-156`; pre-check `fatturapa.go:29-35` returns 422 |
| €2.00 bollo added by fiskaly | CONFIRMED | `fatturapa.go:64-75`, threshold €77.47 on N1–N5 lines (`:17-36`, `:96`); N6.x and N7 do not count. Legal basis and Natura list: Verification Log #6 |
| SdI 00326 source | PARTLY CONFIRMED | The code is right, but it is defined in the AdE *Specifiche tecniche* v1.9.1 Allegato A, not in Elenco Controlli v2.0; the tabellare V1.4 also lists it. Corrected in §8.2 |
| Appendix 5.5 of the English Technical Rules v2.6 swaps RF18/RF19 | CONFIRMED | English: "RF18 Flat-rate scheme", "RF19 Other". The Italian v2.6 text and the tabellare give RF19 = regime forfettario, which is correct |
| CORRECTION → TD04; no TD05 / upward path | CONFIRMED | `e_invoicing_worker.go:196-197`; `addons/it/sdi/scenarios.go:186-193` |

### Second validation pass (2026-11-03)

Every cell in §3–§8 re-checked against fiskaly `cd19ec8`, gobl v0.501.0, gobl.fatturapa v0.90.0 (its `addon/` diffed against gobl's `addons/it/sdi`), the V1.4 tabellare, *Elenco controlli* v2.0, Allegato A v1.9.1, the AdE Technical Rules v2.6 (A4.1, A4.2, A5), DPR 642/1972 and AdE stamp-duty guidance.

| Area | Checked | Corrected |
| --- | --- | --- |
| fiskaly code citations and behaviour | 156 | 16 |
| gobl addon and converter citations | 105, plus uncited claims | 11 |
| FatturaPA IDs, sizes, EC codes, legal references | 120 IDs, 37 EC codes | 15 |
| BT cells and UAPI paths | 186 BT cells, ~150 paths | 14 |

Corrections that change behaviour statements: `Arrotondamento` is filled by the converter's VAT removal (not "never set"); converter rule 46 limits purchase-order, contract and tender references to 20 ASCII characters (UAPI allows 32), so they fail at Invopop only; the stamp-duty Natura list is wider than AdE's; a taxpayer without `vat_id_number` sends the codice fiscale as `IdCodice`; a BUSINESS recipient without VAT identification is rejected by `EA:64-67` in any country; `liquidation_status` and `shareholder_status` are required, so the `LN` default is never used and `SocioUnico` is always written.
