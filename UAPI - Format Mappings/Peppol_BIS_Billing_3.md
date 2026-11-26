# UAPI v6 → Peppol BIS Billing 3.0 (UBL)

As of 2026-11-16 · Status: draft for review

UAPI v6 invoice and correction fields mapped to Peppol BIS Billing 3.0, release 3.0.21 (UBL 2.1). A BE supplier's invoice becomes BIS 3 when its one BUSINESS recipient has `invoicing.type` `PEPPOL` and a BE or DE address (`RH:3270-3279`): workflow `BE_SEND_INVOICE_PEPPOL_OASIS_UBL_BUSINESS`, Invopop `doc_type` `bis-invoice-v3`; INVOICE becomes `/Invoice`, CORRECTION becomes `/CreditNote`. German suppliers on Peppol get XRechnung, which Invopop enforces on German Peppol invoices (`E/invopop.go:308-312`; [XRechnung_3.md](XRechnung_3.md)). The XPath column is the spec target from the BIS 3.0.21 syntax binding (`structure/syntax/ubl-invoice.xml`, `ubl-creditnote.xml`); credit-note lines use `cac:CreditNoteLine` and `cbc:CreditedQuantity`. The notes column records where today's output departs from it. UAPI paths are relative to `content.operation` (CORRECTION: under `data.`); "API docs: BT-x" is the BT fiskaly's API docs give where it differs from the spec.

