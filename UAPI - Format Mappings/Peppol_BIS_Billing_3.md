# UAPI v6 → Peppol BIS Billing 3.0 (UBL)

As of 2026-11-04 · Status: draft for review

UAPI v6 invoice and correction fields mapped to Peppol BIS Billing 3.0, release 3.0.21 (UBL 2.1). A BE supplier's invoice becomes BIS 3 when its one BUSINESS recipient has `invoicing.type` `PEPPOL` and a BE or DE address (`RH:3270-3279`): workflow `BE_SEND_INVOICE_PEPPOL_OASIS_UBL_BUSINESS`, Invopop `doc_type` `bis-invoice-v3`; INVOICE becomes `/Invoice`, CORRECTION becomes `/CreditNote`. German suppliers on Peppol get XRechnung, which Invopop enforces on German Peppol invoices (`E/invopop.go:308-312`; [XRechnung_3.md](XRechnung_3.md)). The XPath column is the spec target from the BIS 3.0.21 syntax binding (`structure/syntax/ubl-invoice.xml`, `ubl-creditnote.xml`); credit-note lines use `cac:CreditNoteLine` and `cbc:CreditedQuantity`. The notes column records where today's output departs from it. UAPI paths are relative to `content.operation` (CORRECTION: under `data.`); "API docs: BT-x" is the BT fiskaly's API docs give where it differs from the spec.

