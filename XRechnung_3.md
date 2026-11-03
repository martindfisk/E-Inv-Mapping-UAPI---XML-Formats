# UAPI v6 → XRechnung 3.0.2 (UBL)

As of 2026-11-03 · Status: draft for review

UAPI v6 invoice and correction fields mapped to XRechnung 3.0.2 UBL. A DE supplier's invoice becomes XRechnung when the business recipient has `invoicing.type` `PEPPOL`, or `EMAIL` with `invoicing.format` `XRECHNUNG_V3` and a DE address. The XPath column is the spec target from the XRechnung model 2026-08-31 (`ubl-inv` binding; `ubl-cn` in §3); the notes column records where today's output departs from it. UAPI paths are relative to `content.operation` unless a table says otherwise; a value in brackets, e.g. "(OUTSTANDING)", restricts the row to that variant. "fatal" / "warning" are Schematron flags.

Code citations: `G/` = fiskaly `src/unified/einvoicing/gobl/`; `E/` = fiskaly `src/unified/einvoicing/`; `RH` = fiskaly `src/unified/api/v6/record_handler.go`; `EA` = fiskaly `src/unified/domain/record/einvoicing_adaptations.go`; `W` = fiskaly `src/unified/worker/e_invoicing_worker.go`; `WF/` = fiskaly `cmd/invopop-workflow-migration/workflows/` (all at `cd19ec8`); `gobl:` = invopop/gobl v0.501.0; `ubl:` = invopop/gobl.ubl v0.99.0; `xinv:` = invopop/gobl.de.xinvoice v0.0.9; `XR-sch` = XRechnung Schematron 2.6.0; `EN-sch` = EN 16931 validation artefacts v1.3.17.

## 1. Invoice Mapping

### 1.1 Header and References

| UAPI | EN 16931 BT | XRechnung UBL XPath | notes |
| --- | --- | --- | --- |
| `document.number` | BT-1 Invoice number | `/Invoice/cbc:ID` | 1..1. Verbatim (G/invoice.go:153; ubl:invoice.go:127). Peppol: the workflow's `sequence.enumerate` step (`PPL`, WF/DE_SEND_INVOICE_PEPPOL_XRECHNUNG_BUSINESS.json:15-26) may replace it: UNVERIFIED |
| `document.series` | BT-1 Invoice number | `/Invoice/cbc:ID` | ignored: GOBL `series` never set (gobl.ubl would write `series-code`, ubl:invoice.go:223-228) |
| `document.issued_at` | BT-2 Invoice issue date | `/Invoice/cbc:IssueDate` | 1..1. Date part; the time is not written. default: absent → today at each build (G/invoice.go:706-714), so sync check and sent XML can differ |
| `-` | BT-3 Invoice type code | `/Invoice/cbc:InvoiceTypeCode` | fixed `380` |
| `payments[i].details.currency` | BT-5 Invoice currency code | `/Invoice/cbc:DocumentCurrencyCode` | 1..1. derived from `payments[0]` only (G/invoice.go:725-741); default `EUR` set in the domain (`src/unified/domain/record/transaction/receipt/payment/details.go:174`; `src/unified/country/de/v1/record.go:25`). API enum `CZK\|EUR\|SEK` |
| `-` | BT-6 VAT accounting currency code | `/Invoice/cbc:TaxCurrencyCode` | gap: written only with an exchange rate (ubl:invoice.go:141-148), which UAPI cannot supply |
| `document.operation_date` | BT-72 Actual delivery date | `/Invoice/cac:Delivery/cbc:ActualDeliveryDate` | API docs: BT-7; spec: the date of supply (§ 14 Abs. 4 Satz 1 Nr. 6 UStG) is BT-72. As built: GOBL `op_date` + `delivery.date` (G/invoice.go:433-441, 716-719) → BT-72 (ubl:delivery.go:38-41). default: absent → issue date |
| `document.operation_date` | BT-7 Value added tax point date | `/Invoice/cbc:TaxPointDate` | ignored: gobl.ubl writes BT-7 only from GOBL `value_date` (ubl:invoice.go:158-161), never set by fiskaly |
| `-` | BT-8 Value added tax point date code | `/Invoice/cac:InvoicePeriod/cbc:DescriptionCode` | gap (no GOBL `tax.point`) |
| `document.payment_terms` | BT-20 Payment terms | `/Invoice/cac:PaymentTerms/cbc:Note` | Verbatim (G/invoice.go:316-321; ubl:payment.go:225-230). Lines starting with `#` must match `#SKONTO#TAGE=n#PROZENT=n.nn#` (BR-DE-18, fatal) |
| `document.references.buyer` | BT-10 Buyer reference | `/Invoice/cbc:BuyerReference` | **1..1, BR-DE-15 (fatal).** Verbatim (G/invoice.go:331-333; G/xrechnung.go:30-37). No Leitweg-ID default: absent → GOBL `DE-XRECHNUNG-29` (neither `document.references` nor `tax_representative` sent) or `-30` at sync (gobl:addons/de/xrechnung/bill_invoices.go:135-140). API max 32 characters; a Leitweg-ID can have 46 |
| `document.references.project` | BT-11 Project reference | `/Invoice/cac:ProjectReference/cbc:ID` | ignored for DE suppliers "due to an Invopop bug (META-3995)" (G/invoice.go:349-353). gobl.ubl writes `cac:ProjectReference` on credit notes too (ubl:ordering.go:124-129), an element the UBL 2.1 CreditNote XSD lacks; that this is META-3995: UNVERIFIED |
| `document.references.contract` | BT-12 Contract reference | `/Invoice/cac:ContractDocumentReference/cbc:ID` | Verbatim (G/invoice.go:355-357; ubl:ordering.go:143-151) |
| `document.references.purchase_order` | BT-13 Purchase order reference | `/Invoice/cac:OrderReference/cbc:ID` | Verbatim (G/invoice.go:358-360; ubl:ordering.go:107-112) |
| `document.references.despatch_advice` | BT-16 Despatch advice reference | `/Invoice/cac:DespatchDocumentReference/cbc:ID` | Verbatim (G/invoice.go:361-363) |
| `document.references.tender` | BT-17 Tender or lot reference | `/Invoice/cac:OriginatorDocumentReference/cbc:ID` | Verbatim (G/invoice.go:364-366) |
| `document.references.buyer_routing` | BT-19 Buyer accounting reference | `/Invoice/cbc:AccountingCost` | Verbatim (G/invoice.go:334-336; ubl:ordering.go:80-83) |
| `-` | BT-14 / BT-15 / BT-18 | `/Invoice/cac:OrderReference/cbc:SalesOrderID`; `/Invoice/cac:ReceiptDocumentReference/cbc:ID`; `/Invoice/cac:AdditionalDocumentReference[cbc:DocumentTypeCode = '130']/cbc:ID` | gap |
| `document.text` | BT-22 Invoice note | `/Invoice/cbc:Note` | ignored (never read). The only note written is the correction reason (§3) |
| `document.references.preceding_document.number` | BT-25 Preceding Invoice reference | `/Invoice/cac:BillingReference/cac:InvoiceDocumentReference/cbc:ID` | `INVOICE` only (e.g. final invoice after advance invoices). Verbatim (G/invoice.go:197-205, 371-383; ubl:ordering.go:53-72). On `CORRECTION` see §3 |
| `document.references.preceding_document.issued_at` | BT-26 Preceding Invoice issue date | `/Invoice/cac:BillingReference/cac:InvoiceDocumentReference/cbc:IssueDate` | as BT-25 |
| `-` | BG-14 BT-73 / BT-74 Invoicing period | `/Invoice/cac:InvoicePeriod/cbc:StartDate`, `/Invoice/cac:InvoicePeriod/cbc:EndDate` | gap. BR-DE-TMP-32 (information) is met because BT-72 is always written |
| `document.activity_code`, `document.offline.reason`, `document.offline.reference` | — | — | ignored; no EN 16931 counterpart |

### 1.2 Buyer and Delivery

