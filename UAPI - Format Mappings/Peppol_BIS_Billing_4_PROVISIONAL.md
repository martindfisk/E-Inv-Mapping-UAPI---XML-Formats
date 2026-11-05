# UAPI v6 → Peppol BIS Billing 4.0 (PROVISIONAL)

As of 2026-11-04 · Status: provisional draft for review

OpenPeppol has not published Peppol BIS Billing 4.0 (its tags end at 3.0.21), so every row on this page is provisional. Terms, cardinalities and rules come from KoSIT's non-normative *Vorversion XRechnung 4.0.0* (2026-09-15), which stands in for EN 16931-1:2026; the CEN text and its UBL binding CEN/TS 16931-3-2:2026 are published but paywalled and not held. No fiskaly flow produces BIS 4.0 yet: today a BE taxpayer whose record has one BUSINESS recipient with `invoicing.type` `PEPPOL`, sent from an e-invoice system with a `peppol_id`, gets BIS 3 (`doc_type` `bis-invoice-v3`, see [Peppol_BIS_Billing_3.md](Peppol_BIS_Billing_3.md)), and the "today via BIS 3" notes describe that output. For terms carried over from 2017 the XPath column holds the UBL 2.1 path of the XRechnung 3.0.2 model (`ubl-inv`; credit notes in §3), assumed unchanged; for terms new in 2026 it reads "not obtained", and a "candidate" in the notes is a UBL 2.1 element inferred by the author. UAPI paths are relative to `content.operation` (§2: to the taxpayer or system object). Each note starts with `prov.`, then `NEW in 2026`, `CHANGED vs 2017` or `unchanged`, then the `README.md` vocabulary; quoted cardinalities are XRechnung CIUS cardinalities.

Code citations: `G/` = fiskaly `src/unified/einvoicing/gobl/`; `E/` = fiskaly `src/unified/einvoicing/`; `RH` = fiskaly `src/unified/api/v6/record_handler.go`; `EA` = fiskaly `src/unified/domain/record/einvoicing_adaptations.go`; `SV1` = fiskaly `src/unified/domain/record/service_v1.go`; `W` = fiskaly `src/unified/worker/e_invoicing_worker.go`; `WF/` = fiskaly `cmd/invopop-workflow-migration/workflows/` (all at `cd19ec8`); `gobl:` = invopop/gobl v0.501.0; `ubl:` = invopop/gobl.ubl v0.99.0.

## 1. Invoice Mapping

### 1.1 Header and References

| UAPI | EN 16931-1:2026 BT | BIS 4.0 XPath (provisional) | notes |
| --- | --- | --- | --- |
| `document.number` | BT-1 Invoice number | `/Invoice/cbc:ID` | prov. · unchanged · effect of the workflow's `sequence.enumerate` `PPL` step (`WF/BE_SEND_INVOICE_PEPPOL_OASIS_UBL_BUSINESS.json:15-26`) UNVERIFIED |
| `document.series` | — | — | prov. · ignored; put the series into BT-1 |
| `document.issued_at` (date) | BT-2 Invoice issue date | `/Invoice/cbc:IssueDate` | prov. · unchanged · default today (`G/invoice.go:706-722`) |
| `document.issued_at` (time) | BT-166 Invoice issue time | not obtained (CEN/TS 16931-3-2:2026) | prov. · NEW in 2026 · candidate `/Invoice/cbc:IssueTime` · today via BIS 3: ignored (`ubl:invoice.go:50` declared, never set) |
| `-` | BT-3 Invoice type code | `/Invoice/cbc:InvoiceTypeCode` | prov. · unchanged · fixed `380` |
| `payments[i].details.currency` (first payment) | BT-5 Invoice currency code | `/Invoice/cbc:DocumentCurrencyCode` | prov. · unchanged · derived from the first payment (`G/invoice.go:725-741`); default `EUR` (`src/unified/domain/record/transaction/receipt/payment/details.go:174`); for BE an explicit non-EUR currency fails record validation (`SV1:1968`; `src/unified/domain/record/capabilities.go:186-225`) |
| `document.operation_date` | BT-7 Value added tax point date | `/Invoice/cbc:TaxPointDate` | prov. · unchanged · today via BIS 3: lands in BT-72 instead (`G/invoice.go:433-441`); `cbc:TaxPointDate` comes only from GOBL `value_date` (`ubl:invoice.go:158-161`) |
| `document.text` | BT-22 Invoice note | `/Invoice/cbc:Note` | prov. · unchanged · today ignored |
| `document.references.buyer` | BT-10 Buyer reference | `/Invoice/cbc:BuyerReference` | prov. · CHANGED vs 2017 (repeatable, optional UNTDID 1153 scheme) · UAPI: one value, ≤ 32 chars |
| `document.references.project` | BT-11 Project reference | `/Invoice/cac:ProjectReference/cbc:ID` | prov. · unchanged · ignored for DE suppliers only (`G/invoice.go:337-354`) · today via BIS 3: credit notes get the same `cac:ProjectReference` (`ubl:ordering.go:124-129`), which the UBL 2.1 CreditNote XSD lacks (§3); from code and XSD, not executed |
| `document.references.contract` | BT-12 Contract reference | `/Invoice/cac:ContractDocumentReference/cbc:ID` | prov. · unchanged |
| `document.references.purchase_order` | BT-13 Purchase order reference | `/Invoice/cac:OrderReference/cbc:ID` | prov. · unchanged · BR-CO-28: not together with BT-188 |
| `document.references.despatch_advice` | BT-16 Despatch advice reference | `/Invoice/cac:DespatchDocumentReference/cbc:ID` | prov. · unchanged · BR-CO-29: not together with BT-189 |
| `document.references.tender` | BT-17 Tender or lot reference | `/Invoice/cac:OriginatorDocumentReference/cbc:ID` | prov. · unchanged |
| `document.references.buyer_routing` | BT-19 Buyer accounting reference | `/Invoice/cbc:AccountingCost` | prov. · unchanged |
| `document.payment_terms` | BT-20 Payment term text (BG-33 PAYMENT TERMS) | `/Invoice/cac:PaymentTerms/cbc:Note` | prov. · CHANGED vs 2017: now inside BG-33 (0..*); 2026 binding not obtained, 2017 path shown |
| `document.references.preceding_document.number` | BT-25 Preceding Invoice reference | `/Invoice/cac:BillingReference/cac:InvoiceDocumentReference/cbc:ID` | prov. · unchanged · invoices only; ignored on credit notes (`G/invoice.go:191-205`) |
| `document.references.preceding_document.issued_at` | BT-26 Preceding Invoice issue date | `/Invoice/cac:BillingReference/cac:InvoiceDocumentReference/cbc:IssueDate` | prov. · unchanged |
| `-` | BT-202 Preceding Invoice type code | not obtained (CEN/TS 16931-3-2:2026) | prov. · NEW in 2026 · gap · candidate `/Invoice/cac:BillingReference/cac:InvoiceDocumentReference/cbc:DocumentTypeCode` (gobl.ubl writes it when the reference has `untdid-document-type`, `ubl:ordering.go:53-72`) |
| `document.activity_code`, `document.offline.*`, `details.*` | — | — | prov. · ignored (record metadata) |

### 1.2 Buyer

Only `recipients[i].type: BUSINESS` with `invoicing.type: PEPPOL` applies; a consumer recipient cannot carry PEPPOL invoicing (`RH:3557-3571`), and one without `invoicing` is ignored (`G/invoice.go:400-418`).

