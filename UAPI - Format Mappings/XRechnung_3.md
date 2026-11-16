# UAPI v6 → XRechnung 3.0.2 (UBL)

As of 2026-11-16 · Status: draft for review

UAPI v6 invoice and correction fields mapped to XRechnung 3.0.2 (UBL 2.1), which Invopop generates with `doc_type` `xrechnung-invoice-v2`. A DE supplier's invoice becomes XRechnung when the business recipient has `invoicing.type` `PEPPOL` (always, BE addresses included: E/invopop.go:300-313), or `EMAIL` with `invoicing.format` `XRECHNUNG_V3` and a DE address; with a BE address the format is dropped without an error and the invoice goes out as ZUGFeRD (RH:3366-3369), although the API docs say `XRECHNUNG_V3` e-mail always yields XRechnung. The XPath column is the spec target from the XRechnung model 2026-08-31 (`ubl-inv` binding; `ubl-cn` in §3), cardinalities are the model's (`m:structure id="invoice"`). The UAPI column names the field whose meaning is that BT, i.e. the mapping fiskaly must have; `NEW field needed: `path`` marks a field the API lacks (today's wrong or partial source in brackets), and its notes start with the requirement level (`law`, `format-mandatory`, `conditional`, `optional`). The notes column records where today's output departs from the target. UAPI paths are relative to `content.operation` unless a table says otherwise; a value in brackets after a path, e.g. "(OUTSTANDING)", restricts the row to that variant; "fatal" / "warning" are Schematron flags.

Code citations: `G/` = fiskaly `src/unified/einvoicing/gobl/`; `E/` = fiskaly `src/unified/einvoicing/`; `RH` = fiskaly `src/unified/api/v6/record_handler.go`; `EA` = fiskaly `src/unified/domain/record/einvoicing_adaptations.go`; `W` = fiskaly `src/unified/worker/e_invoicing_worker.go`; `SV1` = fiskaly `src/unified/domain/record/service_v1.go`; `WF/` = fiskaly `cmd/invopop-workflow-migration/workflows/` (all at `cd19ec8`); `gobl:` = invopop/gobl v0.501.0; `ubl:` = invopop/gobl.ubl v0.99.0; `xinv:` = invopop/gobl.de.xinvoice v0.0.9; `XR-sch` = XRechnung Schematron 2.6.0; `EN-sch` = EN 16931 validation artefacts v1.3.17.

## 1. Invoice Mapping

### 1.1 Header and References

| UAPI | EN 16931 BT | XRechnung UBL XPath | notes |
| --- | --- | --- | --- |
| `document.number` | BT-1 Invoice number | `/Invoice/cbc:ID` | 1..1. Must be unique across all series (§ 14 Abs. 4 Satz 1 Nr. 4 UStG); any series prefix belongs in the number. G/invoice.go:153; ubl:invoice.go:127. Peppol: the workflow's `sequence.enumerate` step (`PPL`, WF/DE_SEND_INVOICE_PEPPOL_XRECHNUNG_BUSINESS.json:15-26) may replace it: UNVERIFIED |
| `document.series` | — | — | No BT; `document.number` carries any series prefix. ignored: GOBL `series` never set (gobl.ubl would write `series-code`, ubl:invoice.go:223-228) |
| `document.issued_at` | BT-2 Invoice issue date | `/Invoice/cbc:IssueDate` | 1..1. Date part; the time is not written. default: absent → today at each build (G/invoice.go:706-714), so sync check and sent XML can differ |
| `type` = INVOICE | BT-3 Invoice type code | `/Invoice/cbc:InvoiceTypeCode` | derived: `380` (GOBL `standard`, W:192-193; gobl:addons/eu/en16931/scenarios.go:21-28). No other type code can be requested |
| NEW field needed: `document.invoice_currency` (`payments[0].details.currency`) | BT-5 Invoice currency code | `/Invoice/cbc:DocumentCurrencyCode` | 1..1. derived from `payments[0]` only (G/invoice.go:725-741); default `EUR` set in the domain (`src/unified/domain/record/transaction/receipt/payment/details.go:174`; `src/unified/country/de/v1/record.go:25`). API enum `CZK\|EUR\|SEK`, but a DE taxpayer's explicit non-EUR payment currency is rejected at record validation ("Payment currency '…' not supported for fiscalization country …", `src/unified/domain/record/capabilities.go:188-220`) |
| `-` | BT-6 VAT accounting currency code | `/Invoice/cbc:TaxCurrencyCode` | derived: `EUR` when BT-5 ≠ `EUR` (Art. 230 Directive 2006/112/EC); then BT-111 is required (BR-53). As built: not written; reachable only once non-EUR is enabled. gobl.ubl writes it only with an exchange rate (ubl:invoice.go:141-148) |
| `document.operation_date` | BT-7 Value added tax point date | `/Invoice/cbc:TaxPointDate` | API docs: BT-7. Excludes BT-8 (BR-CO-3). § 14 Abs. 4 Satz 1 Nr. 6 UStG needs the date of supply; BT-72 or BG-14 is the usual carrier, BT-7 is accepted. DE rule: `operation_date` is also copied into BT-72 when neither `recipients[i].shipping.date` nor an invoicing or line period is given. ignored: gobl.ubl writes BT-7 only from GOBL `value_date` (ubl:invoice.go:158-161), never set by fiskaly. As built, `operation_date` becomes GOBL `op_date` + `delivery.date` (G/invoice.go:433-441, 716-719) and is written to BT-72 (ubl:delivery.go:38-41) |
| NEW field needed: `document.tax_point_date_code` | BT-8 Value added tax point date code | `/Invoice/cac:InvoicePeriod/cbc:DescriptionCode` | optional. UNTDID 2005; XRechnung uses `3`, `35`, `432`. Excludes BT-7 (BR-CO-3). As built: not written (no GOBL `tax.point`) |
| `document.payment_terms` | BT-20 Payment terms | `/Invoice/cac:PaymentTerms/cbc:Note` | G/invoice.go:316-321; ubl:payment.go:225-230. Lines starting with `#` must match `#SKONTO#TAGE=n#PROZENT=n.nn#` (BR-DE-18, fatal) |
| NEW field needed: `document.early_payment_discounts[i].end_date`, `document.early_payment_discounts[i].percentage` | BT-20 Payment terms (Skonto line) | `/Invoice/cac:PaymentTerms/cbc:Note` | law (§ 14 Abs. 4 Satz 1 Nr. 7 UStG: an agreed reduction such as Skonto). Rendered as `#SKONTO#TAGE=n#PROZENT=n.nn#` (BR-DE-18): TAGE = `end_date` − BT-2, PROZENT = `percentage`. Today only possible as free text in `document.payment_terms` |
| `document.references.buyer` | BT-10 Buyer reference | `/Invoice/cbc:BuyerReference` | **1..1, BR-DE-15 (fatal).** G/invoice.go:331-333; G/xrechnung.go:30-37. No Leitweg-ID default: absent → GOBL `DE-XRECHNUNG-29` (neither `document.references` nor `tax_representative` sent) or `-30` at sync (gobl:addons/de/xrechnung/bill_invoices.go:135-140). API max 32 characters; a Leitweg-ID can have 46 (gap, §5) |
| `document.references.project` | BT-11 Project reference | `/Invoice/cac:ProjectReference/cbc:ID` | ignored for DE suppliers "due to an Invopop bug (META-3995)" (G/invoice.go:349-353). gobl.ubl writes `cac:ProjectReference` on credit notes too (ubl:ordering.go:124-129), an element the UBL 2.1 CreditNote XSD lacks; that this is META-3995: UNVERIFIED |
| `document.references.contract` | BT-12 Contract reference | `/Invoice/cac:ContractDocumentReference/cbc:ID` | G/invoice.go:355-357; ubl:ordering.go:143-151 |
| `document.references.purchase_order` | BT-13 Purchase order reference | `/Invoice/cac:OrderReference/cbc:ID` | G/invoice.go:358-360; ubl:ordering.go:107-112 |
| NEW field needed: `document.references.sales_order` | BT-14 Sales order reference | `/Invoice/cac:OrderReference/cbc:SalesOrderID` | optional |
| NEW field needed: `document.references.receiving_advice` | BT-15 Receiving advice reference | `/Invoice/cac:ReceiptDocumentReference/cbc:ID` | optional |
| `document.references.despatch_advice` | BT-16 Despatch advice reference | `/Invoice/cac:DespatchDocumentReference/cbc:ID` | G/invoice.go:361-363 |
| `document.references.tender` | BT-17 Tender or lot reference | `/Invoice/cac:OriginatorDocumentReference/cbc:ID` | G/invoice.go:364-366 |
| NEW field needed: `document.references.invoiced_object` | BT-18 Invoiced object identifier | `/Invoice/cac:AdditionalDocumentReference[cbc:DocumentTypeCode = '130']/cbc:ID` | optional. CreditNote: `[cbc:DocumentTypeCode = 'ATS']` (`ubl-cn` binding) |
| `document.references.buyer_routing` | BT-19 Buyer accounting reference | `/Invoice/cbc:AccountingCost` | G/invoice.go:334-336; ubl:ordering.go:80-83 |
| NEW field needed: `document.text_subject_code` | BT-21 Invoice note subject code | `/Invoice/cbc:Note` | optional. UNTDID 4451, written as the `#code#` prefix of the BT-22 note. As built: only `ACD` for the correction reason (§3) |
| `document.text` | BT-22 Invoice note | `/Invoice/cbc:Note` | Carrier for the notes UStG requires in context: the retention notice (§ 14 Abs. 4 Satz 1 Nr. 9 with § 14b Abs. 1 Satz 5 UStG) and the hint on the Fiskalvertretung (§ 22c Nr. 1 UStG). ignored (never read). The only note written is the correction reason (§3) |
| `document.references.preceding_document.number` | BT-25 Preceding Invoice reference | `/Invoice/cac:BillingReference/cac:InvoiceDocumentReference/cbc:ID` | `INVOICE` only. G/invoice.go:197-205, 371-383; ubl:ordering.go:53-72. On `CORRECTION` see §3 |
| `entries[i].details.prepayment.document_number` (RETURN) | BT-25 Preceding Invoice reference | `/Invoice/cac:BillingReference/cac:InvoiceDocumentReference/cbc:ID` | Final invoice: the number of each advance invoice whose amount and tax are deducted (§ 14 Abs. 5 Satz 2 UStG); the deduction itself is a negative line (§1.3). ignored |
| `document.references.preceding_document.issued_at` | BT-26 Preceding Invoice issue date | `/Invoice/cac:BillingReference/cac:InvoiceDocumentReference/cbc:IssueDate` | as BT-25 |
| NEW field needed: `document.period.start` | BG-14 INVOICING PERIOD, BT-73 Invoicing period start date | `/Invoice/cac:InvoicePeriod/cbc:StartDate` | law (§ 14 Abs. 4 Satz 1 Nr. 6 UStG: time of supply, for a supply over a period; § 31 Abs. 4 UStDV allows the calendar month). BR-CO-19, BR-29. As built: not written; BR-DE-TMP-32 (information) is met because BT-72 is always written |
| NEW field needed: `document.period.end` | BT-74 Invoicing period end date | `/Invoice/cac:InvoicePeriod/cbc:EndDate` | law (as BT-73). BR-29: end ≥ start |
| NEW field needed: `document.attachments[i].id` | BG-24 ADDITIONAL SUPPORTING DOCUMENTS, BT-122 Supporting document reference | `/Invoice/cac:AdditionalDocumentReference/cbc:ID` | format-mandatory (1..1 within BG-24). As built: no BG-24 (§4) |
| NEW field needed: `document.attachments[i].description` | BT-123 Supporting document description | `/Invoice/cac:AdditionalDocumentReference/cbc:DocumentDescription` | optional |
| NEW field needed: `document.attachments[i].uri` | BT-124 External document location | `/Invoice/cac:AdditionalDocumentReference/cac:Attachment/cac:ExternalReference/cbc:URI` | optional |
| NEW field needed: `document.attachments[i].content` | BT-125 Attached document | `/Invoice/cac:AdditionalDocumentReference/cac:Attachment/cbc:EmbeddedDocumentBinaryObject` | optional. Base64 |
| NEW field needed: `document.attachments[i].mime_code` | BT-125 Attached document (mime code) | `/Invoice/cac:AdditionalDocumentReference/cac:Attachment/cbc:EmbeddedDocumentBinaryObject/@mimeCode` | format-mandatory (1..1 when BT-125 is sent) |
| NEW field needed: `document.attachments[i].filename` | BT-125 Attached document (filename) | `/Invoice/cac:AdditionalDocumentReference/cac:Attachment/cbc:EmbeddedDocumentBinaryObject/@filename` | format-mandatory (1..1 when BT-125 is sent); unique per invoice, case-insensitive (BR-DE-22) |
| `document.activity_code`, `document.offline.reason`, `document.offline.reference` | — | — | ignored; no EN 16931 counterpart |

### 1.2 Buyer

| UAPI | EN 16931 BT | XRechnung UBL XPath | notes |
| --- | --- | --- | --- |
| `recipients[i]` (BUSINESS) | BG-7 BUYER | `/Invoice/cac:AccountingCustomerParty` | exactly one (G/invoice.go:400-418) |
| `recipients[i]` (CONSUMER) | BG-7 BUYER | `/Invoice/cac:AccountingCustomerParty` | Never XRechnung. An EMAIL or PEPPOL block fails with 400 on the empty SDI destination code (RH:3565-3568); no `invoicing` fails at sync (EA:30-33); an SDI block from a DE supplier is typed as ZUGFeRD (E/invopop.go:300-313), but no `DE_SEND_INVOICE_SDI_*` workflow exists, so the job most likely fails at the worker: UNVERIFIED (inferred) |
| `recipients[i].name` | BT-44 Buyer name | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PartyLegalEntity/cbc:RegistrationName` | 1..1. G/recipient.go:175; ubl:party.go:207-216 |
| NEW field needed: `recipients[i].trade_name` | BT-45 Buyer trading name | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PartyName/cbc:Name` | optional. Only when it differs from BT-44. As built: gobl.ubl copies `recipients[i].name` here (ubl:party.go:208-211) |
| `recipients[i].buyer_id` | BT-46 Buyer identifier | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PartyIdentification/cbc:ID` | The value must carry the ISO 6523 ICD prefix (`^[0-9]{4}:`, as `invoicing.identifier`), which becomes `@schemeID`. As built: GOBL identity `{key: other, label: buyer}`, no scheme (G/recipient.go:159-165; ubl:party.go:322-349) |
| `recipients[i].company_id` | BT-47 Buyer legal registration identifier | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PartyLegalEntity/cbc:CompanyID` | ICD prefix as BT-46 → `@schemeID`. As built: the identity has no legal scope (G/recipient.go:166-172), so gobl.ubl writes it as a second BT-46 `cac:PartyIdentification` instead. With `buyer_id` also set → UBL-SR-16 (fatal) |
| `recipients[i].identification.number` (VAT) | BT-48 Buyer VAT identifier | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PartyTaxScheme/cbc:CompanyID` | The full VAT ID with its country prefix is required in `identification.number` (the issuing state's prefix, e.g. `EL`, `XI`; Art. 226(4) Directive 2006/112/EC); `address.country` only as fallback. As built: prefix = recipient `address.country` (G/identification.go:15-36; ubl:party.go:229-249); `cac:TaxScheme/cbc:ID` `VAT`. Format checked at sync (EA:71-93) |
| `recipients[i].identification.number` (TAX, PASSPORT, DOCUMENT, OTHER) | BT-46 Buyer identifier / BT-47 Buyer legal registration identifier | — | Fails sync validation: "business recipient identification is not a valid VAT identification" (EA:66). A buyer without VAT ID (e.g. a public body) cannot be invoiced |
| `recipients[i].invoicing.identifier` (PEPPOL) | BT-49 Buyer electronic address | `/Invoice/cac:AccountingCustomerParty/cac:Party/cbc:EndpointID` | 1..1, PEPPOL-EN16931-R010. `ICD:id` → `@schemeID` + value (G/inbox.go:37-76; ubl:party.go:26-62). Leitweg-ID endpoints (ICD `0204`) fit the API pattern |
| `recipients[i].invoicing.email` (EMAIL) | BT-49 Buyer electronic address | `/Invoice/cac:AccountingCustomerParty/cac:Party/cbc:EndpointID` | derived: XRechnung only, EAS `EM` (G/xrechnung.go:68-72; ubl:party.go:47-52) |
| `recipients[i].invoicing.format` | — | — | Selects the flow (intro). Kept only for a DE recipient address (RH:3366-3369), after it has been validated (RH:3361-3364) |
| `recipients[i].address.line.street`, `recipients[i].address.line.number` | BT-50 Buyer address line 1 | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PostalAddress/cbc:StreetName` | `street + " " + number` (G/address.go:33-40; ubl:party.go:429-432) |
| `recipients[i].address.line.suffix` | BT-51 Buyer address line 2 | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PostalAddress/cbc:AdditionalStreetName` | GOBL `street_extra` (G/address.go:18-21, 36) |
| `recipients[i].address.line.prefix`, `recipients[i].address.line.infix` | BT-163 Buyer address line 3 | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PostalAddress/cac:AddressLine/cbc:Line` | ignored |
| `recipients[i].address.city` | BT-52 Buyer city | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PostalAddress/cbc:CityName` | 1..1, BR-DE-8 |
| `recipients[i].address.code` | BT-53 Buyer post code | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PostalAddress/cbc:PostalZone` | 1..1, BR-DE-9 |
| `recipients[i].address.region` | BT-54 Buyer country subdivision | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PostalAddress/cbc:CountrySubentity` | GOBL `region` (G/address.go:42-48) |
| `recipients[i].address.country` | BT-55 Buyer country code | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PostalAddress/cac:Country/cbc:IdentificationCode` | 1..1. Also the BT-48 prefix fallback and flow choice |
| NEW field needed: `recipients[i].contact.name` | BG-9 BUYER CONTACT, BT-56 Buyer contact point | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:Contact/cbc:Name` | optional |
| NEW field needed: `recipients[i].contact.phone` | BT-57 Buyer contact telephone number | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:Contact/cbc:Telephone` | optional |
| NEW field needed: `recipients[i].contact.email` (`recipients[i].invoicing.email`) | BT-58 Buyer contact email address | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:Contact/cbc:ElectronicMail` | optional. As built: the EMAIL delivery address is copied here (G/recipient.go:149-156; ubl:party.go:259-261); it is the BT-49 inbox, not a contact |
| `recipients[i].origin` | — | — | ignored |
| `recipients[i].shipping.name` | BT-70 Deliver to party name | `/Invoice/cac:Delivery/cac:DeliveryParty/cac:PartyName/cbc:Name` | ignored: cleared for XRechnung (G/xrechnung.go:79-94) because gobl.ubl also writes it as `cac:DeliveryParty/cac:PartyLegalEntity` (ubl:party.go:375-385; UBL-CR-397, warning). Valid BT-70 lost |
| NEW field needed: `recipients[i].shipping.location_id` | BT-71 Deliver to location identifier | `/Invoice/cac:Delivery/cac:DeliveryLocation/cbc:ID` | optional |
| `recipients[i].shipping.date`, else `document.operation_date` | BT-72 Actual delivery date | `/Invoice/cac:Delivery/cbc:ActualDeliveryDate` | API docs: BT-72. The date of supply (§ 14 Abs. 4 Satz 1 Nr. 6 UStG). default: `operation_date` when no `shipping.date` and no invoicing or line period is given (DE rule; BT-7 alone does not show the date of supply); no issue-date default. ignored. As built (deviation): BT-72 is written from `document.operation_date` (G/invoice.go:433-441, 716-719; ubl:delivery.go:38-41), default the issue date when that is absent |
| `recipients[i].shipping.address.line.street`, `recipients[i].shipping.address.line.number` | BT-75 Deliver to address line 1 | `/Invoice/cac:Delivery/cac:DeliveryLocation/cac:Address/cbc:StreetName` | as the buyer address (G/invoice.go:443-467; ubl:delivery.go:53-58). default: no shipping → buyer address copied to BG-15 (G/xrechnung.go:48-53) |
| `recipients[i].shipping.address.line.suffix` | BT-76 Deliver to address line 2 | `/Invoice/cac:Delivery/cac:DeliveryLocation/cac:Address/cbc:AdditionalStreetName` | as BT-75 |
| `recipients[i].shipping.address.line.prefix`, `recipients[i].shipping.address.line.infix` | BT-165 Deliver to address line 3 | `/Invoice/cac:Delivery/cac:DeliveryLocation/cac:Address/cac:AddressLine/cbc:Line` | ignored |
| `recipients[i].shipping.address.city` | BT-77 Deliver to city | `/Invoice/cac:Delivery/cac:DeliveryLocation/cac:Address/cbc:CityName` | BR-DE-10 |
| `recipients[i].shipping.address.code` | BT-78 Deliver to post code | `/Invoice/cac:Delivery/cac:DeliveryLocation/cac:Address/cbc:PostalZone` | BR-DE-11 |
| `recipients[i].shipping.address.region` | BT-79 Deliver to country subdivision | `/Invoice/cac:Delivery/cac:DeliveryLocation/cac:Address/cbc:CountrySubentity` | as BT-75 |
| `recipients[i].shipping.address.country` | BT-80 Deliver to country code | `/Invoice/cac:Delivery/cac:DeliveryLocation/cac:Address/cac:Country/cbc:IdentificationCode` | 1..1 within BG-15 |

### 1.3 Lines

| UAPI | EN 16931 BT | XRechnung UBL XPath | notes |
| --- | --- | --- | --- |
| `entries[i]` | BG-25 INVOICE LINE | `/Invoice/cac:InvoiceLine` | one line per entry (G/entries.go:76-84); GROUP entries see below |
| `entries[i].details.number` | BT-126 Invoice line identifier | `/Invoice/cac:InvoiceLine/cbc:ID` | 1..1. ignored. As built: ID = index + 1 (G/entries.go:76-84; ubl:lines.go:76-77). Correction: §3 |
| `entries[i].type` | — | — | ignored: `RETURN` not negated (§3) |
| `entries[i].type` = RETURN with `entries[i].details.prepayment.document_number` | BG-25 INVOICE LINE (negative), BT-131 Invoice line net amount | `/Invoice/cac:InvoiceLine`; `/Invoice/cac:InvoiceLine/cbc:LineExtensionAmount` | Final invoice: each advance received is deducted as a negative line in the advance's VAT category, so BG-23 shows the deducted tax (§ 14 Abs. 5 Satz 2 UStG); the number goes to BT-25 (§1.1). Negative quantity with a positive BT-146 (BR-27). A DEPOSIT payment is not counted again in BT-113 (§1.4). As built: not negated, no reference |
| `entries[i].data.type` (GROUP) | BG-25 INVOICE LINE | `/Invoice/cac:InvoiceLine` | `entries[i].data.items[]` with different VAT → one BG-25 line per VAT category and rate; same VAT → one line from the group-level fields (§ 14 Abs. 4 Satz 1 Nr. 7-8 UStG: breakdown by rate). `items[i].*` map as the `entries[i].data.*` rows below. As built: one line from the group-level fields; `items[]` ignored entirely |
| `entries[i].data.text` | BT-153 Item name | `/Invoice/cac:InvoiceLine/cac:Item/cbc:Name` | 1..1. G/entries.go:299-306 |
| `entries[i].data.unit.quantity` | BT-129 Invoiced quantity | `/Invoice/cac:InvoiceLine/cbc:InvoicedQuantity` | 1..1. Up to 8 decimals |
| `entries[i].data.unit.measure` | BT-130 Invoiced quantity unit of measure code | `/Invoice/cac:InvoiceLine/cbc:InvoicedQuantity/@unitCode` | Must be a UN/ECE Rec 20/21 code, validated at sync. As built: GOBL unit key or UN/ECE Rec 20 code passed through, else dropped (G/entries.go:309-323); the warning log is written in the worker only (E/invopop.go:729-751). default: `C62` (GOBL unit `one`: gobl:addons/eu/en16931/org.go:79-86; gobl:org/unit.go:201) |
| `entries[i].data.unit.price.exclusive` | BT-146 Item net price | `/Invoice/cac:InvoiceLine/cac:Price/cbc:PriceAmount` | API docs: BT-146. ignored. As built: derived from `price.inclusive`, VAT removed by `RemoveIncludedTaxes` (ubl:ubl.go:160-163), full precision (ubl:lines.go:496-499); may be nudged for rounding (§1.6) |
| `entries[i].data.unit.factor` | BT-149 Item price base quantity | `/Invoice/cac:InvoiceLine/cac:Price/cbc:BaseQuantity` | API docs: BT-149. ignored |
| NEW field needed: `entries[i].data.unit.factor_measure` | BT-150 Item price base quantity unit of measure code | `/Invoice/cac:InvoiceLine/cac:Price/cbc:BaseQuantity/@unitCode` | optional. Must equal BT-130 (PEPPOL-EN16931-R130) |
| NEW field needed: `entries[i].data.unit.price.gross` | BT-148 Item gross price | `/Invoice/cac:InvoiceLine/cac:Price/cac:AllowanceCharge[cbc:ChargeIndicator = 'false']/cbc:BaseAmount` | optional. Net of VAT, before the item price discount; not `price.inclusive` |
| NEW field needed: `entries[i].data.unit.price.discount` | BT-147 Item price discount | `/Invoice/cac:InvoiceLine/cac:Price/cac:AllowanceCharge[cbc:ChargeIndicator = 'false']/cbc:Amount` | optional. BT-146 = BT-148 − BT-147 |
| `entries[i].data.value.discount.exclusive` | BG-27 INVOICE LINE ALLOWANCES, BT-136 Invoice line allowance amount | `/Invoice/cac:InvoiceLine/cac:AllowanceCharge[cbc:ChargeIndicator = 'false']/cbc:Amount` | API docs: BT-136. ignored. As built: `value.discount.inclusive`, if ≠ 0, 2 dp, converted to net (G/entries.go:45-55; ubl:lines.go:467-489) |
| NEW field needed: `entries[i].data.value.discount.base` | BT-137 Invoice line allowance base amount | `/Invoice/cac:InvoiceLine/cac:AllowanceCharge[cbc:ChargeIndicator = 'false']/cbc:BaseAmount` | optional |
| NEW field needed: `entries[i].data.value.discount.percentage` | BT-138 Invoice line allowance percentage | `/Invoice/cac:InvoiceLine/cac:AllowanceCharge[cbc:ChargeIndicator = 'false']/cbc:MultiplierFactorNumeric` | optional |
| NEW field needed: `entries[i].data.value.discount.reason` (fixed "Special Discount") | BT-139 Invoice line allowance reason | `/Invoice/cac:InvoiceLine/cac:AllowanceCharge[cbc:ChargeIndicator = 'false']/cbc:AllowanceChargeReason` | conditional (BR-42: reason or code). As built: fixed "Special Discount" (META-2347) |
| NEW field needed: `entries[i].data.value.discount.reason_code` | BT-140 Invoice line allowance reason code | `/Invoice/cac:InvoiceLine/cac:AllowanceCharge[cbc:ChargeIndicator = 'false']/cbc:AllowanceChargeReasonCode` | conditional (BR-42: reason or code). UNTDID 5189 |
| `entries[i].data.value.surcharge.exclusive` | BG-28 INVOICE LINE CHARGES, BT-141 Invoice line charge amount | `/Invoice/cac:InvoiceLine/cac:AllowanceCharge[cbc:ChargeIndicator = 'true']/cbc:Amount` | API docs: BT-141. ignored. As built: `value.surcharge.inclusive`, converted as BT-136 (G/entries.go:57-67) |
| NEW field needed: `entries[i].data.value.surcharge.base` | BT-142 Invoice line charge base amount | `/Invoice/cac:InvoiceLine/cac:AllowanceCharge[cbc:ChargeIndicator = 'true']/cbc:BaseAmount` | optional |
| NEW field needed: `entries[i].data.value.surcharge.percentage` | BT-143 Invoice line charge percentage | `/Invoice/cac:InvoiceLine/cac:AllowanceCharge[cbc:ChargeIndicator = 'true']/cbc:MultiplierFactorNumeric` | optional |
| NEW field needed: `entries[i].data.value.surcharge.reason` (fixed "Special Surcharge") | BT-144 Invoice line charge reason | `/Invoice/cac:InvoiceLine/cac:AllowanceCharge[cbc:ChargeIndicator = 'true']/cbc:AllowanceChargeReason` | conditional (BR-44: reason or code). As built: fixed "Special Surcharge" |
| NEW field needed: `entries[i].data.value.surcharge.reason_code` | BT-145 Invoice line charge reason code | `/Invoice/cac:InvoiceLine/cac:AllowanceCharge[cbc:ChargeIndicator = 'true']/cbc:AllowanceChargeReasonCode` | conditional (BR-44: reason or code). UNTDID 7161 |
| `entries[i].data.value.base.exclusive` | BT-131 Invoice line net amount | `/Invoice/cac:InvoiceLine/cbc:LineExtensionAmount` | 1..1. `value.base.exclusive` is the line amount after line allowances and charges, on every `vat.type` (the API docs must say so). Must equal BT-129 × BT-146 − BT-136 + BT-141 within rounding. ignored. As built: computed from the line values (§1.6) |
| `entries[i].data.vat.exclusive`, `entries[i].data.value.base.inclusive`, `entries[i].data.unit.price.inclusive`, `entries[i].data.value.discount.inclusive`, `entries[i].data.value.surcharge.inclusive` | — | — | No BT: BT-131 comes from `value.base.exclusive`, the `.inclusive` amounts include VAT. As built: `price.inclusive`, `discount.inclusive` and `surcharge.inclusive` are the values read for BT-146, BT-136 and BT-141 |
| `entries[i].data.vat.type` = VAT_RATE, with `entries[i].data.vat.code` (STANDARD, REDUCED_n) and `entries[i].data.vat.percentage` | BG-30 LINE VAT INFORMATION, BT-151 Invoiced item VAT category code (→ BT-118 VAT category code), BT-152 Invoiced item VAT rate | `/Invoice/cac:InvoiceLine/cac:Item/cac:ClassifiedTaxCategory/cbc:ID`, `/Invoice/cac:InvoiceLine/cac:Item/cac:ClassifiedTaxCategory/cbc:Percent` | Category per code: code table maintained separately. As built: `S` for any rate > 0, `Z` for 0 % (G/entries.go:101-122; gobl:addons/eu/en16931/tax_combo.go:34-58; ubl:lines.go:338-361); `vat.code` unused |
| `entries[i].data.vat.type` = VAT_EXEMPTION, with `entries[i].data.vat.code` | BT-151 Invoiced item VAT category code (→ BT-118 VAT category code), BT-121 VAT exemption reason code | `/Invoice/cac:InvoiceLine/cac:Item/cac:ClassifiedTaxCategory/cbc:ID`; `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cac:TaxCategory/cbc:TaxExemptionReasonCode` | Category and VATEX code per `vat.code`: code table maintained separately. As built: G/german_exemption.go:24-60 (`CAUSE_n` > 31 → error); mixed VATEX per key are dropped (G/german_exemption.go:99-120). BT-152 = 0 for E/K/G, absent for O |
| `entries[i].data.vat.reason` (VAT_EXEMPTION) | BT-120 VAT exemption reason text | `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cac:TaxCategory/cbc:TaxExemptionReason` | Line field, document element: one reason per VAT category; sync must reject differing reasons within a category. Default text per code: code table maintained separately. As built: one GOBL tax note per VAT key, distinct reasons joined with `"; "`, German default text per code (G/entries.go:203-269; ubl:totals.go:122-126) |
| `entries[i].data.vat.type` = VAT_REVERSE_CHARGE | BT-151 Invoiced item VAT category code = `AE` (→ BT-118 VAT category code), BT-120 VAT exemption reason text, BT-121 VAT exemption reason code | `/Invoice/cac:InvoiceLine/cac:Item/cac:ClassifiedTaxCategory/cbc:ID`; `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cac:TaxCategory/cbc:TaxExemptionReason`; `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cac:TaxCategory/cbc:TaxExemptionReasonCode` | Category `AE`. BT-120 legal text ("Steuerschuldnerschaft des Leistungsempfängers", § 14a Abs. 5 UStG) and BT-121: code table maintained separately. As built: fails the GOBL mapping: "unsupported VAT type … for line mapping" (G/entries.go:164-169). gap (§5) |
| `entries[i].data.vat.amount`, `entries[i].data.vat.inclusive` | — | — | No line-level BT: BT-117 belongs to BG-23 and BT-112 to the document totals (API docs tag them BT-117 / BT-112). ignored; GOBL derives BG-23 (§1.5) |
| `entries[i].data.vat.fiscal_location`, `entries[i].data.vat.regime`, `entries[i].data.vat.regime_base`, `entries[i].data.vat.surcharge.rate`, `entries[i].data.vat.surcharge.amount` | — | — | ignored ("Not applicable" for DE) |
| `entries[i].data.product.details.label` | BT-154 Item description | `/Invoice/cac:InvoiceLine/cac:Item/cbc:Description` | API docs: BT-154. ignored: no `product` field is read |
| `entries[i].data.product.number` (SKU, MPN, OTHER) | BT-155 Item Sellers identifier | `/Invoice/cac:InvoiceLine/cac:Item/cac:SellersItemIdentification/cbc:ID` | ignored. API docs put BT-155 / BT-157 on type OTHER |
| `entries[i].data.product.number` (GTIN-8, GTIN-12, GTIN-13, GTIN-14) | BT-157 Item standard identifier | `/Invoice/cac:InvoiceLine/cac:Item/cac:StandardItemIdentification/cbc:ID` | ignored. Needs `@schemeID` `0160` (BR-64) |
| `entries[i].data.product.code` | BT-158 Item classification identifier | `/Invoice/cac:InvoiceLine/cac:Item/cac:CommodityClassification/cbc:ItemClassificationCode` | API docs: BT-158. ignored. Needs `@listID` (BR-65) |
| NEW field needed: `entries[i].data.product.code_scheme` | BT-158 Item classification identifier (scheme identifier) | `/Invoice/cac:InvoiceLine/cac:Item/cac:CommodityClassification/cbc:ItemClassificationCode/@listID` | conditional (BR-65, when BT-158 is sent). UNTDID 7143 |
| `entries[i].data.product.details.*` (name, brand, model, serial, batch, dates) | BG-32 ITEM ATTRIBUTES, BT-160 Item attribute name / BT-161 Item attribute value | `/Invoice/cac:InvoiceLine/cac:Item/cac:AdditionalItemProperty/cbc:Name`, `/Invoice/cac:InvoiceLine/cac:Item/cac:AdditionalItemProperty/cbc:Value` | ignored |
| NEW field needed: `entries[i].data.product.attributes[i].name` | BT-160 Item attribute name | `/Invoice/cac:InvoiceLine/cac:Item/cac:AdditionalItemProperty/cbc:Name` | optional. Carrier for the § 1b Abs. 2-3 characteristics that § 14a Abs. 4 UStG requires on an intra-Community supply of a new vehicle |
| NEW field needed: `entries[i].data.product.attributes[i].value` | BT-161 Item attribute value | `/Invoice/cac:InvoiceLine/cac:Item/cac:AdditionalItemProperty/cbc:Value` | optional (as BT-160) |
| NEW field needed: `entries[i].data.product.buyer_number` | BT-156 Item Buyers identifier | `/Invoice/cac:InvoiceLine/cac:Item/cac:BuyersItemIdentification/cbc:ID` | optional |
| NEW field needed: `entries[i].data.product.origin_country` | BT-159 Item country of origin | `/Invoice/cac:InvoiceLine/cac:Item/cac:OriginCountry/cbc:IdentificationCode` | optional |
| `entries[i].details.description` | BT-127 Invoice line note | `/Invoice/cac:InvoiceLine/cbc:Note` | API docs: BT-127. ignored |
| NEW field needed: `entries[i].details.object_id` | BT-128 Invoice line object identifier | `/Invoice/cac:InvoiceLine/cac:DocumentReference[cbc:DocumentTypeCode = '130']/cbc:ID` | optional |
| NEW field needed: `entries[i].details.order_line_reference` | BT-132 Referenced purchase order line reference | `/Invoice/cac:InvoiceLine/cac:OrderLineReference/cbc:LineID` | optional |
| NEW field needed: `entries[i].details.buyer_accounting_reference` | BT-133 Invoice line Buyer accounting reference | `/Invoice/cac:InvoiceLine/cbc:AccountingCost` | optional |
| NEW field needed: `entries[i].details.period.start` | BG-26 INVOICE LINE PERIOD, BT-134 Invoice line period start date | `/Invoice/cac:InvoiceLine/cac:InvoicePeriod/cbc:StartDate` | law (§ 14 Abs. 4 Satz 1 Nr. 6 UStG, when lines have different supply periods). BR-CO-20 |
| NEW field needed: `entries[i].details.period.end` | BT-135 Invoice line period end date | `/Invoice/cac:InvoiceLine/cac:InvoicePeriod/cbc:EndDate` | law (as BT-134). BR-30: end ≥ start |
| `entries[i].details.*` (concept, purpose, regulatory, label, activity_code, properties, voucher, applied_vouchers) | — | — | ignored |

### 1.4 Payment

| UAPI | EN 16931 BT | XRechnung UBL XPath | notes |
| --- | --- | --- | --- |
| `payments[i].details.date` (OUTSTANDING, first) | BT-9 Payment due date | `/Invoice/cbc:DueDate` | required for OUTSTANDING ("payment due date not set"); later OUTSTANDING entries dropped (G/invoice.go:274-289, 607-621; ubl:payment.go:232-235), with a warning log in the worker only (E/invopop.go:709-727). Credit note: §3 |
| `payments[i].details.amount` (OUTSTANDING) | — | — | GOBL due-date amount; no BT. Used for rounding reconciliation |
| `payments[i].type`, `payments[i].kind` (CARD), `payments[i].instruction.type` (OUTSTANDING) | BG-16 PAYMENT INSTRUCTIONS, BT-81 Payment means type code | `/Invoice/cac:PaymentMeans/cbc:PaymentMeansCode` | 1..1, BR-DE-1. derived: CASH → `10`; CARD → `54` / `55` / `48` (`kind` CREDIT / DEBIT / PREPAID); ONLINE → `68`; credit transfer → `30` / `58` (SEPA); direct debit → `49` / `59` (SEPA); several methods → `1`; CARD `kind` TICKET, SINGLE_PURPOSE_VOUCHER, MULTI_PURPOSE_VOUCHER, MEAL_VOUCHER, OTHER and `instruction.type` UNKNOWN → `1` (not defined), with BT-82 naming the payment type. Card codes need exactly BG-18 (BR-DE-24). As built: default `1` (instrument not defined) via GOBL key `any` (G/xrechnung.go:39-46; gobl:addons/eu/en16931/pay.go:12-27) whenever there is no OUTSTANDING instruction |
| `payments[i].instruction.type` (CREDIT_TRANSFER) | BT-81 Payment means type code | `/Invoice/cac:PaymentMeans/cbc:PaymentMeansCode` | derived: `58` for a SEPA transfer, else `30`. As built: fixed `30`, so BR-DE-19 never runs; BR-DE-23-a met by BG-17 |
| `payments[i].instruction.account` (CREDIT_TRANSFER) | BG-17 CREDIT TRANSFER, BT-84 Payment account identifier | `/Invoice/cac:PaymentMeans/cac:PayeeFinancialAccount/cbc:ID` | GOBL `credit_transfer.iban` (G/invoice.go:530-537; ubl:payment.go:195-203) |
| `payments[i].instruction.name` | BT-85 Payment account name | `/Invoice/cac:PaymentMeans/cac:PayeeFinancialAccount/cbc:Name` | GOBL `credit_transfer.name` (G/invoice.go:535) |
| `payments[i].instruction.payment_service_provider` | BT-86 Payment service provider identifier | `/Invoice/cac:PaymentMeans/cac:PayeeFinancialAccount/cac:FinancialInstitutionBranch/cbc:ID` | GOBL `credit_transfer.bic` (G/invoice.go:534); no BIC format check |
| `-` | BT-82 Payment means text | `/Invoice/cac:PaymentMeans/cbc:PaymentMeansCode/@name` | derived: text for the payment type (cash, card, online). `payments[i].name` (API docs: BT-82) is not the source; ignored. As built: `instruction.text` lands here (BT-83 row) |
| `payments[i].instruction.text` | BT-83 Remittance information | `/Invoice/cac:PaymentMeans/cbc:PaymentID` | API docs: BT-83. As built: GOBL `instructions.detail` (G/invoice.go:538-540, 563-565) → `cbc:PaymentMeansCode/@name` = **BT-82** (ubl:payment.go:160-162) |
| `payments[i].instruction.properties` | — | — | No BT. As built: GOBL `instructions.ref` (G/invoice.go:541-543) → BT-83 (ubl:payment.go:157-159); `text` and `properties` are swapped against the docs |
| `payments[i].instruction.type` (DIRECT_DEBIT) | BT-81 Payment means type code | `/Invoice/cac:PaymentMeans/cbc:PaymentMeansCode` | derived: `59` for a SEPA direct debit, else `49`. As built: fixed `49`, so BR-DE-20 / BR-DE-25 never run; PEPPOL-EN16931-R061 applies |
| `payments[i].instruction.mandate_reference` | BG-19 DIRECT DEBIT, BT-89 Mandate reference identifier | `/Invoice/cac:PaymentMeans/cac:PaymentMandate/cbc:ID` | G/invoice.go:555-561; ubl:payment.go:166-171 |
| `payments[i].instruction.creditor_id.identifier` | BT-90 Bank assigned creditor identifier | `/Invoice/cac:PayeeParty/cac:PartyIdentification/cbc:ID[@schemeID = 'SEPA']` | As built: no payee, so written to `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PartyIdentification/cbc:ID[@schemeID = 'SEPA']` (ubl:payment.go:121-136); BR-DE-30 accepts both |
| `payments[i].instruction.debited_account` | BT-91 Debited account identifier | `/Invoice/cac:PaymentMeans/cac:PaymentMandate/cac:PayerFinancialAccount/cbc:ID` | G/invoice.go:560; ubl:payment.go:172-176. BR-DE-31 |
| `payments[i].instruction.type` (UNKNOWN) | BT-81 Payment means type code | `/Invoice/cac:PaymentMeans/cbc:PaymentMeansCode` | derived: `1`. As built: GOBL key `other` (G/invoice.go:571-583) has no UNTDID 4461 code (gobl:addons/eu/en16931/pay.go:12-38; gobl:cbc/code.go:273-283) → GOBL fault "payment means extension is required (BR-49)" at sync. From code, not executed |
| `payments[i].details.amount` (CARD, CASH, ONLINE, SINGLE_PURPOSE_VOUCHER, MULTI_PURPOSE_VOUCHER, OTHER) | BT-113 Paid amount | `/Invoice/cac:LegalMonetaryTotal/cbc:PrepaidAmount` | summed as GOBL advances (G/invoice.go:290-299; ubl:totals.go:77-79). A DEPOSIT already deducted as a prepayment line (§1.3) is not counted again. As built: the method is not expressed; BT-81 stays `1` without an OUTSTANDING instruction |
| `payments[i].number` (CARD) | BG-18 PAYMENT CARD INFORMATION, BT-87 Payment card primary account number | `/Invoice/cac:PaymentMeans/cac:CardAccount/cbc:PrimaryAccountNumberID` | Masked: only the last 4-6 digits (BR-51). Required with card codes in BT-81 (BR-DE-24). API docs: BT-87. ignored |
| NEW field needed: `payments[i].holder_name` | BT-88 Payment card holder name | `/Invoice/cac:PaymentMeans/cac:CardAccount/cbc:HolderName` | optional |
| `payments[i].type` (MEAL_VOUCHER, DEPOSIT) | — | — | Fails the GOBL mapping: "unsupported payment type" (G/invoice.go:300-302). A DEPOSIT's `document_number` is the advance invoice: the deduction is a RETURN prepayment line (§1.3), not BT-113 |
| `payments[i].details.channel`, `payments[i].details.properties`, `payments[i].concept`, `payments[i].count`, `payments[i].document_number`; advance `payments[i].details.date` | — | — | ignored; gobl.ubl writes no `cac:PrepaidPayment`. `payments[i].details.discount`: §1.5 |
| NEW field needed: `payee.name` | BG-10 PAYEE, BT-59 Payee name | `/Invoice/cac:PayeeParty/cac:PartyName/cbc:Name` | conditional (BR-17: when the payee differs from the seller) |
| NEW field needed: `payee.identifier` | BT-60 Payee identifier | `/Invoice/cac:PayeeParty/cac:PartyIdentification/cbc:ID` | optional. ICD prefix → `@schemeID`, as BT-46 |
| NEW field needed: `payee.company_id` | BT-61 Payee legal registration identifier | `/Invoice/cac:PayeeParty/cac:PartyLegalEntity/cbc:CompanyID` | optional. ICD prefix → `@schemeID`, as BT-46 |

### 1.5 VAT Breakdown and Totals

| UAPI | EN 16931 BT | XRechnung UBL XPath | notes |
| --- | --- | --- | --- |
| `breakdown[i]` | BG-23 VAT BREAKDOWN | `/Invoice/cac:TaxTotal/cac:TaxSubtotal` | 1..n. Calculated from the lines; sync rejects a difference above the rounding tolerance. As built: ignored; GOBL rebuilds BG-23 from the lines |
| `breakdown[i].exclusive` | BT-116 VAT category taxable amount | `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cbc:TaxableAmount` | Calculated from the lines; sync rejects a difference above the rounding tolerance. As built: ignored; derived (§1.6) |
| `breakdown[i].amount` | BT-117 VAT category tax amount | `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cbc:TaxAmount` | Calculated from the lines; sync rejects a difference above the rounding tolerance. As built: ignored; derived (§1.6) |
| `breakdown[i].type`, `breakdown[i].code` | BT-118 VAT category code | `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cac:TaxCategory/cbc:ID` | Calculated from the line categories (§1.3; code table maintained separately). As built: ignored; derived from the line categories |
| `breakdown[i].percentage` | BT-119 VAT category rate | `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cac:TaxCategory/cbc:Percent` | 1..1, BR-DE-14 (fatal). Calculated from the lines; sync rejects a difference. As built: from the lines; `0` for E/K/G, **omitted for O** (ubl:totals.go:128-136) |
| `breakdown[i].inclusive` | — | — | no BT |
| `-` | BT-106 Sum of Invoice line net amount | `/Invoice/cac:LegalMonetaryTotal/cbc:LineExtensionAmount` | derived (§1.6) |
| `totals.vat.exclusive` | BT-109 Invoice total amount without VAT | `/Invoice/cac:LegalMonetaryTotal/cbc:TaxExclusiveAmount` | Calculated from the lines; sync rejects a difference above the rounding tolerance. As built: ignored; derived (§1.6). A mismatch with the derived value only writes a warning log, in the worker (E/invopop.go:753-782) |
| `totals.vat.amount` | BT-110 Invoice total VAT amount | `/Invoice/cac:TaxTotal/cbc:TaxAmount[@currencyID = /Invoice/cbc:DocumentCurrencyCode]` | Calculated from the lines; sync rejects a difference above the rounding tolerance. As built: ignored; derived (§1.6); a mismatch is logged as for BT-109 |
| `totals.vat.inclusive` | BT-112 Invoice total amount with VAT | `/Invoice/cac:LegalMonetaryTotal/cbc:TaxInclusiveAmount` | Calculated from the lines; sync rejects a difference above the rounding tolerance. As built: ignored; derived (§1.6); a mismatch is logged as for BT-109 |
| `-` | BT-107 Sum of allowances on document level / BT-108 Sum of charges on document level | `/Invoice/cac:LegalMonetaryTotal/cbc:AllowanceTotalAmount`, `/Invoice/cac:LegalMonetaryTotal/cbc:ChargeTotalAmount` | derived: Σ BT-92 / Σ BT-99 (BR-CO-11, BR-CO-12). As built: not written (no BG-20 / BG-21 source) |
| NEW field needed: `document.allowances[i].amount` | BG-20 DOCUMENT LEVEL ALLOWANCES, BT-92 Document level allowance amount | `/Invoice/cac:AllowanceCharge[cbc:ChargeIndicator = 'false']/cbc:Amount` | law (§ 14 Abs. 4 Satz 1 Nr. 7 UStG: an agreed reduction not already in the line prices). BR-31 |
| NEW field needed: `document.allowances[i].base` | BT-93 Document level allowance base amount | `/Invoice/cac:AllowanceCharge[cbc:ChargeIndicator = 'false']/cbc:BaseAmount` | optional |
| NEW field needed: `document.allowances[i].percentage` | BT-94 Document level allowance percentage | `/Invoice/cac:AllowanceCharge[cbc:ChargeIndicator = 'false']/cbc:MultiplierFactorNumeric` | optional |
| NEW field needed: `document.allowances[i].vat_type` | BT-95 Document level allowance VAT category code | `/Invoice/cac:AllowanceCharge[cbc:ChargeIndicator = 'false']/cac:TaxCategory[cac:TaxScheme/cbc:ID = 'VAT']/cbc:ID` | format-mandatory (BR-32). Category from the `vat.type` values as on the lines (§1.3) |
| NEW field needed: `document.allowances[i].vat_percentage` | BT-96 Document level allowance VAT rate | `/Invoice/cac:AllowanceCharge[cbc:ChargeIndicator = 'false']/cac:TaxCategory/cbc:Percent` | optional (0..1; the per-category BR rules decide when a rate is needed) |
| NEW field needed: `document.allowances[i].reason` | BT-97 Document level allowance reason | `/Invoice/cac:AllowanceCharge[cbc:ChargeIndicator = 'false']/cbc:AllowanceChargeReason` | conditional (BR-33: reason or code) |
| NEW field needed: `document.allowances[i].reason_code` | BT-98 Document level allowance reason code | `/Invoice/cac:AllowanceCharge[cbc:ChargeIndicator = 'false']/cbc:AllowanceChargeReasonCode` | conditional (BR-33: reason or code). UNTDID 5189 |
| `payments[i].details.discount` | BG-20 DOCUMENT LEVEL ALLOWANCES, BT-92 Document level allowance amount | `/Invoice/cac:AllowanceCharge[cbc:ChargeIndicator = 'false']/cbc:Amount` | A granted payment discount is a document allowance (§ 14 Abs. 4 Satz 1 Nr. 7 UStG). ignored |
| NEW field needed: `payments[i].details.discount_vat.type` | BT-95 Document level allowance VAT category code | `/Invoice/cac:AllowanceCharge[cbc:ChargeIndicator = 'false']/cac:TaxCategory[cac:TaxScheme/cbc:ID = 'VAT']/cbc:ID` | format-mandatory (BR-32) once the payment discount is mapped |
| NEW field needed: `payments[i].details.discount_vat.percentage` | BT-96 Document level allowance VAT rate | `/Invoice/cac:AllowanceCharge[cbc:ChargeIndicator = 'false']/cac:TaxCategory/cbc:Percent` | optional (as BT-96 above) |
| `-` | BT-97 Document level allowance reason | `/Invoice/cac:AllowanceCharge[cbc:ChargeIndicator = 'false']/cbc:AllowanceChargeReason` | derived: "Payment discount" for the `payments[i].details.discount` allowance (meets BR-33) |
| NEW field needed: `document.charges[i].amount` | BG-21 DOCUMENT LEVEL CHARGES, BT-99 Document level charge amount | `/Invoice/cac:AllowanceCharge[cbc:ChargeIndicator = 'true']/cbc:Amount` | optional. BR-36 within BG-21 |
| NEW field needed: `document.charges[i].base` | BT-100 Document level charge base amount | `/Invoice/cac:AllowanceCharge[cbc:ChargeIndicator = 'true']/cbc:BaseAmount` | optional |
| NEW field needed: `document.charges[i].percentage` | BT-101 Document level charge percentage | `/Invoice/cac:AllowanceCharge[cbc:ChargeIndicator = 'true']/cbc:MultiplierFactorNumeric` | optional |
| NEW field needed: `document.charges[i].vat_type` | BT-102 Document level charge VAT category code | `/Invoice/cac:AllowanceCharge[cbc:ChargeIndicator = 'true']/cac:TaxCategory[cac:TaxScheme/cbc:ID = 'VAT']/cbc:ID` | format-mandatory (BR-37) |
| NEW field needed: `document.charges[i].vat_percentage` | BT-103 Document level charge VAT rate | `/Invoice/cac:AllowanceCharge[cbc:ChargeIndicator = 'true']/cac:TaxCategory/cbc:Percent` | optional (as BT-96) |
| NEW field needed: `document.charges[i].reason` | BT-104 Document level charge reason | `/Invoice/cac:AllowanceCharge[cbc:ChargeIndicator = 'true']/cbc:AllowanceChargeReason` | conditional (BR-38: reason or code) |
| NEW field needed: `document.charges[i].reason_code` | BT-105 Document level charge reason code | `/Invoice/cac:AllowanceCharge[cbc:ChargeIndicator = 'true']/cbc:AllowanceChargeReasonCode` | conditional (BR-38: reason or code). UNTDID 7161 |
| NEW field needed: `totals.vat.amount_accounting` | BT-111 Invoice total VAT amount in accounting currency | `/Invoice/cac:TaxTotal/cbc:TaxAmount[@currencyID = /Invoice/cbc:TaxCurrencyCode]` | conditional (BR-53; Art. 230 Directive 2006/112/EC). Required once non-EUR invoices are enabled (BT-6). As built: not written; a DE taxpayer's non-EUR currency is rejected (BT-5) |
| NEW field needed: `totals.rounding` | BT-114 Rounding amount | `/Invoice/cac:LegalMonetaryTotal/cbc:PayableRoundingAmount` | optional. As built: derived from GOBL `totals.rounding`; fiskaly absorbs rounding drift in the line prices instead (G/line_rounding.go:44-52); the converter's `RoundToCurrency()` can still put ±0.01 here (ubl:ubl.go:160-166) |
| `-` | BT-115 Amount due for payment | `/Invoice/cac:LegalMonetaryTotal/cbc:PayableAmount` | 1..1, derived (§1.6) |

### 1.6 Calculated Fields

| UAPI | EN 16931 BT | XRechnung UBL XPath | formula | notes |
| --- | --- | --- | --- | --- |
| `entries[i].data.unit.price.exclusive` | BT-146 Item net price | `/Invoice/cac:InvoiceLine/cac:Price/cbc:PriceAmount` | client value | As built: `price.inclusive / (1 + rate)` (GOBL `prices_include = VAT`); more than 2 decimals kept |
| `entries[i].data.value.base.exclusive` | BT-131 Invoice line net amount | `/Invoice/cac:InvoiceLine/cbc:LineExtensionAmount` | client value; check `BT-129 × BT-146 − Σ BT-136 + Σ BT-141` | Sync rejects a difference above the rounding tolerance. As built: computed from quantity, the derived net price and the converted inclusive discount / surcharge, rounded to currency; `tax.rounding = currency` (G/invoice.go:165-168). PEPPOL-EN16931-R120 (warning) can fire on residue (inferred) |
| `-` | BT-106 Sum of Invoice line net amount | `/Invoice/cac:LegalMonetaryTotal/cbc:LineExtensionAmount` | `Σ BT-131` | BR-CO-10 |
| `-` | BT-109 Invoice total amount without VAT | `/Invoice/cac:LegalMonetaryTotal/cbc:TaxExclusiveAmount` | `BT-106 − BT-107 + BT-108` | BR-CO-13. Checked against `totals.vat.exclusive` (§1.5). As built: `BT-106` (no document-level allowances / charges) |
| `-` | BT-116 VAT category taxable amount | `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cbc:TaxableAmount` | `Σ BT-131 − Σ BT-92 + Σ BT-99` per (BT-118, BT-119) | Checked against `breakdown[i].exclusive` (§1.5). As built: `Σ BT-131` per (BT-151, BT-152) |
| `-` | BT-117 VAT category tax amount | `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cbc:TaxAmount` | `round2(BT-116 × BT-119 / 100)` | BR-CO-17; the CEN Schematron tolerates ±1 currency unit (EN16931-UBL-model.sch:76) |
| `-` | BT-110 Invoice total VAT amount | `/Invoice/cac:TaxTotal/cbc:TaxAmount[@currencyID = /Invoice/cbc:DocumentCurrencyCode]` | `Σ BT-117` | BR-CO-14 |
| `-` | BT-112 Invoice total amount with VAT | `/Invoice/cac:LegalMonetaryTotal/cbc:TaxInclusiveAmount` | `BT-109 + BT-110` | BR-CO-15 |
| `payments[i].details.amount` (non-OUTSTANDING) | BT-113 Paid amount | `/Invoice/cac:LegalMonetaryTotal/cbc:PrepaidAmount` | `Σ advances` | 2 dp each. Advances deducted as prepayment lines (§1.3) are excluded |
| `-` | BT-115 Amount due for payment | `/Invoice/cac:LegalMonetaryTotal/cbc:PayableAmount` | `BT-112 − BT-113 + BT-114` | BR-CO-16. If > 0 on an INVOICE, GOBL requires BT-9 or BT-20 (BR-CO-25); credit notes are exempt (gobl:addons/eu/en16931/bill.go:143-151, 259-262). Not checked by the KoSIT validator configuration (§6.2) |
| `payments[i].details.amount` | BT-146 Item net price (adjusted) | `/Invoice/cac:InvoiceLine/cac:Price/cbc:PriceAmount` | prices rewritten (≤ 8 dp) so BT-112 = `Σ payments[i].details.amount` | drift = target − GOBL `TotalWithTax` (G/line_rounding.go:58); only when it is explained by rounding (G/line_rounding.go:21-159) |

## 2. Supplier Data (Taxpayer, System)

Paths are relative to the taxpayer root (`POST/PATCH /taxpayers`) or the system annotations; "(invoice)" marks paths under `content.operation`. Built by G/taxpayer.go:25-81 and G/invoice.go:469-497.

| UAPI | EN 16931 BT | XRechnung UBL XPath | notes |
| --- | --- | --- | --- |
| `fiscalization.type` | — | — | `DE` selects the regime, flow and exemption table, and the BT-31 / BT-40 country |
| `name.legal` | BT-27 Seller name | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PartyLegalEntity/cbc:RegistrationName` | 1..1. API docs: BT-44 (buyer name); spec: BT-27. G/taxpayer.go:62 |
| `name.trade` | BT-28 Seller trading name | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PartyName/cbc:Name` | ignored (required for COMPANY, never read). As built: gobl.ubl copies `name.legal` here (ubl:party.go:208-211) |
| `name.person.forename`, `name.person.surname` | BT-27 Seller name | — | ignored; `name.legal` is used for INDIVIDUAL too |
| NEW field needed: `seller.identifier` (invoice) | BT-29 Seller identifier | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PartyIdentification/cbc:ID` | conditional (BR-CO-26: BT-29, BT-30 or BT-31 must be present). Value with ISO 6523 ICD prefix (`nnnn:`) → `@schemeID`. A SEPA creditor ID does not count for BR-CO-26. As built: BT-29 carries only the SEPA creditor ID with direct debit (§1.4) |
| NEW field needed: `fiscalization.registration.company_id` (`fiscalization` = DE) | BT-30 Seller legal registration identifier | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PartyLegalEntity/cbc:CompanyID` | conditional (BR-CO-26). Handelsregister number and court, e.g. "HRB 12345, Amtsgericht München". Today the field exists for IT only |
| `fiscalization.vat_id_number` | BT-31 Seller VAT identifier | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PartyTaxScheme[cac:TaxScheme/cbc:ID = 'VAT']/cbc:CompanyID` | `DE` + 9 digits. **VAT ID wins** over `tax_id_number` (G/taxpayer.go:34-45; ubl:party.go:229-249). Meets BR-DE-16, BR-CO-26, BR-S-02 |
| NEW field needed: `fiscalization.tax_number` (`fiscalization.tax_id_number`) | BT-32 Seller tax registration identifier | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PartyTaxScheme[cac:TaxScheme/cbc:ID != 'VAT']/cbc:CompanyID` | law (§ 14 Abs. 4 Satz 1 Nr. 2 UStG: the Finanzamt Steuernummer, when no USt-IdNr is given). BR-DE-16. `cac:TaxScheme/cbc:ID` `FC` (GOBL identity `de-tax-number`, gobl:regimes/de/org_identities.go:20, with type `FC`; without the type gobl.ubl writes `LOC`, ubl:party.go:308-320, 354-361). As built: `tax_id_number` is the 11-digit personal Steuer-ID (BZSt), not a Steuernummer; ignored when a VAT ID exists, so BT-32 is never written; without VAT ID it becomes the BT-31 code and fails GOBL's `^[1-9]\d{8}$` (gobl:regimes/de/tax_identity.go:17-19) |
| NEW field needed: `seller.legal_information` (invoice) | BT-33 Seller additional legal information | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PartyLegalEntity/cbc:CompanyLegalForm` | optional. Company-law disclosures on business letters (§ 35a GmbHG, § 37a HGB) may require it: UNVERIFIED (texts not held) |
| `address.line.street`, `address.line.number` | BT-35 Seller address line 1 | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PostalAddress/cbc:StreetName` | `street + " " + number` |
| `address.line.suffix` | BT-36 Seller address line 2 | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PostalAddress/cbc:AdditionalStreetName` | GOBL `street_extra` (G/address.go:18-21, 36) |
| `address.line.prefix`, `address.line.infix` | BT-162 Seller address line 3 | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PostalAddress/cac:AddressLine/cbc:Line` | ignored |
| `address.city` | BT-37 Seller city | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PostalAddress/cbc:CityName` | 1..1, BR-DE-3 |
| `address.code` | BT-38 Seller post code | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PostalAddress/cbc:PostalZone` | 1..1, BR-DE-4 |
| `address.region` | BT-39 Seller country subdivision | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PostalAddress/cbc:CountrySubentity` | GOBL `region` (G/address.go:42-48) |
| `address.country` | BT-40 Seller country code | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PostalAddress/cac:Country/cbc:IdentificationCode` | 1..1. Overwritten by the tax-ID country (`DE`) (ubl:party.go:250-256) |
| `peppol_id` (system annotation) | BT-34 Seller electronic address | `/Invoice/cac:AccountingSupplierParty/cac:Party/cbc:EndpointID` | 1..1, PEPPOL-EN16931-R020. API docs: BT-49; spec: BT-34. Set on any channel when present (W:284-298; G/inbox.go:17-35) |
| `seller.email` (invoice) | BT-34 Seller electronic address | `/Invoice/cac:AccountingSupplierParty/cac:Party/cbc:EndpointID` | default without `peppol_id`: EAS `EM` (G/xrechnung.go:61-65). The sync check never has the Peppol ID (E/invopop.go:330-338), so at sync `seller.email` is the only BT-34 source (§6.2) |
| `seller.name` (invoice) | BG-6 SELLER CONTACT, BT-41 Seller contact point | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:Contact/cbc:Name` | 1..1, BR-DE-5. A contact person or department, not the seller's name: the API docs description ("Seller's legal or trading name") must be changed to "contact person or department". default `name.legal` (G/xrechnung.go:55-60) |
| `seller.phone` (invoice) | BT-42 Seller contact telephone number | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:Contact/cbc:Telephone` | **1..1, BR-DE-6 (fatal)**, optional in the API. BG-6 itself is mandatory by BR-DE-2; BR-DE-5/6/7 apply inside it. Absent → GOBL `DE-XRECHNUNG-09` (gobl:addons/de/xrechnung/bill_invoices.go:58-60). BR-DE-27 (warning) |
| `seller.email` (invoice) | BT-43 Seller contact email address | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:Contact/cbc:ElectronicMail` | **1..1, BR-DE-7 (fatal)**, optional in the API. Absent → `DE-XRECHNUNG-10` (bill_invoices.go:61-63). BR-DE-28 (warning) |
| `tax_representative.name` (invoice) | BG-11 SELLER TAX REPRESENTATIVE PARTY, BT-62 Seller tax representative name | `/Invoice/cac:TaxRepresentativeParty/cac:PartyName/cbc:Name` | GOBL `ordering.seller` (G/invoice.go:208-213, 499-519; ubl:ordering.go:87-89); also written as `cac:PartyLegalEntity` → UBL-CR-335 (warning) |
| `tax_representative.vat_number` (invoice) | BT-63 Seller tax representative VAT identifier | `/Invoice/cac:TaxRepresentativeParty/cac:PartyTaxScheme/cbc:CompanyID` | prefixed with `tax_representative.country` |
| NEW field needed: `tax_representative.address.line.street`, `tax_representative.address.line.number` (invoice) | BG-12 SELLER TAX REPRESENTATIVE POSTAL ADDRESS, BT-64 Tax representative address line 1 | `/Invoice/cac:TaxRepresentativeParty/cac:PostalAddress/cbc:StreetName` | law (§ 22c Nr. 2 UStG: name and address of the Fiskalvertreter; Art. 226(15) Directive 2006/112/EC). Optional inside BG-12 for the format (BR-19 needs only BT-69) |
| NEW field needed: `tax_representative.address.line.suffix` (invoice) | BT-65 Tax representative address line 2 | `/Invoice/cac:TaxRepresentativeParty/cac:PostalAddress/cbc:AdditionalStreetName` | optional |
| NEW field needed: `tax_representative.address.line.prefix`, `tax_representative.address.line.infix` (invoice) | BT-164 Tax representative address line 3 | `/Invoice/cac:TaxRepresentativeParty/cac:PostalAddress/cac:AddressLine/cbc:Line` | optional |
| NEW field needed: `tax_representative.address.city` (invoice) | BT-66 Tax representative city | `/Invoice/cac:TaxRepresentativeParty/cac:PostalAddress/cbc:CityName` | law (as BT-64) |
| NEW field needed: `tax_representative.address.code` (invoice) | BT-67 Tax representative post code | `/Invoice/cac:TaxRepresentativeParty/cac:PostalAddress/cbc:PostalZone` | law (as BT-64) |
| NEW field needed: `tax_representative.address.region` (invoice) | BT-68 Tax representative country subdivision | `/Invoice/cac:TaxRepresentativeParty/cac:PostalAddress/cbc:CountrySubentity` | optional |
| `tax_representative.country` (invoice) | BT-69 Tax representative country code | `/Invoice/cac:TaxRepresentativeParty/cac:PostalAddress/cac:Country/cbc:IdentificationCode` | derived: kept in GOBL as the tax-ID country, which gobl.ubl writes into `cac:PostalAddress/cac:Country` (ubl:party.go:250-256). BG-12 holds BT-69 only; BR-19 / BR-20 pass |

## 3. Correction (Credit Note)

XPaths from the `ubl-cn` binding. A `CORRECTION` with `data.type` = `INVOICE` carries a full invoice under `data.`; every §1 row applies with that prefix and the `/CreditNote` root (`cac:CreditNoteLine` instead of `cac:InvoiceLine`). A receipt correction fails in the worker (§6.1).

| UAPI | EN 16931 BT | XRechnung UBL XPath | notes |
| --- | --- | --- | --- |
| `type` = CORRECTION, `data.type` = INVOICE | — | `/CreditNote` | GOBL `credit-note` (W:196-197) → root `CreditNote` (ubl:invoice.go:150-156; ubl:lines.go:84-88) |
| `type` = CORRECTION, `data.type` = INVOICE | BT-3 Invoice type code | `/CreditNote/cbc:CreditNoteTypeCode` | derived: `381` (GOBL `credit-note`, W:196-197; gobl:addons/eu/en16931/scenarios.go:29-36). No `384` or `383` path; upward corrections are `381` too |
| `record.id` | BG-3 PRECEDING INVOICE REFERENCE, BT-25 Preceding Invoice reference | `/CreditNote/cac:BillingReference/cac:InvoiceDocumentReference/cbc:ID` | derived: referenced record → `used_in` → Invopop entry → original GOBL `code` (W:208-218; E/invopop.go:1101-1129). Missing `used_in` → worker error; at sync GOBL `DE-XRECHNUNG-02` fails. On Peppol this may be the `PPL` value: UNVERIFIED |
| `record.id` | BT-26 Preceding Invoice issue date | `/CreditNote/cac:BillingReference/cac:InvoiceDocumentReference/cbc:IssueDate` | derived: original issue date from Invopop |
| `data.document.references.preceding_document.number`, `data.document.references.preceding_document.issued_at` | BT-25 Preceding Invoice reference / BT-26 Preceding Invoice issue date | — | ignored with a warning log (G/invoice.go:191-205; E/invopop.go:706) |
| `record.offset` | — | — | ignored |
| `reason.text` | BT-22 Invoice note (BT-21 Invoice note subject code `ACD`) | `/CreditNote/cbc:Note` | GOBL note key `reason` → `#ACD#<text>` (G/invoice.go:215-222; gobl:addons/eu/en16931/org.go:28; ubl:utils.go:77-86) |
| `reason.type` | BT-22 Invoice note (BT-21 Invoice note subject code `ACD`) | `/CreditNote/cbc:Note` | default when no text: the raw enum, e.g. `#ACD#FACTUAL_ERROR` (`src/unified/service/einvoicing/correction.go:8-14`) |
| `-` | BT-126 Invoice line identifier | `/CreditNote/cac:CreditNoteLine/cbc:ID` | derived: index + 1 (G/entries.go:76-84; ubl:lines.go:76-77) |
| `data.entries[i].data.unit.quantity` | BT-129 Invoiced quantity | `/CreditNote/cac:CreditNoteLine/cbc:CreditedQuantity` | sign as supplied |
| `data.entries[i].data.unit.measure` | BT-130 Invoiced quantity unit of measure code | `/CreditNote/cac:CreditNoteLine/cbc:CreditedQuantity/@unitCode` | as §1.3 |
| `data.entries[i].type` | — | — | ignored; nothing is negated. API docs require positive amounts on RETURN lines; a negative unit price fails BR-27 (fatal) |
| `data.entries[i].details.number`, `data.entries[i].details.prepayment.document_number` | — | — | `details.number` is the corrected invoice's line number; EN 16931:2017 has no element for it (BT-220 exists only in EN 16931-1:2026). ignored: the corrected line / prepayment document is not referenced |
| `data.payments[i].details.date` (OUTSTANDING) | BT-9 Payment due date | `/CreditNote/cac:PaymentMeans/cbc:PaymentDueDate` | ubl:payment.go:188-191 (`cbc:DueDate` does not exist in CreditNote) |
| `data.document.references.project` | BT-11 Project reference | `/CreditNote/cac:AdditionalDocumentReference/cbc:ID` | ignored (DE). If re-enabled: `cac:AdditionalDocumentReference` with `cbc:DocumentTypeCode` `50`, a code UBL-SR-43 allows only on a CreditNote. The UBL 2.1 CreditNote XSD has no `cac:ProjectReference` |
| `data.recipients[i]` | BG-7 BUYER | `/CreditNote/cac:AccountingCustomerParty` | must equal the original's recipients (EA:345, 374) |

**Sign convention.** EN 16931 expresses the reduction through BT-3 `381` with positive amounts. fiskaly passes signs through unchanged (the only negation in G/ is the rounding direction, G/line_rounding.go:110-113), so the API rule "positive amounts on RETURN lines" keeps the credit note valid. BR-DE-26 (warning) needs BG-3 only for `384`; the GOBL addon requires a preceding reference for every credit note (gobl:addons/de/xrechnung/bill_invoices.go:35-40). An upward correction sent this way (`381`, positive amounts) passes validation and credits the buyer instead of charging more.

## 4. Fixed and Derived Values

| UAPI | EN 16931 BT | XRechnung UBL XPath | notes |
| --- | --- | --- | --- |
| `-` | BT-24 Specification identifier | `/Invoice/cbc:CustomizationID` | fixed `urn:cen.eu:en16931:2017#compliant#urn:xeinkauf.de:kosit:xrechnung_3.0` (xinv:xinvoice.go:25). Also over Peppol, BE recipient addresses included: the receiver must accept the XRechnung 3.0 document type, so a receiver registered only for Peppol BIS Billing 3 is likely unreachable (inferred). The Peppol document type identifier is set inside Invopop's `peppol.send`: UNVERIFIED |
| `-` | BT-23 Business process type | `/Invoice/cbc:ProfileID` | fixed `urn:fdc:peppol.eu:2017:poacc:billing:01:1.0` (xinv:xinvoice.go:36) |
| `type` | BT-3 Invoice type code | `/Invoice/cbc:InvoiceTypeCode`, `/CreditNote/cbc:CreditNoteTypeCode` | derived: INVOICE → `380`, CORRECTION → `381` (W:192-197) |
| `-` | BT-81 Payment means type code | `/Invoice/cac:PaymentMeans/cbc:PaymentMeansCode` | target: derived from `payments[i].type` + `payments[i].kind` (§1.4). As built: default `1`; fixed `30` / `49` |
| `-` | BT-41 Seller contact point | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:Contact/cbc:Name` | default taxpayer `name.legal` |
| `-` | BT-34 Seller electronic address / BT-49 Buyer electronic address | `/Invoice/cac:AccountingSupplierParty/cac:Party/cbc:EndpointID`; `/Invoice/cac:AccountingCustomerParty/cac:Party/cbc:EndpointID` | derived: Peppol ID, else an e-mail address with `@schemeID` `EM` |
| `-` | BT-72 Actual delivery date | `/Invoice/cac:Delivery/cbc:ActualDeliveryDate` | As built (deviation from the target `recipients[i].shipping.date` without default, §1.2): always written, from `document.operation_date`, else the issue date |
| `-` | BG-15 DELIVER TO ADDRESS | `/Invoice/cac:Delivery/cac:DeliveryLocation/cac:Address` | default, always written: shipping address, else buyer address (GOBL addon rule 25 requires a receiver) |
| `-` | BT-130 Invoiced quantity unit of measure code | `/Invoice/cac:InvoiceLine/cbc:InvoicedQuantity/@unitCode` | default `C62` |
| `-` | BT-151 Invoiced item VAT category code / BT-118 VAT category code / BT-121 VAT exemption reason code / BT-120 VAT exemption reason text | `/Invoice/cac:InvoiceLine/cac:Item/cac:ClassifiedTaxCategory/cbc:ID`; `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cac:TaxCategory/cbc:ID`; `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cac:TaxCategory/cbc:TaxExemptionReasonCode`; `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cac:TaxCategory/cbc:TaxExemptionReason` | derived per `vat.type` and `vat.code` from the code table maintained separately (§1.3) |
| `-` | BT-119 VAT category rate / BT-152 Invoiced item VAT rate | `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cac:TaxCategory/cbc:Percent`; `/Invoice/cac:InvoiceLine/cac:Item/cac:ClassifiedTaxCategory/cbc:Percent` | derived: `0` for E/K/G; omitted for O |
| `-` | BT-139 Invoice line allowance reason / BT-144 Invoice line charge reason | `/Invoice/cac:InvoiceLine/cac:AllowanceCharge[cbc:ChargeIndicator = 'false']/cbc:AllowanceChargeReason`, `/Invoice/cac:InvoiceLine/cac:AllowanceCharge[cbc:ChargeIndicator = 'true']/cbc:AllowanceChargeReason` | fixed "Special Discount" / "Special Surcharge" (target: NEW reason fields, §1.3) |
| `-` | BT-21 Invoice note subject code | `/Invoice/cbc:Note` | fixed `ACD` for the correction reason |
| `-` | BT-28 Seller trading name / BT-45 Buyer trading name | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PartyName/cbc:Name`; `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PartyName/cbc:Name` | derived: copies of BT-27 / BT-44 (target: `name.trade` §2, `recipients[i].trade_name` §1.2) |
| `-` | Tax scheme of BT-31 Seller VAT identifier, BT-48 Buyer VAT identifier, BT-63 Seller tax representative VAT identifier, BG-23 VAT BREAKDOWN, BG-30 LINE VAT INFORMATION | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PartyTaxScheme/cac:TaxScheme/cbc:ID`; `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PartyTaxScheme/cac:TaxScheme/cbc:ID`; `/Invoice/cac:TaxRepresentativeParty/cac:PartyTaxScheme/cac:TaxScheme/cbc:ID`; `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cac:TaxCategory/cac:TaxScheme/cbc:ID`; `/Invoice/cac:InvoiceLine/cac:Item/cac:ClassifiedTaxCategory/cac:TaxScheme/cbc:ID` | fixed `VAT` |
| `-` | BT-31 Seller VAT identifier / BT-48 Buyer VAT identifier / BT-63 Seller tax representative VAT identifier (country prefix) | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PartyTaxScheme[cac:TaxScheme/cbc:ID = 'VAT']/cbc:CompanyID`; `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PartyTaxScheme/cbc:CompanyID`; `/Invoice/cac:TaxRepresentativeParty/cac:PartyTaxScheme/cbc:CompanyID` | derived from `fiscalization.type` / recipient `address.country` / `tax_representative.country`. BT-48 target: the prefix sent in `identification.number`, address country only as fallback (§1.2) |
| `-` | BT-2 Invoice issue date | `/Invoice/cbc:IssueDate` | default today |
| `-` | BT-1 Invoice number | `/Invoice/cbc:ID` | Peppol: possible `PPL` sequence value (UNVERIFIED) |
| `-` | BG-24 ADDITIONAL SUPPORTING DOCUMENTS | `/Invoice/cac:AdditionalDocumentReference` | fixed: none. The e-mail workflow sets `attach_invoice_pdf: false` (WF/DE_SEND_INVOICE_EMAIL_XRECHNUNG_BUSINESS.json:26), so the buyer gets the XML without a PDF; the Peppol workflow attaches nothing either |
| `-` | — (Invopop document type) | — | fixed `doc_type` `xrechnung-invoice-v2` with `validate_ubl: true` in both workflows (WF/DE_SEND_INVOICE_EMAIL_XRECHNUNG_BUSINESS.json:26-29; WF/DE_SEND_INVOICE_PEPPOL_XRECHNUNG_BUSINESS.json:38-40); corrections use the same workflow (E/invopop.go:398-404). That it runs the gobl.de.xinvoice format `ubl+de-xrechnung-v3` (xinv:ubl_format.go:15-33): UNVERIFIED (Invopop-side) |

## 5. Format Elements Without a UAPI Source (Gaps)

### 5.1 NEW fields (index)

Every field the target mapping in §1–§2 needs and the API lacks. Paths relative to `content.operation`; "(taxpayer)" = taxpayer root.

| NEW UAPI field | EN 16931 BT | Required? | section |
| --- | --- | --- | --- |
| `document.invoice_currency` | BT-5 | format-mandatory (1..1) | §1.1 |
| `document.tax_point_date_code` | BT-8 | optional | §1.1 |
| `document.early_payment_discounts[i].end_date`, `.percentage` | BT-20 (Skonto line) | law (§ 14 Abs. 4 Satz 1 Nr. 7 UStG) | §1.1 |
| `document.references.sales_order` | BT-14 | optional | §1.1 |
| `document.references.receiving_advice` | BT-15 | optional | §1.1 |
| `document.references.invoiced_object` | BT-18 | optional | §1.1 |
| `document.text_subject_code` | BT-21 | optional | §1.1 |
| `document.period.start`, `.end` | BG-14 BT-73, BT-74 | law (§ 14 Abs. 4 Satz 1 Nr. 6 UStG) | §1.1 |
| `document.attachments[i].id` | BG-24 BT-122 | format-mandatory (1..1 within BG-24) | §1.1 |
| `document.attachments[i].description`, `.uri`, `.content` | BT-123, BT-124, BT-125 | optional | §1.1 |
| `document.attachments[i].mime_code`, `.filename` | BT-125 attributes | format-mandatory (with BT-125); BR-DE-22 | §1.1 |
| `recipients[i].trade_name` | BT-45 | optional | §1.2 |
| `recipients[i].contact.name`, `.phone`, `.email` | BG-9 BT-56, BT-57, BT-58 | optional | §1.2 |
| `recipients[i].shipping.location_id` | BT-71 | optional | §1.2 |
| `entries[i].data.unit.factor_measure` | BT-150 | optional (PEPPOL-EN16931-R130) | §1.3 |
| `entries[i].data.unit.price.gross`, `entries[i].data.unit.price.discount` | BT-148, BT-147 | optional | §1.3 |
| `entries[i].data.value.discount.base`, `.percentage` | BT-137, BT-138 | optional | §1.3 |
| `entries[i].data.value.discount.reason`, `.reason_code` | BT-139, BT-140 | conditional (BR-42) | §1.3 |
| `entries[i].data.value.surcharge.base`, `.percentage` | BT-142, BT-143 | optional | §1.3 |
| `entries[i].data.value.surcharge.reason`, `.reason_code` | BT-144, BT-145 | conditional (BR-44) | §1.3 |
| `entries[i].data.product.code_scheme` | BT-158 scheme identifier | conditional (BR-65) | §1.3 |
| `entries[i].data.product.attributes[i].name`, `.value` | BG-32 BT-160, BT-161 | optional | §1.3 |
| `entries[i].data.product.buyer_number` | BT-156 | optional | §1.3 |
| `entries[i].data.product.origin_country` | BT-159 | optional | §1.3 |
| `entries[i].details.object_id` | BT-128 | optional | §1.3 |
| `entries[i].details.order_line_reference` | BT-132 | optional | §1.3 |
| `entries[i].details.buyer_accounting_reference` | BT-133 | optional | §1.3 |
| `entries[i].details.period.start`, `.end` | BG-26 BT-134, BT-135 | law (§ 14 Abs. 4 Satz 1 Nr. 6 UStG) | §1.3 |
| `payments[i].holder_name` | BT-88 | optional | §1.4 |
| `payee.name` | BG-10 BT-59 | conditional (BR-17) | §1.4 |
| `payee.identifier`, `payee.company_id` | BT-60, BT-61 | optional | §1.4 |
| `document.allowances[i].amount` | BG-20 BT-92 | law (§ 14 Abs. 4 Satz 1 Nr. 7 UStG) | §1.5 |
| `document.allowances[i].base`, `.percentage`, `.vat_percentage` | BT-93, BT-94, BT-96 | optional | §1.5 |
| `document.allowances[i].vat_type` | BT-95 | format-mandatory (BR-32) | §1.5 |
| `document.allowances[i].reason`, `.reason_code` | BT-97, BT-98 | conditional (BR-33) | §1.5 |
| `payments[i].details.discount_vat.type` | BT-95 (payment discount) | format-mandatory (BR-32) | §1.5 |
| `payments[i].details.discount_vat.percentage` | BT-96 (payment discount) | optional | §1.5 |
| `document.charges[i].amount`, `.base`, `.percentage`, `.vat_percentage` | BG-21 BT-99, BT-100, BT-101, BT-103 | optional | §1.5 |
| `document.charges[i].vat_type` | BT-102 | format-mandatory (BR-37) | §1.5 |
| `document.charges[i].reason`, `.reason_code` | BT-104, BT-105 | conditional (BR-38) | §1.5 |
| `totals.vat.amount_accounting` | BT-111 | conditional (BR-53; Art. 230 Directive 2006/112/EC) | §1.5 |
| `totals.rounding` | BT-114 | optional | §1.5 |
| `seller.identifier` | BT-29 | conditional (BR-CO-26) | §2 |
| `fiscalization.registration.company_id` (taxpayer, DE) | BT-30 | conditional (BR-CO-26) | §2 |
| `fiscalization.tax_number` (taxpayer) | BT-32 | law (§ 14 Abs. 4 Satz 1 Nr. 2 UStG) | §2 |
| `seller.legal_information` | BT-33 | optional (company law UNVERIFIED) | §2 |
| `tax_representative.address.line.street`, `.line.number`, `.city`, `.code` | BG-12 BT-64, BT-66, BT-67 | law (§ 22c Nr. 2 UStG) | §2 |
| `tax_representative.address.line.suffix`, `.line.prefix`, `.line.infix`, `.region` | BT-65, BT-164, BT-68 | optional | §2 |

### 5.2 Other gaps

Mandatory first (1..x or made mandatory by a rule), then mandatory-in-context, then important optional BTs.

| BT | element | cardinality / rule | status today | proposal |
| --- | --- | --- | --- | --- |
| BT-10 Buyer reference | `/Invoice/cbc:BuyerReference` | 1..1, BR-DE-15 (fatal) | optional in the API, no default; missing → GOBL failure at sync, no 4xx. Max 32 < Leitweg-ID 46 | Require `document.references.buyer` (4xx) for XRechnung; raise max length to ≥ 46 |
| BT-42 Seller contact telephone number | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:Contact/cbc:Telephone` | 1..1, BR-DE-6 (fatal) | `seller.phone` optional; missing → sync failure | Require for XRechnung, or hold seller contact data on the taxpayer |
| BT-43 Seller contact email address / BT-34 Seller electronic address | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:Contact/cbc:ElectronicMail`; `/Invoice/cac:AccountingSupplierParty/cac:Party/cbc:EndpointID` | 1..1, BR-DE-7 (fatal); PEPPOL-EN16931-R020 | `seller.email` optional; it is also the BT-34 fallback, and at sync the only BT-34 source | as BT-42 |
| BT-3 Invoice type code = `384`, `389`, `326`, `875`, `876`, `877` (and `383`, `386`) | `/Invoice/cbc:InvoiceTypeCode` | BR-DE-17 lists 326, 380, 384, 389, 381, 875, 876, 877; BR-DE-26 | `type` yields only `380` / `381`; an upward correction credits the buyer (§3) | BT-3 stays governed by `type` (no type-code field). Not expressible: `384` corrected invoice (Rechnungsberichtigung, § 31 Abs. 5 UStDV; `383` debit note is not in the BR-DE-17 list); `389` self-billed invoice (Gutschrift, § 14 Abs. 2 Satz 5 and Abs. 4 Satz 1 Nr. 10 UStG); `326` partial invoice and `875`–`877` construction invoices (advance and final invoices, § 14 Abs. 5 UStG; `386` prepayment invoice is not in the BR-DE-17 list) |
| BT-119 VAT category rate | `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cac:TaxCategory/cbc:Percent` | 1..1, BR-DE-14 (fatal) | omitted for O | Converter writes `0` for O (Invopop change) |
| BT-151 Invoiced item VAT category code = O | `/Invoice/cac:InvoiceLine/cac:Item/cac:ClassifiedTaxCategory/cbc:ID` | BR-O-02 (fatal) | `NOT_SUBJECT` → O, but BT-31 and BT-48 are always present | Reject `NOT_SUBJECT` for XRechnung at sync until the code table maps it (cross-border B2B services belong to AE) |
| BT-151 Invoiced item VAT category code = AE, BT-120 VAT exemption reason text, BT-121 VAT exemption reason code | `/Invoice/cac:InvoiceLine/cac:Item/cac:ClassifiedTaxCategory/cbc:ID`; `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cac:TaxCategory/cbc:TaxExemptionReason`; `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cac:TaxCategory/cbc:TaxExemptionReasonCode` | BR-AE-01…10 | `VAT_REVERSE_CHARGE` rejected | Map to GOBL `reverse-charge` (AE); BT-120 / BT-121 from the code table maintained separately (§1.3) |
| BG-16 PAYMENT INSTRUCTIONS / BT-81 Payment means type code | `/Invoice/cac:PaymentMeans`; `/Invoice/cac:PaymentMeans/cbc:PaymentMeansCode` | 1..1, BR-DE-1 | default `1`; advances' method lost | Derive from `payments[i].type` + `kind` (§1.4), with BG-18 for card codes (BR-DE-24) |
| BT-81 Payment means type code = 58 / 59 | `/Invoice/cac:PaymentMeans/cbc:PaymentMeansCode` | BR-DE-19 / BR-DE-20 (warning) | SEPA payments sent as `30` / `49` | Use GOBL `credit-transfer+sepa` / `direct-debit+sepa` |
| BT-47 Buyer legal registration identifier | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PartyLegalEntity/cbc:CompanyID` | 0..1; UBL-SR-16 (fatal) | `company_id` written as a second BT-46 | Set identity scope `legal` for `company_id` |
| BT-82 Payment means text / BT-83 Remittance information | `/Invoice/cac:PaymentMeans/cbc:PaymentMeansCode/@name`; `/Invoice/cac:PaymentMeans/cbc:PaymentID` | 0..1 | `instruction.text` → BT-82, `instruction.properties` → BT-83 | Map `instruction.text` to GOBL `ref` (BT-83); BT-82 from the payment type; `instruction.properties` to no BT |
| BT-70 Deliver to party name | `/Invoice/cac:Delivery/cac:DeliveryParty/cac:PartyName/cbc:Name` | 0..1 | stripped | Converter writes only `cac:PartyName`; then stop stripping |
| BT-11 Project reference | `/Invoice/cac:ProjectReference/cbc:ID` | 0..1 | dropped for all DE invoices | Re-enable for Invoice; CreditNote via `AdditionalDocumentReference` type `50` |
| BT-22 Invoice note | `/Invoice/cbc:Note` | 0..n (BG-1) | `document.text` ignored | Map to a GOBL note |
| BT-7 Value added tax point date / BT-72 Actual delivery date | `/Invoice/cbc:TaxPointDate`; `/Invoice/cac:Delivery/cbc:ActualDeliveryDate` | 0..1; § 14 Abs. 4 Satz 1 Nr. 6 UStG needs the date of supply | `operation_date` → BT-72, issue date when absent; `shipping.date` ignored | `operation_date` → BT-7, `shipping.date` → BT-72, else `operation_date` copied to BT-72 when no period is given; no silent issue-date fallback; BG-14 from the NEW period fields (§5.1) |
| BT-127 Invoice line note | `/Invoice/cac:InvoiceLine/cbc:Note` | 0..1 | `details.description` ignored | Map to line notes |
| BT-154 Item description / BT-155 Item Sellers identifier / BT-157 Item standard identifier / BT-158 Item classification identifier, BG-32 ITEM ATTRIBUTES | `/Invoice/cac:InvoiceLine/cac:Item/cbc:Description`; `/Invoice/cac:InvoiceLine/cac:Item/cac:SellersItemIdentification/cbc:ID`; `/Invoice/cac:InvoiceLine/cac:Item/cac:StandardItemIdentification/cbc:ID`; `/Invoice/cac:InvoiceLine/cac:Item/cac:CommodityClassification/cbc:ItemClassificationCode`; `/Invoice/cac:InvoiceLine/cac:Item/cac:AdditionalItemProperty` | 0..1 / 0..n | `product.*` ignored | label → BT-154; SKU / MPN / OTHER → BT-155; GTIN → BT-157 `@schemeID` `0160`; code → BT-158 + `@listID` from `product.code_scheme`; brand/model/serial/batch → BG-32 |
| BT-149 Item price base quantity | `/Invoice/cac:InvoiceLine/cac:Price/cbc:BaseQuantity` | 0..1 | `unit.factor` ignored | Map to GOBL `item.per` |
| BT-28 Seller trading name | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PartyName/cbc:Name` | 0..1 | holds `name.legal`; `name.trade` ignored | Map `name.trade` to GOBL `alias` |
| BT-131 / BT-136 / BT-141 / BT-146 | `/Invoice/cac:InvoiceLine/cbc:LineExtensionAmount`; `/Invoice/cac:InvoiceLine/cac:AllowanceCharge[cbc:ChargeIndicator = 'false']/cbc:Amount`; `/Invoice/cac:InvoiceLine/cac:AllowanceCharge[cbc:ChargeIndicator = 'true']/cbc:Amount`; `/Invoice/cac:InvoiceLine/cac:Price/cbc:PriceAmount` | 1..1 / 0..1 | derived from the VAT-inclusive values | Read the `.exclusive` fields (§1.3); reject a BT-131 that differs from BT-129 × BT-146 − BT-136 + BT-141 beyond rounding |

## 6. Common Validation Failures

### 6.1 fiskaly-side errors for this flow

| Stage | Message (abridged) | Trigger | Source |
| --- | --- | --- | --- |
| A: API (501) | `<TYPE> E_INVOICE_SERVICE is not supported for <cc>` | EMAIL or PEPPOL recipient address outside DE/BE | RH:3332-3336; `src/unified/api/errmap/errmap.go:29` |
| A: API (400) | `destination code '' must match the format '^[A-Z0-9]{7}$'` | CONSUMER recipient with an EMAIL or PEPPOL block: parsed as SDI without a destination code. `consumer recipients only support SDI invoicing` fires only on an invoicing body that does not decode | RH:3557-3568; `pkg/identifier/sdi/destination_code.go:49-54` |
| B: sync (2xx; record `COMPLETED` with one error log of at most 256 characters; nothing is sent) | `TRANSACTION::INVOICE needs exactly 1 business recipient with invoicing…`; `business recipient identification is not a valid VAT identification`; `recipients[0].identification.number: …` | recipient shape / VAT format | EA:60-93; SV1:3739-3766 |
| B | `supplier is missing a Peppol identifier` / `supplier is not registered for PEPPOL` | PEPPOL without registration | EA:145-186 |
| B | `invoice correction recipients do not match the original invoice` | correction | EA:345, 374 |
| C: GOBL (sync and worker) | `invalid gobl invoice: ` plus the fault texts, with code and path dropped (G/error_translation.go:21-46): `ordering is required (BR-DE-15)` (`DE-XRECHNUNG-29`), `ordering code is required (BR-DE-15)` (`-30`), `either party.telephones or party.people[0].telephones is required (BR-DE-6)` (`-09`), `either party.emails or party.people[0].emails is required (BR-DE-7)` (`-10`), `supplier inboxes are required (PEPPOL-EN16931-R020)` (`-08`), `preceding documents are required for corrective and credit note invoices (BR-DE-26)` (`-02`) | missing BT-10, BT-42, BT-43, BT-34 (at sync: no `seller.email`), BG-3 | gobl:addons/de/xrechnung/bill_invoices.go:35-140; E/invopop.go:674-677 |
| C | `invalid German VAT identity code`; `taxpayer fiscalization does not have tax code` | DE taxpayer with only / no tax ID | gobl:regimes/de/tax_identity.go:17-31; G/taxpayer.go:43-44 |
| C | `unsupported VAT type … 'REVERSE_CHARGE'`; `unsupported VAT exemption code '<c>' for Germany`; `missing VAT exemption reason for code '<c>'` | reverse charge; `CAUSE_n` > 31; `CAUSE_31` without reason | G/entries.go:164-169, 239-244; G/german_exemption.go:62-74 |
| C | `unsupported payment type: MEAL_VOUCHER\|DEPOSIT`; `payment due date not set` | those payment types; OUTSTANDING without date | G/invoice.go:300-302, 610-613 |
| C | `payment terms are required when amount is due (BR-CO-25)` (EN 16931 rule 07), as GOBL words it. The friendlier text in E/error_translation.go:15-24 is used only for taxpayer-registration errors (E/service.go:509, 886-905) | standard invoice with amount due > 0 and neither BT-9 nor BT-20; credit notes are exempt | gobl:addons/eu/en16931/bill.go:143-151, 259-262; G/error_translation.go:26-47 (called from E/invopop.go:677) |
| C | EN 16931 pay instructions 01 (BR-49) | `instruction.type` UNKNOWN (from code) | gobl:addons/eu/en16931/pay.go:40-48 |
| D: worker | `supplier has no Peppol identifier yet…`; `referenced record is missing used_in identifier`; `no commissioned E_INVOICE_SERVICE found for taxpayer …`; `TRANSACTION::CORRECTION with receipt correction is not supported` | PEPPOL before proof of ownership; correction of an untransmitted invoice; no commissioned e-invoice system; correction of a receipt | E/service.go:166; W:213-216, 277-278, 199-201 |
| E: Invopop (webhook, after `COMPLETED`) | Schematron / XSD faults (§6.2), logged as Invopop words them | — | `validate_ubl: true` (§4); `buildErrorLogs` (E/service.go:662, 860-884). Invopop's artefact versions: UNVERIFIED |

### 6.2 Rules most likely to trip

| Rule | Severity | UAPI trigger | Caught at |
| --- | --- | --- | --- |
| BR-DE-15 | fatal | `document.references.buyer` missing | sync (GOBL) |
| BR-DE-6 / BR-DE-7 | fatal | `seller.phone` / `seller.email` missing | sync (GOBL) |
| PEPPOL-EN16931-R020 | fatal | no `seller.email`: the sync check never has `peppol_id` (E/invopop.go:330-338) | sync (GOBL) |
| BR-O-02 | fatal | `vat.code` `NOT_SUBJECT` (O with BT-31 and BT-48) | Invopop only; from code + EN-sch, not executed |
| BR-DE-14 | fatal | `NOT_SUBJECT`: BT-119 omitted for O | Invopop only |
| UBL-SR-16 | fatal | `buyer_id` and `company_id` both set | Invopop only |
| BR-27 | fatal | negative unit price (e.g. RETURN with negative values) | Invopop only |
| BR-DE-18 | fatal | `document.payment_terms` lines starting with `#` outside the Skonto syntax | Invopop only |
| BR-CO-25 | — (removed from the CEN UBL Schematron in v1.3.16, which the KoSIT validator configuration 2026-08-31 bundles; Invopop's artefact version UNVERIFIED) | standard invoice with amount due > 0 and no OUTSTANDING date or `payment_terms`; credit notes exempt | sync (GOBL) |
| BR-DE-27 / BR-DE-28 | warning | phone with < 3 digits / odd email | Invopop |
| PEPPOL-EN16931-R120 | warning | BT-131 ≠ BT-129 × BT-146 after rounding | Invopop |
| UBL-CR-335 | warning | `tax_representative` present | Invopop |
| BR-DE-16, BR-DE-21, PEPPOL-EN16931-R001, PEPPOL-EN16931-R061 | fatal / warning | cannot occur today: BT-31, BT-24, BT-23 always written; mandate ID required by the API | — |

## Sources

| Artefact | Version / pin | Path or URL | Read |
| --- | --- | --- | --- |
| XRechnung CIUS model (terms, cardinalities, BR-DE rules, `ubl-inv` / `ubl-cn` bindings) | model 2026-08-31 in bundle 3.0.2 2026-08-31 | formats repo `Germany/xrechnung-3.0.2-bundle-2026-08-31/xrechnung-3.0.2-xrechnung-model-2026-08-31/model/xrechnung-cius-model.xml` (`xrechnung-extension-model.xml` for context) | 2026-11-03 |
| XRechnung specification | 3.0.2, published 2024-06-20 (KoSIT) | bundle `XRechnung-v3.0.2.pdf` | 2026-11-03 |
| XRechnung Schematron | 2.6.0 | bundle `xrechnung-3.0.2-schematron-2.6.0/schematron/ubl/XRechnung-UBL-validation.sch`, `schematron/common.sch` | 2026-11-03 |
| XRechnung validator configuration, UBL 2.1 XSD, test suite | 2026-08-31 | bundle `xrechnung-3.0.2-validator-configuration-2026-08-31/` (`scenarios.xml`, `resources/ubl/2.1/xsd/maindoc/`); `xrechnung-3.0.2-testsuite-2026-08-31/instances/standard/` | 2026-11-03 |
| EN 16931 validation artefacts (UBL) | v1.3.17 (2026-10-12), commit `6e35e0a` | formats repo `eInvoicing-EN16931/ubl/schematron/`; https://github.com/ConnectingEurope/eInvoicing-EN16931/releases/tag/validation-1.3.17. The XRechnung validator configuration 2026-08-31 bundles v1.3.16 (`scenarios.xml`) | 2026-11-03 |
| Leitweg-ID Formatspezifikation (KoSIT) | v2.0.2, Fassung 2021-07-28 | https://xeinkauf.de/app/uploads/2022/11/Leitweg-ID-Formatspezifikation-v2-0-2-1.pdf (§2.1: "maximal 46 Stellen"; §3.4). xeinkauf.de still lists 2.0.2 as current | retrieved 2026-11-03 |
| Umsatzsteuergesetz, § 14 Abs. 2, 4, 5, § 14a Abs. 4, 5, § 22c | consolidated, Stand Art. 5 G v. 29.6.2026 I Nr. 197 | formats repo `Germany/Laws/01_UStG_konsolidiert.md` (gesetze-im-internet.de) | 2026-11-16 |
| Umsatzsteuer-Durchführungsverordnung, § 31 Abs. 4, 5 | consolidated | formats repo `Germany/Laws/02_UStDV_konsolidiert.md` (gesetze-im-internet.de) | 2026-11-16 |
| Directive 2006/112/EC, Art. 226, 230 | consolidated; Art. 226 unchanged since Directive 2010/45/EU | EUR-Lex answered with a bot challenge; read from the legislation.gov.uk EU-derived copy (`/eudr/2006/112/article/226`, `/230`) | 2026-11-10 |
| UAPI v6 definition / API docs | `fiskaly/meta@cd19ec88cb299ee41a44302448006340dd028bf0`; docs version 2026-06-01 | [`UAPI_v6_Fields.md`](UAPI_v6_Fields.md); [`fiskaly.unified-api.BE_EI-DE_EI-IT_EI.2026-06-01 (1).yaml`](../fiskaly.unified-api.BE_EI-DE_EI-IT_EI.2026-06-01%20(1).yaml) | 2026-11-03 |
| fiskaly code and Invopop workflows (`DE_SEND_INVOICE_EMAIL_XRECHNUNG_BUSINESS`, `DE_SEND_INVOICE_PEPPOL_XRECHNUNG_BUSINESS`, both `doc_type` `xrechnung-invoice-v2`) | `fiskaly/meta@cd19ec8` (paths in the citation legend) | GitHub `fiskaly/meta` | 2026-11-04 |
| GOBL (addons `de-xrechnung-v3`, `eu-en16931-v2017`, regime DE) | v0.501.0, commit `225e39d` (2026-06-16) | https://github.com/invopop/gobl/tree/v0.501.0 | 2026-11-03 |
| gobl.ubl | v0.99.0, commit `540f201` (2026-10-28); latest tag, production version not visible. Its `go.mod` requires gobl v0.511.0, and it calls `RoundToCurrency` (ubl:ubl.go:164), which gobl v0.501.0 lacks, so it does not build against fiskaly's pin | https://github.com/invopop/gobl.ubl/tree/v0.99.0 | 2026-11-03 |
| gobl.de.xinvoice (`ubl+de-xrechnung-v3`) | v0.0.9, commit `8116d89` (2026-10-15); its `go.mod` requires gobl v0.510.0 and gobl.ubl v0.93.0, so Invopop's GOBL may be newer than fiskaly's. Its `addon/xrechnung` rules are identical to gobl v0.501.0's. That `xrechnung-invoice-v2` runs this format: UNVERIFIED | https://github.com/invopop/gobl.de.xinvoice/tree/v0.0.9 | 2026-11-03 |
| As-built trace UAPI → GOBL (secondary, spot-checked) | 2026-11-03 | internal working note | 2026-11-03 |