| UAPI | EN 16931 BT | XRechnung UBL XPath | notes |
| --- | --- | --- | --- |
| `recipients[i]` (BUSINESS) | BG-7 BUYER | `/Invoice/cac:AccountingCustomerParty` | exactly one (G/invoice.go:400-418) |
| `recipients[i]` (CONSUMER) | BG-7 BUYER | `/Invoice/cac:AccountingCustomerParty` | not reachable. An EMAIL or PEPPOL block fails with 4xx on the empty SDI destination code (RH:3565-3568); no `invoicing` fails at sync (EA:30-33); an SDI block from a DE supplier goes to ZUGFeRD (E/invopop.go:300-313) |
| `recipients[i].name` | BT-44 Buyer name | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PartyLegalEntity/cbc:RegistrationName` | 1..1. Verbatim (G/recipient.go:175; ubl:party.go:207-216) |
| `-` | BT-45 Buyer trading name | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PartyName/cbc:Name` | derived: gobl.ubl copies the name here too (ubl:party.go:208-211) |
| `recipients[i].buyer_id` | BT-46 Buyer identifier | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PartyIdentification/cbc:ID` | GOBL identity `{key: other, label: buyer}`, no scheme (G/recipient.go:159-165; ubl:party.go:322-349) |
| `recipients[i].company_id` | BT-47 Buyer legal registration identifier | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PartyLegalEntity/cbc:CompanyID` | wrong element: identity without legal scope (G/recipient.go:166-172) is written as a second `cac:PartyIdentification`. With `buyer_id` also set → UBL-SR-16 (fatal) |
| `recipients[i].identification.number` (VAT) | BT-48 Buyer VAT identifier | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PartyTaxScheme/cbc:CompanyID` | prefix = recipient `address.country` (G/identification.go:15-36; ubl:party.go:229-249); `cac:TaxScheme/cbc:ID` `VAT`. Format checked at sync (EA:71-93) |
| `recipients[i].identification.number` (TAX, PASSPORT, DOCUMENT, OTHER) | BT-46 / BT-47 | — | rejected: "business recipient identification is not a valid VAT identification" (EA:66). A buyer without VAT ID (e.g. a public body) cannot be invoiced |
| `recipients[i].invoicing.identifier` (PEPPOL) | BT-49 Buyer electronic address | `/Invoice/cac:AccountingCustomerParty/cac:Party/cbc:EndpointID` | 1..1, PEPPOL-EN16931-R010. `ICD:id` → `@schemeID` + value (G/inbox.go:37-76; ubl:party.go:26-62). Leitweg-ID endpoints (ICD `0204`) fit the API pattern |
| `recipients[i].invoicing.email` (EMAIL) | BT-49 Buyer electronic address | `/Invoice/cac:AccountingCustomerParty/cac:Party/cbc:EndpointID` | derived: XRechnung only, EAS `EM` (G/xrechnung.go:68-72; ubl:party.go:47-52) |
| `recipients[i].invoicing.email` (EMAIL) | BT-58 Buyer contact email address | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:Contact/cbc:ElectronicMail` | derived (G/recipient.go:149-156; ubl:party.go:259-261). BT-56 / BT-57: gap |
| `recipients[i].invoicing.format` | — | — | selects the flow (see intro) |
| `recipients[i].address.line.street`, `recipients[i].address.line.number` | BT-50 Buyer address line 1 | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PostalAddress/cbc:StreetName` | `street + " " + number` (G/address.go:33-40; ubl:party.go:429-432) |
| `recipients[i].address.line.suffix` | BT-51 Buyer address line 2 | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PostalAddress/cbc:AdditionalStreetName` | Verbatim |
| `recipients[i].address.line.prefix`, `recipients[i].address.line.infix` | BT-163 Buyer address line 3 | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PostalAddress/cac:AddressLine/cbc:Line` | ignored |
| `recipients[i].address.city` | BT-52 Buyer city | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PostalAddress/cbc:CityName` | 1..1, BR-DE-8 |
| `recipients[i].address.code` | BT-53 Buyer post code | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PostalAddress/cbc:PostalZone` | 1..1, BR-DE-9 |
| `recipients[i].address.region` | BT-54 Buyer country subdivision | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PostalAddress/cbc:CountrySubentity` | Verbatim |
| `recipients[i].address.country` | BT-55 Buyer country code | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PostalAddress/cac:Country/cbc:IdentificationCode` | 1..1. Also BT-48 prefix and flow choice |
| `recipients[i].origin` | — | — | ignored |
| `recipients[i].shipping.name` | BT-70 Deliver to party name | `/Invoice/cac:Delivery/cac:DeliveryParty/cac:PartyName/cbc:Name` | ignored: cleared for XRechnung (G/xrechnung.go:79-94) because gobl.ubl also writes it as `cac:DeliveryParty/cac:PartyLegalEntity` (ubl:party.go:375-385; UBL-CR-397, warning). Valid BT-70 lost |
| `recipients[i].shipping.date` | BT-72 Actual delivery date | `/Invoice/cac:Delivery/cbc:ActualDeliveryDate` | ignored; BT-72 comes from `document.operation_date` or the issue date |
| `-` | BT-71 Deliver to location identifier | `/Invoice/cac:Delivery/cac:DeliveryLocation/cbc:ID` | gap |
| `recipients[i].shipping.address.line.street`, `recipients[i].shipping.address.line.number` | BT-75 Deliver to address line 1 | `/Invoice/cac:Delivery/cac:DeliveryLocation/cac:Address/cbc:StreetName` | as the buyer address (G/invoice.go:443-467; ubl:delivery.go:53-58). default: no shipping → buyer address copied to BG-15 (G/xrechnung.go:48-53) |
| `recipients[i].shipping.address.line.suffix` | BT-76 Deliver to address line 2 | `/Invoice/cac:Delivery/cac:DeliveryLocation/cac:Address/cbc:AdditionalStreetName` | as BT-75 |
| `recipients[i].shipping.address.city` | BT-77 Deliver to city | `/Invoice/cac:Delivery/cac:DeliveryLocation/cac:Address/cbc:CityName` | BR-DE-10 |
| `recipients[i].shipping.address.code` | BT-78 Deliver to post code | `/Invoice/cac:Delivery/cac:DeliveryLocation/cac:Address/cbc:PostalZone` | BR-DE-11 |
| `recipients[i].shipping.address.region` | BT-79 Deliver to country subdivision | `/Invoice/cac:Delivery/cac:DeliveryLocation/cac:Address/cbc:CountrySubentity` | as BT-75 |
| `recipients[i].shipping.address.country` | BT-80 Deliver to country code | `/Invoice/cac:Delivery/cac:DeliveryLocation/cac:Address/cac:Country/cbc:IdentificationCode` | 1..1 within BG-15 |

### 1.3 Lines

| UAPI | EN 16931 BT | XRechnung UBL XPath | notes |
| --- | --- | --- | --- |
| `entries[i]` | BG-25, BT-126 Invoice line identifier | `/Invoice/cac:InvoiceLine/cbc:ID` | one line per entry, ID = index + 1 (G/entries.go:76-84; ubl:lines.go:76-77) |
| `entries[i].details.number` | BT-126 Invoice line identifier | `/Invoice/cac:InvoiceLine/cbc:ID` | ignored |
| `entries[i].type` | — | — | ignored: `RETURN` not negated (§3) |
| `entries[i].data.type` (GROUP) | BG-25 | `/Invoice/cac:InvoiceLine` | one line from the group-level fields; `entries[i].data.items[]` is ignored entirely |
| `entries[i].data.text` | BT-153 Item name | `/Invoice/cac:InvoiceLine/cac:Item/cbc:Name` | 1..1. Verbatim (G/entries.go:299-306) |
| `entries[i].data.unit.quantity` | BT-129 Invoiced quantity | `/Invoice/cac:InvoiceLine/cbc:InvoicedQuantity` | 1..1. Up to 8 decimals |
| `entries[i].data.unit.measure` | BT-130 Invoiced quantity unit of measure code | `/Invoice/cac:InvoiceLine/cbc:InvoicedQuantity/@unitCode` | GOBL unit key or UN/ECE Rec 20 code verbatim, else dropped (G/entries.go:309-323); the warning log is written in the worker only (E/invopop.go:729-751). default: `C62` (GOBL unit `one`: gobl:addons/eu/en16931/org.go:79-86; gobl:org/unit.go:201) |
| `entries[i].data.unit.price.inclusive` | BT-146 Item net price | `/Invoice/cac:InvoiceLine/cac:Price/cbc:PriceAmount` | derived: gross price, VAT removed by `RemoveIncludedTaxes` (ubl:ubl.go:160-163), full precision (ubl:lines.go:496-499); may be nudged for rounding (§1.6) |
| `entries[i].data.unit.price.exclusive` | BT-146 Item net price | `/Invoice/cac:InvoiceLine/cac:Price/cbc:PriceAmount` | API docs: BT-146. ignored |
| `entries[i].data.unit.factor` | BT-149 Item price base quantity | `/Invoice/cac:InvoiceLine/cac:Price/cbc:BaseQuantity` | API docs: BT-149. ignored |
| `-` | BT-147 / BT-148 Item price discount / gross price | `/Invoice/cac:InvoiceLine/cac:Price/cac:AllowanceCharge[cbc:ChargeIndicator = 'false']/cbc:Amount`, `…/cbc:BaseAmount` | gap |
| `entries[i].data.value.discount.inclusive` | BG-27 BT-136 Invoice line allowance amount | `/Invoice/cac:InvoiceLine/cac:AllowanceCharge[cbc:ChargeIndicator = 'false']/cbc:Amount` | if ≠ 0, 2 dp, converted to net (G/entries.go:45-55; ubl:lines.go:467-489). BT-139 fixed "Special Discount" (META-2347) |
| `entries[i].data.value.surcharge.inclusive` | BG-28 BT-141 Invoice line charge amount | `/Invoice/cac:InvoiceLine/cac:AllowanceCharge[cbc:ChargeIndicator = 'true']/cbc:Amount` | as BT-136 (G/entries.go:57-67). BT-144 fixed "Special Surcharge" |
| `entries[i].data.value.discount.exclusive`, `entries[i].data.value.surcharge.exclusive`, `entries[i].data.value.base.exclusive`, `entries[i].data.value.base.inclusive` | BT-136 / BT-141 / BT-131 | `/Invoice/cac:InvoiceLine/cbc:LineExtensionAmount` | ignored; BT-131 is calculated (§1.6) |
| `entries[i].data.vat.percentage` (VAT_RATE) | BG-30 BT-151 / BT-152 Invoiced item VAT category code / rate | `/Invoice/cac:InvoiceLine/cac:Item/cac:ClassifiedTaxCategory/cbc:ID`, `…/cbc:Percent` | `S` for any rate > 0, `Z` for 0 % (G/entries.go:101-122; gobl:addons/eu/en16931/tax_combo.go:34-58; ubl:lines.go:338-361) |
| `entries[i].data.vat.code` (VAT_RATE) | — | — | ignored (`STANDARD` / `REDUCED_n` unused) |
| `entries[i].data.vat.amount`, `entries[i].data.vat.exclusive`, `entries[i].data.vat.inclusive` | BT-117 / BT-116 / BT-112 (API docs) | — | ignored at line level; GOBL recalculates (§1.5) |
| `entries[i].data.vat.fiscal_location`, `entries[i].data.vat.regime`, `entries[i].data.vat.regime_base`, `entries[i].data.vat.surcharge.rate`, `entries[i].data.vat.surcharge.amount` | — | — | ignored ("Not applicable" for DE) |
| `entries[i].data.vat.code` (VAT_EXEMPTION) | BT-151 (→ BT-118), BT-121 VAT exemption reason code | `/Invoice/cac:InvoiceLine/cac:Item/cac:ClassifiedTaxCategory/cbc:ID`; `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cac:TaxCategory/cbc:TaxExemptionReasonCode` | G/german_exemption.go:24-60: `CAUSE_1` → `E` (§ 19 UStG, no VATEX); `CAUSE_2` → `K` + `VATEX-EU-IC`; `CAUSE_3` → `G` + `VATEX-EU-G`; `CAUSE_4`…`30` → `E` + code-specific VATEX (none for 16); `CAUSE_31` → `E`, reason required; `NOT_SUBJECT` → `O` + `VATEX-EU-O` (fails, §6); `NOT_TAXABLE` → `E`; `CAUSE_n` > 31 → error. Mixed VATEX per key are dropped (G/german_exemption.go:99-120). BT-152 = 0 for E/K/G, absent for O |
| `entries[i].data.vat.reason` (VAT_EXEMPTION) | BT-120 VAT exemption reason text | `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cac:TaxCategory/cbc:TaxExemptionReason` | one GOBL tax note per VAT key, distinct reasons joined with `"; "`, German default text per code (G/entries.go:203-269; ubl:totals.go:122-126) |
| `entries[i].data.vat.type` (VAT_REVERSE_CHARGE) | BT-151 = `AE` | `/Invoice/cac:InvoiceLine/cac:Item/cac:ClassifiedTaxCategory/cbc:ID` | rejected: "unsupported VAT type … for line mapping" (G/entries.go:164-169) |
| `entries[i].data.product.details.label` | BT-154 Item description | `/Invoice/cac:InvoiceLine/cac:Item/cbc:Description` | API docs: BT-154. ignored: no `product` field is read |
| `entries[i].data.product.number` (SKU) | BT-155 Item Sellers identifier | `/Invoice/cac:InvoiceLine/cac:Item/cac:SellersItemIdentification/cbc:ID` | ignored. API docs put BT-155 / BT-157 on type OTHER |
| `entries[i].data.product.number` (GTIN-8, GTIN-12, GTIN-13, GTIN-14) | BT-157 Item standard identifier | `/Invoice/cac:InvoiceLine/cac:Item/cac:StandardItemIdentification/cbc:ID` | ignored. Needs `@schemeID` `0160` (BR-64) |
| `entries[i].data.product.code` | BT-158 Item classification identifier | `/Invoice/cac:InvoiceLine/cac:Item/cac:CommodityClassification/cbc:ItemClassificationCode` | API docs: BT-158. ignored. Needs `@listID` (BR-65) |
| `entries[i].data.product.details.*` (name, brand, model, serial, batch, dates) | BG-32 BT-160 / BT-161 Item attribute name / value | `/Invoice/cac:InvoiceLine/cac:Item/cac:AdditionalItemProperty/cbc:Name`, `…/cbc:Value` | ignored |
| `entries[i].details.description` | BT-127 Invoice line note | `/Invoice/cac:InvoiceLine/cbc:Note` | API docs: BT-127. ignored |
| `entries[i].details.*` (concept, purpose, regulatory, label, activity_code, properties, voucher, applied_vouchers) | — | — | ignored |
| `-` | BT-128, BT-132, BT-133, BG-26, BT-156, BT-159 | `/Invoice/cac:InvoiceLine/…` | gap |

### 1.4 Payment

| UAPI | EN 16931 BT | XRechnung UBL XPath | notes |
| --- | --- | --- | --- |
| `payments[i].details.date` (OUTSTANDING, first) | BT-9 Payment due date | `/Invoice/cbc:DueDate` | required for OUTSTANDING ("payment due date not set"); later OUTSTANDING entries dropped (G/invoice.go:274-289, 607-621; ubl:payment.go:232-235), with a warning log in the worker only (E/invopop.go:709-727). Credit note: §3 |
| `payments[i].details.amount` (OUTSTANDING) | — | — | GOBL due-date amount; no BT. Used for rounding reconciliation |
| `-` (no instruction) | BG-16 BT-81 Payment means type code | `/Invoice/cac:PaymentMeans/cbc:PaymentMeansCode` | 1..1, BR-DE-1. default `1` (instrument not defined) via GOBL key `any` (G/xrechnung.go:39-46; gobl:addons/eu/en16931/pay.go:12-27) |
| `payments[i].instruction.type` (CREDIT_TRANSFER) | BT-81 Payment means type code | `/Invoice/cac:PaymentMeans/cbc:PaymentMeansCode` | fixed `30`, not `58`, so BR-DE-19 never runs; BR-DE-23-a met by BG-17 |
| `payments[i].instruction.account` (CREDIT_TRANSFER) | BG-17 BT-84 Payment account identifier | `/Invoice/cac:PaymentMeans/cac:PayeeFinancialAccount/cbc:ID` | Verbatim (G/invoice.go:530-537; ubl:payment.go:195-203) |
| `payments[i].instruction.name` | BT-85 Payment account name | `/Invoice/cac:PaymentMeans/cac:PayeeFinancialAccount/cbc:Name` | Verbatim |
| `payments[i].instruction.payment_service_provider` | BT-86 Payment service provider identifier | `/Invoice/cac:PaymentMeans/cac:PayeeFinancialAccount/cac:FinancialInstitutionBranch/cbc:ID` | Verbatim; no BIC format check |
| `payments[i].instruction.text` | BT-83 Remittance information | `/Invoice/cac:PaymentMeans/cbc:PaymentID` | API docs: BT-83. As built: GOBL `instructions.detail` (G/invoice.go:538-540, 563-565) → `cbc:PaymentMeansCode/@name` = **BT-82** (ubl:payment.go:160-162) |
| `payments[i].instruction.properties` | BT-83 Remittance information | `/Invoice/cac:PaymentMeans/cbc:PaymentID` | As built: GOBL `instructions.ref` (G/invoice.go:541-543) → BT-83 (ubl:payment.go:157-159). `text` and `properties` are swapped against the docs |
| `payments[i].instruction.type` (DIRECT_DEBIT) | BT-81 Payment means type code | `/Invoice/cac:PaymentMeans/cbc:PaymentMeansCode` | fixed `49`, not `59`, so BR-DE-20 / BR-DE-25 never run; PEPPOL-EN16931-R061 applies |
| `payments[i].instruction.mandate_reference` | BG-19 BT-89 Mandate reference identifier | `/Invoice/cac:PaymentMeans/cac:PaymentMandate/cbc:ID` | Verbatim (G/invoice.go:555-561; ubl:payment.go:166-171) |
| `payments[i].instruction.creditor_id.identifier` | BT-90 Bank assigned creditor identifier | `/Invoice/cac:PayeeParty/cac:PartyIdentification/cbc:ID[@schemeID = 'SEPA']` | As built: no payee, so written to `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PartyIdentification/cbc:ID[@schemeID = 'SEPA']` (ubl:payment.go:121-136); BR-DE-30 accepts both |
| `payments[i].instruction.debited_account` | BT-91 Debited account identifier | `/Invoice/cac:PaymentMeans/cac:PaymentMandate/cac:PayerFinancialAccount/cbc:ID` | Verbatim (ubl:payment.go:172-176); BR-DE-31 |
| `payments[i].instruction.type` (UNKNOWN) | BT-81 Payment means type code | `/Invoice/cac:PaymentMeans/cbc:PaymentMeansCode` | GOBL key `other` (G/invoice.go:571-583) has no UNTDID 4461 code (gobl:addons/eu/en16931/pay.go:12-38; gobl:cbc/code.go:273-283) → GOBL fault "payment means extension is required (BR-49)" at sync. From code, not executed |
| `payments[i].details.amount` (CARD, CASH, ONLINE, SINGLE_PURPOSE_VOUCHER, MULTI_PURPOSE_VOUCHER, OTHER) | BT-113 Paid amount | `/Invoice/cac:LegalMonetaryTotal/cbc:PrepaidAmount` | summed as GOBL advances (G/invoice.go:290-299; ubl:totals.go:77-79). Method not expressed: BT-81 stays `1` without an OUTSTANDING instruction |
| `payments[i].number`, `payments[i].kind` (CARD) | BG-18 BT-87 Payment card primary account number | `/Invoice/cac:PaymentMeans/cac:CardAccount/cbc:PrimaryAccountNumberID` | API docs: BT-87 / BT-81. ignored |
| `payments[i].name` (ONLINE, OTHER) | BT-82 Payment means text | `/Invoice/cac:PaymentMeans/cbc:PaymentMeansCode/@name` | API docs: BT-82. ignored |
| `payments[i].type` (MEAL_VOUCHER, DEPOSIT) | — | — | rejected: "unsupported payment type" (G/invoice.go:300-302) |
| `payments[i].details.discount`, `payments[i].details.channel`, `payments[i].details.properties`, `payments[i].concept`, `payments[i].count`, `payments[i].document_number`; advance `payments[i].details.date` | — | — | ignored; gobl.ubl writes no `cac:PrepaidPayment` |
| `-` | BG-10 PAYEE | `/Invoice/cac:PayeeParty` | gap |

### 1.5 VAT Breakdown and Totals

| UAPI | EN 16931 BT | XRechnung UBL XPath | notes |
| --- | --- | --- | --- |
| `breakdown[i]` | BG-23 VAT BREAKDOWN | `/Invoice/cac:TaxTotal/cac:TaxSubtotal` | 1..n. ignored: GOBL rebuilds BG-23 from the lines |
| `breakdown[i].exclusive` | BT-116 VAT category taxable amount | `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cbc:TaxableAmount` | ignored → calculated (§1.6) |
| `breakdown[i].amount` | BT-117 VAT category tax amount | `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cbc:TaxAmount` | ignored → calculated |
| `breakdown[i].type`, `breakdown[i].code` | BT-118 VAT category code | `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cac:TaxCategory/cbc:ID` | ignored → from line categories |
| `breakdown[i].percentage` | BT-119 VAT category rate | `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cac:TaxCategory/cbc:Percent` | 1..1, BR-DE-14 (fatal). From the lines; `0` for E/K/G, **omitted for O** (ubl:totals.go:128-136) |
| `breakdown[i].inclusive` | — | — | no BT |
| `totals.vat.exclusive` | BT-109 Invoice total amount without VAT | `/Invoice/cac:LegalMonetaryTotal/cbc:TaxExclusiveAmount` | compared only; mismatch = warning log (E/invopop.go:753-782) |
| `totals.vat.amount` | BT-110 Invoice total VAT amount | `/Invoice/cac:TaxTotal/cbc:TaxAmount[@currencyID = /Invoice/cbc:DocumentCurrencyCode]` | compared only |
| `totals.vat.inclusive` | BT-112 Invoice total amount with VAT | `/Invoice/cac:LegalMonetaryTotal/cbc:TaxInclusiveAmount` | compared only |
| `-` | BT-106 Sum of Invoice line net amount | `/Invoice/cac:LegalMonetaryTotal/cbc:LineExtensionAmount` | calculated |
| `-` | BT-107 / BT-108 | `/Invoice/cac:LegalMonetaryTotal/cbc:AllowanceTotalAmount`, `…/cbc:ChargeTotalAmount` | not written: no BG-20 / BG-21 source |
| `-` | BT-111 Invoice total VAT amount in accounting currency | `/Invoice/cac:TaxTotal/cbc:TaxAmount[@currencyID = /Invoice/cbc:TaxCurrencyCode]` | gap |
| `-` | BT-114 Rounding amount | `/Invoice/cac:LegalMonetaryTotal/cbc:PayableRoundingAmount` | GOBL `totals.rounding`, when present |
| `-` | BT-115 Amount due for payment | `/Invoice/cac:LegalMonetaryTotal/cbc:PayableAmount` | 1..1, calculated |

### 1.6 Calculated Fields

| UAPI | EN 16931 BT | XRechnung UBL XPath | formula | notes |
| --- | --- | --- | --- | --- |
| `entries[i].data.unit.price.inclusive`, `entries[i].data.vat.percentage` | BT-146 | `/Invoice/cac:InvoiceLine/cac:Price/cbc:PriceAmount` | `price.inclusive / (1 + rate)` | GOBL `prices_include = VAT`; more than 2 decimals kept |
| `entries[i].data.unit.quantity`, `entries[i].data.value.discount.inclusive`, `entries[i].data.value.surcharge.inclusive` | BT-131 | `/Invoice/cac:InvoiceLine/cbc:LineExtensionAmount` | `BT-129 × BT-146 − Σ BT-136 + Σ BT-141`, rounded to currency | `tax.rounding = currency` (G/invoice.go:165-168). PEPPOL-EN16931-R120 (warning) can fire on residue (inferred) |
| `-` | BT-106 | `/Invoice/cac:LegalMonetaryTotal/cbc:LineExtensionAmount` | `Σ BT-131` | BR-CO-10 |
| `-` | BT-109 | `/Invoice/cac:LegalMonetaryTotal/cbc:TaxExclusiveAmount` | `BT-106` | BR-CO-13 (no document-level allowances / charges) |
| `-` | BT-116 | `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cbc:TaxableAmount` | `Σ BT-131` per (BT-151, BT-152) | — |
| `-` | BT-117 | `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cbc:TaxAmount` | `round2(BT-116 × BT-119 / 100)` | BR-CO-17 |
| `-` | BT-110 | `/Invoice/cac:TaxTotal/cbc:TaxAmount[@currencyID = /Invoice/cbc:DocumentCurrencyCode]` | `Σ BT-117` | BR-CO-14 |
| `-` | BT-112 | `/Invoice/cac:LegalMonetaryTotal/cbc:TaxInclusiveAmount` | `BT-109 + BT-110` | BR-CO-15 |
| `payments[i].details.amount` (non-OUTSTANDING) | BT-113 | `/Invoice/cac:LegalMonetaryTotal/cbc:PrepaidAmount` | `Σ advances` | 2 dp each |
| `-` | BT-115 | `/Invoice/cac:LegalMonetaryTotal/cbc:PayableAmount` | `BT-112 − BT-113 + BT-114` | BR-CO-16. If > 0, GOBL requires BT-9 or BT-20 (BR-CO-25; not checked by the KoSIT validator, §6.2) |
| `payments[i].details.amount` | BT-146 (adjusted) | `/Invoice/cac:InvoiceLine/cac:Price/cbc:PriceAmount` | prices rewritten (≤ 8 dp) so BT-112 = `Σ payments[i].details.amount` | drift = target − GOBL `TotalWithTax` (G/line_rounding.go:58); only when it is explained by rounding (G/line_rounding.go:21-159) |

## 2. Supplier Data (Taxpayer, System)

Paths are relative to the taxpayer root (`POST/PATCH /taxpayers`) or the system annotations; "(invoice)" marks paths under `content.operation`. Built by G/taxpayer.go:25-81 and G/invoice.go:469-497.

| UAPI | EN 16931 BT | XRechnung UBL XPath | notes |
| --- | --- | --- | --- |
| `fiscalization.type` | — | — | `DE` selects the regime, flow and exemption table, and the BT-31 / BT-40 country |
| `name.legal` | BT-27 Seller name | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PartyLegalEntity/cbc:RegistrationName` | 1..1. API docs: BT-44 (buyer name); spec: BT-27. Verbatim (G/taxpayer.go:62) |
| `name.legal` | BT-28 Seller trading name | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PartyName/cbc:Name` | derived: copied by gobl.ubl (ubl:party.go:208-211) |
| `name.trade` | BT-28 Seller trading name | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PartyName/cbc:Name` | ignored (required for COMPANY, never read) |
| `name.person.forename`, `name.person.surname` | BT-27 Seller name | — | ignored; `name.legal` is used for INDIVIDUAL too |
| `fiscalization.vat_id_number` | BT-31 Seller VAT identifier | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PartyTaxScheme[cac:TaxScheme/cbc:ID = 'VAT']/cbc:CompanyID` | `DE` + 9 digits. **VAT ID wins** over `tax_id_number` (G/taxpayer.go:34-45; ubl:party.go:229-249). Meets BR-DE-16, BR-CO-26, BR-S-02 |
| `fiscalization.tax_id_number` | BT-32 Seller tax registration identifier | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PartyTaxScheme[cac:TaxScheme/cbc:ID != 'VAT']/cbc:CompanyID` | ignored when a VAT ID exists, so BT-32 is never written. Without VAT ID the 11-digit value becomes the BT-31 code and fails GOBL's `^[1-9]\d{8}$` (gobl:regimes/de/tax_identity.go:17-19). It is documented as the Steuer-ID (BZSt), not the Finanzamt Steuernummer that § 14 Abs. 4 Satz 1 Nr. 2 UStG accepts instead of the USt-IdNr |
| `address.line.street`, `address.line.number` | BT-35 Seller address line 1 | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PostalAddress/cbc:StreetName` | `street + " " + number` |
| `address.line.suffix` | BT-36 Seller address line 2 | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PostalAddress/cbc:AdditionalStreetName` | Verbatim |
| `address.line.prefix`, `address.line.infix` | BT-162 Seller address line 3 | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PostalAddress/cac:AddressLine/cbc:Line` | ignored |
| `address.city` | BT-37 Seller city | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PostalAddress/cbc:CityName` | 1..1, BR-DE-3 |
| `address.code` | BT-38 Seller post code | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PostalAddress/cbc:PostalZone` | 1..1, BR-DE-4 |
| `address.region` | BT-39 Seller country subdivision | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PostalAddress/cbc:CountrySubentity` | Verbatim |
| `address.country` | BT-40 Seller country code | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PostalAddress/cac:Country/cbc:IdentificationCode` | 1..1. Overwritten by the tax-ID country (`DE`) (ubl:party.go:250-256) |
| `-` | BT-29 / BT-30 / BT-33 | `…/cac:PartyIdentification/cbc:ID`; `…/cac:PartyLegalEntity/cbc:CompanyID`; `…/cac:PartyLegalEntity/cbc:CompanyLegalForm` | gap: registration fields exist for IT only. BT-29 carries the SEPA creditor ID with direct debit (§1.4) |
| `peppol_id` (system annotation) | BT-34 Seller electronic address | `/Invoice/cac:AccountingSupplierParty/cac:Party/cbc:EndpointID` | 1..1, PEPPOL-EN16931-R020. API docs: BT-49; spec: BT-34. Set on any channel when present (W:284-298; G/inbox.go:17-35) |
| `seller.email` (invoice) | BT-34 Seller electronic address | `/Invoice/cac:AccountingSupplierParty/cac:Party/cbc:EndpointID` | default without `peppol_id`: EAS `EM` (G/xrechnung.go:61-65). The sync check never has the Peppol ID (E/invopop.go:330-336) |
| `seller.name` (invoice) | BG-6 BT-41 Seller contact point | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:Contact/cbc:Name` | 1..1, BR-DE-5. default `name.legal` (G/xrechnung.go:55-60) |
| `seller.phone` (invoice) | BT-42 Seller contact telephone number | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:Contact/cbc:Telephone` | **1..1, BR-DE-6 (fatal)**, optional in the API. BG-6 itself is mandatory by BR-DE-2; BR-DE-5/6/7 apply inside it. Absent → GOBL `DE-XRECHNUNG-09` (gobl:addons/de/xrechnung/bill_invoices.go:58-60). BR-DE-27 (warning) |
| `seller.email` (invoice) | BT-43 Seller contact email address | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:Contact/cbc:ElectronicMail` | **1..1, BR-DE-7 (fatal)**, optional in the API. Absent → `DE-XRECHNUNG-10` (bill_invoices.go:61-63). BR-DE-28 (warning) |
| `tax_representative.name` (invoice) | BG-11 BT-62 Seller tax representative name | `/Invoice/cac:TaxRepresentativeParty/cac:PartyName/cbc:Name` | GOBL `ordering.seller` (G/invoice.go:208-213, 499-519; ubl:ordering.go:87-89); also written as `cac:PartyLegalEntity` → UBL-CR-335 (warning) |
| `tax_representative.vat_number` (invoice) | BT-63 Seller tax representative VAT identifier | `/Invoice/cac:TaxRepresentativeParty/cac:PartyTaxScheme/cbc:CompanyID` | prefixed with `tax_representative.country` |
| `tax_representative.country` (invoice) | BG-12 BT-69 Tax representative country code | `/Invoice/cac:TaxRepresentativeParty/cac:PostalAddress/cac:Country/cbc:IdentificationCode` | derived: kept in GOBL as the tax-ID country, which gobl.ubl writes into `cac:PostalAddress/cac:Country` (ubl:party.go:250-256). BG-12 holds BT-69 only; BR-19 / BR-20 pass |
| `-` | BT-64…BT-68, BT-164 | `/Invoice/cac:TaxRepresentativeParty/cac:PostalAddress/…` | gap |