| UAPI | EN 16931-1:2026 BT | BIS 4.0 XPath (provisional) | notes |
| --- | --- | --- | --- |
| `recipients[i].name` | BT-44 Buyer name | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PartyLegalEntity/cbc:RegistrationName` | prov. · unchanged |
| `recipients[i].buyer_id` | BT-46 Buyer Identifier | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PartyIdentification/cbc:ID` | prov. · CHANGED vs 2017: 0..*, `@schemeID` mandatory (BR-71) · gap: no ICD · today via BIS 3: written without a scheme (`G/recipient.go:158-165`; `ubl:party.go:322-350`) |
| `recipients[i].company_id` | BT-47 Buyer legal registration Identifier | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PartyLegalEntity/cbc:CompanyID` | prov. · CHANGED vs 2017: scheme mandatory (BR-72) · today via BIS 3: ignored as BT-47; written as a second BT-46 without a scheme, because the identity has no legal scope (`G/recipient.go:166-172`; `ubl:party.go:286-306`). With `buyer_id` also set this fails UBL-SR-16 on BIS 3; under the 2026 model BT-46 is 0..*, so it fails only on BR-71 |
| `recipients[i].identification.number` (`type: VAT`) | BT-48 Buyer VAT Identifier | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PartyTaxScheme/cbc:CompanyID` | prov. · unchanged · prefix from the address country (`G/identification.go:15-42`); other identification types fail record validation (`EA:64-67`) |
| `recipients[i].invoicing.identifier` | BT-49 Buyer electronic address | `/Invoice/cac:AccountingCustomerParty/cac:Party/cbc:EndpointID` | prov. · unchanged · `<scheme>:<id>` → `@schemeID` + value (`gobl:org/inbox.go:90-115`); mandatory (BR-DE-38 in the Vorversion, scheme BR-63; BIS 3: PEPPOL-EN16931-R010) |
| `recipients[i].address.line.street`, `.number` | BT-50 Buyer address line 1 | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PostalAddress/cbc:StreetName` | prov. · unchanged |
| `recipients[i].address.line.suffix` | BT-51 Buyer address line 2 | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PostalAddress/cbc:AdditionalStreetName` | prov. · unchanged |
| `recipients[i].address.line.prefix`, `.infix` | BT-163 Buyer address line 3 | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PostalAddress/cac:AddressLine/cbc:Line` | prov. · unchanged · today ignored |
| `recipients[i].address.city` | BT-52 Buyer city | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PostalAddress/cbc:CityName` | prov. · unchanged |
| `recipients[i].address.code` | BT-53 Buyer post code | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PostalAddress/cbc:PostalZone` | prov. · unchanged |
| `recipients[i].address.region` | BT-54 Buyer country subdivision | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PostalAddress/cbc:CountrySubentity` | prov. · unchanged |
| `recipients[i].address.country` | BT-55 Buyer country code | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PostalAddress/cac:Country/cbc:IdentificationCode` | prov. · unchanged |
| `recipients[i].shipping.name` | BT-70 Deliver to party name | `/Invoice/cac:Delivery/cac:DeliveryParty/cac:PartyName/cbc:Name` | prov. · unchanged |
| `recipients[i].shipping.date` | BT-72 Actual delivery date | `/Invoice/cac:Delivery/cbc:ActualDeliveryDate` | prov. · unchanged · today via BIS 3: ignored; BT-72 is the operation date, else the issue date (`G/invoice.go:433-441`) |
| `recipients[i].shipping.address.line.street`, `.number` | BT-75 Deliver to address line 1 | `/Invoice/cac:Delivery/cac:DeliveryLocation/cac:Address/cbc:StreetName` | prov. · unchanged |
| `recipients[i].shipping.address.line.suffix` | BT-76 Deliver to address line 2 | `/Invoice/cac:Delivery/cac:DeliveryLocation/cac:Address/cbc:AdditionalStreetName` | prov. · unchanged |
| `recipients[i].shipping.address.city` | BT-77 Deliver to city | `/Invoice/cac:Delivery/cac:DeliveryLocation/cac:Address/cbc:CityName` | prov. · unchanged |
| `recipients[i].shipping.address.code` | BT-78 Deliver to post code | `/Invoice/cac:Delivery/cac:DeliveryLocation/cac:Address/cbc:PostalZone` | prov. · unchanged |
| `recipients[i].shipping.address.region` | BT-79 Deliver to country subdivision | `/Invoice/cac:Delivery/cac:DeliveryLocation/cac:Address/cbc:CountrySubentity` | prov. · unchanged |
| `recipients[i].shipping.address.country` | BT-80 Deliver to country code | `/Invoice/cac:Delivery/cac:DeliveryLocation/cac:Address/cac:Country/cbc:IdentificationCode` | prov. · unchanged · shipping stays at header level (BG-13); BR-CO-40 forbids BG-37 alongside |
| `recipients[i].invoicing.email`, `.format`, `.destination_code`, `.pec`, `recipients[i].origin` | — | — | prov. · ignored (e-mail and SDI fields) |

### 1.3 Lines

There is one `cac:InvoiceLine` per `entries[i]` (`G/entries.go:39-85`). A `GROUP` entry yields one line from its group values (`RH:4034-4070`); its `data.items[]` are parsed (`RH:4072-4080`) and then ignored (`G/entries.go:299-307`). The 2026 model has no sub-lines.

| UAPI | EN 16931-1:2026 BT | BIS 4.0 XPath (provisional) | notes |
| --- | --- | --- | --- |
| `entries[i]` | BG-25 INVOICE LINE | `/Invoice/cac:InvoiceLine` | prov. · unchanged · `entries[i].type` ignored; signs as supplied |
| `entries[i].details.number` | BT-126 Invoice line Identifier | `/Invoice/cac:InvoiceLine/cbc:ID` | prov. · unchanged · today via BIS 3: ignored; derived `i+1` (`G/entries.go:77`) |
| `entries[i].details.description` | BT-127 Invoice line note | `/Invoice/cac:InvoiceLine/cbc:Note` | prov. · unchanged · today ignored |
| `entries[i].data.unit.quantity` | BT-129 Invoiced quantity | `/Invoice/cac:InvoiceLine/cbc:InvoicedQuantity` | prov. · unchanged |
| `entries[i].data.unit.measure` | BT-130 Invoiced quantity unit of measure code | `/Invoice/cac:InvoiceLine/cbc:InvoicedQuantity/@unitCode` | prov. · unchanged · default `C62` (`G/entries.go:309-323`) |
| `entries[i].data.value.base.exclusive`, `.discount.exclusive`, `.surcharge.exclusive` | BT-131 Invoice line net amount | `/Invoice/cac:InvoiceLine/cbc:LineExtensionAmount` | prov. · unchanged · derived: base − discount + surcharge (BR-CO-32) · today via BIS 3: GOBL recomputes the amount; the `.exclusive` values are ignored |
| `entries[i].data.value.discount.exclusive` | BT-136 Invoice line allowance amount | `/Invoice/cac:InvoiceLine/cac:AllowanceCharge[cbc:ChargeIndicator = 'false']/cbc:Amount` | prov. · unchanged · today via BIS 3: taken from `.discount.inclusive`, 2 dp (`G/entries.go:45-55`); reason BT-139 fixed `Special Discount` |
| `entries[i].data.value.surcharge.exclusive` | BT-141 Invoice line charge or tax amount | `/Invoice/cac:InvoiceLine/cac:AllowanceCharge[cbc:ChargeIndicator = 'true']/cbc:Amount` | prov. · CHANGED vs 2017 (renamed; BT-193 added) · today via BIS 3: from `.surcharge.inclusive` (`G/entries.go:57-67`); reason BT-144 fixed `Special Surcharge` |
| `entries[i].data.unit.price.exclusive` | BT-146 Item net price | `/Invoice/cac:InvoiceLine/cac:Price/cbc:PriceAmount` | prov. · unchanged · today via BIS 3: ignored; the net price is derived from `price.inclusive` (`G/entries.go:300`; `ubl:ubl.go:161`) |
| `entries[i].data.unit.price.inclusive` | — | — | prov. · no EN 16931 term (BT-148 is the pre-discount price, not VAT-inclusive); today the GOBL price source, derived into BT-146 (§4) |
| `entries[i].data.unit.factor` | BT-149 Item price base quantity | `/Invoice/cac:InvoiceLine/cac:Price/cbc:BaseQuantity` | prov. · unchanged · today ignored |
| `entries[i].data.vat.type: VAT_RATE`, `.percentage` | BT-151 Invoiced item VAT category code, BT-152 Invoiced item VAT rate | `/Invoice/cac:InvoiceLine/cac:Item/cac:ClassifiedTaxCategory/cbc:ID`, `/Invoice/cac:InvoiceLine/cac:Item/cac:ClassifiedTaxCategory/cbc:Percent` | prov. · unchanged · derived `S` (> 0) or `Z` (`G/entries.go:101-122`); rate `code` ignored |
| `entries[i].data.vat.code` (`type: VAT_EXEMPTION`) | BT-151 Invoiced item VAT category code; BT-195 Invoiced item VAT exemption reason and specification code | `/Invoice/cac:InvoiceLine/cac:Item/cac:ClassifiedTaxCategory/cbc:ID`; BT-195 not obtained (CEN/TS 16931-3-2:2026) | prov. · NEW in 2026 (BT-195) · candidate `/Invoice/cac:InvoiceLine/cac:Item/cac:ClassifiedTaxCategory/cbc:TaxExemptionReasonCode` · today via BIS 3: BE → `E`/`O` without VATEX (`G/entries.go:148-189`) |
| `entries[i].data.vat.reason` | BT-194 Invoiced item Exemption reason text | not obtained (CEN/TS 16931-3-2:2026) | prov. · NEW in 2026 · candidate `/Invoice/cac:InvoiceLine/cac:Item/cac:ClassifiedTaxCategory/cbc:TaxExemptionReason` · today via BIS 3: goes to BT-120 (`G/entries.go:203-269`); mandatory for BE |
| `entries[i].data.vat.type: VAT_REVERSE_CHARGE` | BT-151 Invoiced item VAT category code = `AE` | `/Invoice/cac:InvoiceLine/cac:Item/cac:ClassifiedTaxCategory/cbc:ID` | prov. · unchanged · gap: today the GOBL mapping fails (`G/entries.go:164-169`) |
| `entries[i].data.vat.regime`, `.regime_base` | BT-151 Invoiced item VAT category code = `D`/`F`/`I`/`J`/`N`; BT-116 VAT category taxable amount | — | prov. · NEW in 2026 (categories D, F, I, J for margin schemes, N for national schemes) · ignored: "not applicable" for BE/DE/IT in the API docs |
| `entries[i].data.vat.amount`, `.exclusive`, `.inclusive`, `.fiscal_location`, `.surcharge.*` | — | — | prov. · ignored (no line-level term) |
| `entries[i].data.text` | BT-153 Item name | `/Invoice/cac:InvoiceLine/cac:Item/cbc:Name` | prov. · unchanged |
| `entries[i].data.product.details.label` | BT-154 Item description | `/Invoice/cac:InvoiceLine/cac:Item/cbc:Description` | prov. · unchanged · today ignored |
| `entries[i].data.product.number` (`SKU`, `MPN`, `OTHER`) | BT-155 Item Seller's Identifier | `/Invoice/cac:InvoiceLine/cac:Item/cac:SellersItemIdentification/cbc:ID` | prov. · unchanged · today ignored |
| `entries[i].data.product.number` (`GTIN-8/12/13/14`) | BT-157 Item standard Identifier | `/Invoice/cac:InvoiceLine/cac:Item/cac:StandardItemIdentification/cbc:ID` | prov. · unchanged · derived `@schemeID` `0160` (BR-64) · today ignored |
| `entries[i].data.product.code` | BT-158 Item classification Identifier | `/Invoice/cac:InvoiceLine/cac:Item/cac:CommodityClassification/cbc:ItemClassificationCode` | prov. · unchanged · gap: `@listID` mandatory (BR-65) · today ignored |
| `entries[i].data.product.details.name`, `.brand`, `.model`, `.serial`, `.batch`, `.date_of_*` | BT-160 Item attribute name, BT-161 Item attribute value (BG-32 ITEM ATTRIBUTE) | `/Invoice/cac:InvoiceLine/cac:Item/cac:AdditionalItemProperty/cbc:Name`, `/Invoice/cac:InvoiceLine/cac:Item/cac:AdditionalItemProperty/cbc:Value` | prov. · CHANGED vs 2017: BT-160 0..1, BT-211 alternative · today ignored |
| `entries[i].details.concept` | BT-196 Goods/services code | not obtained (CEN/TS 16931-3-2:2026) | prov. · NEW in 2026 · derived; code values UNVERIFIED |
| `entries[i].details.prepayment.document_number` (`RETURN`) | BT-217 Line-level preceding invoice reference, BT-219 Line-level preceding invoice type code = `386` (BG-39) | not obtained (CEN/TS 16931-3-2:2026) | prov. · NEW in 2026 · candidate `/Invoice/cac:InvoiceLine/cac:BillingReference/cac:InvoiceDocumentReference/cbc:ID` · today ignored |
| `entries[i].details.purpose`, `.regulatory`, `.label`, `.activity_code`, `.properties`, `.voucher.*`, `.applied_vouchers[i].*`; `entries[i].data.items[i].*` | — | — | prov. · ignored (no EN 16931 term) |

### 1.4 Payment

| UAPI | EN 16931-1:2026 BT | BIS 4.0 XPath (provisional) | notes |
| --- | --- | --- | --- |
| `payments[i].details.date` (first `OUTSTANDING`) | BT-9 Payment due date | `/Invoice/cbc:DueDate` | prov. · unchanged (single, at root) · later OUTSTANDING payments ignored (`G/invoice.go:276-289`); BG-33 has no per-term date, so instalments stay in BT-20 |
| `payments[i].instruction.type` | BT-81 Payment means type code | `/Invoice/cac:PaymentMeans/cbc:PaymentMeansCode` | prov. · unchanged · derived: `CREDIT_TRANSFER` → `30`, `DIRECT_DEBIT` → `49` (`gobl:addons/eu/en16931/pay.go:12-38`) · today via BIS 3: `UNKNOWN` → GOBL `other` (`G/invoice.go:576`), which has no UNTDID 4461 code, so the export fails BR-49 (`gobl:addons/eu/en16931/pay.go:40-47`; `ubl:payment.go:143-151`) at Invopop, after the record is `COMPLETED` (§6); code `1` is set only on the XRechnung route and only when there is no instruction at all (`G/xrechnung.go:42-46`), so `UNKNOWN` fails there too |
| `payments[i].instruction.text` | BT-83 Remittance information | `/Invoice/cac:PaymentMeans/cbc:PaymentID` | prov. · unchanged · today via BIS 3: lands in BT-82 `@name` (`G/invoice.go:538-540`; `ubl:payment.go:161`) |
| `payments[i].instruction.properties` | — | — | prov. · no EN 16931 term in the API docs · today via BIS 3: written to `/Invoice/cac:PaymentMeans/cbc:PaymentID`, the BT-83 element (`G/invoice.go:541-543`; `ubl:payment.go:158`) |
| `payments[i].name` (`ONLINE`, `OTHER`) | BT-82 Payment means text | `/Invoice/cac:PaymentMeans/cbc:PaymentMeansCode/@name` | prov. · unchanged · today ignored |
| `payments[i].instruction.account` | BT-84 Payment account Identifier | `/Invoice/cac:PaymentMeans/cac:PayeeFinancialAccount/cbc:ID` | prov. · unchanged · BR-76 makes it mandatory on intra-community invoices |
| `payments[i].instruction.name` | BT-85 Payment account name | `/Invoice/cac:PaymentMeans/cac:PayeeFinancialAccount/cbc:Name` | prov. · unchanged |
| `payments[i].instruction.payment_service_provider` | BT-86 Payment service provider Identifier | `/Invoice/cac:PaymentMeans/cac:PayeeFinancialAccount/cac:FinancialInstitutionBranch/cbc:ID` | prov. · unchanged |
| `payments[i].instruction.mandate_reference` | BT-89 Mandate reference Identifier | `/Invoice/cac:PaymentMeans/cac:PaymentMandate/cbc:ID` | prov. · unchanged |
| `payments[i].instruction.creditor_id.identifier` | BT-90 Bank assigned creditor Identifier | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PartyIdentification/cbc:ID[@schemeID = 'SEPA']`, `/Invoice/cac:PayeeParty/cac:PartyIdentification/cbc:ID[@schemeID = 'SEPA']` | prov. · unchanged · today via BIS 3: there is no payee, so the seller path (`ubl:payment.go:123-136`) |
| `payments[i].instruction.debited_account` | BT-91 Debited account Identifier | `/Invoice/cac:PaymentMeans/cac:PaymentMandate/cac:PayerFinancialAccount/cbc:ID` | prov. · unchanged |
| `payments[i].number` (`CARD`) | BT-87 Payment card primary account number | `/Invoice/cac:PaymentMeans/cac:CardAccount/cbc:PrimaryAccountNumberID` | prov. · unchanged · today ignored (CARD is a paid advance); BR-51 allows only the last 4–6 digits |
| `payments[i].kind` (`CARD`) | BT-81 Payment means type code | `/Invoice/cac:PaymentMeans/cbc:PaymentMeansCode` | prov. · unchanged · today ignored; would need a derived code (`48`/`54`/`55`) |
| `payments[i].details.amount` (`CARD`, `CASH`, `ONLINE`, `OTHER`, vouchers) | BT-113 Paid amount | `/Invoice/cac:LegalMonetaryTotal/cbc:PrepaidAmount` | prov. · unchanged · derived sum of advances (`ubl:totals.go:78`); a `MEAL_VOUCHER` or `DEPOSIT` payment makes the GOBL mapping fail (`G/invoice.go:300-302`) |
| `payments[i].document_number` (`DEPOSIT`) | BT-25 Preceding Invoice reference, BT-202 Preceding Invoice type code = `386` | `/Invoice/cac:BillingReference/cac:InvoiceDocumentReference/cbc:ID` | prov. · NEW in 2026 (BT-202) · gap: would reference the deposit invoice; today a `DEPOSIT` payment makes the GOBL mapping fail (`G/invoice.go:300-302`) |
| `payments[i].details.discount` | BT-172 Discount amount (BG-35 EARLY PAYMENT DISCOUNT) | not obtained (CEN/TS 16931-3-2:2026) | prov. · NEW in 2026 · UNVERIFIED fit (UAPI semantics undocumented) · today ignored |
| `payments[i].details.channel`, `.properties`, `payments[i].concept`, `.count` | — | — | prov. · ignored |