Code citations: `G/` = fiskaly `src/unified/einvoicing/gobl/`; `E/` = fiskaly `src/unified/einvoicing/`; `RH` = fiskaly `src/unified/api/v6/record_handler.go`; `EA` = fiskaly `src/unified/domain/record/einvoicing_adaptations.go`; `SV1` = fiskaly `src/unified/domain/record/service_v1.go`; `W` = fiskaly `src/unified/worker/e_invoicing_worker.go`; `WF/` = fiskaly `cmd/invopop-workflow-migration/workflows/` (all at `cd19ec8`); `gobl:` = invopop/gobl v0.501.0 (fiskaly's `go.mod` pin); "gobl v0.511" = invopop/gobl v0.511.0, the minimum gobl.ubl v0.99.0 requires, cited for code that runs inside the converter; `ubl:` = invopop/gobl.ubl v0.99.0. Which converter and GOBL versions Invopop runs is not visible to fiskaly.

## 1. Invoice Mapping

### 1.1 Header and References

| UAPI | EN 16931 BT | BIS 3.0 UBL XPath | notes |
| --- | --- | --- | --- |
| `document.number` | BT-1 Invoice number | `/Invoice/cbc:ID` | must be unique across all series of the taxpayer (AR n° 1 art. 5 § 1, 1°). GOBL `code` (`G/invoice.go:153`); written as `series-code`, series never set (`ubl:invoice.go:127`, `223-228`). UNVERIFIED: the `sequence.enumerate` PPL step may replace it (§4) |
| `document.series` | — | — | no BT: BT-1 is `document.number` alone. ignored as-built (no `Series` in `G/`) |
| `document.issued_at` | BT-2 Invoice issue date | `/Invoice/cbc:IssueDate` | date part; time has no BIS element. default: absent → `cal.Today()` at each build, so the sync and worker builds can differ (`G/invoice.go:704-722`) |
| `type` = INVOICE | BT-3 Invoice type code | `/Invoice/cbc:InvoiceTypeCode` | derived: `380` (GOBL `standard`, `W:192-193`; `gobl:addons/eu/en16931/scenarios.go:21-28`; `ubl:common.go:168-179`). No other type code can be requested |
| NEW field needed: `document.invoice_currency` (`payments[0].details.currency`) | BT-5 Invoice currency code | `/Invoice/cbc:DocumentCurrencyCode` | first payment only, domain default EUR (`G/invoice.go:725-741`). The API enum allows CZK\|EUR\|SEK, but for BE any explicit non-EUR payment currency is rejected at record validation (`src/unified/country/be/v1/record.go:25`; `src/unified/domain/record/capabilities.go:188-220`, called from `src/unified/domain/record/service_v1.go:1968`) |
| `-` | BT-6 VAT accounting currency code | `/Invoice/cbc:TaxCurrencyCode` | derived: `EUR` when BT-5 ≠ EUR (AR n° 1 art. 5 § 1, 9°: total VAT in the national currency; PEPPOL-EN16931-R005). Not emitted today, since BE suppliers are EUR-only (BT-5); gobl.ubl writes it only with a matching exchange rate (`ubl:invoice.go:141-148`) |
| `document.operation_date` | BT-7 Value added tax point date | `/Invoice/cbc:TaxPointDate` | date of the chargeable event or of payment when it differs from the issue date (AR n° 1 art. 5 § 1, 5°; Art. 226(7) Directive 2006/112/EC); BR-CO-03: BT-7 or BT-8, not both. As-built: ignored for BT-7. gobl.ubl writes BT-7 only from GOBL `value_date` (`ubl:invoice.go:158-161`), which fiskaly never sets; GOBL `op_date` (`G/invoice.go:152`) has no UBL output; the operation date goes to BT-72 instead (`G/invoice.go:433-441`) |
| NEW field needed: `document.tax_point_date_code` | BT-8 Value added tax point date code | `/Invoice/cac:InvoicePeriod/cbc:DescriptionCode` | optional. UNTDID 2005 restriction (BR-CL-06, PEPPOL-EN16931-CL006); excluded when BT-7 is sent (BR-CO-03). As-built: `tax.point` never set (`ubl:invoice.go:208-221`) |
| NEW field needed: `document.period.start` | BT-73 Invoicing period start date | `/Invoice/cac:InvoicePeriod/cbc:StartDate` | conditional (BR-IC-11: BT-72 or BG-14 on an intra-EU supply). Supply period when the date of supply is a period (AR n° 1 art. 5 § 1, 5°); BR-29 |
| NEW field needed: `document.period.end` | BT-74 Invoicing period end date | `/Invoice/cac:InvoicePeriod/cbc:EndDate` | conditional (BR-IC-11). BR-29: not before BT-73 |
| `document.text` | BT-22 Invoice note | `/Invoice/cbc:Note` | ignored: `G/` never reads `Document().Text()`. Proposal: GOBL note → `cbc:Note`; R002 allows one, gobl.ubl merges |
| `document.payment_terms` | BT-20 Payment terms | `/Invoice/cac:PaymentTerms/cbc:Note` | `payment.terms.notes` (`G/invoice.go:316-321`; `ubl:payment.go:225-231`) |
| `document.references.buyer` | BT-10 Buyer reference | `/Invoice/cbc:BuyerReference` | `ordering.code` (`G/invoice.go:331-333`; `ubl:ordering.go:76-78`); R003 needs BT-10 or BT-13 |
| `document.references.purchase_order` | BT-13 Purchase order reference | `/Invoice/cac:OrderReference/cbc:ID` | `ordering.purchases[0]` (`ubl:ordering.go:107-112`). default `NA` when BT-10 and BT-13 are both absent (`ubl:ordering.go:181-187`) |
| NEW field needed: `document.references.sales_order` | BT-14 Sales order reference | `/Invoice/cac:OrderReference/cbc:SalesOrderID` | optional. `cac:OrderReference/cbc:ID` is 1..1 in UBL, so without BT-13 it carries `NA` |
| NEW field needed: `document.references.receiving_advice` | BT-15 Receiving advice reference | `/Invoice/cac:ReceiptDocumentReference/cbc:ID` | optional |
| `document.references.project` | BT-11 Project reference | `/Invoice/cac:ProjectReference/cbc:ID` | attached for BE suppliers (the DE drop, META-3995, does not apply) (`G/invoice.go:337-354`; `ubl:ordering.go:124-129`). Credit note: §3 |
| `document.references.contract` | BT-12 Contract reference | `/Invoice/cac:ContractDocumentReference/cbc:ID` | `ubl:ordering.go:143-151` |
| `document.references.despatch_advice` | BT-16 Despatch advice reference | `/Invoice/cac:DespatchDocumentReference/cbc:ID` | `ubl:ordering.go:131-135` |
| `document.references.tender` | BT-17 Tender or lot reference | `/Invoice/cac:OriginatorDocumentReference/cbc:ID` | `ubl:ordering.go:153-157`; credit-note element order: §3 |
| NEW field needed: `document.references.invoiced_object` | BT-18 Invoiced object identifier | `/Invoice/cac:AdditionalDocumentReference[cbc:DocumentTypeCode='130']/cbc:ID` | optional. `@schemeID` (BT-18-1) has no source |
| `document.references.buyer_routing` | BT-19 Buyer accounting reference | `/Invoice/cbc:AccountingCost` | `ordering.cost` (`ubl:ordering.go:80-83`) |
| `document.references.preceding_document.number` | BT-25 Preceding Invoice reference | `/Invoice/cac:BillingReference/cac:InvoiceDocumentReference/cbc:ID` | INVOICE only (`G/invoice.go:197-205`, `371-383`; `ubl:ordering.go:53-72`); ignored on credit notes (§3). Advance invoices deducted on a final invoice add further BG-3 entries (§1.3) |
| `document.references.preceding_document.issued_at` | BT-26 Preceding Invoice issue date | `/Invoice/cac:BillingReference/cac:InvoiceDocumentReference/cbc:IssueDate` | as above |
| NEW field needed: `document.attachments[i].id` | BT-122 Supporting document reference | `/Invoice/cac:AdditionalDocumentReference/cbc:ID` | format-mandatory (1..1 in BG-24, BR-52). No `cbc:DocumentTypeCode` (`130` marks BT-18) |
| NEW field needed: `document.attachments[i].description` | BT-123 Supporting document description | `/Invoice/cac:AdditionalDocumentReference/cbc:DocumentDescription` | optional |
| NEW field needed: `document.attachments[i].uri` | BT-124 External document location | `/Invoice/cac:AdditionalDocumentReference/cac:Attachment/cac:ExternalReference/cbc:URI` | optional |
| NEW field needed: `document.attachments[i].content` | BT-125 Attached document | `/Invoice/cac:AdditionalDocumentReference/cac:Attachment/cbc:EmbeddedDocumentBinaryObject` | optional. Base64; no PDF is attached in the Peppol workflow today |
| NEW field needed: `document.attachments[i].mime_code` | BT-125-1 Attached document Mime code | `/Invoice/cac:AdditionalDocumentReference/cac:Attachment/cbc:EmbeddedDocumentBinaryObject/@mimeCode` | conditional (UBL-DT-06 when BT-125 is sent; BR-CL-24, PEPPOL-EN16931-CL001) |
| NEW field needed: `document.attachments[i].filename` | BT-125-2 Attached document Filename | `/Invoice/cac:AdditionalDocumentReference/cac:Attachment/cbc:EmbeddedDocumentBinaryObject/@filename` | conditional (UBL-DT-07 when BT-125 is sent) |
| `document.activity_code`, `document.offline.reason`, `document.offline.reference` | — | — | ignored; no BT |
| `details.creators[]`, `details.training`, `details.properties` | — | — | ignored (record metadata) |

### 1.2 Buyer

| UAPI | EN 16931 BT | BIS 3.0 UBL XPath | notes |
| --- | --- | --- | --- |
| `recipients[i].type` = BUSINESS | BG-7 Buyer | `/Invoice/cac:AccountingCustomerParty` | exactly one BUSINESS recipient with `invoicing` (`EA:60-62`; `G/invoice.go:400-418`) |
| `recipients[i].name` | BT-44 Buyer name | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PartyLegalEntity/cbc:RegistrationName` | `ubl:party.go:207-216` |
| NEW field needed: `recipients[i].trade_name` (`recipients[i].name`) | BT-45 Buyer trading name | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PartyName/cbc:Name` | optional. `recipients[i].name` is BT-44. As-built: copy of BT-44 (`ubl:party.go:208-211`) |
| `recipients[i].identification.number` (type VAT) | BT-48 Buyer VAT identifier | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PartyTaxScheme/cbc:CompanyID` | the client must send the full VAT ID with its country prefix (AR n° 1 art. 5 § 1, 3°); the address country is only a fallback. As-built: `customer.tax_id`, country = recipient **address** country, scheme `VAT` (`G/identification.go:15-42`); prefix stripped (`gobl:tax/identity.go:174-185`) and rewritten as country + code (`ubl:party.go:229-249`); BR-CO-09 holds |
| `recipients[i].type` = CONSUMER | — | — | no PEPPOL: a CONSUMER invoicing block is parsed as SDI, so PEPPOL fails on the missing destination code (`RH:3557-3568`; §6) |
| `recipients[i].identification.type` = TAX \| PASSPORT \| DOCUMENT \| OTHER | — | — | the sync check fails (`EA:64-67`; §6.1); buyers without a VAT ID cannot be invoiced in this flow |
| `recipients[i].buyer_id` | BT-46 Buyer identifier | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PartyIdentification/cbc:ID` | the value must carry the ISO 6523 ICD prefix (`^[0-9]{4}:`, as `invoicing.identifier`); prefix → `@schemeID`, rest → value. As-built: identity `{key: other, label: buyer}` (`G/recipient.go:158-165`) → `PartyIdentification` without `@schemeID` (`ubl:party.go:322-349`) |
| `recipients[i].company_id` | BT-47 Buyer legal registration identifier | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PartyLegalEntity/cbc:CompanyID` | the value must carry the ISO 6523 ICD prefix (`^[0-9]{4}:`), split as for BT-46. As-built: second `cac:PartyIdentification` (BT-46 slot), identity has no legal scope (`G/recipient.go:166-172`; `ubl:party.go:322-349`; only `scope: legal` reaches `PartyLegalEntity/cbc:CompanyID`, `286-306`); with `buyer_id` → UBL-SR-16 fatal. Proposal: `scope: legal` + ICD |
| `recipients[i].invoicing.type` = PEPPOL | — | — | selects the workflow network (`E/invopop.go:388-412`) |
| `recipients[i].invoicing.identifier` | BT-49 Buyer electronic address | `/Invoice/cac:AccountingCustomerParty/cac:Party/cbc:EndpointID` | customer inbox `peppol` (`G/inbox.go:43-52`); GOBL splits `<scheme>:<id>` (`gobl:org/inbox.go:105-113`) and the en16931 addon copies it to an `iso6523-actorid-upis` endpoint (`gobl:addons/eu/en16931/org.go:141-160`); value = part after `:`, `@schemeID` = the 4-digit prefix (`ubl:party.go:27-36`; inbox fallback `47-59`). R010, BR-63 |
| `recipients[i].invoicing.identifier` (first 4 digits) | BT-49-1 Buyer electronic address identification scheme identifier | `/Invoice/cac:AccountingCustomerParty/cac:Party/cbc:EndpointID/@schemeID` | derived EAS code (BE: `0208` KBO/BCE or `9925` Belgium VAT number, per `structure/codelist/eas.xml` at tag 3.0.21). CL008 (EAS list); PEPPOL-COMMON-R043 checks `0208` (10 digits, mod 97) |
| `recipients[i].invoicing.email`, `.format`, `.destination_code`, `.pec` | — | — | other channels; not this flow |
| `recipients[i].address.line.street` + `.number` | BT-50 Buyer address line 1 | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PostalAddress/cbc:StreetName` | joined `street num` (`G/address.go:33-40`; `gobl:org/address.go:63-77`; `ubl:party.go:429-432`) |
| `recipients[i].address.line.suffix` | BT-51 Buyer address line 2 | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PostalAddress/cbc:AdditionalStreetName` | `street_extra` (`G/address.go:18-21`) |
| `recipients[i].address.line.prefix`, `.infix` | BT-163 Buyer address line 3 | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PostalAddress/cac:AddressLine/cbc:Line` | ignored |
| `recipients[i].address.city` | BT-52 Buyer city | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PostalAddress/cbc:CityName` | — |
| `recipients[i].address.code` | BT-53 Buyer post code | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PostalAddress/cbc:PostalZone` | must pass `cbc.Code.Validate` (`G/address.go:23-26`) |
| `recipients[i].address.region` | BT-54 Buyer country subdivision | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PostalAddress/cbc:CountrySubentity` | optional for BE/DE |
| `recipients[i].address.country` | BT-55 Buyer country code | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PostalAddress/cac:Country/cbc:IdentificationCode` | derived: overwritten with the tax-ID country, which fiskaly also takes from the address (`ubl:party.go:250-256`); BE or DE only (§6, stage A) |
| NEW field needed: `recipients[i].contact.name` | BT-56 Buyer contact point | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:Contact/cbc:Name` | optional |
| NEW field needed: `recipients[i].contact.phone` | BT-57 Buyer contact telephone number | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:Contact/cbc:Telephone` | optional |
| NEW field needed: `recipients[i].contact.email` (`recipients[i].invoicing.email`) | BT-58 Buyer contact email address | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:Contact/cbc:ElectronicMail` | optional. `invoicing.email` is the EMAIL delivery channel, not a contact |
| `recipients[i].shipping.name` | BT-70 Deliver to party name | `/Invoice/cac:Delivery/cac:DeliveryParty/cac:PartyName/cbc:Name` | `delivery.receiver.name` (`G/invoice.go:454-467`); also `PartyLegalEntity` → UBL-CR-397 warning (`ubl:party.go:376-385`) |
| NEW field needed: `recipients[i].shipping.location_id` | BT-71 Deliver to location identifier | `/Invoice/cac:Delivery/cac:DeliveryLocation/cbc:ID` | optional. `@schemeID` (BT-71-1) has no source |
| `recipients[i].shipping.date` | BT-72 Actual delivery date | `/Invoice/cac:Delivery/cbc:ActualDeliveryDate` | API docs: BT-72. Emitted only when the client sends it; no issue-date default. BR-IC-11 needs BT-72 or BG-14 on an intra-EU supply. As-built: ignored; BT-72 is filled from `document.operation_date`, else the issue date (`G/invoice.go:433-441`; `ubl:delivery.go:38-41`), a deviation (§4) |
| `recipients[i].shipping.address.*` | BG-15 Deliver to address | `/Invoice/cac:Delivery/cac:DeliveryLocation/cac:Address` | BT-75 to BT-80 and BT-165 in `cbc:StreetName`, `cbc:AdditionalStreetName`, `cbc:CityName`, `cbc:PostalZone`, `cbc:CountrySubentity`, `cac:Country/cbc:IdentificationCode`, `cac:AddressLine/cbc:Line`; same sub-mapping as the buyer address above (street + number, suffix, city, code, region, country; prefix/infix ignored) (`G/invoice.go:443-449`; `ubl:delivery.go:53-58`); BR-57 |
| `recipients[i].origin` | — | — | ignored |

### 1.3 Lines

| UAPI | EN 16931 BT | BIS 3.0 UBL XPath | notes |
| --- | --- | --- | --- |
| `entries[i]` | BG-25 Invoice line | `/Invoice/cac:InvoiceLine` | one line per entry (`G/entries.go:34-88`) |
| `entries[i].details.number` | BT-126 Invoice line identifier | `/Invoice/cac:InvoiceLine/cbc:ID` | INVOICE only; on CORRECTION derived `i+1` (§3). As-built: derived `i+1` (`G/entries.go:77`; `ubl:lines.go:77`); `details.number` ignored |
| `entries[i].type` (SALE \| RETURN) | — | — | no BT. As-built: RETURN lines map like SALE, signs as supplied |
| `entries[i].type` = RETURN with `entries[i].details.prepayment.document_number` | BG-25 Invoice line | `/Invoice/cac:InvoiceLine` | deduction of an advance on the final invoice: a negative line (negative BT-129, BT-146 not negative per BR-27) in the VAT category and rate of the advance, so the VAT already charged is not charged again (AR n° 1 art. 5 § 1, 11°). As-built: maps like SALE |
| `entries[i].details.prepayment.document_number` | BT-25 Preceding Invoice reference | `/Invoice/cac:BillingReference/cac:InvoiceDocumentReference/cbc:ID` | one further BG-3 per advance invoice (BG-3 0..n); reference to the earlier document required by AR n° 1 art. 5 § 1, 11°. BT-26 has no source (optional). As-built: ignored |
| `entries[i].data.type` = GROUP | BG-25 Invoice line | `/Invoice/cac:InvoiceLine` | `data.items[]` with different VAT → one line per VAT category and rate; items with the same VAT → one line. As-built: one line from group-level values (`RH:4034-4070`); `data.items[]` are parsed (`RH:4072-4080`) and then ignored (`G/entries.go:299-307`) |
| `entries[i].data.text` | BT-153 Item name | `/Invoice/cac:InvoiceLine/cac:Item/cbc:Name` | `G/entries.go:299-306` |
| `entries[i].data.unit.quantity` | BT-129 Invoiced quantity | `/Invoice/cac:InvoiceLine/cbc:InvoicedQuantity` | written as GOBL holds it (`ubl:lines.go:233-245`); UAPI input allows ≤ 8 dp |
| `entries[i].data.unit.measure` | BT-130 Invoiced quantity unit of measure code | `/Invoice/cac:InvoiceLine/cbc:InvoicedQuantity/@unitCode` | must be a UN/ECE Rec 20/21 code (BR-CL-23), validated at sync. As-built: GOBL unit key or Rec 20 code as given, else dropped with a warning (`G/entries.go:312-323`); default `C62` (`gobl:addons/eu/en16931/org.go:79-86`) |
| `entries[i].data.unit.price.exclusive` | BT-146 Item net price | `/Invoice/cac:InvoiceLine/cac:Price/cbc:PriceAmount` | API docs: BT-146. BR-27: not negative. As-built: ignored; BT-146 is recomputed from `unit.price.inclusive` (`tax.prices_include: VAT`, `G/invoice.go:165-166`) as `gross / (1 + rate/100)` by GOBL `RemoveIncludedTaxes` (`ubl:ubl.go:161`) and written at GOBL precision (`ubl:lines.go:496-499`) |
| NEW field needed: `entries[i].data.unit.price.gross` | BT-148 Item gross price | `/Invoice/cac:InvoiceLine/cac:Price/cac:AllowanceCharge/cbc:BaseAmount` | optional. Net of VAT, before the price discount; PEPPOL-EN16931-R046: BT-146 = BT-148 − BT-147. `unit.price.inclusive` is not BT-148 |
| NEW field needed: `entries[i].data.unit.price.discount` | BT-147 Item price discount | `/Invoice/cac:InvoiceLine/cac:Price/cac:AllowanceCharge/cbc:Amount` | optional. `cbc:ChargeIndicator` `false` (PEPPOL-EN16931-R044) |
| `entries[i].data.unit.factor` | BT-149 Item price base quantity | `/Invoice/cac:InvoiceLine/cac:Price/cbc:BaseQuantity` | `@unitCode` must equal BT-130 (PEPPOL-EN16931-R130). ignored (API docs: BT-149); GOBL v0.501.0 `org.Item` has no `per` (`ubl:lines.go:500-502` needs it) |
| `entries[i].data.value.base.exclusive` | BT-131 Invoice line net amount | `/Invoice/cac:InvoiceLine/cbc:LineExtensionAmount` | `value.base.exclusive` is the line amount after line allowances and charges, on every `vat.type` (the API docs must say so); must equal BT-129 × BT-146 / BT-149 − BT-136 + BT-141 within rounding (PEPPOL-EN16931-R120, 0.02). As-built: ignored; BT-131 derived (§1.6) |
| `entries[i].data.vat.exclusive`, `entries[i].data.value.base.inclusive` | — | — | no BT: BT-131 comes from `value.base.exclusive`; `.inclusive` includes VAT. ignored |
| `entries[i].data.unit.price.inclusive`, `entries[i].data.value.discount.inclusive`, `entries[i].data.value.surcharge.inclusive` | — | — | no BT (VAT-inclusive amounts). As-built: the sources of BT-146, BT-136 and BT-141 |
| `entries[i].data.value.discount.exclusive` | BT-136 Invoice line allowance amount | `/Invoice/cac:InvoiceLine/cac:AllowanceCharge[cbc:ChargeIndicator='false']/cbc:Amount` | if ≠ 0. As-built: `.inclusive` net-converted, 2 dp (`G/entries.go:45-55`; `ubl:lines.go:467-477`); `.exclusive` ignored |
| NEW field needed: `entries[i].data.value.discount.base` | BT-137 Invoice line allowance base amount | `/Invoice/cac:InvoiceLine/cac:AllowanceCharge[cbc:ChargeIndicator='false']/cbc:BaseAmount` | optional. Together with BT-138 or neither (PEPPOL-EN16931-R041, R042) |
| NEW field needed: `entries[i].data.value.discount.percentage` | BT-138 Invoice line allowance percentage | `/Invoice/cac:InvoiceLine/cac:AllowanceCharge[cbc:ChargeIndicator='false']/cbc:MultiplierFactorNumeric` | optional. As BT-137 |
| NEW field needed: `entries[i].data.value.discount.reason` (fixed `Special Discount`) | BT-139 Invoice line allowance reason | `/Invoice/cac:InvoiceLine/cac:AllowanceCharge[cbc:ChargeIndicator='false']/cbc:AllowanceChargeReason` | conditional (BR-42: BT-139 or BT-140). As-built: fixed `Special Discount` (META-2347) |
| NEW field needed: `entries[i].data.value.discount.reason_code` | BT-140 Invoice line allowance reason code | `/Invoice/cac:InvoiceLine/cac:AllowanceCharge[cbc:ChargeIndicator='false']/cbc:AllowanceChargeReasonCode` | conditional (BR-42). UNTDID 5189 (BR-CL-19) |
| `entries[i].data.value.surcharge.exclusive` | BT-141 Invoice line charge amount | `/Invoice/cac:InvoiceLine/cac:AllowanceCharge[cbc:ChargeIndicator='true']/cbc:Amount` | if ≠ 0. As-built: as discount (`G/entries.go:57-67`); `.exclusive` ignored |
| NEW field needed: `entries[i].data.value.surcharge.base` | BT-142 Invoice line charge base amount | `/Invoice/cac:InvoiceLine/cac:AllowanceCharge[cbc:ChargeIndicator='true']/cbc:BaseAmount` | optional. Together with BT-143 or neither (PEPPOL-EN16931-R041, R042) |
| NEW field needed: `entries[i].data.value.surcharge.percentage` | BT-143 Invoice line charge percentage | `/Invoice/cac:InvoiceLine/cac:AllowanceCharge[cbc:ChargeIndicator='true']/cbc:MultiplierFactorNumeric` | optional. As BT-142 |
| NEW field needed: `entries[i].data.value.surcharge.reason` (fixed `Special Surcharge`) | BT-144 Invoice line charge reason | `/Invoice/cac:InvoiceLine/cac:AllowanceCharge[cbc:ChargeIndicator='true']/cbc:AllowanceChargeReason` | conditional (BR-44: BT-144 or BT-145). As-built: fixed `Special Surcharge` |
| NEW field needed: `entries[i].data.value.surcharge.reason_code` | BT-145 Invoice line charge reason code | `/Invoice/cac:InvoiceLine/cac:AllowanceCharge[cbc:ChargeIndicator='true']/cbc:AllowanceChargeReasonCode` | conditional (BR-44). UNTDID 7161 (BR-CL-20) |
| `entries[i].data.vat.type` = VAT_RATE (with `vat.code` STANDARD \| REDUCED_n and `vat.percentage`) | BT-151 Invoiced item VAT category code | `/Invoice/cac:InvoiceLine/cac:Item/cac:ClassifiedTaxCategory/cbc:ID` | category from `vat.type` + `vat.code` + `vat.percentage`; code table maintained separately. As-built: `S` for any non-zero rate, `Z` (GOBL key `zero`) for 0 (`G/entries.go:101-122`; `gobl:addons/eu/en16931/tax_combo.go:33-42`); `vat.code` unused |
| `entries[i].data.vat.percentage` | BT-152 Invoiced item VAT rate | `/Invoice/cac:InvoiceLine/cac:Item/cac:ClassifiedTaxCategory/cbc:Percent` | `ubl:lines.go:349-357` |
| `entries[i].data.vat.type` = VAT_EXEMPTION (with `vat.code`, `vat.reason`) | BT-151 Invoiced item VAT category code | `/Invoice/cac:InvoiceLine/cac:Item/cac:ClassifiedTaxCategory/cbc:ID` | category from `vat.code` (exempt, intra-EU supply, export, out of scope, special regimes; AR n° 1 art. 5 § 1, 10° to 10° quater); code table maintained separately. As-built: every BE code → `E`, Percent `0` (`G/entries.go:148-161`, `175-189`); NOT_SUBJECT → `O` without Percent, which always fails BR-O-02 (BT-31 and BT-48 always present, §6) |
| `entries[i].data.vat.code` (VAT_EXEMPTION) | BT-121 VAT exemption reason code | `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cac:TaxCategory/cbc:TaxExemptionReasonCode` | VATEX code from `vat.code`; code table maintained separately. BR-E-10, BR-AE-10: BT-120 or BT-121. As-built: gap, no VATEX code for BE (no `cef-vatex` in the BE branch) |
| `entries[i].data.vat.reason` | BT-120 VAT exemption reason text | `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cac:TaxCategory/cbc:TaxExemptionReason` | the exemption mention of AR n° 1 art. 5 § 1, 10°. One reason per VAT category: the sync check must reject differing reasons within one category. As-built: one tax note per category, reasons joined with `; ` (`G/entries.go:208-269`); mandatory for BE; matched by category (`gobl:addons/eu/en16931/bill.go:52-72`; `ubl:totals.go:122-126`) |
| `entries[i].data.vat.type` = VAT_REVERSE_CHARGE | BT-151 Invoiced item VAT category code | `/Invoice/cac:InvoiceLine/cac:Item/cac:ClassifiedTaxCategory/cbc:ID` | `AE`; BT-120 carries "Autoliquidation" (AR n° 1 art. 5 § 1, 9° bis) and BT-121 the matching code, both from the code table maintained separately (BR-AE-10). As-built: the GOBL build fails at sync: `unsupported VAT type … 'REVERSE_CHARGE'` (`G/entries.go:164-169`). API docs: not applicable for BE/DE/IT |
| `entries[i].data.vat.amount`, `.inclusive` | — | — | no BT: line VAT amounts are not BT-110/BT-117/BT-112 sources (API docs: BT-110/117, BT-112). ignored |
| `entries[i].data.vat.regime`, `.regime_base`, `.surcharge.rate`, `.surcharge.amount` | — | — | ignored (API docs: not applicable for BE/DE/IT) |
| `entries[i].data.vat.fiscal_location.country`, `.region` | — | — | ignored; no BT. API docs: defaults to the taxpayer's fiscal region |
| `entries[i].data.product.number` (type SKU, MPN, OTHER) | BT-155 Item Seller's identifier | `/Invoice/cac:InvoiceLine/cac:Item/cac:SellersItemIdentification/cbc:ID` | ignored (API docs: BT-155, BT-157 for `product.type` OTHER only; none for SKU, MPN); proposal: GOBL `item.ref` (`ubl:lines.go:418-423`) |
| `entries[i].data.product.number` (type GTIN-8/12/13/14) | BT-157 Item standard identifier | `/Invoice/cac:InvoiceLine/cac:Item/cac:StandardItemIdentification/cbc:ID` (`@schemeID` `0160`) | ignored; proposal: identity key `gtin` → 0160 (`gobl:addons/eu/en16931/org.go:50-53`) |
| `entries[i].data.product.code` (type OTHER) | BT-158 Item classification identifier | `/Invoice/cac:InvoiceLine/cac:Item/cac:CommodityClassification/cbc:ItemClassificationCode` | ignored (API docs: BT-158) |
| NEW field needed: `entries[i].data.product.code_scheme` | BT-158-1 Item classification identifier identification scheme identifier | `/Invoice/cac:InvoiceLine/cac:Item/cac:CommodityClassification/cbc:ItemClassificationCode/@listID` | conditional (BR-65: required with BT-158). UNTDID 7143 (BR-CL-13) |
| NEW field needed: `entries[i].data.product.buyer_number` | BT-156 Item Buyer's identifier | `/Invoice/cac:InvoiceLine/cac:Item/cac:BuyersItemIdentification/cbc:ID` | optional |
| NEW field needed: `entries[i].data.product.origin_country` | BT-159 Item country of origin | `/Invoice/cac:InvoiceLine/cac:Item/cac:OriginCountry/cbc:IdentificationCode` | optional |
| `entries[i].data.product.details.label` | BT-154 Item description | `/Invoice/cac:InvoiceLine/cac:Item/cbc:Description` | ignored (API docs: BT-154) |
| `entries[i].data.product.details.name` | BT-153 Item name | `/Invoice/cac:InvoiceLine/cac:Item/cbc:Name` | ignored; BT-153 comes from `data.text` |
| `entries[i].data.product.details.brand`, `.model`, `.serial`, `.batch`, `.date_of_entry`, `.date_of_production` | BT-160 Item attribute name, BT-161 Item attribute value | `/Invoice/cac:InvoiceLine/cac:Item/cac:AdditionalItemProperty/cbc:Name`, `/Invoice/cac:InvoiceLine/cac:Item/cac:AdditionalItemProperty/cbc:Value` | one `cac:AdditionalItemProperty` per field. ignored |
| NEW field needed: `entries[i].data.product.attributes[i].name` | BT-160 Item attribute name | `/Invoice/cac:InvoiceLine/cac:Item/cac:AdditionalItemProperty/cbc:Name` | law (AR n° 1 art. 5 § 1, 7°, for vehicles: cylinder capacity, engine power; registration plate for works on motor vehicles). Optional otherwise |
| NEW field needed: `entries[i].data.product.attributes[i].value` | BT-161 Item attribute value | `/Invoice/cac:InvoiceLine/cac:Item/cac:AdditionalItemProperty/cbc:Value` | law (AR n° 1 art. 5 § 1, 7°, as BT-160) |
| `entries[i].details.description` | BT-127 Invoice line note | `/Invoice/cac:InvoiceLine/cbc:Note` | ignored (API docs: BT-127) |
| NEW field needed: `entries[i].details.object_id` | BT-128 Invoice line object identifier | `/Invoice/cac:InvoiceLine/cac:DocumentReference[cbc:DocumentTypeCode='130']/cbc:ID` | optional. One per line (PEPPOL-EN16931-R100, R101) |
| NEW field needed: `entries[i].details.order_line_reference` | BT-132 Referenced purchase order line reference | `/Invoice/cac:InvoiceLine/cac:OrderLineReference/cbc:LineID` | optional |
| NEW field needed: `entries[i].details.buyer_accounting_reference` | BT-133 Invoice line Buyer accounting reference | `/Invoice/cac:InvoiceLine/cbc:AccountingCost` | optional |
| NEW field needed: `entries[i].details.period.start` | BT-134 Invoice line period start date | `/Invoice/cac:InvoiceLine/cac:InvoicePeriod/cbc:StartDate` | optional. Within BG-14 (PEPPOL-EN16931-R110) |
| NEW field needed: `entries[i].details.period.end` | BT-135 Invoice line period end date | `/Invoice/cac:InvoiceLine/cac:InvoicePeriod/cbc:EndDate` | optional. Within BG-14 (PEPPOL-EN16931-R111); BR-30 |
| `entries[i].details.concept`, `.purpose`, `.regulatory`, `.label`, `.activity_code`, `.properties` | — | — | ignored |
| `entries[i].details.voucher.*`, `.applied_vouchers[]` | — | — | ignored |

### 1.4 Payment

| UAPI | EN 16931 BT | BIS 3.0 UBL XPath | notes |
| --- | --- | --- | --- |
| `payments[i].details.date` (first OUTSTANDING) | BT-9 Payment due date | `/Invoice/cbc:DueDate` | optional in the schema, required for OUTSTANDING at conversion (`payment due date not set`, `G/invoice.go:607-621`); `ubl:payment.go:233-235`. Later OUTSTANDING entries dropped with a warning. Credit note: §3 |
| `payments[i].details.amount` (OUTSTANDING) | — | — | `due_dates[0].amount`; no UBL element. The sum of all payments drives line rounding (§1.6) |
| `payments[i].type`, `payments[i].kind` (CARD), `payments[i].instruction.type` | BT-81 Payment means type code | `/Invoice/cac:PaymentMeans/cbc:PaymentMeansCode` | derived: CASH → `10`; CARD → `54` (CREDIT), `55` (DEBIT), `48` (PREPAID); ONLINE → `68`; credit transfer → `30` (`58` SEPA); direct debit → `49` (`59` SEPA); several methods → `1`; CARD `kind` TICKET, SINGLE_PURPOSE_VOUCHER, MULTI_PURPOSE_VOUCHER, MEAL_VOUCHER, OTHER and `instruction.type` UNKNOWN → `1` (not defined), with BT-82 naming the payment type. As-built: CREDIT_TRANSFER fixed `30`, DIRECT_DEBIT fixed `49` (`gobl:addons/eu/en16931/pay.go:12-27`); UNKNOWN has no UNTDID 4461 code → `payment means extension is required (BR-49)` / converter error inside Invopop (`gobl:addons/eu/en16931/pay.go:40-48`; `ubl:payment.go:143-151`); CARD, CASH and ONLINE become advances without `cac:PaymentMeans`; `kind` ignored (API docs: BT-81) |
| `-` | BT-82 Payment means text | `/Invoice/cac:PaymentMeans/cbc:PaymentMeansCode/@name` | derived: text from the payment type (cash, card, online …). `payments[i].name` (ONLINE, OTHER; API docs: BT-82) is not a BT-82 source. As-built: `instruction.text` lands here (BT-83 row) |
| `payments[i].instruction.account` (CREDIT_TRANSFER) | BT-84 Payment account identifier | `/Invoice/cac:PaymentMeans/cac:PayeeFinancialAccount/cbc:ID` | IBAN (`G/invoice.go:530-537`; `ubl:payment.go:195-212`); BR-50, BR-61 |
| `payments[i].instruction.name` | BT-85 Payment account name | `/Invoice/cac:PaymentMeans/cac:PayeeFinancialAccount/cbc:Name` | — |
| `payments[i].instruction.payment_service_provider` | BT-86 Payment service provider identifier | `/Invoice/cac:PaymentMeans/cac:PayeeFinancialAccount/cac:FinancialInstitutionBranch/cbc:ID` | free text 1–128, no BIC check |
| `payments[i].instruction.text` | BT-83 Remittance information | `/Invoice/cac:PaymentMeans/cbc:PaymentID` | API docs: BT-83. As-built: GOBL `instructions.detail` (`G/invoice.go:538-540`, `563-565`) → `cbc:PaymentMeansCode/@name`, i.e. BT-82 Payment means text (`ubl:payment.go:160-162`) |
| `payments[i].instruction.properties` | — | — | no BT (API docs: none). As-built: GOBL `instructions.ref` → `cbc:PaymentID` (BT-83) (`G/invoice.go:541-543`; `ubl:payment.go:157-159`), so a Belgian structured communication (`+++…+++`) only reaches BT-83 through `properties` today |
| `payments[i].instruction.mandate_reference` | BT-89 Mandate reference identifier | `/Invoice/cac:PaymentMeans/cac:PaymentMandate/cbc:ID` | `ubl:payment.go:166-171`; PEPPOL-EN16931-R061 |
| `payments[i].instruction.creditor_id.identifier` (`.type` SEPA) | BT-90 Bank assigned creditor identifier | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PartyIdentification/cbc:ID` (`@schemeID` `SEPA`) | no payee party, so it goes on the seller (`ubl:payment.go:121-136`) |
| `payments[i].instruction.debited_account` | BT-91 Debited account identifier | `/Invoice/cac:PaymentMeans/cac:PaymentMandate/cac:PayerFinancialAccount/cbc:ID` | `ubl:payment.go:172-176` |
| `payments[i].number` (CARD) | BT-87 Payment card primary account number | `/Invoice/cac:PaymentMeans/cac:CardAccount/cbc:PrimaryAccountNumberID` | masked PAN only (the binding: never the full number; the UAPI pattern allows `*`). BG-18 also needs `cbc:NetworkID` (§5). ignored (API docs: BT-87) |
| NEW field needed: `payments[i].holder_name` | BT-88 Payment card holder name | `/Invoice/cac:PaymentMeans/cac:CardAccount/cbc:HolderName` | optional |
| `payments[i].details.amount` (CARD, CASH, ONLINE, SINGLE_PURPOSE_VOUCHER, MULTI_PURPOSE_VOUCHER, OTHER) | BT-113 Paid amount | `/Invoice/cac:LegalMonetaryTotal/cbc:PrepaidAmount` | DEPOSIT amounts are not counted again: the advance is deducted through the RETURN line (§1.3). As-built: `payment.advances[]` (`G/invoice.go:290-299`, `623-636`) summed into `totals.advance` (`ubl:totals.go:77-79`); no `cac:PaymentMeans`, no `cac:PrepaidPayment` |
| `payments[i].type` = DEPOSIT, `payments[i].document_number` | — | — | no BT of its own: the advance invoice is referenced from the RETURN entry (§1.3). As-built: the GOBL build fails at sync: `unsupported payment type` (`G/invoice.go:300-302`) |
| `payments[i].type` = MEAL_VOUCHER | — | — | the GOBL build fails at sync: `unsupported payment type` (`G/invoice.go:300-302`) |
| `payments[i].details.discount` | BT-92 Document level allowance amount | `/Invoice/cac:AllowanceCharge[cbc:ChargeIndicator='false']/cbc:Amount` | a document-level allowance (BG-20) granted for payment. ignored |
| NEW field needed: `payments[i].details.discount_vat.type` | BT-95 Document level allowance VAT category code | `/Invoice/cac:AllowanceCharge[cbc:ChargeIndicator='false']/cac:TaxCategory/cbc:ID` | format-mandatory (BR-32, when `details.discount` is sent). Category per the code table maintained separately |
| NEW field needed: `payments[i].details.discount_vat.percentage` | BT-96 Document level allowance VAT rate | `/Invoice/cac:AllowanceCharge[cbc:ChargeIndicator='false']/cac:TaxCategory/cbc:Percent` | conditional (BR-S-06 for category `S`) |
| `-` | BT-97 Document level allowance reason | `/Invoice/cac:AllowanceCharge[cbc:ChargeIndicator='false']/cbc:AllowanceChargeReason` | derived `Payment discount` for `payments[i].details.discount` (BR-33) |
| `payments[i].details.date` (non-OUTSTANDING), `.details.channel`, `.details.properties`; `payments[i].concept`, `.count`; voucher `payments[i].number` | — | — | ignored or no BIS element (advance date) |
| NEW field needed: `payee.name` | BT-59 Payee name | `/Invoice/cac:PayeeParty/cac:PartyName/cbc:Name` | conditional (BR-17: required when BG-10 is used; BG-10 only when the payee differs from the seller) |
| NEW field needed: `payee.identifier` | BT-60 Payee identifier | `/Invoice/cac:PayeeParty/cac:PartyIdentification/cbc:ID` | optional. Value with the ISO 6523 ICD prefix, split into `@schemeID` as for BT-46 |
| NEW field needed: `payee.company_id` | BT-61 Payee legal registration identifier | `/Invoice/cac:PayeeParty/cac:PartyLegalEntity/cbc:CompanyID` | optional. ICD prefix as BT-60 |

### 1.5 VAT Breakdown and Totals

| UAPI | EN 16931 BT | BIS 3.0 UBL XPath | notes |
| --- | --- | --- | --- |
| `breakdown[i]` | BG-23 VAT breakdown | `/Invoice/cac:TaxTotal/cac:TaxSubtotal` | calculated from the lines; sync rejects a difference above the rounding tolerance. As-built: ignored; one subtotal per GOBL rate, derived from the lines (`ubl:totals.go:102-145`) |
| `breakdown[i].exclusive` | BT-116 VAT category taxable amount | `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cbc:TaxableAmount` | calculated from the lines; sync rejects a difference above the rounding tolerance. As-built: ignored; derived |
| `breakdown[i].amount` | BT-117 VAT category tax amount | `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cbc:TaxAmount` | calculated from the lines; sync rejects a difference above the rounding tolerance. As-built: ignored; derived |
| `breakdown[i].type`, `.code` | BT-118 VAT category code | `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cac:TaxCategory/cbc:ID` | same enums as `entries[i].data.vat.type`, `.code`; category per the code table maintained separately; calculated from the lines. As-built: ignored; derived from line combos |
| `breakdown[i].percentage` | BT-119 VAT category rate | `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cac:TaxCategory/cbc:Percent` | calculated from the lines; sync rejects a difference above the rounding tolerance. As-built: ignored; derived; `0` for E/Z, omitted for O (`ubl:totals.go:129-136`) |
| `breakdown[i].inclusive` | — | — | no BT (API docs: BT-112). ignored |
| `totals.vat.exclusive` | BT-109 Invoice total amount without VAT | `/Invoice/cac:LegalMonetaryTotal/cbc:TaxExclusiveAmount` | calculated from the lines; sync rejects a difference above the rounding tolerance. As-built: ignored; BT-109 is GOBL `totals.total`; the UAPI value is only compared, with a warning log (`E/invopop.go:753-782`) |
| `totals.vat.exclusive` | BT-106 Sum of Invoice line net amount | `/Invoice/cac:LegalMonetaryTotal/cbc:LineExtensionAmount` | equal to BT-109 only while no document-level allowances or charges are sent; otherwise BT-106 = BT-109 + BT-107 − BT-108. As-built: ignored; BT-106 is GOBL `totals.sum` |
| `totals.vat.amount` | BT-110 Invoice total VAT amount | `/Invoice/cac:TaxTotal/cbc:TaxAmount` | calculated from the lines; sync rejects a difference above the rounding tolerance. As-built: ignored; BT-110 is GOBL `totals.tax` (compared, warning log, §1.6) |
| `totals.vat.inclusive` | BT-112 Invoice total amount with VAT | `/Invoice/cac:LegalMonetaryTotal/cbc:TaxInclusiveAmount` | calculated from the lines; sync rejects a difference above the rounding tolerance. As-built: ignored; BT-112 is GOBL `totals.total_with_tax` (compared, warning log, §1.6) |
| NEW field needed: `totals.vat.amount_accounting` | BT-111 Invoice total VAT amount in accounting currency | `/Invoice/cac:TaxTotal/cbc:TaxAmount[@currencyID = /Invoice/cbc:TaxCurrencyCode]` | conditional (BR-53; Art. 230 Dir. 2006/112/EC). Required once a non-EUR BT-5 is enabled for BE: AR n° 1 art. 5 § 1, 9° wants the total VAT in EUR. Second `cac:TaxTotal` without subtotals (PEPPOL-EN16931-R054). The API docs describe `totals.vat.amount` as "in EUR (€)", which fits BT-110 only while BT-5 is EUR. As-built: BE is EUR-only; gobl.ubl writes it only together with BT-6 (`ubl:invoice.go:141-148`) |
| NEW field needed: `document.allowances[i].amount` | BT-92 Document level allowance amount | `/Invoice/cac:AllowanceCharge[cbc:ChargeIndicator='false']/cbc:Amount` | format-mandatory (BR-31, within BG-20) |
| NEW field needed: `document.allowances[i].base` | BT-93 Document level allowance base amount | `/Invoice/cac:AllowanceCharge[cbc:ChargeIndicator='false']/cbc:BaseAmount` | optional. Together with BT-94 or neither (PEPPOL-EN16931-R041, R042) |
| NEW field needed: `document.allowances[i].percentage` | BT-94 Document level allowance percentage | `/Invoice/cac:AllowanceCharge[cbc:ChargeIndicator='false']/cbc:MultiplierFactorNumeric` | optional. As BT-93 |
| NEW field needed: `document.allowances[i].vat_type` | BT-95 Document level allowance VAT category code | `/Invoice/cac:AllowanceCharge[cbc:ChargeIndicator='false']/cac:TaxCategory/cbc:ID` | format-mandatory (BR-32, within BG-20). Category per the code table maintained separately |
| NEW field needed: `document.allowances[i].vat_percentage` | BT-96 Document level allowance VAT rate | `/Invoice/cac:AllowanceCharge[cbc:ChargeIndicator='false']/cac:TaxCategory/cbc:Percent` | conditional (BR-S-06 for category `S`) |
| NEW field needed: `document.allowances[i].reason` | BT-97 Document level allowance reason | `/Invoice/cac:AllowanceCharge[cbc:ChargeIndicator='false']/cbc:AllowanceChargeReason` | conditional (BR-33: BT-97 or BT-98). Rebates and discounts not in the unit price are a legal mention (AR n° 1 art. 5 § 1, 8°) |
| NEW field needed: `document.allowances[i].reason_code` | BT-98 Document level allowance reason code | `/Invoice/cac:AllowanceCharge[cbc:ChargeIndicator='false']/cbc:AllowanceChargeReasonCode` | conditional (BR-33). UNTDID 5189 (BR-CL-19) |
| NEW field needed: `document.charges[i].amount` | BT-99 Document level charge amount | `/Invoice/cac:AllowanceCharge[cbc:ChargeIndicator='true']/cbc:Amount` | format-mandatory (BR-36, within BG-21) |
| NEW field needed: `document.charges[i].base` | BT-100 Document level charge base amount | `/Invoice/cac:AllowanceCharge[cbc:ChargeIndicator='true']/cbc:BaseAmount` | optional. Together with BT-101 or neither (PEPPOL-EN16931-R041, R042) |
| NEW field needed: `document.charges[i].percentage` | BT-101 Document level charge percentage | `/Invoice/cac:AllowanceCharge[cbc:ChargeIndicator='true']/cbc:MultiplierFactorNumeric` | optional. As BT-100 |
| NEW field needed: `document.charges[i].vat_type` | BT-102 Document level charge VAT category code | `/Invoice/cac:AllowanceCharge[cbc:ChargeIndicator='true']/cac:TaxCategory/cbc:ID` | format-mandatory (BR-37, within BG-21). Category per the code table maintained separately |
| NEW field needed: `document.charges[i].vat_percentage` | BT-103 Document level charge VAT rate | `/Invoice/cac:AllowanceCharge[cbc:ChargeIndicator='true']/cac:TaxCategory/cbc:Percent` | conditional (BR-S-07 for category `S`) |
| NEW field needed: `document.charges[i].reason` | BT-104 Document level charge reason | `/Invoice/cac:AllowanceCharge[cbc:ChargeIndicator='true']/cbc:AllowanceChargeReason` | conditional (BR-38: BT-104 or BT-105) |
| NEW field needed: `document.charges[i].reason_code` | BT-105 Document level charge reason code | `/Invoice/cac:AllowanceCharge[cbc:ChargeIndicator='true']/cbc:AllowanceChargeReasonCode` | conditional (BR-38). UNTDID 7161 (BR-CL-20) |
| `-` | BT-107 Sum of allowances on document level, BT-108 Sum of charges on document level | `/Invoice/cac:LegalMonetaryTotal/cbc:AllowanceTotalAmount`, `/Invoice/cac:LegalMonetaryTotal/cbc:ChargeTotalAmount` | calculated (§1.6). Not emitted today: no BG-20/BG-21 |
| `-` | BT-113 Paid amount | `/Invoice/cac:LegalMonetaryTotal/cbc:PrepaidAmount` | from non-OUTSTANDING payments (§1.4) |
| NEW field needed: `totals.rounding` | BT-114 Rounding amount | `/Invoice/cac:LegalMonetaryTotal/cbc:PayableRoundingAmount` | optional. As-built: derived from GOBL `totals.rounding` (`ubl:totals.go:74-76`), set inside the converter. `RemoveIncludedTaxes` and `RoundToCurrency` recalculate while keeping the original payable amount (gobl v0.511 `bill/calculator.go:337-338`, `bill/rounding.go:33-68`, `239-258`), so VAT-inclusive UAPI input can produce a ±0.01 BT-114 that keeps BT-115 at the gross total (converter sample `peppol/out/invoice-prices-include-vat.xml:93`) |
| `-` | BT-115 Amount due for payment | `/Invoice/cac:LegalMonetaryTotal/cbc:PayableAmount` | calculated (§1.6). As-built: `totals.due` when advances exist, else `totals.payable` (`ubl:totals.go:61-82`) |

### 1.6 Calculated Fields

| UAPI | EN 16931 BT | BIS 3.0 UBL XPath | formula | notes |
| --- | --- | --- | --- | --- |
| `entries[i].data.value.base.exclusive` (check) | BT-131 Invoice line net amount | `/Invoice/cac:InvoiceLine/cbc:LineExtensionAmount` | `BT-129 × BT-146 / BT-149 − BT-136 + BT-141` | PEPPOL-EN16931-R120 tolerance 0.02. As-built: `round2(qty × net price − allowances + charges)` from the gross-to-net price, rounded per line by GOBL's `currency` rule, which fiskaly sets on the document (`G/invoice.go:160-168`). The converter keeps a rule set on the document: its switch to `precise` applies only when none is set (gobl v0.511 `bill/calculator.go:329-335`), so it never applies to fiskaly invoices, and `RoundToCurrency` (`ubl:ubl.go:164`) only re-rounds amounts that still exceed the currency precision |
| all lines | BT-106 Sum of Invoice line net amount | `/Invoice/cac:LegalMonetaryTotal/cbc:LineExtensionAmount` | `Σ BT-131` | BR-CO-10 |
| BG-20, BG-21 | BT-107 Sum of allowances on document level, BT-108 Sum of charges on document level | `/Invoice/cac:LegalMonetaryTotal/cbc:AllowanceTotalAmount`, `/Invoice/cac:LegalMonetaryTotal/cbc:ChargeTotalAmount` | `Σ BT-92`, `Σ BT-99` | BR-CO-11, BR-CO-12 |
| — | BT-109 Invoice total amount without VAT | `/Invoice/cac:LegalMonetaryTotal/cbc:TaxExclusiveAmount` | `BT-106 − BT-107 + BT-108` | BR-CO-13 |
| lines grouped by BT-151 + BT-152 | BT-116 VAT category taxable amount | `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cbc:TaxableAmount` | `Σ BT-131 per category and rate` (± document-level allowances and charges of that category) | BR-S-08, BR-Z-08, BR-E-08, BR-O-08 |
| — | BT-117 VAT category tax amount | `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cbc:TaxAmount` | `round2(BT-116 × BT-119 / 100)`; 0 for E/Z/O | BR-S-09 |
| — | BT-110 Invoice total VAT amount | `/Invoice/cac:TaxTotal/cbc:TaxAmount` | `Σ BT-117` | BR-CO-14 |
| — | BT-112 Invoice total amount with VAT | `/Invoice/cac:LegalMonetaryTotal/cbc:TaxInclusiveAmount` | `BT-109 + BT-110` | BR-CO-15 |
| `payments[i].details.amount` (non-OUTSTANDING, not DEPOSIT) | BT-113 Paid amount | `/Invoice/cac:LegalMonetaryTotal/cbc:PrepaidAmount` | `Σ advances` (2 dp each) | `G/invoice.go:623-636` |
| — | BT-115 Amount due for payment | `/Invoice/cac:LegalMonetaryTotal/cbc:PayableAmount` | `BT-112 − BT-113 + BT-114` | BR-CO-16 |
| `payments[*].details.amount` | BT-146 Item net price (indirect) | `/Invoice/cac:InvoiceLine/cac:Price/cbc:PriceAmount` | line prices nudged by cents until GOBL `totals.total_with_tax` = Σ payments (each rounded to 2 dp), only when all payments share the invoice currency, the drift is pure rounding and no price turns negative | as-built only (`G/line_rounding.go:21-185`); can move BT-131 and so R120/BR-CO-10 |
| `totals.vat.*`, `breakdown[i].*` | BT-109 Invoice total amount without VAT, BT-110 Invoice total VAT amount, BT-112 Invoice total amount with VAT | `/Invoice/cac:LegalMonetaryTotal/cbc:TaxExclusiveAmount`, `/Invoice/cac:TaxTotal/cbc:TaxAmount`, `/Invoice/cac:LegalMonetaryTotal/cbc:TaxInclusiveAmount` | difference UAPI vs calculated | target: sync rejects a difference above the rounding tolerance. As-built: warning log only (`E/invopop.go:753-782`) |

## 2. Supplier Data (Taxpayer, System)

| UAPI | EN 16931 BT | BIS 3.0 UBL XPath | notes |
| --- | --- | --- | --- |
| taxpayer `name.legal` | BT-27 Seller name | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PartyLegalEntity/cbc:RegistrationName` | `supplier.name` (`G/taxpayer.go:62`; `ubl:party.go:207-216`). API docs: BT-44 |
| taxpayer `name.trade` | BT-28 Seller trading name | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PartyName/cbc:Name` | As-built: ignored; BT-28 is a copy of `name.legal` (BT-27). Proposal: GOBL `alias`, which gobl.ubl writes here (`ubl:party.go:280-284`) |
| taxpayer `name.person.*` (INDIVIDUAL) | BT-27 Seller name | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PartyLegalEntity/cbc:RegistrationName` | ignored; `name.legal` used |
| NEW field needed: `seller.identifier` | BT-29 Seller identifier | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PartyIdentification/cbc:ID` | optional (BR-CO-26 is met by BT-31). Value with the ISO 6523 ICD prefix (`0000:`), split into `@schemeID` and value. Today this element carries only BT-90 (`SEPA`) |
| taxpayer `address.line.street` + `.number` | BT-35 Seller address line 1 | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PostalAddress/cbc:StreetName` | taxpayer address, not location (`G/taxpayer.go:51-54`) |
| taxpayer `address.line.suffix` | BT-36 Seller address line 2 | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PostalAddress/cbc:AdditionalStreetName` | — |
| taxpayer `address.line.prefix`, `.infix` | BT-162 Seller address line 3 | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PostalAddress/cac:AddressLine/cbc:Line` | ignored |
| taxpayer `address.city` | BT-37 Seller city | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PostalAddress/cbc:CityName` | — |
| taxpayer `address.code` | BT-38 Seller post code | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PostalAddress/cbc:PostalZone` | — |
| taxpayer `address.region` | BT-39 Seller country subdivision | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PostalAddress/cbc:CountrySubentity` | — |
| taxpayer `address.country` | BT-40 Seller country code | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PostalAddress/cac:Country/cbc:IdentificationCode` | derived: overwritten with the fiscalization country `BE` via the tax ID (`ubl:party.go:250-256`) |
| taxpayer `fiscalization.vat_id_number` (BE) | BT-31 Seller VAT identifier | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PartyTaxScheme[cac:TaxScheme/cbc:ID='VAT']/cbc:CompanyID` | UAPI: 10 digits without prefix; written as `BE` + 10 digits. VAT ID always wins over the tax ID (`G/taxpayer.go:37-45`) |
| taxpayer `fiscalization.tax_id_number` (BE, KBO/BCE) | BT-30 Seller legal registration identifier | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PartyLegalEntity/cbc:CompanyID` (`@schemeID` `0208`) | ignored: used only without VAT ID, and BE requires both (`G/taxpayer.go:37-45`). API docs: BT-32. KBO/BCE is the legal registration (ICD 0208), so BT-30 fits semantically. **UNVERIFIED** as Belgian practice: no Belgian primary source gives field-level guidance, and Peppol 3.0.21 accepts `0208` equally in BT-29, BT-30, BT-34, BT-46, BT-47 and BT-49 (PEPPOL-COMMON-R043). Proposal: `scope: legal` plus extension `iso-scheme-id: 0208` (`ubl:party.go:291-302`), the number left-padded to 10 digits (UAPI allows 9, `^[01]?\d{9}$`; PEPPOL-COMMON-R043 needs 10 plus mod 97); GOBL does not map the BE `BCE` identity type to 0208 (`gobl:addons/eu/en16931/org.go:57-61`), and a bare `BCE` identity would be written as `PartyIdentification/@schemeID="BCE"` (`ubl:party.go:339-344`) |
| taxpayer `fiscalization.tax_id_number` (BE) | BT-32 Seller tax registration identifier | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PartyTaxScheme/cbc:CompanyID` (TaxScheme ≠ `VAT`) | Not needed: Belgium has no tax registration separate from the VAT number |
| NEW field needed: `seller.legal_information` | BT-33 Seller additional legal information | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PartyLegalEntity/cbc:CompanyLegalForm` | optional in BIS 3. UNVERIFIED: whether Belgian company law (CSA art. 2:20, text not held) makes legal form, registered seat and RPM/RPR mentions mandatory on invoices |
| taxpayer `fiscalization.type` = BE | — | — | GOBL regime BE: the supplier needs a tax ID code or a `BCE`-type identity (`gobl:regimes/be/bill_invoices.go:12-36`); fiskaly always sends the VAT ID, so the tax-ID branch applies |
| taxpayer `fiscalization.credentials.type`, `type` | — | — | ignored |
| system annotation `peppol_id` | BT-34 Seller electronic address | `/Invoice/cac:AccountingSupplierParty/cac:Party/cbc:EndpointID` | supplier inbox `peppol`. fiskaly writes `peppol_id` on the system once the Peppol registration workflow (`BE_REGISTER_TAXPAYER_PEPPOL`) confirms proof of ownership (`src/unified/domain/system/einvoicing_adaptations.go:148`, called from `SV1:1197`) (`G/inbox.go:17-35`; `E/service.go:149-179`; `W:284-298`). API docs: BT-49 "of the recipient" (a DE_EI note on the schema shared with `invoicing.identifier`; no BE hint). R020, BR-62. Absent at sync validation (`E/invopop.go:334`) |
| system annotation `peppol_id` (first 4 digits) | BT-34-1 Seller electronic address identification scheme identifier | `/Invoice/cac:AccountingSupplierParty/cac:Party/cbc:EndpointID/@schemeID` | derived EAS code (BE usually `0208` or `9925`); PEPPOL-COMMON-R043 for `0208` |
| `seller.name` | BT-41 Seller contact point | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:Contact/cbc:Name` | `supplier.people[0].name.given` (`G/invoice.go:479-485`). The API docs description ("Seller's legal or trading name") must be changed to "contact person or department" |
| `seller.phone` | BT-42 Seller contact telephone number | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:Contact/cbc:Telephone` | `G/invoice.go:486-489` |
| `seller.email` | BT-43 Seller contact email address | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:Contact/cbc:ElectronicMail` | `G/invoice.go:490-493`; never used as endpoint in this flow (that fallback is XRechnung-only) |
| `tax_representative.name` | BT-62 Seller tax representative name | `/Invoice/cac:TaxRepresentativeParty/cac:PartyName/cbc:Name` | `ordering.seller.name` (`G/invoice.go:387-398`, `499-519`; `ubl:ordering.go:85-89`); gobl.ubl also writes `PartyLegalEntity/cbc:RegistrationName` → UBL-CR-335 warning |
| `tax_representative.vat_number` | BT-63 Seller tax representative VAT identifier | `/Invoice/cac:TaxRepresentativeParty/cac:PartyTaxScheme/cbc:CompanyID` | country + code, TaxScheme `VAT` |
| NEW field needed: `tax_representative.address.line.street` + `tax_representative.address.line.number` | BT-64 Tax representative address line 1 | `/Invoice/cac:TaxRepresentativeParty/cac:PostalAddress/cbc:StreetName` | law (AR n° 1 art. 5 § 1, 2° bis: identity and address of the tax representative, when one is used); BR-19 requires BG-12 |
| NEW field needed: `tax_representative.address.line.suffix` | BT-65 Tax representative address line 2 | `/Invoice/cac:TaxRepresentativeParty/cac:PostalAddress/cbc:AdditionalStreetName` | optional |
| NEW field needed: `tax_representative.address.city` | BT-66 Tax representative city | `/Invoice/cac:TaxRepresentativeParty/cac:PostalAddress/cbc:CityName` | law (AR n° 1 art. 5 § 1, 2° bis) |
| NEW field needed: `tax_representative.address.code` | BT-67 Tax representative post code | `/Invoice/cac:TaxRepresentativeParty/cac:PostalAddress/cbc:PostalZone` | law (AR n° 1 art. 5 § 1, 2° bis) |
| NEW field needed: `tax_representative.address.region` | BT-68 Tax representative country subdivision | `/Invoice/cac:TaxRepresentativeParty/cac:PostalAddress/cbc:CountrySubentity` | optional |
| `tax_representative.country` | BT-69 Tax representative country code | `/Invoice/cac:TaxRepresentativeParty/cac:PostalAddress/cac:Country/cbc:IdentificationCode` | derived: GOBL keeps only the tax-ID country; gobl.ubl writes it as postal country (`ubl:party.go:250-256`), so BT-69 is present (BR-19, BR-20) |
| location `name`, `address.*`, `coordinates.*`, `label` | — | — | ignored |
| system `software.*`, `registrations[]` | — | — | stored registrations gate PEPPOL at sync; systems without annotations or stored registrations skip the check and fail in the worker if `peppol_id` is missing (`EA:145-186`); no XML element |

## 3. Correction (Credit Note)

| UAPI | EN 16931 BT | BIS 3.0 UBL XPath | notes |
| --- | --- | --- | --- |
| `type` = CORRECTION, `data.type` = INVOICE | BT-3 Invoice type code | `/CreditNote/cbc:CreditNoteTypeCode` | derived: GOBL `credit-note` (`W:196-197`) → `CreditNote`, `381` (`ubl:invoice.go:150-156`; `gobl:addons/eu/en16931/scenarios.go:29-36`); P0101 allows 381. No `Invoice` + 381 variant; 384 fails P0112 outside DE–DE/SK–SK. An upward correction is also sent as 381 with positive amounts: it passes validation and silently credits the buyer (no 383 path) |
| `record.id` | BG-3 Preceding Invoice reference | `/CreditNote/cac:BillingReference` | original record's `UsedIn` → original GOBL invoice fetched from Invopop (`W:208-218`; `E/invopop.go:1104-1129`) |
| `record.id` → original `code` | BT-25 Preceding Invoice reference | `/CreditNote/cac:BillingReference/cac:InvoiceDocumentReference/cbc:ID` | derived: code stored by Invopop, i.e. what was sent (follows any PPL override) (`ubl:ordering.go:59-61`). BR-55 |
| `record.id` → original `issue_date` | BT-26 Preceding Invoice issue date | `/CreditNote/cac:BillingReference/cac:InvoiceDocumentReference/cbc:IssueDate` | derived |
| `data.document.references.preceding_document.*` | BT-25 Preceding Invoice reference, BT-26 Preceding Invoice issue date | `/CreditNote/cac:BillingReference/cac:InvoiceDocumentReference/cbc:ID`, `/CreditNote/cac:BillingReference/cac:InvoiceDocumentReference/cbc:IssueDate` | ignored on corrections; BG-3 comes from `record.id` only. Warning log (`G/invoice.go:191-205`; `E/invopop.go:690-707`) |
| `record.offset` | — | — | ignored |
| `details.creators[]`, `details.training`, `details.properties` (CORRECTION level, not `data.details`) | — | — | ignored (record metadata) |
| `reason.text` | BT-22 Invoice note | `/CreditNote/cbc:Note` | GOBL note `key: reason` (`G/invoice.go:215-222`) → subject `ACD` (`gobl:addons/eu/en16931/org.go:28`) → `#ACD#<text>` (`ubl:utils.go:77-86`). The prefix encodes BT-21 Invoice note subject code, which has no element in the BIS 3.0.21 binding; it is the CEN UBL convention and passes BR-CL-08 |
| `reason.type` | BT-22 Invoice note | `/CreditNote/cbc:Note` | default: without `reason.text` the raw enum (e.g. `FACTUAL_ERROR`) is the note (`src/unified/service/einvoicing/correction.go:8-14`). `text` is required for OTHER, so the fallback covers the other types; without `reason` there is no note (`W:205`) |
| `-` | BT-22 Invoice note | `/CreditNote/cbc:Note` | fixed: "TVA à reverser à l'État dans la mesure où elle a été initialement déduite" on every CORRECTION from a BE supplier. UNVERIFIED: the source was not found in AR n° 1 or the AR of 2025-07-08. R002 allows one `cbc:Note`, so it shares the note with `reason.text`. As-built: not emitted |
| `data.payments[i].details.date` (OUTSTANDING) | BT-9 Payment due date | `/CreditNote/cac:PaymentMeans/cbc:PaymentDueDate` | only when an instruction exists (`ubl:payment.go:188-191`), else dropped; CreditNote has no `cbc:DueDate` |
| `data.document.references.project` | BT-11 Project reference | `/CreditNote/cac:AdditionalDocumentReference[cbc:DocumentTypeCode='50']/cbc:ID` | as-built `cac:ProjectReference` (`ubl:ordering.go:124-129`), not defined in `UBL-CreditNote-2.1.xsd` → schema failure. Inferred, not executed; UNVERIFIED for Invopop production |
| `data.document.references.tender` + `.contract` | BT-17 Tender or lot reference, BT-12 Contract reference | `/CreditNote/cac:OriginatorDocumentReference/cbc:ID`, `/CreditNote/cac:ContractDocumentReference/cbc:ID` | The CreditNote schema orders Contract, Additional, Statement, Originator; the as-built order puts Originator first (`ubl:invoice.go:69-75`), so a schema order error follows when both are set. Inferred; UNVERIFIED for Invopop production |
| `data.entries[i]` | BG-25 Invoice line | `/CreditNote/cac:CreditNoteLine` | §1.3; quantity in `cbc:CreditedQuantity` (`ubl:lines.go:241-245`); `entries[i].type` ignored |
| `-` (array index) | BT-126 Invoice line identifier | `/CreditNote/cac:CreditNoteLine/cbc:ID` | derived `i+1`. `data.entries[i].details.number` (the corrected original line, API docs) has no BT in BIS 3 (EN 16931:2017 has no preceding-line reference) |
| amount signs | BT-129 Invoiced quantity, BT-131 Invoice line net amount, BT-146 Item net price, BT-115 Amount due for payment | `/CreditNote/cac:CreditNoteLine/cbc:CreditedQuantity`, `/CreditNote/cac:CreditNoteLine/cbc:LineExtensionAmount`, `/CreditNote/cac:CreditNoteLine/cac:Price/cbc:PriceAmount`, `/CreditNote/cac:LegalMonetaryTotal/cbc:PayableAmount` | as supplied, nothing negated (`G/line_rounding.go:110-113` only). API docs instruct RETURN lines with positive amounts on CORRECTION (guidance only; the amount patterns allow `-`), which matches the BIS credit-note convention. Negative quantities allowed; negative unit price fails BR-27 |
| `data.recipients[]` | BG-7 Buyer | `/CreditNote/cac:AccountingCustomerParty` | must equal the original's (`EA:306-399`) |
| `data.document.number` | BT-1 Invoice number | `/CreditNote/cbc:ID` | credit-note number; PPL caveat (§4); same workflow as the original (`E/invopop.go:398-404`) |

## 4. Fixed and Derived Values

| UAPI | EN 16931 BT | BIS 3.0 UBL XPath | notes |
| --- | --- | --- | --- |
| `-` | BT-24 Specification identifier | `/Invoice/cbc:CustomizationID` | fixed `urn:cen.eu:en16931:2017#compliant#urn:fdc:peppol.eu:2017:poacc:billing:3.0` |
| `-` | BT-23 Business process type | `/Invoice/cbc:ProfileID` | fixed `urn:fdc:peppol.eu:2017:poacc:billing:01:1.0` |
| `type` | BT-3 Invoice type code | `/Invoice/cbc:InvoiceTypeCode`, `/CreditNote/cbc:CreditNoteTypeCode` | derived: INVOICE → `380`, CORRECTION → `381` (`W:192-197`). Never `389`: fiskaly sets no `self-billed` tag, so gobl.ubl never selects Peppol Self-Billing (`ubl:format.go:316-330`) |
| `document.number` | BT-1 Invoice number | `/Invoice/cbc:ID` | UNVERIFIED override. Before signing, the workflow runs `sequence.enumerate` `{name: PPL, prefix: PPL, padding: 6, start: 1}`, summary `Dynamic · PPL · PPL000001` (`WF/BE_SEND_INVOICE_PEPPOL_OASIS_UBL_BUSINESS.json:15-26`). Invopop docs: the step "generates the next number in the sequence and adds it to your document"; leave `code` empty with such a workflow unless your own system owns the numbering. Nothing documents what happens to a pre-set `code`; fiskaly always sets `code`, never `series`. If overwritten, BT-1 = `PPL000001`-style from a workspace series, not the taxpayer's number |
| `-` | BT-2 Invoice issue date | `/Invoice/cbc:IssueDate` | default today when `issued_at` is absent |
| `-` | BT-6 VAT accounting currency code | `/Invoice/cbc:TaxCurrencyCode` | derived `EUR` when BT-5 ≠ EUR (§1.1) |
| `-` | BT-13 Purchase order reference | `/Invoice/cac:OrderReference/cbc:ID` | default `NA` when no BT-10 and no BT-13 (R003) |
| `-` | BT-72 Actual delivery date | `/Invoice/cac:Delivery/cbc:ActualDeliveryDate` | As-built default, a deviation: issue date when no `operation_date`, always emitted. Target: no default; BT-72 only from `recipients[i].shipping.date` (§1.2) |
| `-` | BT-28 Seller trading name, BT-45 Buyer trading name | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PartyName/cbc:Name`, `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PartyName/cbc:Name` | As-built, a deviation: copies of BT-27 / BT-44. Target sources: taxpayer `name.trade` (§2) and NEW `recipients[i].trade_name` (§1.2) |
| `-` | BT-40 Seller country code, BT-55 Buyer country code, BT-69 Tax representative country code | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PostalAddress/cac:Country/cbc:IdentificationCode`, `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PostalAddress/cac:Country/cbc:IdentificationCode`, `/Invoice/cac:TaxRepresentativeParty/cac:PostalAddress/cac:Country/cbc:IdentificationCode` | derived from the tax-ID country (`ubl:party.go:250-256`) |
| `-` | — | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PartyTaxScheme/cac:TaxScheme/cbc:ID`, `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PartyTaxScheme/cac:TaxScheme/cbc:ID`, `/Invoice/cac:InvoiceLine/cac:Item/cac:ClassifiedTaxCategory/cac:TaxScheme/cbc:ID`, `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cac:TaxCategory/cac:TaxScheme/cbc:ID` | tax scheme, no BT: fixed `VAT` for the buyer, default `VAT` for the seller |
| `-` | BT-130 Invoiced quantity unit of measure code | `/Invoice/cac:InvoiceLine/cbc:InvoicedQuantity/@unitCode` | default `C62` |
| `-` | BT-139 Invoice line allowance reason, BT-144 Invoice line charge reason | `/Invoice/cac:InvoiceLine/cac:AllowanceCharge[cbc:ChargeIndicator='false']/cbc:AllowanceChargeReason`, `/Invoice/cac:InvoiceLine/cac:AllowanceCharge[cbc:ChargeIndicator='true']/cbc:AllowanceChargeReason` | As-built fixed `Special Discount` / `Special Surcharge`. Target: NEW `value.discount.reason` / `value.surcharge.reason` (§1.3) |
| `entries[i].data.vat.type`, `.code` | BT-151 Invoiced item VAT category code, BT-118 VAT category code, BT-152 Invoiced item VAT rate, BT-119 VAT category rate | `/Invoice/cac:InvoiceLine/cac:Item/cac:ClassifiedTaxCategory/cbc:ID`, `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cac:TaxCategory/cbc:ID`, `/Invoice/cac:InvoiceLine/cac:Item/cac:ClassifiedTaxCategory/cbc:Percent`, `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cac:TaxCategory/cbc:Percent` | derived per the code table maintained separately. As-built `S`/`Z`/`E`/`O`; Percent `0` for E and Z, omitted for O |
| `payments[i].type`, `.kind`, `.instruction.type` | BT-81 Payment means type code | `/Invoice/cac:PaymentMeans/cbc:PaymentMeansCode` | derived (§1.4). As-built fixed `30` (credit transfer) / `49` (direct debit) |
| `payments[i].type` | BT-82 Payment means text | `/Invoice/cac:PaymentMeans/cbc:PaymentMeansCode/@name` | derived text from the payment type (§1.4) |
| `payments[i].details.discount` | BT-97 Document level allowance reason | `/Invoice/cac:AllowanceCharge[cbc:ChargeIndicator='false']/cbc:AllowanceChargeReason` | derived `Payment discount` |
| `-` | BT-90 Bank assigned creditor identifier | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PartyIdentification/cbc:ID/@schemeID` | fixed `SEPA` (scheme attribute) |
| `-` | BT-126 Invoice line identifier | `/Invoice/cac:InvoiceLine/cbc:ID` | derived: 1-based array index on CORRECTION; as-built on every document |
| `-` | BT-21 Invoice note subject code | — | No element in the BIS 3.0.21 binding. As-built, correction reasons carry `#ACD#` as a prefix inside BT-22 (§3) |
| `-` | BT-22 Invoice note | `/Invoice/cbc:Note` | derived: several notes merged into one (`ubl:peppol.go:17-20`) |
| `-` | BT-34-1 Seller electronic address identification scheme identifier, BT-49-1 Buyer electronic address identification scheme identifier | `/Invoice/cac:AccountingSupplierParty/cac:Party/cbc:EndpointID/@schemeID`, `/Invoice/cac:AccountingCustomerParty/cac:Party/cbc:EndpointID/@schemeID` | derived: first 4 digits of the Peppol identifier |
| `-` | BT-113 Paid amount | `/Invoice/cac:LegalMonetaryTotal/cbc:PrepaidAmount` | derived: sum of non-OUTSTANDING, non-DEPOSIT payments (§1.6) |

## 5. Format Elements Without a UAPI Source (Gaps)

NEW UAPI fields this page needs. Each has its row in §1–§3.

| NEW UAPI field | EN 16931 BT | Required? | section |
| --- | --- | --- | --- |
| `document.invoice_currency` | BT-5 | format-mandatory (BR-05) | 1.1 |
| `document.tax_point_date_code` | BT-8 | optional | 1.1 |
| `document.period.start` | BT-73 | conditional (BR-IC-11) | 1.1 |
| `document.period.end` | BT-74 | conditional (BR-IC-11) | 1.1 |
| `document.references.sales_order` | BT-14 | optional | 1.1 |
| `document.references.receiving_advice` | BT-15 | optional | 1.1 |
| `document.references.invoiced_object` | BT-18 | optional | 1.1 |
| `document.attachments[i].id` | BT-122 | format-mandatory (BR-52, within BG-24) | 1.1 |
| `document.attachments[i].description` | BT-123 | optional | 1.1 |
| `document.attachments[i].uri` | BT-124 | optional | 1.1 |
| `document.attachments[i].content` | BT-125 | optional | 1.1 |
| `document.attachments[i].mime_code` | BT-125-1 | conditional (UBL-DT-06) | 1.1 |
| `document.attachments[i].filename` | BT-125-2 | conditional (UBL-DT-07) | 1.1 |
| `recipients[i].trade_name` | BT-45 | optional | 1.2 |
| `recipients[i].contact.name` | BT-56 | optional | 1.2 |
| `recipients[i].contact.phone` | BT-57 | optional | 1.2 |
| `recipients[i].contact.email` | BT-58 | optional | 1.2 |
| `recipients[i].shipping.location_id` | BT-71 | optional | 1.2 |
| `entries[i].data.unit.price.gross` | BT-148 | optional | 1.3 |
| `entries[i].data.unit.price.discount` | BT-147 | optional | 1.3 |
| `entries[i].data.value.discount.base` | BT-137 | optional | 1.3 |
| `entries[i].data.value.discount.percentage` | BT-138 | optional | 1.3 |
| `entries[i].data.value.discount.reason` | BT-139 | conditional (BR-42) | 1.3 |
| `entries[i].data.value.discount.reason_code` | BT-140 | conditional (BR-42) | 1.3 |
| `entries[i].data.value.surcharge.base` | BT-142 | optional | 1.3 |
| `entries[i].data.value.surcharge.percentage` | BT-143 | optional | 1.3 |
| `entries[i].data.value.surcharge.reason` | BT-144 | conditional (BR-44) | 1.3 |
| `entries[i].data.value.surcharge.reason_code` | BT-145 | conditional (BR-44) | 1.3 |
| `entries[i].data.product.code_scheme` | BT-158-1 | conditional (BR-65) | 1.3 |
| `entries[i].data.product.buyer_number` | BT-156 | optional | 1.3 |
| `entries[i].data.product.origin_country` | BT-159 | optional | 1.3 |
| `entries[i].data.product.attributes[i].name` | BT-160 | law (AR n° 1 art. 5 § 1, 7°, vehicles) | 1.3 |
| `entries[i].data.product.attributes[i].value` | BT-161 | law (AR n° 1 art. 5 § 1, 7°, vehicles) | 1.3 |
| `entries[i].details.object_id` | BT-128 | optional | 1.3 |
| `entries[i].details.order_line_reference` | BT-132 | optional | 1.3 |
| `entries[i].details.buyer_accounting_reference` | BT-133 | optional | 1.3 |
| `entries[i].details.period.start` | BT-134 | optional | 1.3 |
| `entries[i].details.period.end` | BT-135 | optional | 1.3 |
| `payments[i].holder_name` | BT-88 | optional | 1.4 |
| `payments[i].details.discount_vat.type` | BT-95 | format-mandatory (BR-32) | 1.4 |
| `payments[i].details.discount_vat.percentage` | BT-96 | conditional (BR-S-06) | 1.4 |
| `payee.name` | BT-59 | conditional (BR-17) | 1.4 |
| `payee.identifier` | BT-60 | optional | 1.4 |
| `payee.company_id` | BT-61 | optional | 1.4 |
| `totals.vat.amount_accounting` | BT-111 | conditional (BR-53; Art. 230 Dir. 2006/112/EC) | 1.5 |
| `document.allowances[i].amount` | BT-92 | format-mandatory (BR-31, within BG-20) | 1.5 |
| `document.allowances[i].base` | BT-93 | optional | 1.5 |
| `document.allowances[i].percentage` | BT-94 | optional | 1.5 |
| `document.allowances[i].vat_type` | BT-95 | format-mandatory (BR-32, within BG-20) | 1.5 |
| `document.allowances[i].vat_percentage` | BT-96 | conditional (BR-S-06) | 1.5 |
| `document.allowances[i].reason` | BT-97 | conditional (BR-33) | 1.5 |
| `document.allowances[i].reason_code` | BT-98 | conditional (BR-33) | 1.5 |
| `document.charges[i].amount` | BT-99 | format-mandatory (BR-36, within BG-21) | 1.5 |
| `document.charges[i].base` | BT-100 | optional | 1.5 |
| `document.charges[i].percentage` | BT-101 | optional | 1.5 |
| `document.charges[i].vat_type` | BT-102 | format-mandatory (BR-37, within BG-21) | 1.5 |
| `document.charges[i].vat_percentage` | BT-103 | conditional (BR-S-07) | 1.5 |
| `document.charges[i].reason` | BT-104 | conditional (BR-38) | 1.5 |
| `document.charges[i].reason_code` | BT-105 | conditional (BR-38) | 1.5 |
| `totals.rounding` | BT-114 | optional | 1.5 |
| `seller.identifier` | BT-29 | optional | 2 |
| `seller.legal_information` | BT-33 | optional (law UNVERIFIED) | 2 |
| `tax_representative.address.line.street` + `.line.number` | BT-64 | law (AR n° 1 art. 5 § 1, 2° bis) | 2 |
| `tax_representative.address.line.suffix` | BT-65 | optional | 2 |
| `tax_representative.address.city` | BT-66 | law (AR n° 1 art. 5 § 1, 2° bis) | 2 |
| `tax_representative.address.code` | BT-67 | law (AR n° 1 art. 5 § 1, 2° bis) | 2 |
| `tax_representative.address.region` | BT-68 | optional | 2 |

Other gaps (no NEW field; mapping, code-table or pipeline matters):

| BT | element | rule / legal basis | status today | proposal |
| --- | --- | --- | --- | --- |
| BT-3 = `383` (upward correction) | `/Invoice/cbc:InvoiceTypeCode` | AR n° 1 art. 12 § 1, § 3 (a *document rectificatif* referencing the invoice, in either direction); P0100 allows 383; 384 fails P0112 outside DE–DE/SK–SK | `type` gives only 380 / 381; an upward correction becomes a 381 credit note that credits the buyer (§3) | cannot be expressed: BT-3 stays governed by `type` |
| BT-3 = `386` (prepayment invoice) | `/Invoice/cbc:InvoiceTypeCode` | AR n° 1 art. 5 § 1, 5° (date of payment of an advance); P0100 allows 386 | advance invoices go out as 380 | cannot be expressed: BT-3 stays governed by `type` |
| BT-3 = `389` (self-billing) | `/Invoice/cbc:InvoiceTypeCode` | AR n° 1 art. 5 § 1, 9° ter ("Autofacturation"); P0112 allows 389 only DE–DE | no self-billing (§4) | cannot be expressed: BT-3 stays governed by `type` |
| BT-1 | `/Invoice/cbc:ID` | 1..1, BR-02 | from `document.number`; PPL override UNVERIFIED | confirm with Invopop; if it overwrites, drop the step from fiskaly's Peppol workflows |
| BT-10 or BT-13 | `/Invoice/cbc:BuyerReference`, `/Invoice/cac:OrderReference/cbc:ID` | PEPPOL-EN16931-R003 | met by the `NA` filler when the client sends neither | keep; consider requiring `references.buyer` for public-sector receivers |
| BT-34 | `/Invoice/cac:AccountingSupplierParty/cac:Party/cbc:EndpointID` | 1..1, R020, BR-62 | from annotation `peppol_id`; missing at sync, worker error if absent | pass the supplier inbox to sync validation (`E/invopop.go:334`) |
| BT-30 | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PartyLegalEntity/cbc:CompanyID` | BR-CO-26 (met by BT-31) | `tax_id_number` ignored | legal-scope identity with `iso-scheme-id: 0208`, number left-padded to 10 digits; placement in BT-30 UNVERIFIED (§2) |
| BT-120 / BT-121 | `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cac:TaxCategory/cbc:TaxExemptionReason`, `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cac:TaxCategory/cbc:TaxExemptionReasonCode` | BR-E-10, BR-O-10; AR n° 1 art. 5 § 1, 10° | BT-120 from mandatory `vat.reason`; BT-121 never | BE codes → VATEX via the code table maintained separately |
| BT-151 = AE | `/Invoice/cac:InvoiceLine/cac:Item/cac:ClassifiedTaxCategory/cbc:ID` | BR-AE-01…10; AR n° 1 art. 5 § 1, 9° bis; Art. 196 Directive 2006/112/EC | VAT_REVERSE_CHARGE fails the GOBL build, so Belgian domestic reverse-charge supplies and intra-EU B2B services cannot be invoiced | map VAT_REVERSE_CHARGE to `AE` with the text and code from the code table |
| BT-151 = K / G | `/Invoice/cac:InvoiceLine/cac:Item/cac:ClassifiedTaxCategory/cbc:ID` | BR-IC-*, BR-G-*; Art. 138 Directive 2006/112/EC | unreachable: every BE exemption code becomes `E` or `O` | category from the code table. `K` also needs BT-80 (BR-IC-12) and, without the issue-date default, BT-72 or BG-14 (BR-IC-11) |
| BT-151 = O | `/Invoice/cac:InvoiceLine/cac:Item/cac:ClassifiedTaxCategory/cbc:ID` | BR-O-02, BR-O-11 | NOT_SUBJECT → `O` always fails, since BT-31 and BT-48 are always present | category from the code table |
| BT-9 or BT-20 | `/Invoice/cbc:DueDate`, `/Invoice/cac:PaymentTerms/cbc:Note` | BR-CO-25 (EN 16931; not in the 3.0.21 UBL schematrons) | enforced by GOBL en16931 rule 07 inside Invopop, after `COMPLETED`; rule 06 cannot fire, since `detailsFrom` always returns payment details or an error (`G/invoice.go:262-325`) | add `eu-en16931-v2017` at fiskaly time for BE so it fails at the sync check, before anything is sent |
| BT-11 (credit note) | `/CreditNote/cac:AdditionalDocumentReference[cbc:DocumentTypeCode='50']/cbc:ID` | 0..1, UBL 2.1 CreditNote XSD | `cac:ProjectReference` → XSD (inferred) | fix converter or omit on corrections |
| BT-46 / BT-47 | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PartyIdentification/cbc:ID`, `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PartyLegalEntity/cbc:CompanyID` | UBL-SR-16/17 | both in `PartyIdentification`, no `@schemeID` | split the ICD prefix into `@schemeID`; `company_id` with legal scope |
| BG-18 `cbc:NetworkID` | `/Invoice/cac:PaymentMeans/cac:CardAccount/cbc:NetworkID` | 1..1 in UBL/BIS, no BT | card payments become advances, no BG-18 | no UAPI source for the card network; decide before mapping BT-87 |
| BT-149 | `/Invoice/cac:InvoiceLine/cac:Price/cbc:BaseQuantity` | 0..1, R121, R130 | `unit.factor` ignored; no `item.per` in GOBL v0.501.0 | map after GOBL upgrade |
| BT-7, BT-22, BT-25 (prepayment), BT-28, BT-72, BT-83, BT-87, BT-92 (payment discount), BT-126, BT-127, BT-131, BT-136, BT-141, BT-146, BT-154, BT-155, BT-157, BT-158 | various (§1) | — | the UAPI field exists, but the pipeline ignores it or routes it elsewhere | map as in §1 |

## 6. Common Validation Failures

Stages: A API request, B sync checks, C GOBL build, D worker, E Invopop. A failures return an HTTP error. B and C run synchronously inside `POST /records`; a failure there still returns 2xx, leaves the record `COMPLETED` with an error log and sends nothing (`SV1:3628-3700`, `3739-3766`). fiskaly attaches no GOBL addon to the OASIS UBL document type, so no EN 16931 GOBL rules run for Belgium before stage E (`E/invopop.go:316-317`; `G/addon.go:19-30`). EN 16931 and BIS faults therefore arrive by webhook after the record is `COMPLETED` and are logged raw (`E/service.go:662`, `860-884`).

### 6.1 fiskaly-side errors for this flow

| Stage | Message | Trigger | Source |
| --- | --- | --- | --- |
| A (API, 501) | `PEPPOL E_INVOICE_SERVICE is not supported for <cc>` | recipient address not BE/DE | `RH:3330-3336` |
| A (API, 501) | `E_INVOICE_SERVICE is not supported for <cc>` | `invoicing.type` omitted, address not BE/DE/IT. Likely unreachable: `type` is required by the schema and the v6 request validator rejects its absence with 400 (inferred, not executed) | `RH:3320-3327` |
| A (API, 400) | `destination code '' must match the format '^[A-Z0-9]{7}$'` | CONSUMER with PEPPOL or EMAIL invoicing: the block is parsed as SDI without a destination code. `consumer recipients only support SDI invoicing` fires only on unparseable invoicing JSON (code reading, not executed) | `RH:3557-3568`; `pkg/identifier/sdi/destination_code.go:46-53` |
| B (sync) | `no recipient with invoicing configuration found` / `more than one recipient with invoicing configuration found` | no recipient, or several, with `invoicing` | `EA:27-36` |
| B | `TRANSACTION::INVOICE needs exactly 1 business recipient with invoicing …` | the invoicing recipient is BUSINESS and another BUSINESS recipient (without invoicing) is present | `EA:60-62` |
| B | `business recipient identification is not a valid VAT identification` | identification not VAT | `EA:64-67` |
| B | `recipients[0].identification.number: <reason>` | BE/DE VAT format | `EA:71-93` |
| B | `supplier is missing a Peppol identifier` / `supplier is not registered for PEPPOL` | no `peppol_id` and the system has stored registrations; otherwise the error surfaces at stage D | `EA:145-186` |
| B | `invoice correction recipients do not match the original invoice` | correction | `EA:306-399` |
| C (GOBL) | `unsupported VAT type at value_added_tax.type 'REVERSE_CHARGE' …` | VAT_REVERSE_CHARGE | `G/entries.go:164-169` |
| C | `missing VAT exemption reason for code '<c>'` | BE exemption without `reason` | `G/entries.go:239-244` |
| C | `unsupported payment type: MEAL_VOUCHER` / `DEPOSIT` | those types | `G/invoice.go:300-302` |
| C | `payment due date not set` | OUTSTANDING without date | `G/invoice.go:610-613` |
| C | address, VAT-code, inbox-code, tax-representative format errors | invalid input | `G/address.go:23-31`; `G/inbox.go:44-47`; `G/invoice.go:503-510` |
| D (worker) | `supplier has no Peppol identifier yet: proof of ownership is still pending` | no `peppol_id` | `E/service.go:162-168` |
| B, D | `could not resolve referenced invoice reference` | the original's Invopop entry cannot be fetched | `E/invopop.go:645-650` (sync via `EA:388-398`) |
| D | `referenced record is missing used_in identifier` | correction whose original has no Invopop entry (sync skips the fetch) | `W:213-216` |
| D | `e_invoicing failed: TRANSACTION::CORRECTION with receipt correction is not supported` | CORRECTION whose `data` is a receipt | `W:199-202` |
| E (Invopop, record already `COMPLETED`) | the raw GOBL rule text, e.g. `payment terms are required when amount is due (BR-CO-25)` | standard invoice with an amount due, no OUTSTANDING payment and no `document.payment_terms` (GOBL en16931 rule 07; credit notes exempt, `gobl:addons/eu/en16931/bill.go:259-262`) | `gobl:addons/eu/en16931/bill.go:143-151`; logged without translation (`E/service.go:662`, `860-884`) |
| E | `payment means extension is required (BR-49)` | instruction type UNKNOWN | `gobl:addons/eu/en16931/pay.go:40-48` |

### 6.2 BIS 3.0.21 rules most likely to trip

`ubl.generate` runs with `validate_ubl: true`. gobl.ubl v0.99.0 validates Peppol output against VESID `eu.peppol.bis3:invoice:2026.5` / `eu.peppol.bis3:creditnote:2026.5`, the May 2026 release 3.0.21 (`ubl:format.go:302-305`); the validator Invopop runs in production is UNVERIFIED.

| Rule | Flag | Trigger in this flow | Status |
| --- | --- | --- | --- |
| BR-O-02 | fatal | any NOT_SUBJECT line (`O` with BT-31 and BT-48, always present) | certain on such input |
| BR-O-11 | fatal | NOT_SUBJECT mixed with other categories | certain on such input |
| UBL-SR-16 | fatal | `buyer_id` and `company_id` both set | certain on such input |
| BR-27 | fatal | negative `unit.price.inclusive` (refund priced negative instead of negative quantity) | client-dependent |
| PEPPOL-EN16931-R120 | fatal | BT-131 vs qty × BT-146 ± allowances/charges off by > 0.02 after gross→net conversion and nudging | risk |
| PEPPOL-COMMON-R043 | fatal | `0208` endpoint whose value is not 10 digits passing mod 97 | client/onboarding data |
| PEPPOL-EN16931-CL008 | fatal | identifier prefix not in the EAS list (UAPI accepts any 4 digits) | client data |
| BR-CL-23 | fatal | unit code outside Rec 20/21 | low (invalid measures are dropped) |
| XSD (UBL 2.1 CreditNote) | fatal | credit note with `references.project`, or with `tender` and `contract` | inferred, UNVERIFIED in production |
| BR-SBDH-1 / BR-SBDH-2 | fatal | SBDH receiver/sender ≠ `cbc:EndpointID` + `@schemeID` (new in 3.0.21) | Invopop-side |
| R003, R002, R004, R001/R007, P0100/P0101, R010, R020, R061, BR-61 | fatal | prevented (`NA` filler, note merge, fixed values, API/worker checks) | — |
| BR-CO-25 | — | absent from the 3.0.21 schematrons; enforced only by GOBL | Stage E |
| UBL-CR-335, UBL-CR-397 | warning | `PartyLegalEntity` on tax representative / deliver-to party | — |

National rules: the 3.0.21 PEPPOL schematron has blocks for DE, DK, GR, IS, IT, NL, NO and SE, **none for Belgium**; the only Belgium-specific check is PEPPOL-COMMON-R043 (ICD `0208`). The German block (`german-rules`, DE-R-001 … DE-R-031) fires only when supplier **and** customer addresses are DE, so BE → DE invoices are unaffected.

## Sources

| Artefact | Version / path / URL | Retrieved |
| --- | --- | --- |
| Peppol BIS Billing 3.0 syntax binding | OpenPEPPOL/peppol-bis-invoice-3 tag `3.0.21`: `structure/syntax/ubl-invoice.xml`, `ubl-creditnote.xml`, `structure/syntax/part/*.xml` — https://github.com/OpenPEPPOL/peppol-bis-invoice-3/tree/3.0.21 | 2026-11-03; re-fetched 2026-11-16 and 2026-11-25 (BT ↔ element check) |
| Peppol BIS guide appendices and code lists | same tag: `guide/bis/appendix/national_rules.adoc`, `guide/cius/rules/peppol_rules.adoc`, `structure/codelist/eas.xml`, `structure/codelist/icd.xml` | 2026-11-03 |
| Peppol BIS schematrons | `PEPPOL-EN16931-UBL.sch`, `CEN-EN16931-UBL.sch` (CEN schematron 1.3.16, last update 2026-04-10), `PeppolBIS-Envelope-Rules.sch` — formats repo `Peppol BIS/peppol-bis-invoice-3-3.0.21/rules/sch/` (tag 3.0.21) | 2026-10-29 |
| Peppol BIS release status | "May 2026 Release" = 3.0.21, mandatory from 2026-08-17 — https://docs.peppol.eu/poacc/billing/3.0/release-notes/ (regulatory baseline row verified 2026-10-29) | 2026-10-29 |
| UBL 2.1 schemas | OASIS UBL 2.1 `UBL-Invoice-2.1.xsd`, `UBL-CreditNote-2.1.xsd` (local copy: XRechnung 3.0.2 validator configuration 2026-08-31, `resources/ubl/2.1/xsd/maindoc/`) | 2026-11-03 |
| VAT Directive | Council Directive 2006/112/EC, Art. 138, 196, 226(7), 230 — https://eur-lex.europa.eu/eli/dir/2006/112/oj | 2026-11-03 |
| Belgian invoicing law | AR n° 1 du 29 décembre 1992 (consolidated, Justel), art. 5 § 1 (1°, 2° bis, 3°, 5°, 7°, 8°, 9°, 9° bis, 9° ter, 10° to 10° quater, 11°) and art. 12 § 1, § 3 — formats repo `Belgium/Laws/03_AR_n1_1992_Facturation_TVA_consolide.md` | 2026-10-29 |
| Belgian mandate context | regulatory baseline, Belgium section: B2B Peppol mandate in force since 2026-01-01 (CTVA art. 53 §2bis; loi du 6 février 2024; AR n° 1 art. 13ter; AR du 8 juillet 2025; loi du 10 février 2026) | verified 2026-10-29 |
| UAPI v6 field list | [UAPI_v6_Fields.md](UAPI_v6_Fields.md) from `fiskaly/meta@cd19ec88cb299ee41a44302448006340dd028bf0` `src/unified/api/v6/definition.yaml`; API docs `2026-06-01` | 2026-11-03 |
| fiskaly as-built code | `fiskaly/meta@cd19ec8` (paths in the code-citation legend above; workflows `BE_SEND_INVOICE_PEPPOL_OASIS_UBL_BUSINESS.json`, `BE_REGISTER_TAXPAYER_PEPPOL.json`, `DE_SEND_INVOICE_PEPPOL_XRECHNUNG_BUSINESS.json`) | 2026-11-03 |
| GOBL | invopop/gobl v0.501.0, commit `225e39d` (fiskaly's pin); v0.511.0 for converter-side code | 2026-11-03 |
| GOBL → UBL converter | invopop/gobl.ubl v0.99.0, commit `540f201` (2026-10-28), incl. `test/data/convert/peppol/out/*.xml` | 2026-11-03 |
| Invopop documentation (as-built behaviour only, not compliance) | https://docs.invopop.com/guides/series.md, https://docs.invopop.com/console/series.md, https://docs.invopop.com/llms, https://docs.invopop.com/guides/peppol.md | 2026-11-03 |
| As-built trace | internal working note "UAPI v6 → GOBL bill.Invoice: as-built trace", spot-checked against the code | 2026-11-03 |