## 3. Correction (Credit Note)

XPaths from the `ubl-cn` binding. A `CORRECTION` with `data.type` = `INVOICE` carries a full invoice under `data.`; every §1 row applies with that prefix and the `/CreditNote` root (`cac:CreditNoteLine` instead of `cac:InvoiceLine`).

| UAPI | EN 16931 BT | XRechnung UBL XPath | notes |
| --- | --- | --- | --- |
| `data.type` | — | `/CreditNote` | GOBL `credit-note` (W:196-197) → root `CreditNote` (ubl:invoice.go:150-156; ubl:lines.go:84-88) |
| `-` | BT-3 Invoice type code | `/CreditNote/cbc:CreditNoteTypeCode` | fixed `381` (gobl:addons/eu/en16931/scenarios.go:30-36). No `384` or `383` path; upward corrections are `381` too |
| `record.id` | BG-3 BT-25 Preceding Invoice reference | `/CreditNote/cac:BillingReference/cac:InvoiceDocumentReference/cbc:ID` | derived: referenced record → `used_in` → Invopop entry → original GOBL `code` (W:208-218; E/invopop.go:1101-1129). Missing `used_in` → worker error; at sync GOBL `DE-XRECHNUNG-02` fails. On Peppol this may be the `PPL` value: UNVERIFIED |
| `record.id` | BT-26 Preceding Invoice issue date | `/CreditNote/cac:BillingReference/cac:InvoiceDocumentReference/cbc:IssueDate` | derived: original issue date from Invopop |
| `data.document.references.preceding_document.number`, `data.document.references.preceding_document.issued_at` | BT-25 / BT-26 | — | ignored with a warning log (G/invoice.go:191-205; E/invopop.go:706) |
| `record.offset` | — | — | ignored |
| `reason.text` | BT-22 Invoice note (BT-21 `ACD`) | `/CreditNote/cbc:Note` | GOBL note key `reason` → `#ACD#<text>` (G/invoice.go:215-222; gobl:addons/eu/en16931/org.go:28; ubl:utils.go:77-86) |
| `reason.type` | BT-22 Invoice note (BT-21 `ACD`) | `/CreditNote/cbc:Note` | default when no text: the raw enum, e.g. `#ACD#FACTUAL_ERROR` (`src/unified/service/einvoicing/correction.go:8-14`) |
| `data.entries[i].data.unit.quantity` | BT-129 Invoiced quantity | `/CreditNote/cac:CreditNoteLine/cbc:CreditedQuantity` | sign as supplied |
| `data.entries[i].data.unit.measure` | BT-130 | `/CreditNote/cac:CreditNoteLine/cbc:CreditedQuantity/@unitCode` | as §1.3 |
| `data.entries[i].type` | — | — | ignored; nothing is negated. API docs require positive amounts on RETURN lines; a negative unit price fails BR-27 (fatal) |
| `data.entries[i].details.number`, `data.entries[i].details.prepayment.document_number` | — | — | ignored: the corrected line / prepayment document is not referenced |
| `data.payments[i].details.date` (OUTSTANDING) | BT-9 Payment due date | `/CreditNote/cac:PaymentMeans/cbc:PaymentDueDate` | ubl:payment.go:188-191 (`cbc:DueDate` does not exist in CreditNote) |
| `data.document.references.project` | BT-11 Project reference | `/CreditNote/cac:AdditionalDocumentReference/cbc:ID` | ignored (DE). If re-enabled: `cac:AdditionalDocumentReference` with `cbc:DocumentTypeCode` `50`, a code UBL-SR-43 allows only on a CreditNote. The UBL 2.1 CreditNote XSD has no `cac:ProjectReference` |
| `data.recipients[i]` | BG-7 | `/CreditNote/cac:AccountingCustomerParty` | must equal the original's recipients (EA:345, 374) |