### 1.5 VAT Breakdown and Totals

GOBL recalculates the breakdown and totals from the lines. `totals.vat.*` is ignored apart from a comparison whose mismatch only logs a warning (`E/invopop.go:444`, `753-785`); `breakdown[]` is ignored.

| UAPI | EN 16931-1:2026 BT | BIS 4.0 XPath (provisional) | notes |
| --- | --- | --- | --- |
| `breakdown[i].exclusive` | BT-116 VAT category taxable amount | `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cbc:TaxableAmount` | prov. · unchanged · ignored; derived per the BR-S-8 split (§1.6) |
| `breakdown[i].amount` | BT-117 VAT category tax amount | `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cbc:TaxAmount` | prov. · CHANGED vs 2017: 0..1 (BR-46: margin schemes excepted) · ignored; derived |
| `breakdown[i].type`, `.code` | BT-118 VAT category code | `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cac:TaxCategory/cbc:ID` | prov. · unchanged · ignored; derived from the lines |
| `breakdown[i].percentage` | BT-119 VAT category rate | `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cac:TaxCategory/cbc:Percent` | prov. · CHANGED vs 2017: 0..1 (BR-48) · ignored; derived |
| `breakdown[i].code` (`VAT_EXEMPTION`) | BT-121 VAT exemption reason and specification code | `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cac:TaxCategory/cbc:TaxExemptionReasonCode` | prov. · CHANGED vs 2017 (renamed) · ignored · today via BIS 3: BE none; DE VATEX table (`G/german_exemption.go:25-59`) |
| `entries[i].data.vat.reason` | BT-120 VAT exemption reason text | `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cac:TaxCategory/cbc:TaxExemptionReason` | prov. · unchanged · derived: distinct reasons per GOBL VAT key joined by `; ` (`G/entries.go:257-265`); copied per rate (`ubl:totals.go:122-126`) |
| `entries[i].details.concept` | BT-210 VAT breakdown goods/services code | not obtained (CEN/TS 16931-3-2:2026) | prov. · NEW in 2026 · derived; one BG-23 per value |
| `totals.vat.exclusive` | BT-109 Invoice total amount without VAT (= BT-106 Sum of Invoice line net amount) | `/Invoice/cac:LegalMonetaryTotal/cbc:TaxExclusiveAmount` | prov. · unchanged · ignored: GOBL recalculates; a mismatch logs a warning |
| `totals.vat.amount` | BT-110 Invoice total VAT amount | `/Invoice/cac:TaxTotal/cbc:TaxAmount[@currencyID = /Invoice/cbc:DocumentCurrencyCode]` | prov. · unchanged · ignored: GOBL recalculates; a mismatch logs a warning |
| `totals.vat.inclusive` | BT-112 Invoice total amount with VAT | `/Invoice/cac:LegalMonetaryTotal/cbc:TaxInclusiveAmount` | prov. · unchanged · ignored: GOBL recalculates; a mismatch logs a warning |
| `-` | BT-115 Amount due for payment | `/Invoice/cac:LegalMonetaryTotal/cbc:PayableAmount` | prov. · CHANGED vs 2017 (formula) · derived |
| `-` | BT-114 Rounding amount | `/Invoice/cac:LegalMonetaryTotal/cbc:PayableRoundingAmount` | prov. · unchanged · gap: prices are nudged instead (`G/line_rounding.go:21-159`); the converter's `RoundToCurrency()` can still write ±0.01 (`ubl:ubl.go:164`) |
| `-` | BG-34 CHARGES ON BEHALF OF A THIRD PARTY (BT-179, BT-180), BT-184 VAT breakdown currency, BT-167 VAT accounting currency exchange rate | not obtained (CEN/TS 16931-3-2:2026) | prov. · NEW in 2026 · gap (BG-34); BT-167 and BT-184 are not needed while BE is EUR-only (`src/unified/domain/record/capabilities.go:186-225`) |