Code citations: `G/` = fiskaly `src/unified/einvoicing/gobl/`; `E/` = fiskaly `src/unified/einvoicing/`; `RH` = fiskaly `src/unified/api/v6/record_handler.go`; `EA` = fiskaly `src/unified/domain/record/einvoicing_adaptations.go`; `SV1` = fiskaly `src/unified/domain/record/service_v1.go`; `W` = fiskaly `src/unified/worker/e_invoicing_worker.go`; `WF/` = fiskaly `cmd/invopop-workflow-migration/workflows/` (all at `cd19ec8`); `gobl:` = invopop/gobl v0.501.0 (fiskaly's `go.mod` pin); "gobl v0.511" = invopop/gobl v0.511.0, the minimum gobl.ubl v0.99.0 requires, cited for code that runs inside the converter; `ubl:` = invopop/gobl.ubl v0.99.0. Which converter and GOBL versions Invopop runs is not visible to fiskaly.

## 1. Invoice Mapping

### 1.1 Header and References

| UAPI | EN 16931 BT | BIS 3.0 UBL XPath | notes |
| --- | --- | --- | --- |
| `document.number` | BT-1 Invoice number | `/Invoice/cbc:ID` | GOBL `code` (`G/invoice.go:153`); written as `series-code`, series never set (`ubl:invoice.go:127`, `223-228`). UNVERIFIED: the `sequence.enumerate` PPL step may replace it (§4) |
| `document.series` | BT-1 Invoice number | `/Invoice/cbc:ID` | ignored (no `Series` in `G/`). Proposal: map to GOBL `series` → `cbc:ID` = `SERIES-NUMBER` |
| `document.issued_at` | BT-2 Invoice issue date | `/Invoice/cbc:IssueDate` | date part; time has no BIS element. default: absent → `cal.Today()` at each build, so the sync and worker builds can differ (`G/invoice.go:704-722`) |
| `-` | BT-3 Invoice type code | `/Invoice/cbc:InvoiceTypeCode` | fixed `380` (`gobl:addons/eu/en16931/scenarios.go:20-27`; `ubl:common.go:168-179`) |
| `payments[0].details.currency` | BT-5 Invoice currency code | `/Invoice/cbc:DocumentCurrencyCode` | first payment only, domain default EUR (`G/invoice.go:725-741`). The API enum allows CZK\|EUR\|SEK, but for BE any explicit non-EUR payment currency is rejected at record validation (`src/unified/country/be/v1/record.go:25`; `src/unified/domain/record/capabilities.go:188-220`, called from `src/unified/domain/record/service_v1.go:1968`) |
| `-` | BT-6 VAT accounting currency code | `/Invoice/cbc:TaxCurrencyCode` | Not needed: BE suppliers are EUR-only (BT-5). gobl.ubl writes it only with a matching exchange rate (`ubl:invoice.go:141-148`) |
| `document.operation_date` | BT-72 Actual delivery date | `/Invoice/cac:Delivery/cbc:ActualDeliveryDate` | API docs: BT-7. The UAPI meaning (date of the supply when it differs from the issue date, Art. 226(7) Directive 2006/112/EC) fits BT-72. derived: GOBL `delivery.date` = operation date, else issue date (`G/invoice.go:433-441`); written by `ubl:delivery.go:38-41` |
| `document.operation_date` | BT-7 Value added tax point date | `/Invoice/cbc:TaxPointDate` | ignored: gobl.ubl writes BT-7 only from GOBL `value_date` (`ubl:invoice.go:158-161`), which fiskaly never sets; GOBL `op_date` (`G/invoice.go:152`) has no UBL output. If the operation date is meant as the chargeability date, map it to `value_date` |
| `-` | BT-8 Value added tax point date code | `/Invoice/cac:InvoicePeriod/cbc:DescriptionCode` | gap; `tax.point` never set (`ubl:invoice.go:208-221`) |
| `document.text` | BT-22 Invoice note | `/Invoice/cbc:Note` | ignored: `G/` never reads `Document().Text()`. Proposal: GOBL note → `cbc:Note`; R002 allows one, gobl.ubl merges |
| `document.payment_terms` | BT-20 Payment terms | `/Invoice/cac:PaymentTerms/cbc:Note` | `payment.terms.notes` (`G/invoice.go:316-321`; `ubl:payment.go:225-231`) |
| `document.references.buyer` | BT-10 Buyer reference | `/Invoice/cbc:BuyerReference` | `ordering.code` (`G/invoice.go:331-333`; `ubl:ordering.go:76-78`); R003 needs BT-10 or BT-13 |
| `document.references.purchase_order` | BT-13 Purchase order reference | `/Invoice/cac:OrderReference/cbc:ID` | `ordering.purchases[0]` (`ubl:ordering.go:107-112`). default `NA` when BT-10 and BT-13 are both absent (`ubl:ordering.go:181-187`) |
| `document.references.project` | BT-11 Project reference | `/Invoice/cac:ProjectReference/cbc:ID` | attached for BE suppliers (the DE drop, META-3995, does not apply) (`G/invoice.go:337-354`; `ubl:ordering.go:124-129`). Credit note: §3 |
| `document.references.contract` | BT-12 Contract reference | `/Invoice/cac:ContractDocumentReference/cbc:ID` | `ubl:ordering.go:143-151` |
| `document.references.despatch_advice` | BT-16 Despatch advice reference | `/Invoice/cac:DespatchDocumentReference/cbc:ID` | `ubl:ordering.go:131-135` |
| `document.references.tender` | BT-17 Tender or lot reference | `/Invoice/cac:OriginatorDocumentReference/cbc:ID` | `ubl:ordering.go:153-157`; credit-note element order: §3 |
| `document.references.buyer_routing` | BT-19 Buyer accounting reference | `/Invoice/cbc:AccountingCost` | `ordering.cost` (`ubl:ordering.go:80-83`) |
| `document.references.preceding_document.number` | BT-25 Preceding Invoice reference | `/Invoice/cac:BillingReference/cac:InvoiceDocumentReference/cbc:ID` | INVOICE only (`G/invoice.go:197-205`, `371-383`; `ubl:ordering.go:53-72`); ignored on credit notes (§3) |
| `document.references.preceding_document.issued_at` | BT-26 Preceding Invoice issue date | `/Invoice/cac:BillingReference/cac:InvoiceDocumentReference/cbc:IssueDate` | as above |
| `-` | BT-14 Sales order reference, BT-15 Receiving advice reference, BT-18 Invoiced object identifier | `/Invoice/cac:OrderReference/cbc:SalesOrderID`, `/Invoice/cac:ReceiptDocumentReference/cbc:ID`, `/Invoice/cac:AdditionalDocumentReference[cbc:DocumentTypeCode='130']/cbc:ID` | gap: no UAPI field |
| `document.activity_code`, `document.offline.reason`, `document.offline.reference` | — | — | ignored; no BT |
| `details.creators[]`, `details.training`, `details.properties` | — | — | ignored (record metadata) |

### 1.2 Buyer

| UAPI | EN 16931 BT | BIS 3.0 UBL XPath | notes |
| --- | --- | --- | --- |
| `recipients[i].type` = BUSINESS | BG-7 Buyer | `/Invoice/cac:AccountingCustomerParty` | exactly one BUSINESS recipient with `invoicing` (`EA:60-62`; `G/invoice.go:400-418`) |
| `recipients[i].name` | BT-44 Buyer name | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PartyLegalEntity/cbc:RegistrationName` | `ubl:party.go:207-216` |
| `recipients[i].name` | BT-45 Buyer trading name | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PartyName/cbc:Name` | derived: copy of BT-44 (`ubl:party.go:208-211`); UAPI has no buyer trade name |
| `recipients[i].identification.number` (type VAT) | BT-48 Buyer VAT identifier | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PartyTaxScheme/cbc:CompanyID` | `customer.tax_id`, country = recipient **address** country, scheme `VAT` (`G/identification.go:15-42`); prefix stripped (`gobl:tax/identity.go:174-185`) and rewritten as country + code (`ubl:party.go:229-249`); BR-CO-09 holds |
| `recipients[i].type` = CONSUMER | — | — | no PEPPOL: a CONSUMER invoicing block is parsed as SDI, so PEPPOL fails on the missing destination code (`RH:3557-3568`; §6) |
| `recipients[i].identification.type` = TAX \| PASSPORT \| DOCUMENT \| OTHER | — | — | the sync check fails (`EA:64-67`; §6.1); buyers without a VAT ID cannot be invoiced in this flow |
| `recipients[i].buyer_id` | BT-46 Buyer identifier | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PartyIdentification/cbc:ID` | identity `{key: other, label: buyer}` (`G/recipient.go:158-165`) → `PartyIdentification` without `@schemeID` (`ubl:party.go:322-349`); UAPI has no ICD field |
| `recipients[i].company_id` | BT-47 Buyer legal registration identifier | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PartyLegalEntity/cbc:CompanyID` | as-built: second `cac:PartyIdentification` (BT-46 slot), identity has no legal scope (`G/recipient.go:166-172`; `ubl:party.go:322-349`; only `scope: legal` reaches `PartyLegalEntity/cbc:CompanyID`, `286-306`); with `buyer_id` → UBL-SR-16 fatal. Proposal: `scope: legal` + ICD |
| `recipients[i].invoicing.type` = PEPPOL | — | — | selects the workflow network (`E/invopop.go:388-412`) |
| `recipients[i].invoicing.identifier` | BT-49 Buyer electronic address | `/Invoice/cac:AccountingCustomerParty/cac:Party/cbc:EndpointID` | customer inbox `peppol` (`G/inbox.go:43-52`); GOBL splits `<scheme>:<id>` (`gobl:org/inbox.go:105-113`) and the en16931 addon copies it to an `iso6523-actorid-upis` endpoint (`gobl:addons/eu/en16931/org.go:141-160`); value = part after `:`, `@schemeID` = the 4-digit prefix (`ubl:party.go:27-36`; inbox fallback `47-59`). R010, BR-63 |
| `recipients[i].invoicing.identifier` (first 4 digits) | BT-49-1 Buyer electronic address scheme | `/Invoice/cac:AccountingCustomerParty/cac:Party/cbc:EndpointID/@schemeID` | derived EAS code (BE: `0208` KBO/BCE or `9925` Belgium VAT number, per `structure/codelist/eas.xml` at tag 3.0.21). CL008 (EAS list); PEPPOL-COMMON-R043 checks `0208` (10 digits, mod 97) |
| `recipients[i].invoicing.email`, `.format`, `.destination_code`, `.pec` | — | — | other channels; not this flow |
| `recipients[i].address.line.street` + `.number` | BT-50 Buyer address line 1 | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PostalAddress/cbc:StreetName` | joined `street num` (`G/address.go:33-40`; `gobl:org/address.go:63-77`; `ubl:party.go:429-432`) |
| `recipients[i].address.line.suffix` | BT-51 Buyer address line 2 | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PostalAddress/cbc:AdditionalStreetName` | `street_extra` (`G/address.go:18-21`) |
| `recipients[i].address.line.prefix`, `.infix` | BT-163 Buyer address line 3 | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PostalAddress/cac:AddressLine/cbc:Line` | ignored |
| `recipients[i].address.city` | BT-52 Buyer city | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PostalAddress/cbc:CityName` | — |
| `recipients[i].address.code` | BT-53 Buyer post code | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PostalAddress/cbc:PostalZone` | must pass `cbc.Code.Validate` (`G/address.go:23-26`) |
| `recipients[i].address.region` | BT-54 Buyer country subdivision | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PostalAddress/cbc:CountrySubentity` | optional for BE/DE |
| `recipients[i].address.country` | BT-55 Buyer country code | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PostalAddress/cac:Country/cbc:IdentificationCode` | derived: overwritten with the tax-ID country, which fiskaly also takes from the address (`ubl:party.go:250-256`); BE or DE only (§6, stage A) |
| `-` | BT-56 Buyer contact point, BT-57 Buyer contact telephone number, BT-58 Buyer contact email address | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:Contact/cbc:Name`, `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:Contact/cbc:Telephone`, `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:Contact/cbc:ElectronicMail` | gap: no UAPI source |
| `recipients[i].shipping.name` | BT-70 Deliver to party name | `/Invoice/cac:Delivery/cac:DeliveryParty/cac:PartyName/cbc:Name` | `delivery.receiver.name` (`G/invoice.go:454-467`); also `PartyLegalEntity` → UBL-CR-397 warning (`ubl:party.go:376-385`) |
| `recipients[i].shipping.address.*` | BG-15 Deliver to address | `/Invoice/cac:Delivery/cac:DeliveryLocation/cac:Address` | BT-75 to BT-80 and BT-165 in `cbc:StreetName`, `cbc:AdditionalStreetName`, `cbc:CityName`, `cbc:PostalZone`, `cbc:CountrySubentity`, `cac:Country/cbc:IdentificationCode`, `cac:AddressLine/cbc:Line`; same sub-mapping as the buyer address above (street + number, suffix, city, code, region, country; prefix/infix ignored) (`G/invoice.go:443-449`; `ubl:delivery.go:53-58`); BR-57 |
| `recipients[i].shipping.date` | BT-72 Actual delivery date | `/Invoice/cac:Delivery/cbc:ActualDeliveryDate` | ignored; BT-72 comes from `document.operation_date` or the issue date (§1.1) |
| `-` | BT-71 Deliver to location identifier | `/Invoice/cac:Delivery/cac:DeliveryLocation/cbc:ID` | gap |
| `recipients[i].origin` | — | — | ignored |

### 1.3 Lines

| UAPI | EN 16931 BT | BIS 3.0 UBL XPath | notes |
| --- | --- | --- | --- |
| `entries[i]` | BG-25 Invoice line | `/Invoice/cac:InvoiceLine` | one line per entry (`G/entries.go:34-88`) |
| `-` (array index) | BT-126 Invoice line identifier | `/Invoice/cac:InvoiceLine/cbc:ID` | derived `i+1` (`G/entries.go:77`; `ubl:lines.go:77`); `entries[i].details.number` ignored |
| `entries[i].type` (SALE \| RETURN) | — | — | ignored: RETURN lines map like SALE, signs as supplied |
| `entries[i].data.type` = GROUP | BG-25 Invoice line | `/Invoice/cac:InvoiceLine` | one line from group-level values (`RH:4034-4070`); `data.items[]` are parsed (`RH:4072-4080`) and then ignored (`G/entries.go:299-307`) |
| `entries[i].data.text` | BT-153 Item name | `/Invoice/cac:InvoiceLine/cac:Item/cbc:Name` | `G/entries.go:299-306` |
| `entries[i].data.unit.quantity` | BT-129 Invoiced quantity | `/Invoice/cac:InvoiceLine/cbc:InvoicedQuantity` | written as GOBL holds it (`ubl:lines.go:233-245`); UAPI input allows ≤ 8 dp |
| `entries[i].data.unit.measure` | BT-130 Invoiced quantity unit of measure code | `/Invoice/cac:InvoiceLine/cbc:InvoicedQuantity/@unitCode` | GOBL unit key or Rec 20 code as given, else dropped with a warning (`G/entries.go:312-323`); default `C62` (`gobl:addons/eu/en16931/org.go:79-86`) |
| `entries[i].data.unit.price.inclusive` | BT-146 Item net price | `/Invoice/cac:InvoiceLine/cac:Price/cbc:PriceAmount` | derived: VAT-inclusive price into GOBL (`tax.prices_include: VAT`, `G/invoice.go:165-166`); converter strips VAT (`ubl:ubl.go:161`), writes net at GOBL precision (`ubl:lines.go:496-499`). BR-27: not negative |
| `entries[i].data.unit.price.exclusive` | BT-146 Item net price | `/Invoice/cac:InvoiceLine/cac:Price/cbc:PriceAmount` | ignored (API docs: BT-146); BT-146 is recomputed from the gross price |
| `entries[i].data.unit.factor` | BT-149 Item price base quantity | `/Invoice/cac:InvoiceLine/cac:Price/cbc:BaseQuantity` | ignored (API docs: BT-149); GOBL v0.501.0 `org.Item` has no `per` (`ubl:lines.go:500-502` needs it) |
| `entries[i].data.value.base.exclusive`, `.inclusive` | BT-131 Invoice line net amount | `/Invoice/cac:InvoiceLine/cbc:LineExtensionAmount` | ignored; BT-131 is derived (§1.6) |
| `entries[i].data.value.discount.inclusive` | BT-136 Invoice line allowance amount | `/Invoice/cac:InvoiceLine/cac:AllowanceCharge[cbc:ChargeIndicator='false']/cbc:Amount` | if ≠ 0; inclusive input, net-converted, 2 dp (`G/entries.go:45-55`; `ubl:lines.go:467-477`); `.exclusive` ignored |
| `-` | BT-139 Invoice line allowance reason | `/Invoice/cac:InvoiceLine/cac:AllowanceCharge[cbc:ChargeIndicator='false']/cbc:AllowanceChargeReason` | fixed `Special Discount` (META-2347); satisfies BR-42; BT-140 code gap |
| `entries[i].data.value.surcharge.inclusive` | BT-141 Invoice line charge amount | `/Invoice/cac:InvoiceLine/cac:AllowanceCharge[cbc:ChargeIndicator='true']/cbc:Amount` | as discount (`G/entries.go:57-67`); `.exclusive` ignored |
| `-` | BT-144 Invoice line charge reason | `/Invoice/cac:InvoiceLine/cac:AllowanceCharge[cbc:ChargeIndicator='true']/cbc:AllowanceChargeReason` | fixed `Special Surcharge`; BT-145 code gap |
| `entries[i].data.vat.type` = VAT_RATE, `.percentage` > 0 | BT-151 Invoiced item VAT category code | `/Invoice/cac:InvoiceLine/cac:Item/cac:ClassifiedTaxCategory/cbc:ID` | `S` for any non-zero rate (`G/entries.go:101-122`; `gobl:addons/eu/en16931/tax_combo.go:33-42`); `vat.code` unused |
| `entries[i].data.vat.percentage` | BT-152 Invoiced item VAT rate | `/Invoice/cac:InvoiceLine/cac:Item/cac:ClassifiedTaxCategory/cbc:Percent` | `ubl:lines.go:349-357` |
| `entries[i].data.vat.percentage` = 0 | BT-151 Invoiced item VAT category code | `/Invoice/cac:InvoiceLine/cac:Item/cac:ClassifiedTaxCategory/cbc:ID` | `Z` (GOBL key `zero`) |
| `entries[i].data.vat.type` = VAT_EXEMPTION, `.code` NOT_TAXABLE or CAUSE_n | BT-151 Invoiced item VAT category code | `/Invoice/cac:InvoiceLine/cac:Item/cac:ClassifiedTaxCategory/cbc:ID` | `E`, Percent `0` (`G/entries.go:148-161`, `175-189`); all BE causes → `E`, no VATEX |
| `entries[i].data.vat.type` = VAT_EXEMPTION, `.code` NOT_SUBJECT | BT-151 Invoiced item VAT category code | `/Invoice/cac:InvoiceLine/cac:Item/cac:ClassifiedTaxCategory/cbc:ID` | `O`, no Percent; always fails BR-O-02 (BT-31 and BT-48 always present, §6) |
| `entries[i].data.vat.reason` | BT-120 VAT exemption reason text | `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cac:TaxCategory/cbc:TaxExemptionReason` | one tax note per category, reasons joined with `; ` (`G/entries.go:208-269`); mandatory for BE; matched by category (`gobl:addons/eu/en16931/bill.go:52-72`; `ubl:totals.go:122-126`) |
| `entries[i].data.vat.type` = VAT_REVERSE_CHARGE | BT-151 Invoiced item VAT category code | `/Invoice/cac:InvoiceLine/cac:Item/cac:ClassifiedTaxCategory/cbc:ID` | target `AE`. The GOBL build fails at sync: `unsupported VAT type … 'REVERSE_CHARGE'` (`G/entries.go:164-169`). API docs: not applicable for BE/DE/IT |
| `entries[i].data.vat.amount`, `.exclusive`, `.inclusive` | — | — | ignored (API docs: BT-110/117, BT-106/109/116, BT-112); derived from the lines |
| `entries[i].data.vat.regime`, `.regime_base`, `.surcharge.rate`, `.surcharge.amount` | — | — | ignored (API docs: not applicable for BE/DE/IT) |
| `entries[i].data.vat.fiscal_location.country`, `.region` | — | — | ignored; no BT. API docs: defaults to the taxpayer's fiscal region |
| `entries[i].data.product.number` (type SKU, MPN, OTHER) | BT-155 Item Seller's identifier | `/Invoice/cac:InvoiceLine/cac:Item/cac:SellersItemIdentification/cbc:ID` | ignored (API docs: BT-155, BT-157 for `product.type` OTHER only; none for SKU, MPN); proposal: GOBL `item.ref` (`ubl:lines.go:418-423`) |
| `entries[i].data.product.number` (type GTIN-8/12/13/14) | BT-157 Item standard identifier | `/Invoice/cac:InvoiceLine/cac:Item/cac:StandardItemIdentification/cbc:ID` (`@schemeID` `0160`) | ignored; proposal: identity key `gtin` → 0160 (`gobl:addons/eu/en16931/org.go:50-53`) |
| `entries[i].data.product.code` (type OTHER) | BT-158 Item classification identifier | `/Invoice/cac:InvoiceLine/cac:Item/cac:CommodityClassification/cbc:ItemClassificationCode` | ignored (API docs: BT-158); UAPI has no `@listID`: BR-65 requires it, and BR-CL-13 limits it to UNTDID 7143 |
| `entries[i].data.product.details.label` | BT-154 Item description | `/Invoice/cac:InvoiceLine/cac:Item/cbc:Description` | ignored (API docs: BT-154) |
| `entries[i].data.product.details.name` | BT-153 Item name | `/Invoice/cac:InvoiceLine/cac:Item/cbc:Name` | ignored; BT-153 comes from `data.text` |
| `entries[i].data.product.details.brand`, `.model`, `.serial`, `.batch`, `.date_of_entry`, `.date_of_production` | BT-160 Item attribute name, BT-161 Item attribute value | `/Invoice/cac:InvoiceLine/cac:Item/cac:AdditionalItemProperty/cbc:Name`, `/Invoice/cac:InvoiceLine/cac:Item/cac:AdditionalItemProperty/cbc:Value` | ignored |
| `entries[i].details.description` | BT-127 Invoice line note | `/Invoice/cac:InvoiceLine/cbc:Note` | ignored (API docs: BT-127) |
| `entries[i].details.concept`, `.purpose`, `.regulatory`, `.label`, `.activity_code`, `.properties` | — | — | ignored |
| `entries[i].details.voucher.*`, `.applied_vouchers[]`, `.prepayment.document_number` | — | — | ignored |
| `-` | BT-128 Invoice line object identifier, BT-132 Referenced purchase order line reference, BT-133 Invoice line Buyer accounting reference, BT-134 Invoice line period start date, BT-135 Invoice line period end date, BT-156 Item Buyer's identifier, BT-159 Item country of origin | `/Invoice/cac:InvoiceLine/cac:DocumentReference[cbc:DocumentTypeCode='130']/cbc:ID`, `/Invoice/cac:InvoiceLine/cac:OrderLineReference/cbc:LineID`, `/Invoice/cac:InvoiceLine/cbc:AccountingCost`, `/Invoice/cac:InvoiceLine/cac:InvoicePeriod/cbc:StartDate`, `/Invoice/cac:InvoiceLine/cac:InvoicePeriod/cbc:EndDate`, `/Invoice/cac:InvoiceLine/cac:Item/cac:BuyersItemIdentification/cbc:ID`, `/Invoice/cac:InvoiceLine/cac:Item/cac:OriginCountry/cbc:IdentificationCode` | gap: no UAPI source |
| `-` | BT-147 Item price discount, BT-148 Item gross price | `/Invoice/cac:InvoiceLine/cac:Price/cac:AllowanceCharge/cbc:Amount`, `/Invoice/cac:InvoiceLine/cac:Price/cac:AllowanceCharge/cbc:BaseAmount` | gap: no UAPI source; the VAT-inclusive UAPI price is not BT-148 |

### 1.4 Payment

| UAPI | EN 16931 BT | BIS 3.0 UBL XPath | notes |
| --- | --- | --- | --- |
| `payments[i].details.date` (first OUTSTANDING) | BT-9 Payment due date | `/Invoice/cbc:DueDate` | optional in the schema, required for OUTSTANDING at conversion (`payment due date not set`, `G/invoice.go:607-621`); `ubl:payment.go:233-235`. Later OUTSTANDING entries dropped with a warning. Credit note: §3 |
| `payments[i].details.amount` (OUTSTANDING) | — | — | `due_dates[0].amount`; no UBL element. The sum of all payments drives line rounding (§1.6) |
| `payments[i].instruction.type` = CREDIT_TRANSFER | BT-81 Payment means type code | `/Invoice/cac:PaymentMeans/cbc:PaymentMeansCode` | fixed `30` (`gobl:addons/eu/en16931/pay.go:12-27`); SEPA credit transfer `58` is never produced |
| `payments[i].instruction.account` (CREDIT_TRANSFER) | BT-84 Payment account identifier | `/Invoice/cac:PaymentMeans/cac:PayeeFinancialAccount/cbc:ID` | IBAN (`G/invoice.go:530-537`; `ubl:payment.go:195-212`); BR-50, BR-61 |
| `payments[i].instruction.name` | BT-85 Payment account name | `/Invoice/cac:PaymentMeans/cac:PayeeFinancialAccount/cbc:Name` | — |
| `payments[i].instruction.payment_service_provider` | BT-86 Payment service provider identifier | `/Invoice/cac:PaymentMeans/cac:PayeeFinancialAccount/cac:FinancialInstitutionBranch/cbc:ID` | free text 1–128, no BIC check |
| `payments[i].instruction.text` | BT-83 Remittance information | `/Invoice/cac:PaymentMeans/cbc:PaymentID` | API docs: BT-83. As-built: GOBL `instructions.detail` (`G/invoice.go:538-540`, `563-565`) → `cbc:PaymentMeansCode/@name`, i.e. BT-82 Payment means text (`ubl:payment.go:160-162`) |
| `payments[i].instruction.properties` | BT-83 Remittance information | `/Invoice/cac:PaymentMeans/cbc:PaymentID` | GOBL `instructions.ref` (`G/invoice.go:541-543`; `ubl:payment.go:157-159`). No BT per API docs. `text` and `properties` are swapped against the docs; a Belgian structured communication (`+++…+++`) only reaches BT-83 through `properties` |
| `payments[i].instruction.type` = DIRECT_DEBIT | BT-81 Payment means type code | `/Invoice/cac:PaymentMeans/cbc:PaymentMeansCode` | fixed `49`; SEPA direct debit `59` is never produced |
| `payments[i].instruction.mandate_reference` | BT-89 Mandate reference identifier | `/Invoice/cac:PaymentMeans/cac:PaymentMandate/cbc:ID` | `ubl:payment.go:166-171`; PEPPOL-EN16931-R061 |
| `payments[i].instruction.creditor_id.identifier` (`.type` SEPA) | BT-90 Bank assigned creditor identifier | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PartyIdentification/cbc:ID` (`@schemeID` `SEPA`) | no payee party, so it goes on the seller (`ubl:payment.go:121-136`) |
| `payments[i].instruction.debited_account` | BT-91 Debited account identifier | `/Invoice/cac:PaymentMeans/cac:PaymentMandate/cac:PayerFinancialAccount/cbc:ID` | `ubl:payment.go:172-176` |
| `payments[i].instruction.type` = UNKNOWN | BT-81 Payment means type code | `/Invoice/cac:PaymentMeans/cbc:PaymentMeansCode` | GOBL key `other` has no UNTDID 4461 code → `payment means extension is required (BR-49)` / converter error (`gobl:addons/eu/en16931/pay.go:40-48`; `ubl:payment.go:143-151`); fails inside Invopop |
| `payments[i].details.amount` (CARD, CASH, ONLINE, SINGLE_PURPOSE_VOUCHER, MULTI_PURPOSE_VOUCHER, OTHER) | BT-113 Paid amount | `/Invoice/cac:LegalMonetaryTotal/cbc:PrepaidAmount` | `payment.advances[]` (`G/invoice.go:290-299`, `623-636`) summed into `totals.advance` (`ubl:totals.go:77-79`); no `cac:PaymentMeans`, no `cac:PrepaidPayment` |
| `payments[i].number` (CARD) | BT-87 Payment card primary account number | `/Invoice/cac:PaymentMeans/cac:CardAccount/cbc:PrimaryAccountNumberID` | ignored (API docs: BT-87) |
| `payments[i].kind` (CARD) | BT-81 Payment means type code | `/Invoice/cac:PaymentMeans/cbc:PaymentMeansCode` | ignored (API docs: BT-81) |
| `payments[i].name` (ONLINE, OTHER) | BT-82 Payment means text | `/Invoice/cac:PaymentMeans/cbc:PaymentMeansCode/@name` | ignored (API docs: BT-82); fixed advance descriptions have no UBL element |
| `payments[i].type` = MEAL_VOUCHER, DEPOSIT | — | — | the GOBL build fails at sync: `unsupported payment type` (`G/invoice.go:300-302`) |
| `payments[i].details.date` (non-OUTSTANDING), `.details.discount`, `.details.channel`, `.details.properties`; `payments[i].concept`, `.count`, `.document_number`; voucher `payments[i].number` | — | — | ignored or no BIS element (advance date) |
| `-` | BG-10 Payee | `/Invoice/cac:PayeeParty` | gap: no UAPI source for BT-59, BT-60, BT-61 |

### 1.5 VAT Breakdown and Totals

| UAPI | EN 16931 BT | BIS 3.0 UBL XPath | notes |
| --- | --- | --- | --- |
| `breakdown[i]` | BG-23 VAT breakdown | `/Invoice/cac:TaxTotal/cac:TaxSubtotal` | ignored; one subtotal per GOBL rate, derived from the lines (`ubl:totals.go:102-145`) |
| `breakdown[i].exclusive` | BT-116 VAT category taxable amount | `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cbc:TaxableAmount` | ignored; derived |
| `breakdown[i].amount` | BT-117 VAT category tax amount | `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cbc:TaxAmount` | ignored; derived |
| `breakdown[i].type`, `.code` | BT-118 VAT category code | `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cac:TaxCategory/cbc:ID` | ignored; derived from line combos |
| `breakdown[i].percentage` | BT-119 VAT category rate | `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cac:TaxCategory/cbc:Percent` | ignored; derived; `0` for E/Z, omitted for O (`ubl:totals.go:129-136`) |
| `-` | BT-121 VAT exemption reason code | `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cac:TaxCategory/cbc:TaxExemptionReasonCode` | gap: no VATEX code for BE (no `cef-vatex` in the BE branch); `breakdown[i].inclusive` ignored |
| `totals.vat.exclusive` | BT-109 Invoice total amount without VAT | `/Invoice/cac:LegalMonetaryTotal/cbc:TaxExclusiveAmount` | ignored: BT-109 is GOBL `totals.total`; the UAPI value is only compared, with a warning log (`E/invopop.go:753-782`) |
| `totals.vat.exclusive` | BT-106 Sum of Invoice line net amount | `/Invoice/cac:LegalMonetaryTotal/cbc:LineExtensionAmount` | ignored; BT-106 is GOBL `totals.sum` |
| `totals.vat.amount` | BT-110 Invoice total VAT amount | `/Invoice/cac:TaxTotal/cbc:TaxAmount` | ignored; BT-110 is GOBL `totals.tax` (compared, warning log, §1.6) |
| `totals.vat.inclusive` | BT-112 Invoice total amount with VAT | `/Invoice/cac:LegalMonetaryTotal/cbc:TaxInclusiveAmount` | ignored; BT-112 is GOBL `totals.total_with_tax` (compared, warning log, §1.6) |
| `-` | BT-107 Sum of allowances on document level, BT-108 Sum of charges on document level | `/Invoice/cac:LegalMonetaryTotal/cbc:AllowanceTotalAmount`, `/Invoice/cac:LegalMonetaryTotal/cbc:ChargeTotalAmount` | gap: UAPI has no document-level allowances or charges (BG-20/BG-21) |
| `-` | BT-113 Paid amount | `/Invoice/cac:LegalMonetaryTotal/cbc:PrepaidAmount` | from non-OUTSTANDING payments (§1.4) |
| `-` | BT-114 Rounding amount | `/Invoice/cac:LegalMonetaryTotal/cbc:PayableRoundingAmount` | derived: GOBL `totals.rounding` (`ubl:totals.go:74-76`), set inside the converter. `RemoveIncludedTaxes` and `RoundToCurrency` recalculate while keeping the original payable amount (gobl v0.511 `bill/calculator.go:337-338`, `bill/rounding.go:33-68`, `239-258`), so VAT-inclusive UAPI input can produce a ±0.01 BT-114 that keeps BT-115 at the gross total (converter sample `peppol/out/invoice-prices-include-vat.xml:93`) |
| `-` | BT-115 Amount due for payment | `/Invoice/cac:LegalMonetaryTotal/cbc:PayableAmount` | `totals.due` when advances exist, else `totals.payable` (`ubl:totals.go:61-82`) |
| `-` | BT-111 Invoice total VAT amount in accounting currency | `/Invoice/cac:TaxTotal/cbc:TaxAmount[@currencyID = /Invoice/cbc:TaxCurrencyCode]` | Not needed: BE suppliers are EUR-only (BT-5). It would sit in a second `cac:TaxTotal` without subtotals (PEPPOL-EN16931-R054, BR-53); gobl.ubl writes it only together with BT-6 (`ubl:invoice.go:141-148`) |

### 1.6 Calculated Fields

| UAPI | EN 16931 BT | BIS 3.0 UBL XPath | formula | notes |
| --- | --- | --- | --- | --- |
| `entries[i].data.unit.price.inclusive`, `entries[i].data.vat.percentage` | BT-146 Item net price | `/Invoice/cac:InvoiceLine/cac:Price/cbc:PriceAmount` | `gross / (1 + rate/100)` | GOBL `RemoveIncludedTaxes` (`ubl:ubl.go:161`); precision kept |
| `entries[i].data.unit.quantity`, price, discount, surcharge | BT-131 Invoice line net amount | `/Invoice/cac:InvoiceLine/cbc:LineExtensionAmount` | `round2(qty × net price − allowances + charges)` | rounded per line by GOBL's `currency` rule, which fiskaly sets on the document (`G/invoice.go:160-168`). The converter keeps a rule set on the document: its switch to `precise` applies only when none is set (gobl v0.511 `bill/calculator.go:329-335`), so it never applies to fiskaly invoices, and `RoundToCurrency` (`ubl:ubl.go:164`) only re-rounds amounts that still exceed the currency precision. PEPPOL-EN16931-R120 tolerance 0.02 |
| all lines | BT-106 Sum of Invoice line net amount | `/Invoice/cac:LegalMonetaryTotal/cbc:LineExtensionAmount` | `Σ BT-131` | BR-CO-10 |
| — | BT-109 Invoice total amount without VAT | `/Invoice/cac:LegalMonetaryTotal/cbc:TaxExclusiveAmount` | `BT-106 − BT-107 + BT-108` (= BT-106 here) | BR-CO-13 |
| lines grouped by BT-151 + BT-152 | BT-116 VAT category taxable amount | `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cbc:TaxableAmount` | `Σ BT-131 per category and rate` | BR-S-08, BR-Z-08, BR-E-08, BR-O-08 |
| — | BT-117 VAT category tax amount | `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cbc:TaxAmount` | `round2(BT-116 × BT-119 / 100)`; 0 for E/Z/O | BR-S-09 |
| — | BT-110 Invoice total VAT amount | `/Invoice/cac:TaxTotal/cbc:TaxAmount` | `Σ BT-117` | BR-CO-14 |
| — | BT-112 Invoice total amount with VAT | `/Invoice/cac:LegalMonetaryTotal/cbc:TaxInclusiveAmount` | `BT-109 + BT-110` | BR-CO-15 |
| `payments[i].details.amount` (non-OUTSTANDING) | BT-113 Paid amount | `/Invoice/cac:LegalMonetaryTotal/cbc:PrepaidAmount` | `Σ advances` (2 dp each) | `G/invoice.go:623-636` |
| — | BT-115 Amount due for payment | `/Invoice/cac:LegalMonetaryTotal/cbc:PayableAmount` | `BT-112 − BT-113 + BT-114` | BR-CO-16 |
| `payments[*].details.amount` | BT-146 Item net price (indirect) | `/Invoice/cac:InvoiceLine/cac:Price/cbc:PriceAmount` | line prices nudged by cents until GOBL `totals.total_with_tax` = Σ payments (each rounded to 2 dp), only when all payments share the invoice currency, the drift is pure rounding and no price turns negative | `G/line_rounding.go:21-185`; can move BT-131 and so R120/BR-CO-10 |
| `totals.vat.*` | BT-109 Invoice total amount without VAT, BT-110 Invoice total VAT amount, BT-112 Invoice total amount with VAT | `/Invoice/cac:LegalMonetaryTotal/cbc:TaxExclusiveAmount`, `/Invoice/cac:TaxTotal/cbc:TaxAmount`, `/Invoice/cac:LegalMonetaryTotal/cbc:TaxInclusiveAmount` | difference UAPI vs GOBL logged | warning only (`E/invopop.go:753-782`) |

## 2. Supplier Data (Taxpayer, System)

| UAPI | EN 16931 BT | BIS 3.0 UBL XPath | notes |
| --- | --- | --- | --- |
| taxpayer `name.legal` | BT-27 Seller name | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PartyLegalEntity/cbc:RegistrationName` | `supplier.name` (`G/taxpayer.go:62`; `ubl:party.go:207-216`). API docs: BT-44 |
| taxpayer `name.legal` | BT-28 Seller trading name | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PartyName/cbc:Name` | derived: copy of BT-27 |
| taxpayer `name.trade` | BT-28 Seller trading name | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PartyName/cbc:Name` | ignored. Proposal: GOBL `alias`, which gobl.ubl writes here (`ubl:party.go:280-284`) |
| taxpayer `name.person.*` (INDIVIDUAL) | BT-27 Seller name | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PartyLegalEntity/cbc:RegistrationName` | ignored; `name.legal` used |
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
| taxpayer `fiscalization.type` = BE | — | — | GOBL regime BE: the supplier needs a tax ID code or a `BCE`-type identity (`gobl:regimes/be/bill_invoices.go:12-36`); fiskaly always sends the VAT ID, so the tax-ID branch applies |
| taxpayer `fiscalization.credentials.type`, `type` | — | — | ignored |
| `-` | BT-29 Seller identifier | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PartyIdentification/cbc:ID` | gap; this element only carries BT-90 (`SEPA`) today |
| `-` | BT-33 Seller additional legal information | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PartyLegalEntity/cbc:CompanyLegalForm` | gap |
| system annotation `peppol_id` | BT-34 Seller electronic address | `/Invoice/cac:AccountingSupplierParty/cac:Party/cbc:EndpointID` | supplier inbox `peppol`. fiskaly writes `peppol_id` on the system once the Peppol registration workflow (`BE_REGISTER_TAXPAYER_PEPPOL`) confirms proof of ownership (`src/unified/domain/system/einvoicing_adaptations.go:148`, called from `SV1:1197`) (`G/inbox.go:17-35`; `E/service.go:149-179`; `W:284-298`). API docs: BT-49 "of the recipient" (a DE_EI note on the schema shared with `invoicing.identifier`; no BE hint). R020, BR-62. Absent at sync validation (`E/invopop.go:334`) |
| system annotation `peppol_id` (first 4 digits) | BT-34-1 Seller electronic address scheme | `/Invoice/cac:AccountingSupplierParty/cac:Party/cbc:EndpointID/@schemeID` | derived EAS code (BE usually `0208` or `9925`); PEPPOL-COMMON-R043 for `0208` |
| `seller.name` | BT-41 Seller contact point | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:Contact/cbc:Name` | `supplier.people[0].name.given` (`G/invoice.go:479-485`); the API description ("legal or trading name") does not match BT-41 |
| `seller.phone` | BT-42 Seller contact telephone number | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:Contact/cbc:Telephone` | `G/invoice.go:486-489` |
| `seller.email` | BT-43 Seller contact email address | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:Contact/cbc:ElectronicMail` | `G/invoice.go:490-493`; never used as endpoint in this flow (that fallback is XRechnung-only) |
| `tax_representative.name` | BT-62 Seller tax representative name | `/Invoice/cac:TaxRepresentativeParty/cac:PartyName/cbc:Name` | `ordering.seller.name` (`G/invoice.go:387-398`, `499-519`; `ubl:ordering.go:85-89`); gobl.ubl also writes `PartyLegalEntity/cbc:RegistrationName` → UBL-CR-335 warning |
| `tax_representative.vat_number` | BT-63 Seller tax representative VAT identifier | `/Invoice/cac:TaxRepresentativeParty/cac:PartyTaxScheme/cbc:CompanyID` | country + code, TaxScheme `VAT` |
| `tax_representative.country` | BT-69 Tax representative country code | `/Invoice/cac:TaxRepresentativeParty/cac:PostalAddress/cac:Country/cbc:IdentificationCode` | derived: GOBL keeps only the tax-ID country; gobl.ubl writes it as postal country (`ubl:party.go:250-256`), so BT-69 is present (BR-19, BR-20) |
| `-` | BT-64 Tax representative address line 1, BT-65 Tax representative address line 2, BT-66 Tax representative city, BT-67 Tax representative post code, BT-68 Tax representative country subdivision | `/Invoice/cac:TaxRepresentativeParty/cac:PostalAddress/cbc:StreetName`, `/Invoice/cac:TaxRepresentativeParty/cac:PostalAddress/cbc:AdditionalStreetName`, `/Invoice/cac:TaxRepresentativeParty/cac:PostalAddress/cbc:CityName`, `/Invoice/cac:TaxRepresentativeParty/cac:PostalAddress/cbc:PostalZone`, `/Invoice/cac:TaxRepresentativeParty/cac:PostalAddress/cbc:CountrySubentity` | gap: UAPI has no tax-representative address |
| location `name`, `address.*`, `coordinates.*`, `label` | — | — | ignored |
| system `software.*`, `registrations[]` | — | — | stored registrations gate PEPPOL at sync; systems without annotations or stored registrations skip the check and fail in the worker if `peppol_id` is missing (`EA:145-186`); no XML element |

## 3. Correction (Credit Note)

| UAPI | EN 16931 BT | BIS 3.0 UBL XPath | notes |
| --- | --- | --- | --- |
| `type: CORRECTION`, `data.type: INVOICE` | BT-3 Invoice type code | `/CreditNote/cbc:CreditNoteTypeCode` | GOBL `credit-note` (`W:196-197`) → `CreditNote`, `381` (`ubl:invoice.go:150-156`; `gobl:addons/eu/en16931/scenarios.go:28-35`); P0101 allows 381. No `Invoice` + 381 variant; 384 fails P0112 outside DE–DE/SK–SK. An upward correction is also sent as 381 with positive amounts: it passes validation and silently credits the buyer (no 383 path) |
| `record.id` | BG-3 Preceding Invoice reference | `/CreditNote/cac:BillingReference` | original record's `UsedIn` → original GOBL invoice fetched from Invopop (`W:208-218`; `E/invopop.go:1104-1129`) |
| `record.id` → original `code` | BT-25 Preceding Invoice reference | `/CreditNote/cac:BillingReference/cac:InvoiceDocumentReference/cbc:ID` | derived: code stored by Invopop, i.e. what was sent (follows any PPL override) (`ubl:ordering.go:59-61`). BR-55 |
| `record.id` → original `issue_date` | BT-26 Preceding Invoice issue date | `/CreditNote/cac:BillingReference/cac:InvoiceDocumentReference/cbc:IssueDate` | derived |
| `data.document.references.preceding_document.*` | BT-25 Preceding Invoice reference, BT-26 Preceding Invoice issue date | `/CreditNote/cac:BillingReference/cac:InvoiceDocumentReference/cbc:ID`, `/CreditNote/cac:BillingReference/cac:InvoiceDocumentReference/cbc:IssueDate` | ignored, warning log (`G/invoice.go:191-205`; `E/invopop.go:690-707`) |
| `record.offset` | — | — | ignored |
| `details.creators[]`, `details.training`, `details.properties` (CORRECTION level, not `data.details`) | — | — | ignored (record metadata) |
| `reason.text` | BT-22 Invoice note | `/CreditNote/cbc:Note` | GOBL note `key: reason` (`G/invoice.go:215-222`) → subject `ACD` (`gobl:addons/eu/en16931/org.go:28`) → `#ACD#<text>` (`ubl:utils.go:77-86`). The prefix encodes BT-21 Invoice note subject code, which has no element in the BIS 3.0.21 binding; it is the CEN UBL convention and passes BR-CL-08 |
| `reason.type` | BT-22 Invoice note | `/CreditNote/cbc:Note` | default: without `reason.text` the raw enum (e.g. `FACTUAL_ERROR`) is the note (`src/unified/service/einvoicing/correction.go:8-14`). `text` is required for OTHER, so the fallback covers the other types; without `reason` there is no note (`W:205`) |
| `data.payments[i].details.date` (OUTSTANDING) | BT-9 Payment due date | `/CreditNote/cac:PaymentMeans/cbc:PaymentDueDate` | only when an instruction exists (`ubl:payment.go:188-191`), else dropped; CreditNote has no `cbc:DueDate` |
| `data.document.references.project` | BT-11 Project reference | `/CreditNote/cac:AdditionalDocumentReference[cbc:DocumentTypeCode='50']/cbc:ID` | as-built `cac:ProjectReference` (`ubl:ordering.go:124-129`), not defined in `UBL-CreditNote-2.1.xsd` → schema failure. Inferred, not executed; UNVERIFIED for Invopop production |
| `data.document.references.tender` + `.contract` | BT-17 Tender or lot reference, BT-12 Contract reference | `/CreditNote/cac:OriginatorDocumentReference/cbc:ID`, `/CreditNote/cac:ContractDocumentReference/cbc:ID` | The CreditNote schema orders Contract, Additional, Statement, Originator; the as-built order puts Originator first (`ubl:invoice.go:69-75`), so a schema order error follows when both are set. Inferred; UNVERIFIED for Invopop production |
| `data.entries[i]` | BG-25 Invoice line | `/CreditNote/cac:CreditNoteLine` | §1.3; quantity in `cbc:CreditedQuantity` (`ubl:lines.go:241-245`); `entries[i].type` ignored |
| amount signs | BT-129 Invoiced quantity, BT-131 Invoice line net amount, BT-146 Item net price, BT-115 Amount due for payment | `/CreditNote/cac:CreditNoteLine/cbc:CreditedQuantity`, `/CreditNote/cac:CreditNoteLine/cbc:LineExtensionAmount`, `/CreditNote/cac:CreditNoteLine/cac:Price/cbc:PriceAmount`, `/CreditNote/cac:LegalMonetaryTotal/cbc:PayableAmount` | as supplied, nothing negated (`G/line_rounding.go:110-113` only). API docs instruct RETURN lines with positive amounts on CORRECTION (guidance only; the amount patterns allow `-`), which matches the BIS credit-note convention. Negative quantities allowed; negative unit price fails BR-27 |
| `data.recipients[]` | BG-7 Buyer | `/CreditNote/cac:AccountingCustomerParty` | must equal the original's (`EA:306-399`) |
| `data.document.number` | BT-1 Invoice number | `/CreditNote/cbc:ID` | credit-note number; PPL caveat (§4); same workflow as the original (`E/invopop.go:398-404`) |

## 4. Fixed and Derived Values

| UAPI | EN 16931 BT | BIS 3.0 UBL XPath | notes |
| --- | --- | --- | --- |
| `-` | BT-24 Specification identifier | `/Invoice/cbc:CustomizationID` | fixed `urn:cen.eu:en16931:2017#compliant#urn:fdc:peppol.eu:2017:poacc:billing:3.0` |
| `-` | BT-23 Business process type | `/Invoice/cbc:ProfileID` | fixed `urn:fdc:peppol.eu:2017:poacc:billing:01:1.0` |
| `-` | BT-3 Invoice type code | `/Invoice/cbc:InvoiceTypeCode`, `/CreditNote/cbc:CreditNoteTypeCode` | fixed `380` / `381`. Never `389`: fiskaly sets no `self-billed` tag, so gobl.ubl never selects Peppol Self-Billing (`ubl:format.go:316-330`) |
| `document.number` | BT-1 Invoice number | `/Invoice/cbc:ID` | UNVERIFIED override. Before signing, the workflow runs `sequence.enumerate` `{name: PPL, prefix: PPL, padding: 6, start: 1}`, summary `Dynamic · PPL · PPL000001` (`WF/BE_SEND_INVOICE_PEPPOL_OASIS_UBL_BUSINESS.json:15-26`). Invopop docs: the step "generates the next number in the sequence and adds it to your document"; leave `code` empty with such a workflow unless your own system owns the numbering. Nothing documents what happens to a pre-set `code`; fiskaly always sets `code`, never `series`. If overwritten, BT-1 = `PPL000001`-style from a workspace series, not the taxpayer's number |
| `-` | BT-2 Invoice issue date | `/Invoice/cbc:IssueDate` | default today when `issued_at` is absent |
| `-` | BT-13 Purchase order reference | `/Invoice/cac:OrderReference/cbc:ID` | default `NA` when no BT-10 and no BT-13 (R003) |
| `-` | BT-72 Actual delivery date | `/Invoice/cac:Delivery/cbc:ActualDeliveryDate` | default issue date when no `operation_date`; always emitted |
| `-` | BT-28 Seller trading name, BT-45 Buyer trading name | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PartyName/cbc:Name`, `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PartyName/cbc:Name` | derived: copies of BT-27 / BT-44 |
| `-` | BT-40 Seller country code, BT-55 Buyer country code, BT-69 Tax representative country code | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PostalAddress/cac:Country/cbc:IdentificationCode`, `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PostalAddress/cac:Country/cbc:IdentificationCode`, `/Invoice/cac:TaxRepresentativeParty/cac:PostalAddress/cac:Country/cbc:IdentificationCode` | derived from the tax-ID country (`ubl:party.go:250-256`) |
| `-` | — | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PartyTaxScheme/cac:TaxScheme/cbc:ID`, `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PartyTaxScheme/cac:TaxScheme/cbc:ID`, `/Invoice/cac:InvoiceLine/cac:Item/cac:ClassifiedTaxCategory/cac:TaxScheme/cbc:ID`, `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cac:TaxCategory/cac:TaxScheme/cbc:ID` | tax scheme, no BT: fixed `VAT` for the buyer, default `VAT` for the seller |
| `-` | BT-130 Invoiced quantity unit of measure code | `/Invoice/cac:InvoiceLine/cbc:InvoicedQuantity/@unitCode` | default `C62` |
| `-` | BT-139 Invoice line allowance reason, BT-144 Invoice line charge reason | `/Invoice/cac:InvoiceLine/cac:AllowanceCharge[cbc:ChargeIndicator='false']/cbc:AllowanceChargeReason`, `/Invoice/cac:InvoiceLine/cac:AllowanceCharge[cbc:ChargeIndicator='true']/cbc:AllowanceChargeReason` | fixed `Special Discount` / `Special Surcharge` |
| `-` | BT-151 Invoiced item VAT category code, BT-118 VAT category code, BT-152 Invoiced item VAT rate, BT-119 VAT category rate | `/Invoice/cac:InvoiceLine/cac:Item/cac:ClassifiedTaxCategory/cbc:ID`, `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cac:TaxCategory/cbc:ID`, `/Invoice/cac:InvoiceLine/cac:Item/cac:ClassifiedTaxCategory/cbc:Percent`, `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cac:TaxCategory/cbc:Percent` | derived `S`/`Z`/`E`/`O`; Percent `0` for E and Z, omitted for O |
| `-` | BT-81 Payment means type code | `/Invoice/cac:PaymentMeans/cbc:PaymentMeansCode` | fixed `30` (credit transfer) / `49` (direct debit) |
| `-` | BT-90 Bank assigned creditor identifier | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PartyIdentification/cbc:ID/@schemeID` | fixed `SEPA` (scheme attribute) |
| `-` | BT-126 Invoice line identifier | `/Invoice/cac:InvoiceLine/cbc:ID` | derived: 1-based array index |
| `-` | BT-21 Invoice note subject code | — | No element in the BIS 3.0.21 binding. As-built, correction reasons carry `#ACD#` as a prefix inside BT-22 (§3) |
| `-` | BT-22 Invoice note | `/Invoice/cbc:Note` | derived: several notes merged into one (`ubl:peppol.go:17-20`) |
| `-` | BT-34-1 Seller electronic address scheme, BT-49-1 Buyer electronic address scheme | `/Invoice/cac:AccountingSupplierParty/cac:Party/cbc:EndpointID/@schemeID`, `/Invoice/cac:AccountingCustomerParty/cac:Party/cbc:EndpointID/@schemeID` | derived: first 4 digits of the Peppol identifier |
| `-` | BT-146 Item net price, BT-113 Paid amount | `/Invoice/cac:InvoiceLine/cac:Price/cbc:PriceAmount`, `/Invoice/cac:LegalMonetaryTotal/cbc:PrepaidAmount` | derived: net price from gross (§1.6); sum of non-OUTSTANDING payments |

## 5. Format Elements Without a UAPI Source (Gaps)

Mandatory in BIS 3.0.21 (cardinality 1..x or made mandatory by a rule):

| BT | element | cardinality / rule | status today | proposal |
| --- | --- | --- | --- | --- |
| BT-1 | `/Invoice/cbc:ID` | 1..1, BR-02 | from `document.number`; PPL override UNVERIFIED | confirm with Invopop; if it overwrites, drop the step from fiskaly's Peppol workflows or map `document.series` |
| BT-10 or BT-13 | `/Invoice/cbc:BuyerReference`, `/Invoice/cac:OrderReference/cbc:ID` | PEPPOL-EN16931-R003 | met by the `NA` filler when the client sends neither | keep; consider requiring `references.buyer` for public-sector receivers |
| BT-34 | `/Invoice/cac:AccountingSupplierParty/cac:Party/cbc:EndpointID` | 1..1, R020, BR-62 | from annotation `peppol_id`; missing at sync, worker error if absent | pass the supplier inbox to sync validation (`E/invopop.go:334`) |
| BT-49 | `/Invoice/cac:AccountingCustomerParty/cac:Party/cbc:EndpointID` | 1..1, R010, BR-63 | from `invoicing.identifier` | — |
| BT-29 / BT-30 / BT-31 | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PartyIdentification/cbc:ID`, `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PartyLegalEntity/cbc:CompanyID`, `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PartyTaxScheme[cac:TaxScheme/cbc:ID='VAT']/cbc:CompanyID` | BR-CO-26 | BT-31 always present | add BT-30 (KBO/BCE, `@schemeID` `0208`, 10 digits); placement in BT-30 UNVERIFIED (see §2) |
| BT-120 or BT-121 | `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cac:TaxCategory/cbc:TaxExemptionReason`, `/Invoice/cac:TaxTotal/cac:TaxSubtotal/cac:TaxCategory/cbc:TaxExemptionReasonCode` | BR-E-10, BR-O-10 | BT-120 from mandatory `vat.reason`; BT-121 never | map BE exemption codes to VATEX, as the DE branch does |
| BT-151 = AE (+ BT-31/BT-32/BT-63 and BT-48/BT-47, BR-AE-02; BT-120/121, BR-AE-10) | `/Invoice/cac:InvoiceLine/cac:Item/cac:ClassifiedTaxCategory/cbc:ID` | BR-AE-01…10 | VAT_REVERSE_CHARGE fails the GOBL build, so Belgian domestic reverse-charge supplies and intra-EU B2B services (customer liable, Art. 196 Directive 2006/112/EC) cannot be invoiced | add an AE mapping with VATEX-EU-AE |
| BT-151 = K / G | `/Invoice/cac:InvoiceLine/cac:Item/cac:ClassifiedTaxCategory/cbc:ID` | BR-IC-*, BR-G-* | unreachable: every BE exemption code becomes `E` or `O` | map intra-EU supply (Art. 138) and export causes to `K` / `G` with VATEX-EU-IC / VATEX-EU-G. `K` also needs BT-80 Deliver to country code (BR-IC-12), present only when `shipping.address` is sent; BR-IC-11 is met by the always-emitted BT-72 |
| BT-151 = O | `/Invoice/cac:InvoiceLine/cac:Item/cac:ClassifiedTaxCategory/cbc:ID` | BR-O-02, BR-O-11 | NOT_SUBJECT → `O` always fails, since BT-31 and BT-48 are always present | for BE map NOT_SUBJECT to `E` with reason (as `G/german_exemption.go:57-59` does for NOT_TAXABLE), or omit VAT IDs on genuine out-of-scope invoices |
| BT-9 or BT-20 | `/Invoice/cbc:DueDate`, `/Invoice/cac:PaymentTerms/cbc:Note` | BR-CO-25 (EN 16931; not in the 3.0.21 UBL schematrons) | enforced by GOBL en16931 rule 07 inside Invopop, after `COMPLETED`; rule 06 cannot fire, since `detailsFrom` always returns payment details or an error (`G/invoice.go:262-325`) | add `eu-en16931-v2017` at fiskaly time for BE so it fails at the sync check, before anything is sent |
| BT-69 (BG-12) | `/Invoice/cac:TaxRepresentativeParty/cac:PostalAddress/cac:Country/cbc:IdentificationCode` | 1..1 within BG-12, BR-19, BR-20 | derived from the VAT country; BT-64…BT-68 absent | add a tax-representative address |

Important optional elements UAPI cannot express or the pipeline drops:

| BT | element | cardinality / rule | status today | proposal |
| --- | --- | --- | --- | --- |
| BT-22 | `/Invoice/cbc:Note` | 0..1, R002 | `document.text` ignored | map to a GOBL note |
| BT-7 | `/Invoice/cbc:TaxPointDate` | 0..1, BR-CO-03 | `operation_date` never reaches it (§1.1) | GOBL `value_date` if the operation date is the chargeability date |
| BT-11 (credit note) | `/CreditNote/cac:AdditionalDocumentReference[cbc:DocumentTypeCode='50']/cbc:ID` | 0..1 | `cac:ProjectReference` → XSD (inferred) | fix converter or omit on corrections |
| BT-28 | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PartyName/cbc:Name` | 0..1 | copy of BT-27 | map `name.trade` (GOBL `alias`) |
| BT-30 | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PartyLegalEntity/cbc:CompanyID` | 0..1 | `tax_id_number` ignored | legal-scope identity with `iso-scheme-id: 0208`, number left-padded to 10 digits (§2) |
| BT-33 | `/Invoice/cac:AccountingSupplierParty/cac:Party/cac:PartyLegalEntity/cbc:CompanyLegalForm` | 0..1 | no source | legal form / register mentions (Belgian company-law mentions, CSA art. 2:20: UNVERIFIED) |
| BT-46 / BT-47 | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PartyIdentification/cbc:ID`, `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:PartyLegalEntity/cbc:CompanyID` | 0..1, UBL-SR-16/17 | both in `PartyIdentification`, no `@schemeID` | ICD fields; `company_id` legal scope |
| BT-56 … BT-58 | `/Invoice/cac:AccountingCustomerParty/cac:Party/cac:Contact/*` | 0..1 | no source | buyer contact fields |
| BT-82 / BT-83 | `/Invoice/cac:PaymentMeans/cbc:PaymentMeansCode/@name`, `/Invoice/cac:PaymentMeans/cbc:PaymentID` | 0..1 | `text` → BT-82, `properties` → BT-83 | `text` → BT-83 as documented |
| BT-87 / BT-88 | `/Invoice/cac:PaymentMeans/cac:CardAccount/*` | 0..1, UBL-SR-54 | card payments become advances | BG-18 with code 48/54/55, plus `cbc:NetworkID` (required by UBL/BIS, no BT) |
| BT-127 | `/Invoice/cac:InvoiceLine/cbc:Note` | 0..1 | `details.description` ignored | map |
| BT-149 | `/Invoice/cac:InvoiceLine/cac:Price/cbc:BaseQuantity` | 0..1, R121, R130 | `unit.factor` ignored; no `item.per` in GOBL v0.501.0 | map after GOBL upgrade |
| BT-154/155/157/158/160/161 | `/Invoice/cac:InvoiceLine/cac:Item/*` | 0..1 / 0..n | product data ignored | map `product.*` (§1.3) |
| BT-14/15/18/71/73/74/128/132–135, BG-10, BG-20/21 | various | 0..x | no UAPI source | on demand |
| BG-24 | `/Invoice/cac:AdditionalDocumentReference/cac:Attachment/cbc:EmbeddedDocumentBinaryObject` (BT-125), `/Invoice/cac:AdditionalDocumentReference/cbc:ID` (BT-122, mandatory, BR-52) | 0..n | no PDF in the Peppol workflow | optional `attach_invoice_pdf` |

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
| Peppol BIS Billing 3.0 syntax binding | OpenPEPPOL/peppol-bis-invoice-3 tag `3.0.21`: `structure/syntax/ubl-invoice.xml`, `ubl-creditnote.xml`, `structure/syntax/part/*.xml` — https://github.com/OpenPEPPOL/peppol-bis-invoice-3/tree/3.0.21 | 2026-11-03 |
| Peppol BIS guide appendices and code lists | same tag: `guide/bis/appendix/national_rules.adoc`, `guide/cius/rules/peppol_rules.adoc`, `structure/codelist/eas.xml`, `structure/codelist/icd.xml` | 2026-11-03 |
| Peppol BIS schematrons | `PEPPOL-EN16931-UBL.sch`, `CEN-EN16931-UBL.sch` (CEN schematron 1.3.16, last update 2026-04-10), `PeppolBIS-Envelope-Rules.sch` — formats repo `Peppol BIS/peppol-bis-invoice-3-3.0.21/rules/sch/` (tag 3.0.21) | 2026-10-29 |
| Peppol BIS release status | "May 2026 Release" = 3.0.21, mandatory from 2026-08-17 — https://docs.peppol.eu/poacc/billing/3.0/release-notes/ (regulatory baseline row verified 2026-10-29) | 2026-10-29 |
| UBL 2.1 schemas | OASIS UBL 2.1 `UBL-Invoice-2.1.xsd`, `UBL-CreditNote-2.1.xsd` (local copy: XRechnung 3.0.2 validator configuration 2026-08-31, `resources/ubl/2.1/xsd/maindoc/`) | 2026-11-03 |
| VAT Directive | Council Directive 2006/112/EC, Art. 138, 196, 226(7) — https://eur-lex.europa.eu/eli/dir/2006/112/oj | 2026-11-03 |
| Belgian mandate context | regulatory baseline, Belgium section: B2B Peppol mandate in force since 2026-01-01 (CTVA art. 53 §2bis; loi du 6 février 2024; AR n° 1 art. 13ter; AR du 8 juillet 2025; loi du 10 février 2026) | verified 2026-10-29 |
| UAPI v6 field list | [UAPI_v6_Fields.md](UAPI_v6_Fields.md) from `fiskaly/meta@cd19ec88cb299ee41a44302448006340dd028bf0` `src/unified/api/v6/definition.yaml`; API docs `2026-06-01` | 2026-11-03 |
| fiskaly as-built code | `fiskaly/meta@cd19ec8` (paths in the code-citation legend above; workflows `BE_SEND_INVOICE_PEPPOL_OASIS_UBL_BUSINESS.json`, `BE_REGISTER_TAXPAYER_PEPPOL.json`, `DE_SEND_INVOICE_PEPPOL_XRECHNUNG_BUSINESS.json`) | 2026-11-03 |
| GOBL | invopop/gobl v0.501.0, commit `225e39d` (fiskaly's pin); v0.511.0 for converter-side code | 2026-11-03 |
| GOBL → UBL converter | invopop/gobl.ubl v0.99.0, commit `540f201` (2026-10-28), incl. `test/data/convert/peppol/out/*.xml` | 2026-11-03 |
| Invopop documentation (as-built behaviour only, not compliance) | https://docs.invopop.com/guides/series.md, https://docs.invopop.com/console/series.md, https://docs.invopop.com/llms, https://docs.invopop.com/guides/peppol.md | 2026-11-03 |
| As-built trace | internal working note "UAPI v6 → GOBL bill.Invoice: as-built trace", spot-checked against the code | 2026-11-03 |

## Verification Log

| # | Claim | How checked | Result |
| --- | --- | --- | --- |
| 1 | Emitted CustomizationID and ProfileID equal the 3.0.21 values | `ubl:format.go:16`, `299-300` vs BT-24/BT-23 DEFAULT values in `ubl-invoice.xml` and R004/R001/R007; converter sample `credit-note-peppol.xml` | CONFIRMED |
| 2 | fiskaly sets no addon for BE; EN 16931 GOBL rules first run in the converter | `E/invopop.go:300-319`; `G/addon.go:19-30`; `ubl:ubl.go:157`, `191-220` | CONFIRMED (code); production converter version UNVERIFIED |
| 3 | `sequence.enumerate` (PPL) overwrites `document.number` | workflow JSON lines 15-26; Invopop series guide, console series page, LLM guide: none covers a pre-set `code` | UNVERIFIED |
| 4 | PEPPOL to a recipient outside BE/DE is rejected; default channel follows the recipient address country | `RH:3257-3279`, `3312-3336`, `3508` | CONFIRMED for the PEPPOL rejection; default channel UNVERIFIED (the schema requires `invoicing.type`, so the fallback is likely unreachable) |
| 5 | VAT ID always wins; BT-32 and BT-30 never emitted for BE | `G/taxpayer.go:37-45`, `61-71`; `ubl:party.go:286-320` | CONFIRMED |
| 6 | VAT_REVERSE_CHARGE, MEAL_VOUCHER, DEPOSIT rejected | `G/entries.go:164-169`; `G/invoice.go:300-302` | CONFIRMED |
| 7 | Corrections become `CreditNote`/381 with BG-3 from the Invopop copy of the original | `W:196-218`; `E/invopop.go:1104-1129`; `ubl:invoice.go:150-156`; `ubl:ordering.go:53-72` | CONFIRMED |
| 8 | `document.operation_date` reaches BT-72, never BT-7 | `G/invoice.go:152`, `433-441`; `ubl:invoice.go:158-161`; `ubl:delivery.go:38-41` | CONFIRMED |
| 9 | `instruction.text` → BT-82 (`@name`), `instruction.properties` → BT-83 | `G/invoice.go:538-543`; `ubl:payment.go:157-162` | CONFIRMED |
| 10 | NOT_SUBJECT fails BR-O-02; `buyer_id` + `company_id` fail UBL-SR-16 | `G/entries.go:175-189` + CEN BR-O-02 test; `G/recipient.go:158-172` + `ubl:party.go:322-349` + CEN UBL-SR-16 (fatal); code reading, not executed | CONFIRMED (code reading) |
| 11 | No Belgian national rules; German rules need DE supplier and DE customer; BR-CO-25 absent from the 3.0.21 schematrons | national prefixes in `PEPPOL-EN16931-UBL.sch` (DE, DK, GR, IS, IT, NL, NO, SE); `german-rules` context; grep `BR-CO-25` in all five `.sch`: 0 hits | CONFIRMED |
| 12 | Credit note with project (or tender + contract) breaks the UBL 2.1 CreditNote schema | `ubl:invoice.go:69-75`, `ubl:ordering.go:124-129` vs `UBL-CreditNote-2.1.xsd` sequence | schema side CONFIRMED (no `cac:ProjectReference`; Contract before Originator); emission inferred, not executed |


### Independent verification pass (2026-11-03)

Two separate checkers re-tested the decision-critical claims against code (fiskaly `cd19ec8`, gobl v0.501.0, the vendored converters) and against primary sources. Neither was given the page.

| Claim | Verdict | Evidence / correction |
| --- | --- | --- |
| `VAT_REVERSE_CHARGE`, `MEAL_VOUCHER`, `DEPOSIT` rejected in every country | CONFIRMED | `G/entries.go:164-169`; `G/invoice.go:300-301` |
| VAT ID beats tax ID; BT-30/BT-32 never emitted for BE | CONFIRMED | `G/taxpayer.go:38-43`. BE fiscalization requires both IDs, so the fallback that sends `tax_id_number` as BT-31 does not occur for BE |
| `NOT_SUBJECT` → `O` while BT-31/BT-48 are present (BR-O-02) | CONFIRMED | `G/entries.go:179-180` (`KeyOutsideScope`); `gobl:addons/eu/en16931/tax_combo.go:41`; no converter suppresses the VAT IDs. The job failing at Invopop (`validate_ubl: true`) is inferred |
| `buyer_id` + `company_id` → UBL-SR-16 | CONFIRMED | `G/recipient.go:159-172`; `ubl:party.go:322-349`; neither identity carries `@schemeID` |
| CustomizationID / ProfileID equal 3.0.21 | CONFIRMED | PEPPOL-EN16931-R004 value; `ubl-invoice.xml` default ProfileID (R001/R007) |
| No Belgian national rules except PEPPOL-COMMON-R043 | CONFIRMED | National blocks only for DE, DK, GR, IS, IT, NL, NO, SE; R043 checks 10 digits plus mod 97 for any `@schemeID='0208'`; `icd.xml` and `eas.xml` list 0208 (KBO/BCE) |
| KBO/BCE belongs in BT-30 | **UNVERIFIED** | No Belgian primary source found. Peppol accepts 0208 in BT-29/30/34/46/47/49. Marked in §2 and §5 |
| `operation_date` → BT-72 only; `instruction.text` → BT-82 | CONFIRMED | `G/invoice.go:433-441`, `538-540`; `ubl:payment.go:157-161` |
| CORRECTION → `CreditNote`/381, BG-3 from Invopop | CONFIRMED | `W:196-197`; `E/invopop.go:1104-1129` |
| PPL `sequence.enumerate` effect on BT-1 | UNVERIFIABLE | The step exists (`WF/BE_SEND_INVOICE_PEPPOL_OASIS_UBL_BUSINESS.json:16-25`, before `silo.close`). fiskaly always sets `code` and never `series`; Invopop's docs say dynamic sequences need a `series`. Check a sandbox silo entry or ask Invopop |

### Validation pass (2026-11-03)

Four checks re-read every row against the pinned sources: fiskaly code at `cd19ec8`, gobl v0.501.0 and gobl.ubl v0.99.0, the BIS 3.0.21 bindings, schematrons and UBL 2.1 XSDs, and the UAPI v6 definition with the 2026-06-01 API docs. Rows not listed held as written.

| Area | Corrected |
| --- | --- |
| Currency | BE suppliers are EUR-only at record validation (`src/unified/domain/record/capabilities.go:188-220`), so BT-6/BT-111 are not a gap today; the gap row was removed |
| Recipients | CONSUMER with PEPPOL fails on the SDI destination code, not on `consumer recipients only support SDI invoicing`; the "exactly 1 business recipient" trigger and the two `FindInvoicingRecipient` errors; the default-channel fallback is likely unreachable (`invoicing.type` is schema-required); stage A errors are 501/400, not a generic 4xx |
| Supplier | BE regime needs a tax ID code or a `BCE` identity, not always a tax ID; BT-30 proposal needs the `iso-scheme-id: 0208` extension; the Peppol registration check is skipped for systems without stored registrations |
| Rounding | BT-114 can carry ±0.01 from the converter's recalculation; line nudging targets `total_with_tax` and has more preconditions. The BT-131 claim first logged here (converter rounding instead of fiskaly's `tax.rounding`) was wrong and is withdrawn (see Fix pass) |
| Converter paths | EndpointID comes from the en16931 `iso6523-actorid-upis` endpoint (`ubl:party.go:27-36`); `company_id` → `PartyIdentification` at `ubl:party.go:322-349`; quantity has no converter dp cap |
| Spec | BT-31 TaxScheme predicate; BT-18 and BT-128 `DocumentTypeCode='130'`; BG-3 at `cac:BillingReference`; BG-24 structure (BT-122 mandatory, BR-52); BR-65 for BT-158; PEPPOL-EN16931-R054 (not PEPPOL-COMMON-R054); BR-AE-02 and BR-IC-12 conditions; `cbc:NetworkID` for BG-18; BT-21 has no BIS 3.0.21 element |
| UAPI | `fiscal_location` has no "not applicable" note (only regime and surcharge fields, for BE/DE/IT); BT-155/157 hint applies to product type OTHER only; `payments[i].details.*` paths; the `peppol_id` BT-49 hint is a DE note; positive RETURN amounts are guidance, not schema-enforced; `vat_id_number` is entered without the `BE` prefix |
| BR-CO-25 | triggers only on a standard invoice with an amount due, no OUTSTANDING payment and no `document.payment_terms` |
| Still open | PPL `sequence.enumerate` effect on BT-1; KBO/BCE placement in BT-30; credit-note XSD failures (schema side confirmed, emission inferred); Invopop production converter version |

### Fix pass (2026-11-04)

| Area | Change | Evidence |
| --- | --- | --- |
| Rounding | BT-131: fiskaly's document-level `currency` rule decides; the converter's `precise` switch never applies to fiskaly invoices. BT-114 cited at the converter's GOBL version | gobl v0.511 `bill/calculator.go:329-338`, `bill/rounding.go:33-68`, `239-258`; `G/invoice.go:160-168` |
| Fault logging | BR-CO-25 invoice faults are logged raw; the friendly-rewrite claim is removed; rule 06 cannot fire | `E/service.go:509`, `662`, `860-905`; `G/invoice.go:262-325` |
| §6 | Split into 6.1 / 6.2; sync failures return 2xx with the record `COMPLETED` and an error log; VESID 2026.5 restored; receipt-correction row restored; addon citation corrected | `SV1:3739-3766`; `ubl:format.go:302-305`; `W:199-202`; `E/invopop.go:316-317`, `G/addon.go:19-30` |
| Spec targets | `instruction.text` target is BT-83 `cbc:PaymentID` (as-built BT-82 in notes); BT-21 has no element, so the `#ACD#` prefix moved to notes | XRechnung_3.md §1.4; `structure/syntax/ubl-invoice.xml:85-94` |
| Restored facts | Flow selection incl. BE/DE address; why DE Peppol invoices get XRechnung; origin of `peppol_id`; no self-billing (389); 2026-01-01 mandate and loi du 10 février 2026 in Sources; upward correction as 381 credits the buyer | `RH:3270-3279`; `E/invopop.go:308-312`; `src/unified/domain/system/einvoicing_adaptations.go:148`, `SV1:1197`; `ubl:format.go:316-330`; regulatory baseline (Belgium, verified 2026-10-29) |
| Conventions | Intro in two paragraphs with an inline code legend; full XPaths in §1–§5 (`/CreditNote` for credit-note rows); BT cells as `BT-n Name`; ad-hoc note tags reworded | PROJECT/CLAUDE.md "Page conventions" |
| Nits | BT-30 proposal left-pads to 10 digits; orphaned Art. 230 dropped; log citations prefixed | UAPI `^[01]?\d{9}$`; PEPPOL-COMMON-R043 (`PEPPOL-EN16931-UBL.sch:394`) |