**Sign convention.** EN 16931 expresses the reduction through BT-3 `381` with positive amounts. fiskaly passes signs through unchanged (the only negation in G/ is the rounding direction, G/line_rounding.go:110-113), so the API rule "positive amounts on RETURN lines" keeps the credit note valid. BR-DE-26 (warning) needs BG-3 only for `384`; the GOBL addon requires a preceding reference for every credit note (gobl:addons/de/xrechnung/bill_invoices.go:35-40).

## 4. Fixed and Derived Values

| UAPI | EN 16931 BT | XRechnung UBL XPath | notes |
| --- | --- | --- | --- |
| `-` | BT-24 | `/Invoice/cbc:CustomizationID` | fixed `urn:cen.eu:en16931:2017#compliant#urn:xeinkauf.de:kosit:xrechnung_3.0` |
| `-` | BT-23 | `/Invoice/cbc:ProfileID` | fixed `urn:fdc:peppol.eu:2017:poacc:billing:01:1.0` |
| `-` | BT-3 | `/Invoice/cbc:InvoiceTypeCode` | fixed `380`; `381` on `CreditNote` |
| `-` | BT-81 | `/Invoice/cac:PaymentMeans/cbc:PaymentMeansCode` | default `1`; fixed `30` / `49` |
| `-` | BT-41 | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:Contact/cbc:Name` | default taxpayer `name.legal` |
| `-` | BT-34 / BT-49 | `…/cac:Party/cbc:EndpointID` | Peppol ID, else an email with `@schemeID` `EM` |
| `-` | BT-72 | `/Invoice/cac:Delivery/cbc:ActualDeliveryDate` | always written: `document.operation_date`, else issue date |
| `-` | BG-15 | `/Invoice/cac:Delivery/cac:DeliveryLocation/cac:Address` | always written: shipping address, else buyer address (GOBL addon rule 25 requires a receiver) |
| `-` | BT-130 | `/Invoice/cac:InvoiceLine/cbc:InvoicedQuantity/@unitCode` | default `C62` |
| `-` | BT-151 / BT-118 / BT-121 / BT-120 | `…/cac:ClassifiedTaxCategory/cbc:ID`, `…/cac:TaxSubtotal/cac:TaxCategory/…` | derived from the DE exemption table (§1.3) |
| `-` | BT-119 / BT-152 | `…/cac:TaxCategory/cbc:Percent` | `0` for E/K/G; omitted for O |
| `-` | BT-139 / BT-144 | `/Invoice/cac:InvoiceLine/cac:AllowanceCharge[cbc:ChargeIndicator = 'false']/cbc:AllowanceChargeReason`, `/Invoice/cac:InvoiceLine/cac:AllowanceCharge[cbc:ChargeIndicator = 'true']/cbc:AllowanceChargeReason` | fixed "Special Discount" / "Special Surcharge" |
| `-` | BT-21 | `/Invoice/cbc:Note` | fixed `ACD` for the correction reason |
| `-` | BT-28 / BT-45 | `…/cac:PartyName/cbc:Name` | copies of BT-27 / BT-44 |
| `-` | `cac:TaxScheme/cbc:ID` of BT-31, BT-48, BT-63, BG-23, BG-30 | `…/cac:TaxScheme/cbc:ID` | fixed `VAT` |
| `-` | BT-31 / BT-48 / BT-63 prefix | `…/cac:PartyTaxScheme/cbc:CompanyID` | from `fiscalization.type` / recipient `address.country` / `tax_representative.country` |
| `-` | BT-2 | `/Invoice/cbc:IssueDate` | default today |
| `-` | BT-1 | `/Invoice/cbc:ID` | Peppol: possible `PPL` sequence value (UNVERIFIED) |

## 5. Format Elements Without a UAPI Source (Gaps)

Mandatory first (1..x or made mandatory by a rule), then mandatory-in-context, then important optional BTs.

| BT | element | cardinality / rule | status today | proposal |
| --- | --- | --- | --- | --- |
| BT-10 | `/Invoice/cbc:BuyerReference` | 1..1, BR-DE-15 (fatal) | optional in the API, no default; missing → GOBL failure at sync, no 4xx. Max 32 < Leitweg-ID 46 | Require `document.references.buyer` (4xx) for XRechnung; raise max length to ≥ 46 |
| BT-42 | `…/cac:Contact/cbc:Telephone` | 1..1, BR-DE-6 (fatal) | `seller.phone` optional; missing → sync failure | Require for XRechnung, or hold seller contact data on the taxpayer |
| BT-43 / BT-34 | `…/cac:Contact/cbc:ElectronicMail`; `…/cbc:EndpointID` | 1..1, BR-DE-7 (fatal); PEPPOL-EN16931-R020 | `seller.email` optional; it is also the BT-34 fallback | as BT-42 |
| BT-31 / BT-32 / BG-11 | `…/cac:PartyTaxScheme/cbc:CompanyID` | BR-DE-16 (fatal) for S, Z, E, AE, K, G, L, M; § 14 Abs. 4 Satz 1 Nr. 2 UStG | works only with `vat_id_number`; a DE taxpayer without USt-IdNr (typically § 19 Kleinunternehmer) fails, and no Steuernummer field exists | Add a DE Steuernummer field → GOBL identity `de-tax-number` (gobl:regimes/de/org_identities.go:20) with tax scope and identity type `FC` → BT-32 with `cac:TaxScheme/cbc:ID` `FC` (as in the KoSIT test suite). Without the type, gobl.ubl writes `LOC` (ubl:party.go:308-320, 354-361) |
| BT-119 | `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cac:TaxCategory/cbc:Percent` | 1..1, BR-DE-14 (fatal) | omitted for O | Converter writes `0` for O (Invopop change) |
| BT-151 = O | `…/cac:ClassifiedTaxCategory/cbc:ID` | BR-O-02 (fatal) | `NOT_SUBJECT` → O, but BT-31 and BT-48 are always present | Reject `NOT_SUBJECT` for XRechnung at sync until a mapping is agreed (cross-border B2B services belong to AE) |
| BT-151 = AE | `…/cac:ClassifiedTaxCategory/cbc:ID` + BT-120 / BT-121 | BR-AE-01…10 | `VAT_REVERSE_CHARGE` rejected | Map to GOBL `reverse-charge` (AE), `VATEX-EU-AE`, BT-120 "Steuerschuldnerschaft des Leistungsempfängers" (§ 14a Abs. 5 UStG) |
| BG-16 / BT-81 | `/Invoice/cac:PaymentMeans` | 1..1, BR-DE-1 | default `1`; advances' method lost | Optional: `10` / `48` (+ BG-18 from `payments[i].number`) for fully paid single-method invoices |
| BT-81 = 58 / 59 | `…/cbc:PaymentMeansCode` | BR-DE-19 / BR-DE-20 (warning) | SEPA payments sent as `30` / `49` | Use GOBL `credit-transfer+sepa` / `direct-debit+sepa` |
| BT-47 | `…/cac:PartyLegalEntity/cbc:CompanyID` | 0..1; UBL-SR-16 (fatal) | `company_id` written as a second BT-46 | Set identity scope `legal` for `company_id` |
| BT-82 / BT-83 | `…/cbc:PaymentMeansCode/@name`; `…/cbc:PaymentID` | 0..1 | `instruction.text` → BT-82, `instruction.properties` → BT-83 | Map `instruction.text` to GOBL `ref` (BT-83) as documented |
| BT-70 | `/Invoice/cac:Delivery/cac:DeliveryParty/cac:PartyName/cbc:Name` | 0..1 | stripped | Converter writes only `cac:PartyName`; then stop stripping |
| BT-11 | `/Invoice/cac:ProjectReference/cbc:ID` | 0..1 | dropped for all DE invoices | Re-enable for Invoice; CreditNote via `AdditionalDocumentReference` type `50` |
| BT-22 | `/Invoice/cbc:Note` | 0..n (BG-1) | `document.text` ignored | Map to a GOBL note |
| BT-72 / BG-14 | `/Invoice/cac:Delivery/cbc:ActualDeliveryDate`; `/Invoice/cac:InvoicePeriod` | 0..1; § 14 Abs. 4 Satz 1 Nr. 6 UStG needs the date of supply | issue date used when `operation_date` is absent; `shipping.date` ignored; no period fields | Use `shipping.date`; no silent issue-date fallback; add a service period |
| BT-127 | `/Invoice/cac:InvoiceLine/cbc:Note` | 0..1 | `details.description` ignored | Map to line notes |
| BT-154 / 155 / 157 / 158, BG-32 | `…/cac:Item/…` | 0..1 / 0..n | `product.*` ignored | label → BT-154; SKU → BT-155; GTIN → BT-157 `@schemeID` `0160`; code → BT-158 + `@listID`; brand/model/serial/batch → BG-32 |
| BT-149 / BT-150 | `…/cac:Price/cbc:BaseQuantity` | 0..1 | `unit.factor` ignored | Map to GOBL `item.per` |
| BT-28 | `…/cac:PartyName/cbc:Name` | 0..1 | holds `name.legal`; `name.trade` ignored | Map `name.trade` to GOBL `alias` |
| BT-29 / BT-30 / BT-33 | seller identifier, register number, legal form | 0..n / 0..1 / 0..1 | no DE fields | Add DE register fields to the taxpayer |
| BT-6 / BT-111 | `/Invoice/cbc:TaxCurrencyCode` | 0..1, BR-53 | no exchange rate | Add one when currency ≠ EUR |
| BT-3 = 384 | `/Invoice/cbc:InvoiceTypeCode` | BR-DE-17, BR-DE-26 | every correction is `381` | Decide whether replacing corrections use `384` |
| BG-24, BT-14, BT-15, BT-18, BT-71, BT-128, BT-132, BT-133, BG-26, BG-9, BG-10 | various | 0..x | no UAPI source | Add on demand |

## 6. Common Validation Failures

### 6.1 fiskaly-side errors for this flow

| Stage | Message (abridged) | Trigger | Source |
| --- | --- | --- | --- |
| A: API (4xx) | `<TYPE> E_INVOICE_SERVICE is not supported for <cc>`; `destination code '' must match the format '^[A-Z0-9]{7}$'` | recipient outside DE/BE; CONSUMER recipient with an EMAIL or PEPPOL block | RH:3331-3336, 3565-3568; `pkg/identifier/sdi/destination_code.go:49-54` |
| B: sync (record `COMPLETED` with error log) | `TRANSACTION::INVOICE needs exactly 1 business recipient with invoicing…`; `business recipient identification is not a valid VAT identification`; `recipients[0].identification.number: …` | recipient shape / VAT format | EA:60-93 |
| B | `supplier is missing a Peppol identifier` / `supplier is not registered for PEPPOL` | PEPPOL without registration | EA:145-186 |
| B | `invoice correction recipients do not match the original invoice` | correction | EA:345, 374 |
| C: GOBL (sync and worker) | `DE-XRECHNUNG-29` / `-30` (BR-DE-15), `-09` (BR-DE-6), `-10` (BR-DE-7), `-08` (PEPPOL-EN16931-R020), `-02` (preceding on credit note) | missing BT-10, BT-42, BT-43, BT-34, BG-3 | gobl:addons/de/xrechnung/bill_invoices.go:35-140 |
| C | `invalid German VAT identity code`; `taxpayer fiscalization does not have tax code` | DE taxpayer with only / no tax ID | gobl:regimes/de/tax_identity.go:17-31; G/taxpayer.go:43-44 |
| C | `unsupported VAT type … 'REVERSE_CHARGE'`; `unsupported VAT exemption code '<c>' for Germany`; `missing VAT exemption reason for code '<c>'` | reverse charge; `CAUSE_n` > 31; `CAUSE_31` without reason | G/entries.go:164-169, 239-244; G/german_exemption.go:62-74 |
| C | `unsupported payment type: MEAL_VOUCHER\|DEPOSIT`; `payment due date not set` | those payment types; OUTSTANDING without date | G/invoice.go:300-302, 610-613 |
| C | EN 16931 rule 06 / 07 (BR-CO-25), logged as the raw GOBL message. The friendlier text in E/error_translation.go:15-24 is used only for taxpayer registration errors (E/service.go:509, 886-905) | amount due > 0 without BT-9 and BT-20 | gobl:addons/eu/en16931/bill.go:143-151; G/error_translation.go:26-47 (called from E/invopop.go:677) |
| C | EN 16931 pay instructions 01 (BR-49) | `instruction.type` UNKNOWN (from code) | gobl:addons/eu/en16931/pay.go:40-48 |
| D: worker | `supplier has no Peppol identifier yet…`; `referenced record is missing used_in identifier` | PEPPOL before proof of ownership; correction of an untransmitted invoice | E/service.go:166; W:213-216 |
| E: Invopop (webhook, after `COMPLETED`) | Schematron / XSD faults, §6.2 | — | `validate_ubl: true` |

### 6.2 Rules most likely to trip

| Rule | Severity | UAPI trigger | Caught at |
| --- | --- | --- | --- |
| BR-DE-15 | fatal | `document.references.buyer` missing | sync (GOBL) |
| BR-DE-6 / BR-DE-7 | fatal | `seller.phone` / `seller.email` missing | sync (GOBL) |
| PEPPOL-EN16931-R020 | fatal | no `peppol_id` and no `seller.email` | sync (GOBL) |
| BR-O-02 | fatal | `vat.code` `NOT_SUBJECT` (O with BT-31 and BT-48) | Invopop only; from code + EN-sch, not executed |
| BR-DE-14 | fatal | `NOT_SUBJECT`: BT-119 omitted for O | Invopop only |
| UBL-SR-16 | fatal | `buyer_id` and `company_id` both set | Invopop only |
| BR-27 | fatal | negative unit price (e.g. RETURN with negative values) | Invopop only |
| BR-DE-18 | fatal | `document.payment_terms` lines starting with `#` outside the Skonto syntax | Invopop only |
| BR-CO-25 | — (GOBL only; removed from the CEN UBL Schematron in v1.3.16, which the validator configuration 2026-08-31 bundles) | amount due > 0 without OUTSTANDING date or `payment_terms` | sync (GOBL) |
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
| EN 16931 validation artefacts (UBL) | v1.3.17 (2026-10-09), commit `6e35e0a` | formats repo `eInvoicing-EN16931/ubl/schematron/`; https://github.com/ConnectingEurope/eInvoicing-EN16931/releases/tag/validation-1.3.17. The XRechnung validator configuration 2026-08-31 bundles v1.3.16 (`scenarios.xml`) | 2026-11-03 |
| Leitweg-ID Formatspezifikation (KoSIT) | v2.0.2, Fassung 2021-07-28 | https://xeinkauf.de/app/uploads/2022/11/Leitweg-ID-Formatspezifikation-v2-0-2-1.pdf (§2.1: "maximal 46 Stellen"; §3.4). xeinkauf.de still lists 2.0.2 as current | retrieved 2026-11-03 |
| Umsatzsteuergesetz, § 14 Abs. 4, § 14a Abs. 5 | consolidated, Stand Art. 5 G v. 29.6.2026 I Nr. 197 | formats repo `Germany/Laws/01_UStG_konsolidiert.md` (gesetze-im-internet.de) | 2026-11-03 |
| UAPI v6 definition / API docs | `fiskaly/meta@cd19ec88cb299ee41a44302448006340dd028bf0`; docs version 2026-06-01 | [`UAPI_v6_Fields.md`](UAPI_v6_Fields.md); [`fiskaly.unified-api.BE_EI-DE_EI-IT_EI.2026-06-01 (1).yaml`](fiskaly.unified-api.BE_EI-DE_EI-IT_EI.2026-06-01%20(1).yaml) | 2026-11-03 |
| fiskaly code and Invopop workflows | `fiskaly/meta@cd19ec8` (paths in the citation legend) | GitHub `fiskaly/meta` | 2026-11-03 |
| GOBL (addons `de-xrechnung-v3`, `eu-en16931-v2017`, regime DE) | v0.501.0, commit `225e39d` (2026-06-16) | https://github.com/invopop/gobl/tree/v0.501.0 | 2026-11-03 |
| gobl.ubl | v0.99.0, commit `540f201` (2026-10-28); latest tag, production version not visible. Its `go.mod` requires gobl v0.511.0, and it calls `RoundToCurrency` (ubl:ubl.go:164), which gobl v0.501.0 lacks, so it does not build against fiskaly's pin | https://github.com/invopop/gobl.ubl/tree/v0.99.0 | 2026-11-03 |
| gobl.de.xinvoice (`ubl+de-xrechnung-v3`) | v0.0.9, commit `8116d89` (2026-10-15); its `go.mod` requires gobl v0.510.0 and gobl.ubl v0.93.0, so Invopop's GOBL may be newer than fiskaly's. Its `addon/xrechnung` rules are identical to gobl v0.501.0's | https://github.com/invopop/gobl.de.xinvoice/tree/v0.0.9 | 2026-11-03 |
| As-built trace UAPI → GOBL (secondary, spot-checked) | 2026-11-03 | internal working note | 2026-11-03 |