### 1.6 Calculated Fields

| UAPI | EN 16931-1:2026 BT | BIS 4.0 XPath (provisional) | formula | notes |
| --- | --- | --- | --- | --- |
| `-` | BT-131 Invoice line net amount | `/Invoice/cac:InvoiceLine/cbc:LineExtensionAmount` | BT-129 × (BT-146 ÷ BT-149) + ΣBT-141 − ΣBT-136, tolerance 0.02 (EUR) | prov. · NEW core rule BR-CO-32 (was PEPPOL-EN16931-R120) · today GOBL rounds with `rounding: currency` (`G/invoice.go:165-168`) |
| `-` | BT-106 Sum of Invoice line net amount | `/Invoice/cac:LegalMonetaryTotal/cbc:LineExtensionAmount` | ΣBT-131 | prov. · unchanged (BR-CO-10) |
| `-` | BT-109 Invoice total amount without VAT | `/Invoice/cac:LegalMonetaryTotal/cbc:TaxExclusiveAmount` | BT-106 − BT-107 + BT-108 | prov. · unchanged (BR-CO-13); BT-107/BT-108 always absent |
| `-` | BT-116 VAT category taxable amount | `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cbc:TaxableAmount` | ΣBT-131 + ΣBT-99 − ΣBT-92 per category, rate, exemption code/text and BT-210 | prov. · CHANGED vs 2017 (BR-S-8 split) |
| `-` | BT-117 VAT category tax amount | `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cbc:TaxAmount` | BT-116 × BT-119 ÷ 100 | prov. · unchanged (BR-S-9); BR-CO-17 not listed (UNVERIFIED) |
| `-` | BT-110 Invoice total VAT amount, BT-112 Invoice total amount with VAT | `/Invoice/cac:TaxTotal/cbc:TaxAmount[@currencyID = /Invoice/cbc:DocumentCurrencyCode]`, `/Invoice/cac:LegalMonetaryTotal/cbc:TaxInclusiveAmount` | ΣBT-117; BT-109 + BT-110 | prov. · unchanged (BR-CO-14, BR-CO-15) |
| `-` | BT-115 Amount due for payment | `/Invoice/cac:LegalMonetaryTotal/cbc:PayableAmount` | BT-112 − BT-113 + ΣBT-179 + BT-114 | prov. · CHANGED vs 2017 (BR-CO-16 adds BT-179) |
| `-` | BT-172 Discount amount | not obtained (CEN/TS 16931-3-2:2026) | BT-171 × BT-112 | prov. · NEW in 2026 (BR-CO-36) |

## 2. Supplier Data (Taxpayer, System)

**Taxpayer** (`POST/PATCH /taxpayers`; paths relative to the taxpayer object). The mapping lives in `G/taxpayer.go:25-81`.

| UAPI | EN 16931-1:2026 BT | BIS 4.0 XPath (provisional) | notes |
| --- | --- | --- | --- |
| `name.legal` | BT-27 Seller name | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PartyLegalEntity/cbc:RegistrationName` | prov. · unchanged · the API docs wrongly label it BT-44 |
| `name.trade` | BT-28 Seller trading name | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PartyName/cbc:Name` | prov. · unchanged · today ignored |
| `fiscalization.vat_id_number` | BT-31 Seller VAT Identifier | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PartyTaxScheme[cac:TaxScheme/cbc:ID = 'VAT']/cbc:CompanyID` | prov. · unchanged · derived country prefix (BR-CO-9; `ubl:party.go:229-230`) |
| `fiscalization.tax_id_number` (BE, KBO/BCE) | BT-30 Seller legal registration Identifier (ICD `0208`) | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PartyLegalEntity/cbc:CompanyID` | prov. · CHANGED vs 2017 (scheme mandatory, BR-70) · the API docs say BT-32; which one BIS 4.0 expects is UNVERIFIED · today ignored: the VAT ID wins (`G/taxpayer.go:34-45`), and `vat_id_number` is required for BE |
| `fiscalization.tax_id_number` (DE) | BT-32 Seller tax registration Identifier | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PartyTaxScheme[cac:TaxScheme/cbc:ID != 'VAT']/cbc:CompanyID` | prov. · unchanged · holds the personal Steuer-ID, not the Steuernummer; ignored while a VAT ID exists (`G/taxpayer.go:34-45`); DE suppliers use XRechnung |
| `address.line.street`, `.number` | BT-35 Seller address line 1 | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PostalAddress/cbc:StreetName` | prov. · unchanged · taxpayer address, not location |
| `address.line.suffix` | BT-36 Seller address line 2 | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PostalAddress/cbc:AdditionalStreetName` | prov. · unchanged |
| `address.line.prefix`, `.infix` | BT-162 Seller address line 3 | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PostalAddress/cac:AddressLine/cbc:Line` | prov. · unchanged · today ignored |
| `address.city` | BT-37 Seller city | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PostalAddress/cbc:CityName` | prov. · unchanged |
| `address.code` | BT-38 Seller post code | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PostalAddress/cbc:PostalZone` | prov. · unchanged |
| `address.region` | BT-39 Seller country subdivision | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PostalAddress/cbc:CountrySubentity` | prov. · unchanged |
| `address.country` | BT-40 Seller country code | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PostalAddress/cac:Country/cbc:IdentificationCode` | prov. · unchanged |

**Seller contact and tax representative** (invoice payload, relative to `content.operation`):

| UAPI | EN 16931-1:2026 BT | BIS 4.0 XPath (provisional) | notes |
| --- | --- | --- | --- |
| `seller.name` | BT-41 Seller contact point | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:Contact/cbc:Name` | prov. · unchanged |
| `seller.phone` | BT-42 Seller contact telephone number | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:Contact/cbc:Telephone` | prov. · unchanged |
| `seller.email` | BT-43 Seller contact email address | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:Contact/cbc:ElectronicMail` | prov. · unchanged |
| `tax_representative.name` | BT-62 Seller tax representative name | `/Invoice/cac:TaxRepresentativeParty/cac:PartyName/cbc:Name` | prov. · unchanged |
| `tax_representative.vat_number` | BT-63 Seller tax representative VAT Identifier | `/Invoice/cac:TaxRepresentativeParty/cac:PartyTaxScheme/cbc:CompanyID` | prov. · unchanged |
| `tax_representative.country` | BT-69 Tax representative country code | `/Invoice/cac:TaxRepresentativeParty/cac:PostalAddress/cac:Country/cbc:IdentificationCode` | prov. · unchanged · today via BIS 3: used as the tax-ID country; the address is ignored (`G/invoice.go:503-518`); BR-19/BR-20 need BG-12 |

**E-invoice system** (system annotation):

| UAPI | EN 16931-1:2026 BT | BIS 4.0 XPath (provisional) | notes |
| --- | --- | --- | --- |
| `peppol_id` (annotation) | BT-34 Seller electronic address | `/Invoice/cac:AccountingSupplierParty/cac:Party/cbc:EndpointID` | prov. · unchanged · `@schemeID` from the prefix (`G/inbox.go:17-35`); mandatory (BR-DE-37 in the Vorversion; BIS 3: PEPPOL-EN16931-R020). The API docs wrongly call it BT-49 |

## 3. Correction (Credit Note)

`CORRECTION` with `data.type: INVOICE` becomes a GOBL `credit-note`, then a UBL `CreditNote` (`W:196-197`; `ubl:invoice.go:150-155`), always with type code `381` and with the amounts as supplied (not negated). The `data.*` paths map as in §1, with these 2017 substitutions:

| 2017 Invoice path | 2017 CreditNote path | notes |
| --- | --- | --- |
| `/Invoice`, `/Invoice/cbc:InvoiceTypeCode` | `/CreditNote`, `/CreditNote/cbc:CreditNoteTypeCode` | prov. · 2026 binding not obtained |
| `/Invoice/cac:InvoiceLine`, `/Invoice/cac:InvoiceLine/cbc:InvoicedQuantity` | `/CreditNote/cac:CreditNoteLine`, `/CreditNote/cac:CreditNoteLine/cbc:CreditedQuantity` | prov. · 2026 binding not obtained |
| `/Invoice/cbc:DueDate` (BT-9) | `/CreditNote/cac:PaymentMeans/cbc:PaymentDueDate` | prov. · today via BIS 3: written only inside `cac:PaymentMeans`, so BT-9 is lost when there is no payment instruction (`ubl:payment.go:188-191`, `233-234`); silent, since GOBL checks BR-CO-25 on standard invoices only (`gobl:addons/eu/en16931/bill.go:143-151`) |
| `/Invoice/cac:ProjectReference/cbc:ID` (BT-11) | `/CreditNote/cac:AdditionalDocumentReference/cbc:ID` (BIS 3: with `cbc:DocumentTypeCode` `50`) | prov. · today via BIS 3: written as `cac:ProjectReference` (`ubl:ordering.go:124-129`), which the UBL 2.1 CreditNote XSD lacks, so the Invopop XSD check (`validate_ubl: true`, `WF/BE_SEND_INVOICE_PEPPOL_OASIS_UBL_BUSINESS.json:40`) fails after the record is `COMPLETED`; from code and XSD, not executed |

Whether BIS 4.0 keeps a separate `CreditNote` root is UNVERIFIED; the title of CEN/TS 16931-3-2:2026 still names UBL "invoice and credit note".

| UAPI | EN 16931-1:2026 BT | BIS 4.0 XPath (provisional) | notes |
| --- | --- | --- | --- |
| `-` | BT-3 Invoice type code | `/CreditNote/cbc:CreditNoteTypeCode` | prov. · unchanged · fixed `381`, upward corrections too (see the BT-112 row) |
| `record.id` | BT-25 Preceding Invoice reference | `/CreditNote/cac:BillingReference/cac:InvoiceDocumentReference/cbc:ID` | prov. · unchanged · derived from the Invopop silo entry (`W:208-218`; `E/invopop.go:1101-1129`) |
| `record.id` | BT-26 Preceding Invoice issue date | `/CreditNote/cac:BillingReference/cac:InvoiceDocumentReference/cbc:IssueDate` | prov. · unchanged · derived |
| `record.id` | BT-202 Preceding Invoice type code | not obtained (CEN/TS 16931-3-2:2026) | prov. · NEW in 2026 · derived `380` · candidate `/CreditNote/cac:BillingReference/cac:InvoiceDocumentReference/cbc:DocumentTypeCode` · today via BIS 3: absent, because the reference carries no `untdid-document-type` (`E/invopop.go:1113-1121`; `ubl:ordering.go:65-67`) |
| `record.id` | BT-217 Line-level preceding invoice reference (BG-39) | not obtained (CEN/TS 16931-3-2:2026) | prov. · NEW in 2026 · derived; optional when BG-3 is present |
| `data.entries[i].details.number` | BT-220 Line-level preceding invoice line reference | not obtained (CEN/TS 16931-3-2:2026) | prov. · NEW in 2026 · UAPI: "the original entry number being corrected" · candidate `/CreditNote/cac:CreditNoteLine/cac:BillingReference/cac:BillingReferenceLine/cbc:ID` · today ignored |
| `reason.type`, `reason.text` | BT-22 Invoice note, BT-21 Invoice note subject code | `/CreditNote/cbc:Note` | prov. · unchanged · no correction-reason term in 2026 · today via BIS 3: note `key: reason` (`G/invoice.go:215-222`), text, else the raw enum (`src/unified/service/einvoicing/correction.go:8-14`, called at `W:204-206`); subject `ACD` (`gobl:addons/eu/en16931/org.go:67-77`) |
| `data.document.references.preceding_document.*`, `record.offset` | — | — | prov. · ignored; a warning is logged for `preceding_document` only (`E/invopop.go:690-707`); `record.offset` is parsed (`RH:4950-4959`) and then ignored |
| `data.*` amounts | BT-112 Invoice total amount with VAT | `/CreditNote/cac:LegalMonetaryTotal/cbc:TaxInclusiveAmount` | prov. · NEW rule BR-CO-33 (BT-112 of a credit note not negative) · amounts go out unnegated: a downward correction with positive amounts passes; an upward correction with positive amounts also passes and silently credits the buyer; a correction sent with negative amounts fails BR-CO-33 · gap: `383` / `384` (§5) |

## 4. Fixed and Derived Values

| UAPI | EN 16931-1:2026 BT | BIS 4.0 XPath (provisional) | notes |
| --- | --- | --- | --- |
| `-` | BT-24 Specification identifier | `/Invoice/cbc:CustomizationID` | prov. · fixed; BIS 4.0 value UNVERIFIED. The 2026 prefix is `urn:cen.eu:en16931:2026` (Vorversion BT-24 note); BIS 3 sends `urn:cen.eu:en16931:2017#compliant#urn:fdc:peppol.eu:2017:poacc:billing:3.0` (`ubl:format.go:299`) |
| `-` | BT-23 Business process type | `/Invoice/cbc:ProfileID` | prov. · fixed; BIS 4.0 value UNVERIFIED (`ubl:format.go:16` has the BIS 3 value) |
| `-` | BT-3 Invoice type code | `/Invoice/cbc:InvoiceTypeCode`, `/CreditNote/cbc:CreditNoteTypeCode` | prov. · fixed `380` / `381` |
| `entries[i]` (index) | BT-126 Invoice line Identifier | `/Invoice/cac:InvoiceLine/cbc:ID` | prov. · derived `i+1` (`G/entries.go:77`) |
| `entries[i].data.unit.measure` | BT-130 Invoiced quantity unit of measure code | `/Invoice/cac:InvoiceLine/cbc:InvoicedQuantity/@unitCode` | prov. · default `C62` |
| `-` | BT-139 Invoice line allowance reason, BT-144 Invoice line charge or tax reason | `/Invoice/cac:InvoiceLine/cac:AllowanceCharge[cbc:ChargeIndicator = 'false']/cbc:AllowanceChargeReason`, `/Invoice/cac:InvoiceLine/cac:AllowanceCharge[cbc:ChargeIndicator = 'true']/cbc:AllowanceChargeReason` | prov. · fixed `Special Discount` / `Special Surcharge` (`G/entries.go:51`, `63`) |
| `entries[i].data.vat.*` | BT-151 Invoiced item VAT category code, BT-118 VAT category code, BT-121 VAT exemption reason and specification code | `/Invoice/cac:InvoiceLine/cac:Item/cac:ClassifiedTaxCategory/cbc:ID`, `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cac:TaxCategory/cbc:ID`, `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cac:TaxCategory/cbc:TaxExemptionReasonCode` | prov. · derived categories; VATEX only for DE |
| `payments[i].instruction.type` | BT-81 Payment means type code | `/Invoice/cac:PaymentMeans/cbc:PaymentMeansCode` | prov. · derived `30` / `49`; `UNKNOWN` makes the export fail BR-49 (§1.4) |
| `document.operation_date`, `document.issued_at` | BT-72 Actual delivery date | `/Invoice/cac:Delivery/cbc:ActualDeliveryDate` | prov. · derived: operation date, else issue date (`G/invoice.go:433-441`) |
| `entries[i].data.unit.price.inclusive` | BT-146 Item net price | `/Invoice/cac:InvoiceLine/cac:Price/cbc:PriceAmount` | prov. · derived gross → net; prices adjusted by cents to the payment total (`G/line_rounding.go:21-159`) |

## 5. Format Elements Without a UAPI Source (Gaps)

**A. Terms new in 2026** (UBL element not obtained for all: CEN/TS 16931-3-2:2026)

| BT | element | cardinality / rule | status today | proposal |
| --- | --- | --- | --- | --- |
| BT-166 Invoice issue time | not obtained | 0..1 | prov. · source exists, ignored | Emit once bound |
| BT-167 VAT accounting currency exchange rate | not obtained | 0..1 | prov. · no source; not needed while BE is EUR-only | — |
| BT-197 Delivery note reference | not obtained | 0..1; BR-CO-45 | prov. · no source | Add `references.delivery_note` |
| BT-202 Preceding Invoice type code | not obtained | 0..1 | prov. · derivable for corrections | Derive `380`; add `preceding_document.type` |
| BG-33 PAYMENT TERMS | not obtained | 0..* | prov. · one text field | BT-20 in the first BG-33 |
| BG-35 EARLY PAYMENT DISCOUNT (BT-170..BT-172) | not obtained | 0..*; BR-CO-36 | prov. · no source | Add `{end_date, percentage, amount}` |
| BG-36 LATE PAYMENT PENALTY (BT-181..BT-183) | not obtained | 0..* | prov. · no source | On demand |
| BG-34 CHARGES ON BEHALF OF A THIRD PARTY (BT-179, BT-180) | not obtained | 0..*; BR-CO-16 | prov. · no source | Add `third_party_charges[]` |
| BT-194 Invoiced item Exemption reason text, BT-195 Invoiced item VAT exemption reason and specification code | not obtained | 0..1; BR-68, BR-S-8 | prov. · source exists; mapped to BT-120/BT-121 only | Map per line |
| BT-196 Goods/services code, BT-210 VAT breakdown goods/services code | not obtained | 0..1; BR-S-8 | prov. · derivable (`details.concept`) | Derive; confirm codes |
| BT-188..BT-192, BT-198..BT-201 Invoice line references | not obtained | 0..1; BR-CO-28..30, 42..46 | prov. · header-only | Add per-entry references |
| BG-37 INVOICE LINE DELIVERY INFORMATION, BG-38 INVOICE LINE DELIVER TO ADDRESS | not obtained | 0..1; BR-CO-40 | prov. · header-only | Per-entry shipping |
| BG-39 LINE-LEVEL PRECEDING INVOICE REFERENCE (BT-217..BT-220) | not obtained | 0..*; BT-217 1 | prov. · sources exist; ignored | Map for corrections, prepayments |
| BT-177 Document level non VAT tax code, BT-193 Invoice line tax type code | not obtained | 0..1; BR-CO-34/35 | prov. · no source | If needed |
| BT-211 / BT-212, BT-215 / BT-216, BT-173..BT-176, BT-213 / BT-214, BT-184 | not obtained | 0..1 | prov. · no source | Optional |

**B. 2017 terms UAPI cannot express (or that changed)**