## Verification Log

| # | Claim | How checked | Result |
| --- | --- | --- | --- |
| 1 | BT-24 and BT-23 as emitted match XRechnung 3.0.2 | xinv:xinvoice.go:25, 36; xinv:ubl_format.go:15-36; ubl:invoice.go:106-112, 135-139 vs XR-sch `common.sch` `XR-CIUS-ID` (version `3.0`), BR-DE-21, PEPPOL-EN16931-R001; model note on BT-23 | CONFIRMED |
| 2 | BT-10 is mandatory and the 32-character API cap is below the Leitweg-ID maximum of 46 | model `m:structure` (BT-10 without `min-occurs="0"`); XR-sch BR-DE-15 `fatal`; UAPI `maxLength` 32; Leitweg-ID spec v2.0.2 §2.1 ("maximal 46 Stellen") | CONFIRMED |
| 3 | Missing `seller.phone` / `seller.email` fails BR-DE-6 / BR-DE-7 | G/invoice.go:486-493; gobl:addons/de/xrechnung/bill_invoices.go:58-63; XR-sch BR-DE-6 / BR-DE-7 `fatal` | CONFIRMED |
| 4 | VAT ID wins, BT-32 is never written, and a DE taxpayer with only `tax_id_number` fails GOBL | G/taxpayer.go:34-45; ubl:party.go:229-257; gobl:regimes/de/tax_identity.go:17-19 | CONFIRMED |
| 5 | `VAT_REVERSE_CHARGE`, `MEAL_VOUCHER`, `DEPOSIT` are rejected | G/entries.go:164-169; G/invoice.go:300-302 | CONFIRMED |
| 6 | BT-11 dropped for DE, BT-70 stripped, BT-7 never written | G/invoice.go:349-353; G/xrechnung.go:79-94; ubl:invoice.go:158-161. The META-3995 cause (no `cac:ProjectReference` in the CreditNote XSD) is an inference | CONFIRMED (cause UNVERIFIED) |
| 7 | `NOT_SUBJECT` yields category O with BT-31 and BT-48 and without BT-119, failing BR-O-02 and BR-DE-14 | G/german_exemption.go:56; ubl:party.go:229-249; ubl:totals.go:128-136; EN-sch BR-O-02 and XR-sch BR-DE-14 `fatal`. Static reading of gobl.ubl v0.99.0, not executed | CONFIRMED |
| 8 | `buyer_id` + `company_id` give two buyer identifiers (UBL-SR-16) | G/recipient.go:158-172; ubl:party.go:322-349; EN-sch `UBL-SR-16` `fatal` | CONFIRMED |
| 9 | `instruction.text` lands in BT-82, `instruction.properties` in BT-83 | G/invoice.go:538-543; ubl:payment.go:157-162 | CONFIRMED |
| 10 | The Peppol `sequence.enumerate` step replaces BT-1 (and so the BT-25 of later credit notes) | WF/DE_SEND_INVOICE_PEPPOL_XRECHNUNG_BUSINESS.json:15-26 shows the step; its behaviour on an existing `code` is Invopop-side | UNVERIFIED |