| BT | element | cardinality / rule | status today | proposal |
| --- | --- | --- | --- | --- |
| BT-10 Buyer reference | `/Invoice/cbc:BuyerReference` | XRechnung 1..*, SPFE 0..n | prov. · one value, ≤ 32 chars | Array + scheme; check Leitweg-ID length |
| BT-46 Buyer Identifier, BT-47 Buyer legal registration Identifier (schemes) | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PartyIdentification/cbc:ID/@schemeID`, `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PartyLegalEntity/cbc:CompanyID/@schemeID` | BR-71, BR-72 | prov. · no ICD | Add ICD fields; BT-47 to `PartyLegalEntity` |
| BT-29 Seller Identifier, BT-30 Seller legal registration Identifier | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PartyIdentification/cbc:ID`, `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PartyLegalEntity/cbc:CompanyID` | BR-69, BR-70, BR-CO-26 | prov. · BE enterprise number ignored | Map with ICD `0208` |
| BT-157 Item standard Identifier, BT-158 Item classification Identifier (schemes) | `/Invoice/cac:InvoiceLine/cac:Item/cac:StandardItemIdentification/cbc:ID/@schemeID`, `/Invoice/cac:InvoiceLine/cac:Item/cac:CommodityClassification/cbc:ItemClassificationCode/@listID` | BR-64, BR-65 | prov. · ignored | Derive `0160`; add list ID |
| BG-12 SELLER TAX REPRESENTATIVE POSTAL ADDRESS (BT-64..BT-68) | `/Invoice/cac:TaxRepresentativeParty/cac:PostalAddress` | BR-19, BR-20 | prov. · country only; address ignored | Add the address |
| BG-14 INVOICING PERIOD, BG-26 INVOICE LINE PERIOD | `/Invoice/cac:InvoicePeriod`, `/Invoice/cac:InvoiceLine/cac:InvoicePeriod` | BR-CO-19/20 | prov. · no source. BG-26 now sits inside BG-37 (§7.1); its path here is the 2017 one | Add for services |
| BG-20, BG-21, BG-24, BG-9, BG-10, BT-14, BT-15, BT-18, BT-45, BT-88, BT-128, BT-132, BT-133, BT-147, BT-148 | `/Invoice/cac:AllowanceCharge[cbc:ChargeIndicator = 'false']`, `/Invoice/cac:AllowanceCharge[cbc:ChargeIndicator = 'true']`, `/Invoice/cac:AdditionalDocumentReference`, `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:Contact`, `/Invoice/cac:PayeeParty`, `/Invoice/cac:OrderReference/cbc:SalesOrderID`, `/Invoice/cac:ReceiptDocumentReference/cbc:ID`, `/Invoice/cac:AdditionalDocumentReference[cbc:DocumentTypeCode = '130']/cbc:ID`, `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PartyName/cbc:Name`, `/Invoice/cac:PaymentMeans/cac:CardAccount/cbc:HolderName`, `/Invoice/cac:InvoiceLine/cac:DocumentReference[cbc:DocumentTypeCode = '130']/cbc:ID`, `/Invoice/cac:InvoiceLine/cac:OrderLineReference/cbc:LineID`, `/Invoice/cac:InvoiceLine/cbc:AccountingCost`, `/Invoice/cac:InvoiceLine/cac:Price/cac:AllowanceCharge[cbc:ChargeIndicator = 'false']/cbc:Amount`, `/Invoice/cac:InvoiceLine/cac:Price/cac:AllowanceCharge[cbc:ChargeIndicator = 'false']/cbc:BaseAmount` | 0..1 / 0..* | prov. · no source | Optional |
| BT-114 Rounding amount | `/Invoice/cac:LegalMonetaryTotal/cbc:PayableRoundingAmount` | 0..1 | prov. · prices nudged instead; the converter can still write ±0.01 | Consider |

**C. Mandatory without a UAPI source**

| BT | element | cardinality / rule | status today | proposal |
| --- | --- | --- | --- | --- |
| BT-23 Business process type, BT-24 Specification identifier | `/Invoice/cbc:ProfileID`, `/Invoice/cbc:CustomizationID` | 1 (BR-1, BR-DE-36; BIS 3: PEPPOL-EN16931-R001) | prov. · fixed | BIS 4.0 values needed |
| BT-3 Invoice type code | `/Invoice/cbc:InvoiceTypeCode` | 1 | prov. · `380` / `381` | Add `383` / `384` for upward corrections (BR-CO-33); BIS 3.0.22 allows `384` only for DE and SK, so a BE supplier would need `383` |
| BT-126 Invoice line Identifier, BT-130 Invoiced quantity unit of measure code | `/Invoice/cac:InvoiceLine/cbc:ID`, `/Invoice/cac:InvoiceLine/cbc:InvoicedQuantity/@unitCode` | 1 | prov. · derived / default | Use `details.number` |
| BT-151 Invoiced item VAT category code `AE` | `/Invoice/cac:InvoiceLine/cac:Item/cac:ClassifiedTaxCategory/cbc:ID` | 1 | prov. · reverse charge unsupported | Support it |
| BT-34 Seller electronic address | `/Invoice/cac:AccountingSupplierParty/cac:Party/cbc:EndpointID` | 1 (BR-DE-37; BIS 3: PEPPOL-EN16931-R020) | prov. · from the annotation | — |
| BT-84 Payment account Identifier | `/Invoice/cac:PaymentMeans/cac:PayeeFinancialAccount/cbc:ID` | BR-76 (intra-community invoices; not credit notes) | prov. · only with `CREDIT_TRANSFER` | Require a payment account (BT-84: IBAN or national account number) for `K` invoices |

## 6. Common Validation Failures

All provisional. Rule IDs come from Vorversion chapter 4; BIS 4.0 rules and the CEN 2026 validation artefacts are unpublished. "Today" means the BIS 3 flow, where EN 16931 rules first run at Invopop: fiskaly sets no GOBL addon for BE (`E/invopop.go:300-319`; `G/addon.go:19-30`), so such a fault arrives after the record is `COMPLETED`.

| Rule area | Rule | Trigger in fiskaly data | Status |
| --- | --- | --- | --- |
| Payment means | BR-49 | `instruction.type: UNKNOWN` → GOBL `other`, no UNTDID 4461 code (§1.4) | prov. · fails today at Invopop; rule unchanged in 2026 |
| CreditNote schema | UBL 2.1 CreditNote XSD | `document.references.project` on a correction → `cac:ProjectReference` (§3) | prov. · fails today at the Invopop XSD check; from code and XSD, not executed |
| Line net amount | BR-CO-32 | Net price derived from the gross price; prices nudged by cents | prov. · now core |
| Amount due | BR-CO-16, BR-CO-25 | Amount due > 0 without an OUTSTANDING due date or `payment_terms` (advances that do not cover the total); GOBL checks standard invoices only (`gobl:addons/eu/en16931/bill.go:143-151`) | prov. |
| Credit-note sign | BR-CO-33 | Correction sent with negative amounts as `381`. An upward correction with positive amounts passes and silently credits the buyer (§3) | prov. · new |
| Negative lines | BR-67 | RETURN lines with a negative price on an invoice | prov. · new |
| Exemption texts | BR-68, BR-S-8, BR-CO-18 | Different `vat.reason` per VATEX code; DE `keepSharedVATEX` (`G/german_exemption.go:95-120`) | prov. · new |
| Identifier schemes | BR-69..BR-75, BR-64, BR-65 | `buyer_id` / `company_id` without `@schemeID` | prov. · likely failure |
| Intra-community | BR-76 | `K` invoice without a payment account (BT-84) | prov. · new |
| Header vs line | BR-CO-28..30, 40, 42..46 | None today | prov. |
| VAT categories | BR-S/Z/E/AE/IC/G/O/IG/IP-*, BR-N-2..4 | Reverse charge unsupported. BE `E`/`O` need a reason | prov. · BR-O-2/3/4, BR-O-11..14, BR-S-10, BR-Z-10, BR-IG-10, BR-IP-10, BR-CO-17 not listed |
| Not subject to VAT | — (BR-O-2/3/4 not listed) | `NOT_SUBJECT` → `O` while BT-31 / BT-48 are sent | prov. · README's BR-O-02 finding does not carry over: Vorversion §4.4.7 gives the seller and buyer VAT IDs "in some situations" and allows `O` next to taxed supplies |
| Peppol layer | BIS 3: PEPPOL-EN16931-R001/R004/R007, R010/R020, PEPPOL-COMMON-R043/R062, R140..R276 (3.0.22) | Wrong identifiers; empty elements | prov. · BIS 4.0 rules unpublished |
| Vorversion errata | BR-O-1 says "equal to 'Export outside the EU'"; BT-218 typed `Time`; BT-24 `#compliant#` vs `#conformant#`; BR-CO-30 names BT-180 for the line-level receiving advice (BR-CO-43 has BT-191); BR-CO-28 lists BT-177 in its element column (its text has BT-188); Vorversion §4.4.7 gives the line VAT rate as 0, while BR-O-5 forbids BT-152 | — | prov. · await the final text |

## 7. Changes 2017 → 2026 and Implementation Gaps

### 7.1 What changes from 2017 to 2026

All rows are provisional. The comparison is between the XRechnung 3.0.2 CIUS model and Vorversion chapters 3 and 4, with CIUS cardinalities on both sides. Terms: 216 BT and 39 BG against 164 BT and 32 BG; 52 BTs and 7 BGs are new and none is removed; 8 terms changed structure or cardinality, 16 were only renamed and 12 only changed their data type name. Rules: 39 rule IDs are new (31 core, 8 BR-DE); 20 XRechnung 3.0.2 rule IDs are not listed (12 core, 8 BR-DE), which is UNVERIFIED as a normative removal.

| Area | 2017 | 2026 | Effect for fiskaly |
| --- | --- | --- | --- |
| Standards | EN 16931-1:2017+A1:2019; CEN/TS 16931-3-2:2020 (UBL 2.1); CEN validation artefacts 1.3.x (BIS 3.0.22: 1.3.16) | EN 16931-1:2026 (EVS valid from 2026-05-04) and CEN/TS 16931-3-2:2026 (valid from 2026-08-17), both paywalled and not held; CEN 2026 validation artefacts and BIS 4.0 unpublished | XPaths for new terms not obtained (§5A) |
| BT-10 Buyer reference | 1 (XRechnung), text | 1..* (XRechnung; SPFE 0..n), identifier with optional UNTDID 1153 scheme | One value, no scheme |
| Party identifiers | Scheme optional | Scheme mandatory for BT-29, 30, 46, 47, 60, 61, 71 (BR-69..BR-75); BT-46 now 0..* | `buyer_id`, `company_id` lack an ICD |
| BG-16 PAYMENT INSTRUCTIONS | 1 (XRechnung) | 0..1 | None |
| Payment terms | BT-20 at root | BG-33 (0..*) holds BT-20, BG-35 early payment discount (BT-170..BT-172) and BG-36 late payment penalty (BT-181..BT-183); BT-9 stays at root | BT-20 moves; no discount or penalty source |
| Third-party charges | — | BG-34 (BT-179, BT-180); BR-CO-16: BT-115 = BT-112 − BT-113 + ΣBT-179 + BT-114 | No source |
| Line VAT | BT-151, BT-152 | + BT-194 exemption text, BT-195 VATEX per line; BR-68; BR-S-8 splits BG-23 by exemption and BT-210 | UAPI has both per entry |
| VAT categories | S, Z, E, AE, K, G, O, L, M | + D, F, I, J (margin schemes) and N (national schemes), where the taxable amount differs from the line net amount; BT-117, BT-119 → 0..1 (BR-46, BR-48) | `vat.regime` unused for BE/DE/IT |
| Not subject to VAT (`O`) | BR-O-2/3/4 forbid seller and buyer VAT IDs; BR-O-11..14 forbid other categories alongside | Not listed. Vorversion §4.4.7: the seller and buyer VAT IDs are given "in some situations"; `O` may be combined with supplies in scope of VAT | README's `NOT_SUBJECT` → BR-O-02 finding does not carry over (§6) |
| Line references | BT-132 | BT-188..BT-192, BT-198..BT-201; header or line, not both (BR-CO-28..30, 40, 42..46) | Header-only today |
| Line delivery | BG-26 directly under BG-25 | BG-37 (BT-185..BT-187), BG-38 address (BT-203..BT-209); BG-26 moved into BG-37 | Header-only shipping; the §5B BG-26 path is the 2017 one |
| Preceding invoices | BG-3 (BT-25, BT-26) | + BT-202 type code; BG-39 line level (BT-217..BT-220) | Corrections and prepayments become expressible |
| Header | — | BT-166 issue time, BT-167 exchange rate, BT-197 delivery note reference | `issued_at` carries the time |
| Goods/services | — | BT-196 item, BT-210 breakdown, BT-213/BT-214 allowance/charge | `details.concept` |
| Other taxes | BG-21/BG-28 "charges" | "charges and taxes"; BT-177, BT-193 (UNTDID 5153) | No source |
| Item attributes | BT-160 1 | BT-160 0..1, + BT-211 code, BT-212 unit (BR-54) | Product details could use it |
| Direct debit | BT-89..BT-91 | + BT-215 debtor PSP, BT-216 debited account name | No source |
| Core rules | R120 only in Peppol | BR-CO-32 line net, BR-CO-33 credit-note BT-112 not negative, BR-67 negative lines, BR-76 BT-84 on intra-community invoices | §6 |

### 7.2 Implementation gaps for BIS 4.0

All rows are provisional. Nothing in the pipeline targets BIS 4.0 or EN 16931-1:2026 yet.

| Layer | Today (BIS 3) | Needed for BIS 4.0 |
| --- | --- | --- |
| Invopop workflow | `BE_SEND_INVOICE_PEPPOL_OASIS_UBL_BUSINESS`: `sequence.enumerate` `PPL` → `ubl.generate` with `doc_type` `bis-invoice-v3` and `validate_ubl: true` → `peppol.send` (`WF/BE_SEND_INVOICE_PEPPOL_OASIS_UBL_BUSINESS.json:15-47`) | A BIS 4.0 `doc_type` (name unknown) |
| GOBL → UBL format | `FormatPeppol` (`ubl+peppol+bis-v3`), BIS 3 CustomizationID and ProfileID, VESIDs `eu.peppol.bis3:invoice:2026.5` / `eu.peppol.bis3:creditnote:2026.5` (`ubl:format.go:294-311`) | 2026 CustomizationID and ProfileID, UBL version, VESIDs, bindings for the 59 new BTs and BGs (CEN/TS 16931-3-2:2026) |
| GOBL addon | `eu-en16931-v2017` only (`gobl:addons/eu/en16931/en16931.go:21-22`; also in gobl v0.511.0), added by `ubl.Export` because fiskaly sets no BE addon (`ubl:ubl.go:157`, `191-219`) | An EN 16931-1:2026 addon with the §6 rules |
| fiskaly document type | `OASIS_UBL` for every supplier outside DE and IT (`E/invopop.go:300-319`) | A BIS 3 / BIS 4.0 selector (by date or receiver capability) |
| fiskaly mapper and UAPI | `G/` writes GOBL 2017 structures | Mappings and fields for the gap rows (§5) |
| Timing | BIS 3.0.22: publication 2026-11-23, mandatory from 2027-02-24, still on CEN artefacts 1.3.16 (`guide/release-notes/v3.0.22.adoc`, branch `2026-Q4-MR`) | BIS 4.0: no date announced. KoSIT expects the XRechnung 4.0 bundle in spring 2027; XRechnung 3.0 stays in force until at least 2027-07-31 |

## Sources

| Source | Version / identifier | Location | Retrieved |
| --- | --- | --- | --- |
| XRechnung 4.0 pre-release | *Vorversion XRechnung 4.0.0, Veröffentlicht 15.09.2026* (108 pp.), `XRechnung-4.0.0-Vorversion-2026-09-15.pdf`, SHA-256 `747f402ae8125b5044c65bdd1b5cde306f189394512c0e7e2d87dc7bb76851c8`; no syntax binding | https://xeinkauf.de/app/uploads/2026/09/XRechnung-4.0.0-Vorversion-2026-09-15.pdf | 2026-11-03 |
| EN 16931-1:2026 | EVS-EN 16931-1:2026, valid from 2026-05-04; paywalled, not held | https://www.evs.ee/en/evs-en-16931-1-2026 | 2026-11-03 |
| CEN/TS 16931-3-2:2026 | "Syntax binding of EN 16931-1 to ISO/IEC 19845 (UBL) invoice and credit note", valid from 2026-08-17, replaces CEN/TS 16931-3-2:2020; paywalled, not held | https://www.evs.ee/en/cen-ts-16931-3-2-2026 | 2026-11-03 |
| KoSIT status pages | Vorversion 2026-09-15; bundle spring 2027; 3.0 in force ≥ 2027-07-31 | https://xeinkauf.de/xrechnung/versionen-und-bundles/ ; https://xeinkauf.de/xrechnung/version-4/ | 2026-11-03 |
| KoSIT repositories | No 4.0 branch/tag: GitHub `itplr-kosit`; GitLab `xrechnung/xrechnung-model-semox` (last tag `v2026-08-31`) | https://github.com/itplr-kosit ; https://projekte.kosit.org/xrechnung/xrechnung-model-semox | 2026-11-03 |
| 2017 baseline | XRechnung 3.0.2 bundle 2026-08-31, `xrechnung-cius-model.xml` (`ubl-inv`, `ubl-cn`) | `E-Invoicing-Formats-and-Profiles/Germany/xrechnung-3.0.2-bundle-2026-08-31/xrechnung-3.0.2-xrechnung-model-2026-08-31/model/` | local |
| Peppol BIS 3.0.21 | Tag `3.0.21` (2026-10-28): `structure/syntax/ubl-invoice.xml`, `ubl-creditnote.xml` | https://github.com/OpenPEPPOL/peppol-bis-invoice-3/tree/3.0.21/structure/syntax | 2026-11-03 |
| OpenPeppol status | No 4.0 branch; branch `2026-Q4-MR`: `settings.adoc` `:version: 3.0.22`, `guide/release-notes/v3.0.22.adoc` (publication 2026-11-23, mandatory 2027-02-24, 384 for DE and SK only). docs.peppol.eu `/poacc/billing/4.0/` → 404. peppol.org search: no BIS 4.0 statement | https://github.com/OpenPEPPOL/peppol-bis-invoice-3 ; https://docs.peppol.eu/poacc/billing/4.0/ | 2026-11-03 |
| Spain SPFE (cross-check only) | Anexo I, Orden HAC/1028/2026, AEAT edition 2026-10-20; `factura.xml` (UBL 2.5) | `E-Invoicing-Formats-and-Profiles/Spain/E-Inv Specifications/` | local |
| Regulatory baseline | EN 16931-1:2026 Definitive Text 2026-03-18; rows verified 2026-10-29 | `E-Invoicing-Formats-and-Profiles/regulatory_baseline.md` | local |
| UAPI and as-built code | `fiskaly/meta@cd19ec8`; `invopop/gobl` v0.501.0 (v0.511.0 checked for the converter-side addon); `invopop/gobl.ubl` v0.99.0 | [`UAPI_v6_Fields.md`](UAPI_v6_Fields.md); UAPI → GOBL trace 2026-11-03 | 2026-11-03 |