### Independent verification pass (2026-11-03)

Two separate checkers re-tested the decision-critical claims against code (fiskaly `cd19ec8`, gobl v0.501.0, the vendored converters) and against primary sources. Neither was given the page.

| Claim | Verdict | Evidence / correction |
| --- | --- | --- |
| `VAT_REVERSE_CHARGE`, `MEAL_VOUCHER`, `DEPOSIT` rejected in every country | CONFIRMED | `entries.go:164-169` (also rejects TOTAL/VENTILATION VAT types); `invoice.go:300-301` |
| VAT ID beats tax ID; BT-32 never emitted | CONFIRMED | `taxpayer.go:38-43`. Without a VAT ID, `tax_id_number` becomes BT-31 and fails `^[1-9]\d{8}$` (`gobl/regimes/de/tax_identity.go:17-27`) |
| `NOT_SUBJECT` → category O while BT-31/BT-48 are present (BR-O-02); BR-DE-14 | CONFIRMED | `german_exemption.go:56`; `addons/eu/en16931/tax_combo.go:41`; BR-O-02 text in the EN 16931 artefacts 1.3.17; BR-DE-14 (BT-119 mandatory) in XRechnung Schematron 2.6.0. BR-O-02 triggers per line, allowance or charge |
| `buyer_id` + `company_id` → two `cac:PartyIdentification`; never BT-47 | CONFIRMED | `recipient.go:159-172`; `gobl.ubl/party.go:322-349`; UBL-SR-16 "Buyer identifier shall occur maximum once" |
| `operation_date` → BT-72 only; BT-7 never | CONFIRMED | `invoice.go:433-441` (BT-72 carries the issue date when `operation_date` is empty); BT-7 only from `ValueDate`, which fiskaly never sets |
| Leitweg-ID max 46 characters vs the 32-character API cap | CONFIRMED | KoSIT Leitweg-ID Formatspezifikation v2.0.2 §2.1 and §3.4 (xeinkauf.de) |
| Seller contact mandatory | PARTLY CONFIRMED | BR-DE-2 makes BG-6 mandatory; BR-DE-5/6/7 (fatal) apply inside it. Cite them together |
| CustomizationID `urn:cen.eu:en16931:2017#compliant#urn:xeinkauf.de:kosit:xrechnung_3.0` | CONFIRMED | `common.sch` `XR-CIUS-ID`; BR-DE-21 is a warning only |
| CORRECTION → `CreditNote`, never negated, BG-3 read back from Invopop | CONFIRMED | `e_invoicing_worker.go:196-197`; `gobl.ubl/invoice.go:150-156`; `invopop.go:1104-1129`. The client's `preceding_document` is ignored for credit notes (`invoice.go:191-205`) |
| PPL `sequence.enumerate` effect on BT-1 | UNVERIFIABLE | Invopop's docs do not cover a preset `code`. Check a sandbox silo entry after a Peppol job, or ask Invopop |