## Verification Log

| # | Claim | How checked | Result |
| --- | --- | --- | --- |
| 1 | BIS 4.0 is unpublished, so this page is provisional. The next release is 3.0.22 on the 2017 artefacts (publication 2026-11-23, mandatory 2027-02-24) | `gh api` tags/releases/branches of `OpenPEPPOL/peppol-bis-invoice-3` and `poacc-upgrade-3`; `v3.0.22.adoc`; docs.peppol.eu 4.0 → 404 | CONFIRMED (re-checked 2026-11-03) |
| 2 | OpenPeppol has made no BIS 4.0 / EN 16931-1:2026 statement | OpenPEPPOL repo list; peppol.org search | CONFIRMED |
| 3 | Vorversion published 2026-09-15, not for production; bundle spring 2027 | xeinkauf.de pages; PDF title page and metadata; HTTP Last-Modified | CONFIRMED |
| 4 | The Vorversion has no syntax binding; EN 16931-1:2026 and its UBL binding CEN/TS 16931-3-2:2026 are published, paywalled and not held | Full text: 0 hits for `cbc:`, `cac:`, `ram:`; no 4.0 model on KoSIT GitHub/GitLab; EVS catalogue entries for both standards | CONFIRMED |
| 5 | 216 BT / 39 BG; 52 BT + 7 BG new, 0 removed, 8 changed, 16 renamed, 12 data type only; 39 rule IDs new, 20 not listed | Parsed chapters 3 and 4 against the XRechnung 3.0.2 model (BT-4/168/169/178 unused) | CONFIRMED |
| 6 | 2026 CustomizationID prefix; BIS 4.0 CustomizationID and ProfileID | Vorversion title and BT-24 note (inconsistent); SPFE sample; no Peppol source | Prefix CONFIRMED; BIS 4.0 values UNVERIFIED |
| 7 | BT-10 repeatable with optional scheme | Vorversion §3.1; KoSIT 4.0 page; SPFE Anexo I (0..n) | CONFIRMED; core cardinality UNVERIFIED |
| 8 | Line-level BT-194/195; BR-68; BR-S-8 split | Vorversion §3.26, §4.2, §4.4.1 | CONFIRMED |
| 9 | BR-CO-16 with BT-179; BR-CO-32; BR-CO-33; BR-76 | Vorversion §4.2, §4.3 | CONFIRMED (IDs may change) |
| 10 | Schemes mandatory (BR-69..75); today BT-46 is written once or twice without `@schemeID` and BT-47 is not written | Vorversion §4.2; `ubl:party.go:286-305`, `322-350`; `G/recipient.go:158-172` | CONFIRMED (production converter version unknown) |
| 11 | No EN 16931-1:2026 support in GOBL v0.501.0 / v0.511.0 or gobl.ubl v0.99.0 | grep for `2026`/`v2026`; `en16931.go:21-22`; `format.go:294-311` | CONFIRMED |
| 12 | Rule IDs §6 and §7.1 call "not listed" (e.g. BR-O-2/3/4, BR-CO-17) are dropped in 2026 | Absent from Vorversion chapter 4 | UNVERIFIED as a normative removal |


### Independent verification pass (2026-11-03)

Two separate checkers re-tested the decision-critical claims against code (fiskaly `cd19ec8`, gobl v0.501.0, the vendored converters) and against primary sources. Neither was given the page.

| Claim | Verdict | Evidence / correction |
| --- | --- | --- |
| XRechnung 4.0.0 Vorversion (15.09.2026) has 216 BTs and 39 BGs | CONFIRMED | Counted from the Kapitel 2 index: BT-1..BT-220 without BT-4/168/169/178; BG-1..BG-39 |
| BR-69..BR-75 require scheme identifiers on party identifiers | CONFIRMED | e.g. "BR-70 The Seller legal registration Identifier (BT-30) shall have a Scheme Identifier"; covers BT-29, 30, 46, 47, 60, 61, 71 |
| BR-CO-33 forbids a negative credit note | PARTLY CONFIRMED | It constrains BT-112 (Invoice total amount with VAT) of a credit note, not the amount due (BT-115) |
| BR-76 requires an IBAN on intra-community invoices | PARTLY CONFIRMED | It requires BT-84 (IBAN *or* a national account number), and only on invoices (the BG-17 note excludes credit notes). §5 and §6 corrected |
| `#compliant#` on the title page vs `#conformant#` in the BT-24 note | CONFIRMED | Title: `urn:cen.eu:en16931:2026#compliant#urn:xeinkauf.de:kosit:xrechnung_4.0`; the BT-24 note uses `#conformant#`; the Extension and CVD identifiers are "TBD" |
| `buyer_id` / `company_id` sent without `@schemeID` today | CONFIRMED | `recipient.go:159-172`. The EN16931 normaliser adds a scheme by key (GLN, GTIN) or type (FR SIREN/SIRET, DK CVR) (`gobl:addons/eu/en16931/org.go:50-61`, `88-111`); fiskaly sets key `other` and no type |
| No OpenPeppol BIS 4.0 publication | CONFIRMED (as of 2026-11-03) | OpenPEPPOL tags end at 3.0.21; the next release, 3.0.22, is still built on the 2017 model |

### Full re-validation (2026-11-03)

Every row was re-checked against the Vorversion PDF (SHA-256 matches the Sources row), the XRechnung 3.0.2 `ubl-inv` / `ubl-cn` bindings, `UAPI_v6_Fields.md`, and checkouts of `fiskaly/meta@cd19ec8`, `invopop/gobl` v0.501.0 and `invopop/gobl.ubl` v0.99.0.

| Check | Result | Corrections or additions |
| --- | --- | --- |
| BT/BG IDs and names vs the Vorversion index and chapter 3 | All match; all quoted cardinalities match | — |
| Cited rules vs Vorversion chapter 4 | All match; "not listed" rules confirmed absent | BR-CO-30 erratum added (§6) |
| XPaths vs the XRechnung 3.0.2 binding | All match except three paths without predicates | BT-110 currency predicate; BT-139/144 `ChargeIndicator` predicates; second paths added for BT-29, BT-47, BT-130, BT-158, BG-26 (§5) |
| UAPI paths vs `UAPI_v6_Fields.md` | All exist | — |
| fiskaly code refs (44) | 36 OK | BT-5 default and BE currency check; consumer recipients; `items[]`; `breakdown[]` ignored; BT-30 ref; correction reason ref; `record.offset`; BT-202 resolved (not written) |
| Converter refs (23) | 18 OK | BT-81 `UNKNOWN` fails BR-49; BT-120 join is in fiskaly code; BT-47 never written; normaliser scheme list; BT-11 on credit notes fails the CreditNote schema; BT-9 lost on credit notes without an instruction |

### Fix pass (2026-11-04)

| Area | Change | Evidence |
| --- | --- | --- |
| Standards status | `unpublished` XPath cells → "not obtained (CEN/TS 16931-3-2:2026)"; intro, §5A header, Sources and #4 updated | EVS catalogue: EN 16931-1:2026 (2026-05-04), CEN/TS 16931-3-2:2026 (2026-08-17) |
| Upward correction | §3 BT-112 row and §6: a positive-amount `381` passes and credits the buyer; only negative amounts fail BR-CO-33 | Vorversion BR-CO-33; README correction finding |
| BT-81 / BR-49 | §4 aligned with §1.4; BR-49 row added to §6; the failure surfaces at Invopop after `COMPLETED` | `gobl:addons/eu/en16931/pay.go:40-47`; `ubl:payment.go:143-151`; `E/invopop.go:300-319`; `G/addon.go:19-30`; `SV1:3739-3766` |
| BT-11 and BT-9 on credit notes | Marked "from code and XSD, not executed"; moved into the §3 substitution table; CreditNote schema row in §6; BT-9 loss is silent (GOBL BR-CO-25 on standard invoices only) | `ubl:ordering.go:124-129`; `UBL-CreditNote-2.1.xsd`; `WF/BE_SEND_INVOICE_PEPPOL_OASIS_UBL_BUSINESS.json:40`; `gobl:addons/eu/en16931/bill.go:143-151` |
| Not subject to VAT | §6 row: README's BR-O-02 finding does not carry over to the 2026 model | Vorversion §4.4.7 and chapter 4 (no BR-O-2/3/4) |
| §7 restored | 7.1 change table, corrected for category N, BG-26 under BG-37, Vorversion §4.4.7, the delta counts and the standards status; 7.2 implementation gaps with the 3.0.22 dates; #5 restored with it | `bis4/diff.json`, `bis4/rules.json`; `v3.0.22.adoc`; `ubl:format.go:294-311` |
| Intro and legend | Two paragraphs; flow-selection sentence; `content.operation`; `WF/` legend and the PPL step locator | `WF/BE_SEND_INVOICE_PEPPOL_OASIS_UBL_BUSINESS.json:15-26` |
| Conventions | BT cells plain and named; candidate XPaths written in full; `prov.` on every §3 and §5 row; `instruction.properties` target column emptied; §4 BT-3 and VAT paths completed; ad-hoc note tags reworded | XRechnung 3.0.2 binding (scripted check) |
| Removed | BT-6 / BT-111 gap row: BE suppliers are EUR-only at record validation | `src/unified/domain/record/capabilities.go:186-225` |
| Nits | Errata (BR-CO-28 element column, §4.4.7 vs BR-O-5); `384` DE/SK only in 3.0.22; BT-90 seller path; BT-114 ±0.01; DEPOSIT row; BR-DE-37/38 on BT-34/BT-49 | Vorversion §4.2, §4.4.7, §4.5; `v3.0.22.adoc`; `ubl:payment.go:123-136`; `ubl:ubl.go:164` |