### Validation pass (2026-11-03, after the restructure)

Every mapping row re-checked mechanically against the primary artefacts, and every code citation re-read at its pin.

| Check | Scope | Result |
| --- | --- | --- |
| XPath column vs the `ubl-inv` / `ubl-cn` bindings of `xrechnung-cius-model.xml` 2026-08-31 | all rows in §1–§5 | All match. BT-110 (§1.6) and BT-139 / BT-144 (§4) lacked their predicates; corrected. BT-111 name corrected |
| Cardinalities vs `m:structure id="invoice"` | every 1..1 / 0..x claim | All match; BT-29 is 0..n (§5, corrected) |
| Rule flags and texts vs XR-sch 2.6.0 and EN-sch 1.3.17 | every BR-DE, PEPPOL-EN16931, BR, BR-CO, UBL-SR / UBL-CR rule cited | All match, except BR-CO-25: removed from the CEN UBL and CII Schematron in v1.3.16 (eInvoicing-EN16931 commit `1eb862a`, 2026-04-04, issue #477), absent from the validator configuration's `EN16931-UBL-validation.xsl`. Only GOBL enforces it (§1.6, §6) |
| UBL-SR-43 | BT-11 on CreditNote | The rule allows `DocumentTypeCode` `50` only on a CreditNote; it does not by itself require it (§3, reworded). UBL 2.1 CreditNote XSD confirmed to lack `cac:ProjectReference` |
| UAPI paths vs [`UAPI_v6_Fields.md`](UAPI_v6_Fields.md) | all paths | All exist |
| fiskaly citations at `cd19ec8` | 137 | 130 confirmed. Corrected: CONSUMER 4xx (comes from the SDI destination-code check at RH:3565-3568, not RH:3561); BR-CO-25 is logged raw, not translated; rounding targets BT-112, not the payable amount; EUR default, BT-130 and OUTSTANDING warning logs cited at their actual location |
| Converter citations (gobl v0.501.0, gobl.ubl v0.99.0, gobl.de.xinvoice v0.0.9) | 90 | 88 confirmed. Corrected: missing BT-10 raises `DE-XRECHNUNG-29` when GOBL `ordering` is absent (created only from `document.references` or `tax_representative`, G/invoice.go:188-189, 392-393), `-30` otherwise; the BT-32 proposal also needs identity type `FC`. gobl.ubl v0.99.0 requires gobl v0.511.0 (Sources) |
