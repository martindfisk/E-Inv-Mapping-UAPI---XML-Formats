# UAPI v6 → ZUGFeRD 2 (EN 16931 profile, CII)

As of 2026-11-04 · Status: draft for review

UAPI v6 invoice and correction fields mapped to ZUGFeRD 2 / Factur-X, EN 16931 (COMFORT) profile: CII XML inside a PDF/A-3 file. A DE supplier's invoice becomes ZUGFeRD when its one BUSINESS recipient has `invoicing.type` `EMAIL` (the default for a DE address) with `invoicing.format` `ZUGFERD_V2` or none. The format is kept only for a DE address (`RH`:3366-3369), so `XRECHNUNG_V3` to a BE address also yields ZUGFeRD, without an error. An SDI block from a DE supplier is typed ZUGFeRD too (`E/invopop.go:300-313`) but has no workflow (§6.1). The CII XPath column is the spec target from the locally pinned Factur-X 1.08 / ZUGFeRD 2.4 field table (sheet "Factur-X CII D22B EN16931"). In force since 2026-09-01 is ZUGFeRD 2.5.2 / Factur-X 1.09.2: 2.5 updated the code lists and validation artefacts (FeRD press release 2026-06-10), and 2.5.2 renamed BR-CO-27 to CII-SR-470. The full change log is behind a registration form, so the 1.09.2 rule set is UNVERIFIED. UAPI paths are relative to `content.operation` ([UAPI_v6_Fields.md](UAPI_v6_Fields.md)).

**Citations:** `G/` = fiskaly `src/unified/einvoicing/gobl/`, `E/` = `src/unified/einvoicing/`, `RH` = `src/unified/api/v6/record_handler.go`, `EA` = `src/unified/domain/record/einvoicing_adaptations.go`, `W` = `src/unified/worker/e_invoicing_worker.go`, `SV1` = `src/unified/domain/record/service_v1.go`, `WF` = `cmd/invopop-workflow-migration/workflows/DE_SEND_INVOICE_EMAIL_ZUGFERD_BUSINESS.json` (all `fiskaly/meta@cd19ec8`); `gobl/` = invopop/gobl v0.501.0, `gobl.cii/` = v0.81.0, `gobl.de.xinvoice/` = v0.0.9. `FX-SCH-A-nnnnnn` = assert ID in the Factur-X 1.08 `FACTUR-X_EN16931.sch`; `BR-…` = EN 16931 rule ID.

## 1. Invoice Mapping

### 1.1 Header and References

| UAPI | EN 16931 BT | CII XPath | notes |
| --- | --- | --- | --- |
| `document.number` | BT-1 Invoice number | `/rsm:CrossIndustryInvoice/rsm:ExchangedDocument/ram:ID` | `G/invoice.go:153`; `gobl.cii/header.go:47` |
| `document.series` | BT-1 Invoice number (prefix) | `/rsm:CrossIndustryInvoice/rsm:ExchangedDocument/ram:ID` | ignored: GOBL `series` never set (gobl.cii would write `<series>-<code>`, `gobl.cii/utils.go:81-86`) |
| `document.issued_at` | BT-2 Invoice issue date | `/rsm:CrossIndustryInvoice/rsm:ExchangedDocument/ram:IssueDateTime/udt:DateTimeString` (`@format` `102`) | Date part, YYYYMMDD; time dropped. default: today at build time (`G/invoice.go:706-714`), taken separately by the sync check and the worker |
| `-` | BT-3 Invoice type code | `/rsm:CrossIndustryInvoice/rsm:ExchangedDocument/ram:TypeCode` | fixed `380` (`gobl/addons/eu/en16931/scenarios.go:21-28`); `381` for CORRECTION (§3) |
| `document.text` | BT-22 Invoice note | `/rsm:CrossIndustryInvoice/rsm:ExchangedDocument/ram:IncludedNote/ram:Content` | ignored. gap |
| `document.operation_date` | BT-72 Actual delivery date | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeDelivery/ram:ActualDeliverySupplyChainEvent/ram:OccurrenceDateTime/udt:DateTimeString` | API docs: BT-7. The date of supply (§ 14 Abs. 4 Satz 1 Nr. 6 UStG) belongs in BT-72, BG-14 or BG-26 (Factur-X BR-FX-EN-04, FX-SCH-A-000354; KoSIT BR-DE-TMP-32); EN 16931 refers BT-7 to Art. 226(7) Directive 2006/112/EC, which explains the API docs' hint. derived: fiskaly sets GOBL `op_date` and `delivery.date` (`G/invoice.go:152`, 433-441); gobl.cii writes BT-7 only from `value_date` (`gobl.cii/settlement.go:146-152`), never set, so the value reaches BT-72 only |
| `-` | BT-7 Value added tax point date | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeSettlement/ram:ApplicableTradeTax/ram:TaxPointDate/udt:DateString` | gap: no UAPI source; gobl.cii writes BT-7 only from GOBL `value_date`, which fiskaly never sets. Latent converter bug: gobl.cii writes `TaxPointDate/udt:DateTimeString` (`gobl.cii/invoice.go:40`), the XSD type `udt:DateType` allows only `udt:DateString` |
| `document.payment_terms` | BT-20 Payment terms | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeSettlement/ram:SpecifiedTradePaymentTerms/ram:Description` | GOBL `payment.terms.notes` (`G/invoice.go:316-321`; `gobl.cii/settlement.go:214-220`); shares the one `ram:SpecifiedTradePaymentTerms` with BT-9/BT-89 (FX-SCH-A-000174). Satisfies BR-CO-25 alone |
| `document.references.buyer` | BT-10 Buyer reference | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeAgreement/ram:BuyerReference` | GOBL `ordering.code` (`G/invoice.go:331-333`; `gobl.cii/agreement.go:73-75`); optional here, no Leitweg-ID default |
| `document.references.project` | BT-11 Project reference | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeAgreement/ram:SpecifiedProcuringProject/ram:ID` (+ `ram:Name`, BT-11-0, 1..1) | ignored for DE: built but not attached, "Invopop bug META-3995" (`G/invoice.go:337-354`). Likely cause (inferred): gobl.cii writes the misspelt `ram:SpecifiedProcurringProject` (`gobl.cii/agreement.go:21`), unknown to every Factur-X and CEN CII XSD |
| `document.references.contract` | BT-12 Contract reference | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeAgreement/ram:ContractReferencedDocument/ram:IssuerAssignedID` | First contract only (`G/invoice.go:356`; `gobl.cii/agreement.go:96-101`) |
| `document.references.purchase_order` | BT-13 Purchase order reference | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeAgreement/ram:BuyerOrderReferencedDocument/ram:IssuerAssignedID` | `G/invoice.go:359`; `gobl.cii/agreement.go:102-107` |
| `document.references.despatch_advice` | BT-16 Despatch advice reference | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeDelivery/ram:DespatchAdviceReferencedDocument/ram:IssuerAssignedID` | `G/invoice.go:362`; `gobl.cii/delivery.go:50-55` |
| `document.references.tender` | BT-17 Tender or lot reference | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeAgreement/ram:AdditionalReferencedDocument/ram:IssuerAssignedID` with `ram:TypeCode` `50` | TypeCode fixed `50` (`gobl.cii/agreement.go:123-128`) |
| `document.references.buyer_routing` | BT-19 Buyer accounting reference | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeSettlement/ram:ReceivableSpecifiedTradeAccountingAccount/ram:ID` | GOBL `ordering.cost` (`G/invoice.go:335`; `gobl.cii/settlement.go:181-186`) |
| `document.references.preceding_document.number` | BT-25 Preceding Invoice reference | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeSettlement/ram:InvoiceReferencedDocument/ram:IssuerAssignedID` | INVOICE only (`G/invoice.go:197-205`); first reference only (`gobl.cii/settlement.go:164-176`). CORRECTION: §3 |
| `document.references.preceding_document.issued_at` | BT-26 Preceding Invoice issue date | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeSettlement/ram:InvoiceReferencedDocument/ram:FormattedIssueDateTime/qdt:DateTimeString` | As above; omitted when empty |
| `payments[0].details.currency` | BT-5 Invoice currency code | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeSettlement/ram:InvoiceCurrencyCode` | First payment only (`G/invoice.go:725-741`); default `EUR` (`src/unified/domain/record/transaction/receipt/payment/details.go:174`) |
| `document.activity_code`, `document.offline.*` | - | - | ignored (no EN 16931 term) |
| `details.creators[]`, `details.training`, `details.properties` | - | - | ignored (record metadata) |

### 1.2 Buyer

| UAPI | EN 16931 BT | CII XPath | notes |
| --- | --- | --- | --- |
| `recipients[i].type` = `BUSINESS` | BG-7 Buyer | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeAgreement/ram:BuyerTradeParty` | Exactly one BUSINESS recipient (`G/invoice.go:400-418`) |
| `recipients[i].name` | BT-44 Buyer name | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeAgreement/ram:BuyerTradeParty/ram:Name` | `G/recipient.go:175`; `gobl.cii/party.go:96` |
| `recipients[i].buyer_id` | BT-46 Buyer identifier | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeAgreement/ram:BuyerTradeParty/ram:ID` | Identity without scheme → `ram:ID` (`G/recipient.go:158-165`; `gobl.cii/party.go:159-162`); `ram:GlobalID` (BT-46-1 scheme) never used |
| `recipients[i].company_id` | BT-47 Buyer legal registration identifier | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeAgreement/ram:BuyerTradeParty/ram:SpecifiedLegalOrganization/ram:ID` | As-built a second buyer `ram:ID` (BT-46 slot): no `legal` scope on the identity (`G/recipient.go:166-172`; `gobl.cii/party.go:126-138`). With `buyer_id` set too, FX-SCH-A-000163 (max 1) fails |
| `recipients[i].identification.type` = `VAT`, `.number` | BT-48 Buyer VAT identifier | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeAgreement/ram:BuyerTradeParty/ram:SpecifiedTaxRegistration/ram:ID` (`@schemeID` `VA`) | derived: `<address country><number>` (`G/identification.go:22-36`). Required for BUSINESS, DE/BE format-checked (`EA`:64-92); other identification types fail at sync (`EA`:64-67) |
| `recipients[i].address.line.street`, `recipients[i].address.line.number` | BT-50 Buyer address line 1 | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeAgreement/ram:BuyerTradeParty/ram:PostalTradeAddress/ram:LineOne` | derived: `street number` (`G/address.go:34-35`; GOBL `LineOne()`; `gobl.cii/agreement.go:141-157`) |
| `recipients[i].address.line.suffix` | BT-51 Buyer address line 2 | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeAgreement/ram:BuyerTradeParty/ram:PostalTradeAddress/ram:LineTwo` | GOBL `street_extra` (`G/address.go:18-21`, 36) |
| `recipients[i].address.line.prefix`, `recipients[i].address.line.infix` | BT-163 Buyer address line 3 | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeAgreement/ram:BuyerTradeParty/ram:PostalTradeAddress/ram:LineThree` | ignored |
| `recipients[i].address.code` | BT-53 Buyer post code | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeAgreement/ram:BuyerTradeParty/ram:PostalTradeAddress/ram:PostcodeCode` | `G/address.go:23`, 38 |
| `recipients[i].address.city` | BT-52 Buyer city | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeAgreement/ram:BuyerTradeParty/ram:PostalTradeAddress/ram:CityName` | `G/address.go:37` |
| `recipients[i].address.region` | BT-54 Buyer country subdivision | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeAgreement/ram:BuyerTradeParty/ram:PostalTradeAddress/ram:CountrySubDivisionName` | `G/address.go:46-48`; required only for IT |
| `recipients[i].address.country` | BT-55 Buyer country code | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeAgreement/ram:BuyerTradeParty/ram:PostalTradeAddress/ram:CountryID` | derived: overwritten by the tax-ID country (same value here, `gobl.cii/party.go:116-122`) |
| `recipients[i].invoicing.email` | BT-58 Buyer contact email address | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeAgreement/ram:BuyerTradeParty/ram:DefinedTradeContact/ram:EmailURIUniversalCommunication/ram:URIID` | derived: delivery mailbox → `customer.emails[0]` → buyer contact (`G/recipient.go:149-156`; `gobl.cii/party.go:208-213`) |
| `-` | BT-49 Buyer electronic address | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeAgreement/ram:BuyerTradeParty/ram:URIUniversalCommunication/ram:URIID` (`@schemeID`, BT-49-1) | gap: EMAIL creates no GOBL inbox (`G/inbox.go:53-55`), so the element stays empty. Optional |
| `recipients[i].invoicing.type`, `.format` | - | - | Select the flow: `EMAIL` (default for a DE address, `RH`:3257-3268) with `ZUGFERD_V2` or no format. `XRECHNUNG_V3` is kept only for a DE address (`RH`:3366-3369), so a BE recipient gets ZUGFeRD without an error. Not written to the XML |
| `recipients[i].origin` | - | - | ignored |
| `recipients[i].shipping.name` | BT-70 Deliver to party name | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeDelivery/ram:ShipToTradeParty/ram:Name` | With `shipping` (`G/invoice.go:454-467`); not cleared, unlike XRechnung |
| `recipients[i].shipping.address.line.street`, `recipients[i].shipping.address.line.number` | BT-75 Deliver to address line 1 | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeDelivery/ram:ShipToTradeParty/ram:PostalTradeAddress/ram:LineOne` | Only with `shipping`; derived as BT-50 (`G/invoice.go:454-467`; `gobl.cii/delivery.go:71-79`) |
| `recipients[i].shipping.address.line.suffix` | BT-76 Deliver to address line 2 | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeDelivery/ram:ShipToTradeParty/ram:PostalTradeAddress/ram:LineTwo` | As BT-51 |
| `recipients[i].shipping.address.line.prefix`, `recipients[i].shipping.address.line.infix` | BT-165 Deliver to address line 3 | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeDelivery/ram:ShipToTradeParty/ram:PostalTradeAddress/ram:LineThree` | ignored |
| `recipients[i].shipping.address.code` | BT-78 Deliver to post code | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeDelivery/ram:ShipToTradeParty/ram:PostalTradeAddress/ram:PostcodeCode` | As BT-53 |
| `recipients[i].shipping.address.city` | BT-77 Deliver to city | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeDelivery/ram:ShipToTradeParty/ram:PostalTradeAddress/ram:CityName` | As BT-52 |
| `recipients[i].shipping.address.region` | BT-79 Deliver to country subdivision | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeDelivery/ram:ShipToTradeParty/ram:PostalTradeAddress/ram:CountrySubDivisionName` | As BT-54 |
| `recipients[i].shipping.address.country` | BT-80 Deliver to country code | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeDelivery/ram:ShipToTradeParty/ram:PostalTradeAddress/ram:CountryID` | Only with `shipping`. No buyer-address fallback: gobl.de.xinvoice copies the customer as ship-to party only when GOBL has no `delivery`, which fiskaly always sets (`gobl.de.xinvoice/cii_format.go:124-136`). BR-IC-12 needs it for K |
| `recipients[i].shipping.date` | BT-72 Actual delivery date | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeDelivery/ram:ActualDeliverySupplyChainEvent/ram:OccurrenceDateTime/udt:DateTimeString` | ignored (API docs: BT-72): BT-72 comes from `operation_date`, else `issued_at` |

### 1.3 Lines

One `ram:IncludedSupplyChainTradeLineItem` per `entries[i]` (`G/entries.go:36-88`; `gobl.cii/lines.go:345-383`).

| UAPI | EN 16931 BT | CII XPath | notes |
| --- | --- | --- | --- |
| `entries[i]` (array index) | BT-126 Invoice line identifier | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:IncludedSupplyChainTradeLineItem/ram:AssociatedDocumentLineDocument/ram:LineID` | derived: `i + 1` (`G/entries.go:77`). `entries[i].details.number` (print order) ignored |
| `entries[i].type` (`SALE\|RETURN`) | - | - | ignored: no sign change for RETURN; amounts are used as supplied |
| `entries[i].data.type` = `GROUP` | BG-25 Invoice line | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:IncludedSupplyChainTradeLineItem` | One line from the group-level fields; `data.items[]` never mapped. Sub-lines exist only in EXTENDED, so one line fits this profile, but item detail is lost |
| `entries[i].data.text` | BT-153 Item name | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:IncludedSupplyChainTradeLineItem/ram:SpecifiedTradeProduct/ram:Name` | `G/entries.go:303`; `gobl.cii/lines.go:387` |
| `entries[i].details.description` | BT-127 Invoice line note | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:IncludedSupplyChainTradeLineItem/ram:AssociatedDocumentLineDocument/ram:IncludedNote/ram:Content` | ignored. gap |
| `entries[i].data.unit.quantity` | BT-129 Invoiced quantity | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:IncludedSupplyChainTradeLineItem/ram:SpecifiedLineTradeDelivery/ram:BilledQuantity` | Full precision, up to 8 decimals |
| `entries[i].data.unit.measure` | BT-130 Invoiced quantity unit of measure | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:IncludedSupplyChainTradeLineItem/ram:SpecifiedLineTradeDelivery/ram:BilledQuantity/@unitCode` | GOBL unit key or UN/ECE Rec 20 code; anything else is dropped with a warning log (`G/entries.go:312-323`). default: `C62` (`gobl/addons/eu/en16931/org.go:79-86`) |
| `entries[i].data.unit.factor` | BT-149 Item price base quantity | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:IncludedSupplyChainTradeLineItem/ram:SpecifiedLineTradeAgreement/ram:NetPriceProductTradePrice/ram:BasisQuantity` | ignored (GOBL `item.per` never set). gap |
| `entries[i].data.unit.price.inclusive` | BT-146 Item net price | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:IncludedSupplyChainTradeLineItem/ram:SpecifiedLineTradeAgreement/ram:NetPriceProductTradePrice/ram:ChargeAmount` | derived: mapped as gross with `prices_include: VAT` (`G/invoice.go:165-168`); gobl.cii strips VAT before export (`gobl.cii/cii.go:155`), so BT-146 = gross ÷ (1 + rate), extra decimals. Prices may be shifted by cents to match the payments total (`G/line_rounding.go:21-42`). Negative price fails BR-27. BT-148 (gross price) is the price before BT-147 discount, not a VAT-inclusive price, and is not emitted |
| `entries[i].data.unit.price.exclusive` | BT-146 Item net price | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:IncludedSupplyChainTradeLineItem/ram:SpecifiedLineTradeAgreement/ram:NetPriceProductTradePrice/ram:ChargeAmount` | ignored (API docs: BT-146): recalculated from the inclusive price; may differ by rounding |
| `entries[i].data.value.base.*` | BT-131 Invoice line net amount | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:IncludedSupplyChainTradeLineItem/ram:SpecifiedLineTradeSettlement/ram:SpecifiedTradeSettlementLineMonetarySummation/ram:LineTotalAmount` | ignored: BT-131 is recalculated (§1.6) |
| `entries[i].data.value.discount.inclusive` | BT-136 Invoice line allowance amount | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:IncludedSupplyChainTradeLineItem/ram:SpecifiedLineTradeSettlement/ram:SpecifiedTradeAllowanceCharge/ram:ActualAmount` (`udt:Indicator` `false`) | derived: if ≠ 0, 2 dp, gross then made net by the VAT strip (`G/entries.go:45-55`). BT-139 `ram:Reason` fixed `Special Discount` (META-2347). `.discount.exclusive` ignored |
| `entries[i].data.value.surcharge.inclusive` | BT-141 Invoice line charge amount | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:IncludedSupplyChainTradeLineItem/ram:SpecifiedLineTradeSettlement/ram:SpecifiedTradeAllowanceCharge/ram:ActualAmount` (`udt:Indicator` `true`) | As for the discount (`G/entries.go:57-67`); BT-144 fixed `Special Surcharge`. `.surcharge.exclusive` ignored |
| `entries[i].data.vat.type` = `VAT_RATE`, `.percentage` | BT-151 Invoiced item VAT category code; BT-152 Invoiced item VAT rate | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:IncludedSupplyChainTradeLineItem/ram:SpecifiedLineTradeSettlement/ram:ApplicableTradeTax/ram:CategoryCode`; `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:IncludedSupplyChainTradeLineItem/ram:SpecifiedLineTradeSettlement/ram:ApplicableTradeTax/ram:RateApplicablePercent` | derived: `S` for any rate > 0, `Z` for 0 (`G/entries.go:101-122`). `ram:TypeCode` fixed `VAT`. `vat.code` ignored |
| `entries[i].data.vat.type` = `VAT_EXEMPTION`, `.code` | BT-151 Invoiced item VAT category code | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:IncludedSupplyChainTradeLineItem/ram:SpecifiedLineTradeSettlement/ram:ApplicableTradeTax/ram:CategoryCode` | derived (German table, `G/german_exemption.go:24-60`): `CAUSE_2` → `K`, `CAUSE_3` → `G`, `NOT_SUBJECT` → `O`, other codes → `E`; codes above `CAUSE_31` fail at sync. BT-152 `0`, omitted for `O` |
| `entries[i].data.vat.type` = `VAT_REVERSE_CHARGE` | BT-151 Invoiced item VAT category code | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:IncludedSupplyChainTradeLineItem/ram:SpecifiedLineTradeSettlement/ram:ApplicableTradeTax/ram:CategoryCode` | gap: target `AE`. The request fails at sync with "unsupported VAT type … 'REVERSE_CHARGE' for line mapping" (`G/entries.go:164-169`) |
| `entries[i].data.vat.amount`, `entries[i].data.vat.exclusive`, `entries[i].data.vat.inclusive` | - | - | ignored (API docs: BT-117, BT-116, BT-112): EN 16931 has no line-level VAT amounts; GOBL recalculates |
| `entries[i].data.vat.fiscal_location`, `.regime`, `.regime_base`, `.surcharge.*` | - | - | ignored ([DE] not applicable) |
| `entries[i].data.product.details.label` | BT-154 Item description | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:IncludedSupplyChainTradeLineItem/ram:SpecifiedTradeProduct/ram:Description` | ignored. gap |
| `entries[i].data.product.number` (`GTIN-8/12/13/14`) | BT-157 Item standard identifier | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:IncludedSupplyChainTradeLineItem/ram:SpecifiedTradeProduct/ram:GlobalID` (`@schemeID` `0160`) | ignored. gap |
| `entries[i].data.product.number` (`SKU`, `MPN`, `OTHER`) | BT-155 Item Seller's identifier | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:IncludedSupplyChainTradeLineItem/ram:SpecifiedTradeProduct/ram:SellerAssignedID` | ignored (API docs: BT-155, BT-157). `OTHER` has no scheme for BT-157 (BR-64) |
| `entries[i].data.product.code` | BT-158 Item classification identifier | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:IncludedSupplyChainTradeLineItem/ram:SpecifiedTradeProduct/ram:DesignatedProductClassification/ram:ClassCode` | ignored; no list ID for BR-65 |
| `entries[i].data.product.details.name`, `.brand`, `.model`, `.serial`, `.batch`, `.date_*` | BT-160 Item attribute name; BT-161 Item attribute value | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:IncludedSupplyChainTradeLineItem/ram:SpecifiedTradeProduct/ram:ApplicableProductCharacteristic/ram:Description`; `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:IncludedSupplyChainTradeLineItem/ram:SpecifiedTradeProduct/ram:ApplicableProductCharacteristic/ram:Value` | ignored |
| `entries[i].details.concept`, `.purpose`, `.regulatory`, `.label`, `.activity_code`, `.properties`, `.voucher`, `.applied_vouchers`, `.prepayment` | - | - | ignored |

### 1.4 Payment

| UAPI | EN 16931 BT | CII XPath | notes |
| --- | --- | --- | --- |
| `payments[i].type` = `OUTSTANDING`, `.details.date` | BT-9 Payment due date | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeSettlement/ram:SpecifiedTradePaymentTerms/ram:DueDateDateTime/udt:DateTimeString` | First OUTSTANDING only (`G/invoice.go:274-289`); later ones are dropped, with a warning log written by the worker (`E/invopop.go:709-727`). `details.date` required |
| `payments[i].details.amount` (`OUTSTANDING`) | - | - | ignored: this profile has no element for a partial due amount; BT-115 is calculated (§1.6) |
| `payments[i].instruction.type` | BT-81 Payment means type code | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeSettlement/ram:SpecifiedTradeSettlementPaymentMeans/ram:TypeCode` | derived: `CREDIT_TRANSFER` → `30`, `DIRECT_DEBIT` → `49` (not SEPA `58`/`59`; `gobl/addons/eu/en16931/pay.go:12-38`). `UNKNOWN` → GOBL `other`, no UNTDID code, so GOBL rule BR-49 fails (`G/invoice.go:571-583`; `pay.go:40-47`). No instruction → no BG-16 |
| `payments[i].instruction.text` | BT-82 Payment means text | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeSettlement/ram:SpecifiedTradeSettlementPaymentMeans/ram:Information` | API docs: BT-83. As built BT-82 via `instructions.detail` (`G/invoice.go:538-540`; `gobl.cii/settlement.go:252-255`) |
| `payments[i].instruction.properties` | BT-83 Remittance information | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeSettlement/ram:PaymentReference` | Via `instructions.ref` (`G/invoice.go:541-543`; `gobl.cii/settlement.go:244-246`) |
| `payments[i].instruction.account` (`CREDIT_TRANSFER`) | BT-84 Payment account identifier | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeSettlement/ram:SpecifiedTradeSettlementPaymentMeans/ram:PayeePartyCreditorFinancialAccount/ram:IBANID` | GOBL `credit_transfer.iban` (`G/invoice.go:533`; `gobl.cii/settlement.go:257-274`). Satisfies BR-61 |
| `payments[i].instruction.name` (`CREDIT_TRANSFER`) | BT-85 Payment account name | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeSettlement/ram:SpecifiedTradeSettlementPaymentMeans/ram:PayeePartyCreditorFinancialAccount/ram:AccountName` | ignored by the converter: fiskaly sets GOBL `credit_transfer.name` (`G/invoice.go:535`), which GOBL documents as the bank name; gobl.cii never writes `AccountName` (`gobl.cii/settlement.go:257-274`) |
| `payments[i].instruction.payment_service_provider` | BT-86 Payment service provider identifier | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeSettlement/ram:SpecifiedTradeSettlementPaymentMeans/ram:PayeeSpecifiedCreditorFinancialInstitution/ram:BICID` | GOBL `credit_transfer.bic` (`G/invoice.go:534`; `gobl.cii/settlement.go:276-280`); no BIC check |
| `payments[i].instruction.mandate_reference` (`DIRECT_DEBIT`) | BT-89 Mandate reference identifier | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeSettlement/ram:SpecifiedTradePaymentTerms/ram:DirectDebitMandateID` | GOBL `direct_debit.ref` (`G/invoice.go:558`; `gobl.cii/settlement.go:287-293`) |
| `payments[i].instruction.creditor_id.identifier` | BT-90 Bank assigned creditor identifier | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeSettlement/ram:CreditorReferenceID` | GOBL `direct_debit.creditor` (`G/invoice.go:559`; `gobl.cii/settlement.go:296`); other creditor identifier types fail at sync (`G/invoice.go:551-554`) |
| `payments[i].instruction.debited_account` | BT-91 Debited account identifier | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeSettlement/ram:SpecifiedTradeSettlementPaymentMeans/ram:PayerPartyDebtorFinancialAccount/ram:IBANID` | API docs: BT-84, BT-91. GOBL `direct_debit.account` (`G/invoice.go:560`; `gobl.cii/settlement.go:284-286`) |
| `payments[i].details.amount` (`CARD`, `CASH`, `ONLINE`, `*_VOUCHER`, `OTHER`) | BT-113 Paid amount | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeSettlement/ram:SpecifiedTradeSettlementHeaderMonetarySummation/ram:TotalPrepaidAmount` | derived: Σ advances, 2 dp (`G/invoice.go:290-299`; `gobl.cii/settlement.go:335-337`) |
| `payments[i].kind` (`CARD`) | - | - | ignored (API docs: BT-81): card payments are advances (BT-113) |
| `payments[i].number` (`CARD`) | BT-87 Payment card primary account number | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeSettlement/ram:SpecifiedTradeSettlementPaymentMeans/ram:ApplicableTradeSettlementFinancialCard/ram:ID` | ignored (BR-51: first 6 / last 4 digits at most) |
| `payments[i].name` (`ONLINE`, `OTHER`) | BT-82 Payment means text | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeSettlement/ram:SpecifiedTradeSettlementPaymentMeans/ram:Information` | ignored (API docs: BT-82) |
| `payments[i].details.date` (advances), `.discount`, `.channel`, `.properties`; `payments[i].concept`; voucher `number` | - | - | ignored |
| `payments[i].type` = `MEAL_VOUCHER`, `DEPOSIT` | - | - | The request fails at sync with "unsupported payment type" (`G/invoice.go:300-302`) |
| `-` | BG-10 Payee | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeSettlement/ram:PayeeTradeParty` | gap: no UAPI source; GOBL `payment.payee` is never set |

### 1.5 VAT Breakdown and Totals

BG-23 is built from the line taxes after GOBL recalculation, one `ram:ApplicableTradeTax` per category and rate (`gobl.cii/settlement.go:346-391`).

| UAPI | EN 16931 BT | CII XPath | notes |
| --- | --- | --- | --- |
| `-` (from line taxes) | BT-116 VAT category taxable amount; BT-117 VAT category tax amount | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeSettlement/ram:ApplicableTradeTax/ram:BasisAmount`; `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeSettlement/ram:ApplicableTradeTax/ram:CalculatedAmount` | derived, 2 dp (§1.6); `ram:TypeCode` fixed `VAT` |
| `entries[i].data.vat.type`, `.code` | BT-118 VAT category code | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeSettlement/ram:ApplicableTradeTax/ram:CategoryCode` | derived as BT-151 (§1.3) |
| `entries[i].data.vat.percentage` | BT-119 VAT category rate | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeSettlement/ram:ApplicableTradeTax/ram:RateApplicablePercent` | `0` for E/K/G/Z; omitted for O, which FX-SCH-A-000050 allows (BT-152 is likewise absent on O lines, BR-O-05) |
| `entries[i].data.vat.reason` | BT-120 VAT exemption reason text | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeSettlement/ram:ApplicableTradeTax/ram:ExemptionReason` | One note per VAT key, reasons joined with `; `; default German text per code; `CAUSE_31` needs a reason (`G/entries.go:208-269`). Satisfies BR-E/IC/G/O-10 |
| `entries[i].data.vat.code` (`VAT_EXEMPTION`) | BT-121 VAT exemption reason code | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeSettlement/ram:ApplicableTradeTax/ram:ExemptionReasonCode` | derived: VATEX from the German table (none for `CAUSE_1`, `CAUSE_16`, `CAUSE_31`, `NOT_TAXABLE`); dropped when lines of one key differ (`G/german_exemption.go:99-120`), keeping BR-E-01 |
| `breakdown[i].*` | BG-23 VAT breakdown | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeSettlement/ram:ApplicableTradeTax` | ignored (API docs: BT-106 to BT-117): BG-23 is rebuilt from the line taxes |
| `totals.vat.exclusive` | BT-109 Invoice total amount without VAT | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeSettlement/ram:SpecifiedTradeSettlementHeaderMonetarySummation/ram:TaxBasisTotalAmount` | ignored (API docs also: BT-106, BT-116): a mismatch with GOBL only logs a warning (`E/invopop.go:753-782`) |
| `totals.vat.amount` | BT-110 Invoice total VAT amount | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeSettlement/ram:SpecifiedTradeSettlementHeaderMonetarySummation/ram:TaxTotalAmount` (`@currencyID`) | ignored (warning only) |
| `totals.vat.inclusive` | BT-112 Invoice total amount with VAT | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeSettlement/ram:SpecifiedTradeSettlementHeaderMonetarySummation/ram:GrandTotalAmount` | ignored (warning only) |

### 1.6 Calculated Fields

The VAT strip and currency rounding run inside the converter (`gobl.cii/cii.go:155-160`), so on the gobl version Invopop links: gobl.cii v0.81.0 requires gobl ≥ v0.511.0. The `gobl/bill/*` lines below are v0.501.0. In v0.511.0 the VAT strip switches to `precise` rounding only when the document sets no rule (`bill/calculator.go:329-335`); fiskaly sets `currency` (`G/invoice.go:160-168`), so the result for DE invoices without retained taxes is the same.

| UAPI | EN 16931 BT | CII XPath | formula | notes |
| --- | --- | --- | --- | --- |
| `entries[i].data.unit.price.inclusive`, `entries[i].data.vat.percentage` | BT-146 Item net price | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:IncludedSupplyChainTradeLineItem/ram:SpecifiedLineTradeAgreement/ram:NetPriceProductTradePrice/ram:ChargeAmount` | gross ÷ (1 + rate/100), extra precision | `gobl/bill/line.go:246-258`; exempt lines keep the price |
| `entries[i].data.unit.quantity`, BT-146, line allowances/charges | BT-131 Invoice line net amount | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:IncludedSupplyChainTradeLineItem/ram:SpecifiedLineTradeSettlement/ram:SpecifiedTradeSettlementLineMonetarySummation/ram:LineTotalAmount` | round2(qty × BT-146) − Σ BT-136 + Σ BT-141 | `gobl.cii/lines.go:462-466`, rescaled to 2 dp (BR-DEC-23) |
| `-` | BT-106 Sum of Invoice line net amount | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeSettlement/ram:SpecifiedTradeSettlementHeaderMonetarySummation/ram:LineTotalAmount` | Σ BT-131 | — |
| `-` | BT-109 Invoice total amount without VAT | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeSettlement/ram:SpecifiedTradeSettlementHeaderMonetarySummation/ram:TaxBasisTotalAmount` | BT-106 − BT-107 + BT-108 | BT-107/108 absent (no document-level allowances or charges) |
| `-` | BT-116 VAT category taxable amount; BT-117 VAT category tax amount | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeSettlement/ram:ApplicableTradeTax/ram:BasisAmount` / `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeSettlement/ram:ApplicableTradeTax/ram:CalculatedAmount` | per category and rate: Σ BT-131; BT-116 × BT-119 / 100 | `tax.rounding: currency` (`G/invoice.go:167`). BR-CO-17 allows ±1 currency unit (FX-SCH-A-000052; CEN `EN16931-CII-model.sch:120`); BR-S-08 is exact but holds by construction |
| `-` | BT-110 Invoice total VAT amount | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeSettlement/ram:SpecifiedTradeSettlementHeaderMonetarySummation/ram:TaxTotalAmount` | Σ BT-117 | — |
| `-` | BT-112 Invoice total amount with VAT | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeSettlement/ram:SpecifiedTradeSettlementHeaderMonetarySummation/ram:GrandTotalAmount` | BT-109 + BT-110 | — |
| `-` | BT-114 Rounding amount | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeSettlement/ram:SpecifiedTradeSettlementHeaderMonetarySummation/ram:RoundingAmount` | original gross total − total after VAT strip | Only when ≠ 0 (`gobl/bill/calculator.go:263-307`). gobl.cii then calls `RoundToCurrency()` (`gobl.cii/cii.go:158`, gobl ≥ v0.511.0), which can add ±0.01 to it |
| `-` | BT-115 Amount due for payment | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeSettlement/ram:SpecifiedTradeSettlementHeaderMonetarySummation/ram:DuePayableAmount` | BT-112 − BT-113 + BT-114 | `gobl.cii/settlement.go:318-328`; BR-CO-25 when > 0 |

## 2. Supplier Data (Taxpayer, System)

Taxpayer paths are relative to `POST /taxpayers` content; the system annotation is `peppol_id`; `seller.*` and `tax_representative.*` are invoice payload paths. Supplier mapping: `G/taxpayer.go:25-81`, `G/invoice.go:469-497`; CII party: `gobl.cii/party.go:91-173`.

| UAPI | EN 16931 BT | CII XPath | notes |
| --- | --- | --- | --- |
| taxpayer `name.legal` | BT-27 Seller name | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeAgreement/ram:SellerTradeParty/ram:Name` | API docs: BT-44, which is the Buyer name. `G/taxpayer.go:62`; `gobl.cii/party.go:96` |
| taxpayer `name.trade` | BT-28 Seller trading name | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeAgreement/ram:SellerTradeParty/ram:SpecifiedLegalOrganization/ram:TradingBusinessName` | ignored (GOBL `alias` never set). gap |
| taxpayer `name.person.*` (`INDIVIDUAL`) | BT-27 Seller name | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeAgreement/ram:SellerTradeParty/ram:Name` | ignored: BT-27 comes from `name.legal` |
| taxpayer `address.line.street`, `address.line.number` | BT-35 Seller address line 1 | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeAgreement/ram:SellerTradeParty/ram:PostalTradeAddress/ram:LineOne` | derived as BT-50, from the taxpayer address, not the location (`G/taxpayer.go:51-54`) |
| taxpayer `address.line.suffix` | BT-36 Seller address line 2 | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeAgreement/ram:SellerTradeParty/ram:PostalTradeAddress/ram:LineTwo` | As BT-51 |
| taxpayer `address.line.prefix`, `address.line.infix` | BT-162 Seller address line 3 | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeAgreement/ram:SellerTradeParty/ram:PostalTradeAddress/ram:LineThree` | ignored |
| taxpayer `address.code` | BT-38 Seller post code | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeAgreement/ram:SellerTradeParty/ram:PostalTradeAddress/ram:PostcodeCode` | As BT-53 |
| taxpayer `address.city` | BT-37 Seller city | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeAgreement/ram:SellerTradeParty/ram:PostalTradeAddress/ram:CityName` | As BT-52 |
| taxpayer `address.region` | BT-39 Seller country subdivision | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeAgreement/ram:SellerTradeParty/ram:PostalTradeAddress/ram:CountrySubDivisionName` | As BT-54 |
| taxpayer `address.country` | BT-40 Seller country code | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeAgreement/ram:SellerTradeParty/ram:PostalTradeAddress/ram:CountryID` | derived: tax-ID country (`fiscalization.type`), so `DE` (`gobl.cii/party.go:116-122`) |
| taxpayer `fiscalization.vat_id_number` | BT-31 Seller VAT identifier | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeAgreement/ram:SellerTradeParty/ram:SpecifiedTaxRegistration/ram:ID` (`@schemeID` `VA`) | derived: `DE` + 9 digits; wins over the tax ID (`G/taxpayer.go:37-45`). Optional in UAPI |
| taxpayer `fiscalization.tax_id_number` | BT-32 Seller tax registration identifier | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeAgreement/ram:SellerTradeParty/ram:SpecifiedTaxRegistration/ram:ID` (`@schemeID` `FC`) | ignored with a VAT ID, so BT-32 is never emitted. Without one it goes into BT-31 (`VA`), which the GOBL DE regime rejects: the pattern `^[1-9]\d{8}$` fails for an 11-digit Steuer-ID or a Steuernummer (`gobl/regimes/de/tax_identity.go:17-31`; independent check 2026-11-03). The field is the BZSt Steuer-ID; § 14 Abs. 4 Satz 1 Nr. 2 UStG asks for the Finanzamt Steuernummer or the USt-IdNr. gap |
| `-` | BT-29 Seller identifier; BT-30 Seller legal registration identifier; BT-33 Seller additional legal information | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeAgreement/ram:SellerTradeParty/ram:ID`; `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeAgreement/ram:SellerTradeParty/ram:SpecifiedLegalOrganization/ram:ID`; `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeAgreement/ram:SellerTradeParty/ram:Description` | gap: no UAPI source |
| system annotation `peppol_id` | BT-34 Seller electronic address | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeAgreement/ram:SellerTradeParty/ram:URIUniversalCommunication/ram:URIID` (`@schemeID`, BT-34-1) | API docs: BT-49 "of the recipient". When the annotation exists, also on EMAIL (`W`:284-298). Scheme = 4-digit prefix (`gobl/addons/eu/en16931/org.go:113-120`). Absent in the sync build (`E/invopop.go:334`) |
| system `registrations[i].type`, `software.*`; location `*` | - | - | ignored: no EN 16931 term |
| `seller.name` | BT-41 Seller contact point | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeAgreement/ram:SellerTradeParty/ram:DefinedTradeContact/ram:PersonName` | Whole string as contact name (`G/invoice.go:479-485`) |
| `seller.phone` | BT-42 Seller contact telephone number | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeAgreement/ram:SellerTradeParty/ram:DefinedTradeContact/ram:TelephoneUniversalCommunication/ram:CompleteNumber` | `G/invoice.go:488`; `gobl.cii/party.go:203-206` |
| `seller.email` | BT-43 Seller contact email address | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeAgreement/ram:SellerTradeParty/ram:DefinedTradeContact/ram:EmailURIUniversalCommunication/ram:URIID` | `G/invoice.go:492`; `gobl.cii/party.go:208-213` |
| `tax_representative.name` | BT-62 Seller tax representative name | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeAgreement/ram:SellerTaxRepresentativeTradeParty/ram:Name` | GOBL `ordering.seller` (`G/invoice.go:395`, 513; `gobl.cii/agreement.go:88-93`) |
| `tax_representative.vat_number` | BT-63 Seller tax representative VAT identifier | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeAgreement/ram:SellerTaxRepresentativeTradeParty/ram:SpecifiedTaxRegistration/ram:ID` (`@schemeID` `VA`) | derived: `<country><vat_number>` |
| `tax_representative.country` | BT-69 Tax representative country code | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeAgreement/ram:SellerTaxRepresentativeTradeParty/ram:PostalTradeAddress/ram:CountryID` | derived: set as tax-ID country; gobl.cii copies it into the address (`gobl.cii/party.go:116-122`), so BG-12/BT-69 exist (BR-19, BR-20). BT-64–BT-68: no source |

## 3. Correction (Credit Note)

A `TRANSACTION::CORRECTION` with `data.type: INVOICE` is mapped like an invoice from `data.*` and typed GOBL `credit-note` (`W`:196-221). It uses the same workflow (`E/invopop.go:398-404`). A CORRECTION with `data.type: RECEIPT` fails in the worker (`W`:198-201).

| UAPI | EN 16931 BT | CII XPath | notes |
| --- | --- | --- | --- |
| `-` | BT-3 Invoice type code | `/rsm:CrossIndustryInvoice/rsm:ExchangedDocument/ram:TypeCode` | fixed `381` (`gobl/addons/eu/en16931/scenarios.go:29-36`) |
| `record.id` | BT-25 Preceding Invoice reference | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeSettlement/ram:InvoiceReferencedDocument/ram:IssuerAssignedID` | derived: record → `used_in` → Invopop silo entry → original `code` (`W`:208-218; `E/invopop.go:1104-1129`) |
| `record.id` | BT-26 Preceding Invoice issue date | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeSettlement/ram:InvoiceReferencedDocument/ram:FormattedIssueDateTime/qdt:DateTimeString` | derived from the original `issue_date`; omitted if zero |
| `data.document.references.preceding_document.*` | BT-25 Preceding Invoice reference; BT-26 Preceding Invoice issue date | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeSettlement/ram:InvoiceReferencedDocument/ram:IssuerAssignedID`; `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeSettlement/ram:InvoiceReferencedDocument/ram:FormattedIssueDateTime/qdt:DateTimeString` | ignored, warning log (`G/invoice.go:191-205`) |
| `reason.text`, else `reason.type` | BT-22 Invoice note | `/rsm:CrossIndustryInvoice/rsm:ExchangedDocument/ram:IncludedNote/ram:Content` | Text, else the raw enum, e.g. `FACTUAL_ERROR` (`src/unified/service/einvoicing/correction.go:8-14`) |
| `-` | BT-21 Invoice note subject code | `/rsm:CrossIndustryInvoice/rsm:ExchangedDocument/ram:IncludedNote/ram:SubjectCode` | fixed `ACD` (`gobl/addons/eu/en16931/org.go:28`) |
| `record.offset` | - | - | ignored |
| `data.*` (incl. `data.document.number`, `RETURN` entries) | as §1 / §2 | as §1 / §2 | Same mapping; BT-1 is the credit note's own number; nothing is negated. Recipients must equal the original's (`EA`:340-377) |

**Type code and signs**

| Topic | Spec (Factur-X 1.08 §7.1.6, EN 16931) | As-built | Assessment |
| --- | --- | --- | --- |
| Credit note model | `381`: amounts carry the sign of the cancelled invoice; a negative 381 total is not allowed | `381`, amounts as supplied; docs ask for positive `RETURN` lines | Valid when clients follow the docs; negative quantities give a forbidden negative 381, negative prices fail BR-27 |
| Negative invoice model | `380`/`384` with positive prices, negative quantities and totals | Not producible | Only needed to support that practice |
| Upward correction | `384` (corrected invoice) or `380`, positive amounts | Still `381` with positive amounts | Passes validation and silently credits the buyer. gap |
| Reference to the original | BG-3 0..n; § 31 Abs. 5 UStDV: refer "spezifisch und eindeutig" to the invoice | BT-25 always, BT-26 when known; one reference | Meets BR-55 and § 31 Abs. 5 UStDV |
| BR-CO-25 | Factur-X 1.08 schematron: if BT-115 > 0, BT-9 or BT-20 is required, for any type code | GOBL enforces it only for `standard` invoices (`gobl/addons/eu/en16931/bill.go:143-151`, 259-262) | A credit note with an amount due and no terms passes fiskaly but fails the 1.08 artefacts; 1.09.2 status **UNVERIFIED** |

## 4. Fixed and Derived Values

| UAPI | EN 16931 BT | CII XPath | notes |
| --- | --- | --- | --- |
| `-` | BT-24 Specification identifier | `/rsm:CrossIndustryInvoice/rsm:ExchangedDocumentContext/ram:GuidelineSpecifiedDocumentContextParameter/ram:ID` | fixed `urn:cen.eu:en16931:2017`, the value the field table requires for this profile (`gobl.de.xinvoice/xinvoice.go:28`) |
| `-` | BT-23 Business process type | `/rsm:CrossIndustryInvoice/rsm:ExchangedDocumentContext/ram:BusinessProcessSpecifiedDocumentContextParameter/ram:ID` | gap (optional): no UAPI source, and gobl.de.xinvoice sets no business process ID for ZUGFeRD (`gobl.de.xinvoice/xinvoice.go:111-120`) |
| `-` | BT-3 Invoice type code | `/rsm:CrossIndustryInvoice/rsm:ExchangedDocument/ram:TypeCode` | fixed `380` / `381` |
| `-` | BT-2 Invoice issue date | `/rsm:CrossIndustryInvoice/rsm:ExchangedDocument/ram:IssueDateTime/udt:DateTimeString` | default: today when `issued_at` is absent |
| `document.operation_date` / `document.issued_at` | BT-72 Actual delivery date | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeDelivery/ram:ActualDeliverySupplyChainEvent/ram:OccurrenceDateTime/udt:DateTimeString` | derived: always set, operation date else issue date (`G/invoice.go:433-441`). Satisfies BR-FX-EN-04 (FX-SCH-A-000354) and BR-IC-11; when the supply date differs and `operation_date` is missing, BT-72 shows the issue date |
| `-` | BT-31-0, BT-48-0, BT-63-0 Tax scheme identifier | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeAgreement/ram:SellerTradeParty/ram:SpecifiedTaxRegistration/ram:ID/@schemeID` (same on buyer and tax representative) | fixed `VA` |
| `-` | BT-34-1 Scheme identifier | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeAgreement/ram:SellerTradeParty/ram:URIUniversalCommunication/ram:URIID/@schemeID` | derived: first 4 digits of `peppol_id` |
| `-` | BT-114 Rounding amount | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeSettlement/ram:SpecifiedTradeSettlementHeaderMonetarySummation/ram:RoundingAmount` | derived when the VAT strip leaves a gross-total difference, or `RoundToCurrency()` adds one (§1.6) |
| `-` | - | PDF/A-3 file (ISO 19005-3) | fixed by Factur-X 1.08 §6.1. Invopop's `pdf` step renders the visual part (locale `de`, A4) and `cii.pdf.embed` attaches the XML (`WF`:33-48). PDF/A-3 conformance of the output: UNVERIFIED |
| `-` | - | Embedded file name | fixed `factur-x.xml` (Factur-X 1.08 §6.2; `gobl.de.xinvoice/xinvoice.go:61`) |
| `-` | - | PDF file specification `/AFRelationship` | fixed `Alternative`, the only value allowed in Germany (Factur-X 1.08 §5.3, §6.2.2). Set inside Invopop: UNVERIFIED |
| `-` | - | XMP extension schema `urn:factur-x:pdfa:CrossIndustryDocument:invoice:1p0#` | fixed `fx:DocumentType` `INVOICE`, `fx:DocumentFileName` `factur-x.xml`, `fx:Version` `1.0`, `fx:ConformanceLevel` `EN 16931` (Factur-X 1.08 §6.3.1). Set inside Invopop: UNVERIFIED |
| `-` | BT-125 Attached document | `/rsm:CrossIndustryInvoice/rsm:SupplyChainTradeTransaction/ram:ApplicableHeaderTradeAgreement/ram:AdditionalReferencedDocument/ram:AttachmentBinaryObject` | gap (optional): the PDF is not inside the XML, as `cii.generate` runs before `pdf` (`WF`:23-48) |
| `-` | - | CII syntax version | Factur-X XSDs are D22B-based since Factur-X 1.0.07 / ZUGFeRD 2.3; gobl.cii labels its output `D16B` (`gobl.de.xinvoice/cii_format.go:59`; `gobl.cii/format.go:25`). Metadata only: the namespaces are identical, and every element gobl.cii writes exists in the 1.08 XSDs except `ram:SpecifiedProcurringProject` (BT-11) |

Also fixed or derived, with the details in §1–§3: BT-5 default `EUR`; BT-126 `1..n`; BT-130 default `C62`; BT-139 / BT-144 `Special Discount` / `Special Surcharge`; BT-151-0 / BT-118-0 `VAT`; BT-152 / BT-119 `0` for E, K, G (omitted for O); BT-120 German default texts; BT-121 VATEX per exemption code (all of them exist in the EN 16931 code list v16, "VATEX" sheet); BT-40 / BT-55 / BT-69 from the tax-ID country; BT-81 `30` / `49`; BT-21 `ACD`.

## 5. Format Elements Without a UAPI Source (Gaps)

The element column gives the path below its parent group; full XPaths are in §1–§3. Every element that is mandatory without a condition (BT-1, BT-2, BT-3, BT-5, BT-24, BT-27, BT-40, BT-44, BT-55, BG-22, BG-23, BT-126, BT-129 to BT-131, BT-146, BT-151, BT-153) is emitted today.

| BT | element | cardinality / rule | status today | proposal |
| --- | --- | --- | --- | --- |
| BT-31 / BT-32 | `ram:SellerTradeParty/ram:SpecifiedTaxRegistration/ram:ID` | BR-CO-26; BR-S-02, BR-Z-02, BR-E-02 (BT-31, BT-32 or BT-63) | BT-31 only if the DE taxpayer has a USt-IdNr; no Steuernummer field; a Steuer-ID lands in BT-31 and fails GOBL's DE pattern (`gobl/regimes/de/tax_identity.go:17-31`); BT-32 never emitted | Add a DE Steuernummer field, map it to a GOBL identity with scope `tax` (→ `FC`, `gobl.cii/party.go:140-149`); require USt-IdNr or Steuernummer |
| BT-80 | `ram:ShipToTradeParty/ram:PostalTradeAddress/ram:CountryID` | BR-IC-12 (category K) | Absent without `shipping`, so `CAUSE_2` fails | Default the ship-to from the buyer address for K, as the XRechnung path does (`G/xrechnung.go:48-53`) |
| BT-151 = AE | `ram:ApplicableTradeTax/ram:CategoryCode` | BR-AE-01 to -10; § 13b UStG; note "Steuerschuldnerschaft des Leistungsempfängers" (§ 14a Abs. 5 UStG) | `VAT_REVERSE_CHARGE` rejected | Map to GOBL `reverse-charge` (AE), `VATEX-EU-AE`, § 14a Abs. 5 text as BT-120 |
| BT-151 = O | `ram:ApplicableTradeTax/ram:CategoryCode` | BR-O-02, BR-O-11 | DE `NOT_SUBJECT` → O while BT-31 and BT-48 are always present: fails | Map like `NOT_TAXABLE` (E), or reject at sync |
| BT-47 | `ram:BuyerTradeParty/ram:SpecifiedLegalOrganization/ram:ID` | 0..1; FX-SCH-A-000163 | `company_id` written as a second BT-46 | Identity scope `legal` (`gobl.cii/party.go:126-138`); optional ICD scheme |
| BT-9 / BT-20 | `ram:SpecifiedTradePaymentTerms` | BR-CO-25 (BT-115 > 0) | Checked by GOBL for invoices only | Check credit notes too while 1.08-based validators are in use |
| BT-81 | `ram:SpecifiedTradeSettlementPaymentMeans/ram:TypeCode` | BR-49 | `UNKNOWN` fails GOBL | Map to GOBL `any` (UNTDID `1`) or reject at the API |
| BT-3 | `rsm:ExchangedDocument/ram:TypeCode` | UNTDID 1001 | Upward corrections go out as `381` with positive amounts: they pass validation and silently credit the buyer | Derive `384`/`380` when the amount increases, or add a document-type field |
| BT-72 | `ram:ActualDeliverySupplyChainEvent/ram:OccurrenceDateTime` | Optional; § 14 Abs. 4 Satz 1 Nr. 6 UStG | Issue-date fallback; `shipping.date` ignored | Use `shipping.date` > `operation_date`; document the fallback |
| BG-14 | `ram:ApplicableHeaderTradeSettlement/ram:BillingSpecifiedPeriod` | 0..1; alternative in BR-FX-EN-04; § 31 Abs. 4 UStDV (calendar month) | No UAPI field | Add a document period → GOBL `ordering.period` (`gobl.cii/settlement.go:193-201`) |
| BT-22 | `rsm:ExchangedDocument/ram:IncludedNote/ram:Content` | 0..n | `document.text` ignored | Map to a GOBL note (`general` → `AAI`) |
| BT-11 | `ram:SpecifiedProcuringProject/ram:ID` | 0..1 | Dropped for DE (META-3995) | Report the `SpecifiedProcurringProject` typo to Invopop, then re-enable |
| BT-49 | `ram:BuyerTradeParty/ram:URIUniversalCommunication/ram:URIID` | 0..1; BR-63 | Not emitted for EMAIL | Emit `invoicing.email` with scheme `EM`, as for XRechnung |
| BT-85 | `ram:PayeePartyCreditorFinancialAccount/ram:AccountName` | 0..1 | Dropped by gobl.cii | Upstream fix |
| BT-82 / BT-83 | `ram:Information`; `ram:PaymentReference` | 0..1 | `instruction.text` → BT-82, docs say BT-83 | Align; BT-83 (Verwendungszweck) is the useful one |
| BT-127, BT-154, BT-155, BT-157, BT-158, BT-149 | line note, product identifiers, base quantity | 0..1 / 0..n; BR-64, BR-65 | `details.description`, `product.*`, `unit.factor` ignored | Map: description → BT-127; label → BT-154; SKU/MPN → BT-155; GTIN → BT-157 (`0160`); factor → `item.per`; `code` needs a list ID |
| BT-28, BT-30 | `ram:SpecifiedLegalOrganization/ram:TradingBusinessName`, `/ram:ID` | 0..1 | `name.trade` ignored; no Handelsregister field | GOBL `alias`; optional new taxpayer field |
| BT-7 | `ram:ApplicableTradeTax/ram:TaxPointDate/udt:DateString` | 0..1; BR-CO-03 | No source; converter writes the wrong child | Leave unmapped; report to Invopop |
| BT-14, BT-15, BT-18, BT-87, BT-132, BT-133, BG-20, BG-21, BG-24 | various | 0..1 / 0..n | No UAPI source (card data, document-level allowances, attachments, line references) | None needed now |

## 6. Common Validation Failures

### 6.1 fiskaly-side errors for this flow

Sync failures return 2xx, leave the record `COMPLETED` with an error log and send nothing (`SV1`:3739-3766); worker errors fail the job. Invopop builds the XML with `cii.generate` (`doc_type` `zugferd-v2`, `validate: true`, `WF`:23-27) and then renders and embeds the PDF. gobl.de.xinvoice declares VESID `de.zugferd:en16931:2.5.2` for this format (`gobl.de.xinvoice/cii_format.go:27`); which artefact version Invopop runs is UNVERIFIED. The workflow has no e-mail step: fiskaly fetches the artefact, preferring `application/pdf` (`E/invopop.go:860-898`); who sends the e-mail is UNVERIFIED.

| Stage | Message | Trigger | Source |
| --- | --- | --- | --- |
| API (501) | `<TYPE> E_INVOICE_SERVICE is not supported for <cc>` | EMAIL to an address outside DE/BE | `RH`:3332-3336; `src/unified/api/errmap/errmap.go:29` |
| API (400) | `destination code '' must match the format '^[A-Z0-9]{7}$'` | CONSUMER recipient with an EMAIL or PEPPOL block: the block is parsed as SDI without a destination code. `consumer recipients only support SDI invoicing` fires only on an invoicing body that does not decode (code reading) | `RH`:3557-3568; `pkg/identifier/sdi/destination_code.go:46-53` |
| API (silent) | none | EMAIL recipient in BE with `XRECHNUNG_V3`: ZUGFeRD is sent instead of XRechnung, although the API docs say XRECHNUNG_V3 e-mail always yields XRechnung | `RH`:3366-3369; `E/invopop.go:300-313` |
| Sync | `no recipient with invoicing configuration found`; `more than one recipient with invoicing configuration found` | No recipient, or more than one, with `invoicing` | `EA`:27-35; `src/unified/domain/record/recipient/invoicing_recipient.go:25-58` |
| Sync | `TRANSACTION::INVOICE needs exactly 1 business recipient with invoicing specified …` | More than one BUSINESS recipient, one of them with `invoicing` | `EA`:60-62 |
| Sync | `business recipient identification is not a valid VAT identification`; `recipients[0].identification.number: …` | Non-VAT or malformed DE/BE VAT number | `EA`:64-92 |
| Sync | `invoice correction recipients do not match the original invoice` | Changed recipients | `EA`:343-346, 371-376 |
| GOBL | `taxpayer fiscalization does not have tax code`; `invalid German VAT identity code` | No VAT ID and no tax ID; a Steuer-ID or Steuernummer without a VAT ID, which is sent as BT-31 and fails the pattern `^[1-9]\d{8}$` | `G/taxpayer.go:37-49`; `gobl/regimes/de/tax_identity.go:17-31` |
| GOBL | `unsupported VAT type at value_added_tax.type 'REVERSE_CHARGE' …` | `VAT_REVERSE_CHARGE` | `G/entries.go:164-169` |
| GOBL | `unsupported VAT exemption code '<c>' for Germany`; `missing VAT exemption reason for code 'CAUSE_31'` | Codes above `CAUSE_31`; `CAUSE_31` without reason | `G/german_exemption.go:62-74`; `G/entries.go:235-244` |
| GOBL | `unsupported payment type: MEAL_VOUCHER\|DEPOSIT`; `payment due date not set` | Those types; OUTSTANDING without date | `G/invoice.go:300-302`, 610-613 |
| GOBL | `payment means extension is required (BR-49)` | `instruction.type: UNKNOWN` | `gobl/addons/eu/en16931/pay.go:40-47` |
| GOBL | `payment terms are required when amount is due (BR-CO-25)` | Standard invoice with an amount due and neither an OUTSTANDING date nor `payment_terms`, e.g. advances short of the total. Credit notes are exempt. The sibling assertion 06 ("payment details are required …") cannot fire: `detailsFrom` always returns non-empty payment details or errors (`G/invoice.go:262-325`) | `gobl/addons/eu/en16931/bill.go:143-151`, 259-262 |
| Worker; Sync (second message) | `referenced record is missing used_in identifier`; `could not resolve referenced invoice reference` | Correction of an invoice unknown to Invopop | `W`:213-216; `E/invopop.go:645-650`; `EA`:388-398 |
| Worker | `e_invoicing failed: TRANSACTION::CORRECTION with receipt correction is not supported` | CORRECTION with `data.type: RECEIPT` | `W`:198-201 |
| Worker | not traced (Invopop job creation) | SDI block from a DE supplier: typed ZUGFeRD, but no `DE_SEND_INVOICE_SDI_*` workflow exists (inferred, UNVERIFIED). A BUSINESS recipient with SDI is first rejected at sync (`recipients[0].invoicing.type: supplier is not registered for SDI`) unless the system has an SDI registration or no registrations at all | `E/invopop.go:300-313`, 388-404, 834-857; `EA`:145-157, 178-181 |
| Invopop | Converter or schematron fault via webhook, logged raw | Rules in §6.2 | `E/service.go:639-665`, 860-884. The friendly rewrite in `E/error_translation.go` applies only to taxpayer-registration errors (`E/service.go:509`, 886-905) |

### 6.2 Rules most likely to trip

Whether Invopop's `validate: true` blocks on schematron as well as XSD is **UNVERIFIED**.

| Rule | Text (abridged) | Why it trips here |
| --- | --- | --- |
| BR-O-02, BR-O-11 | O lines exclude BT-31, BT-63, BT-48; an O breakdown excludes other breakdowns | DE `NOT_SUBJECT` |
| BR-IC-12 | K needs BT-80 | `CAUSE_2` without `shipping` |
| FX-SCH-A-000163 | Buyer `ram:ID` at most once | `buyer_id` + `company_id` |
| BR-CO-25 (FX-SCH-A-000155) | BT-115 > 0 needs BT-9 or BT-20 | Credit notes with an amount due and no terms; removed from CEN artefacts 1.3.16, 1.09.2 status **UNVERIFIED** |
| BR-27 (FX-SCH-A-000206) | BT-146 not negative | Negative `price.inclusive` |
| BR-CO-17 (FX-SCH-A-000052), BR-S-08 (FX-SCH-A-000198) | BT-117 = BT-116 × rate, within ±1 currency unit; BT-116 = Σ BT-131 per rate, exact | Not expected to trip: GOBL builds BT-116 from the 2-dp line amounts and rounds BT-117 from it |
| BR-CO-26, BR-S-02, BR-E-02 | Seller identifier needed | DE taxpayer without USt-IdNr |
| XSD | Unknown `ram:SpecifiedProcurringProject` | If BT-11 is re-enabled before the gobl.cii fix |
| CII-SR-470 (was BR-CO-27) | IBAN or proprietary ID in BT-84 | Holds today; new ID in 1.09.2, so match both IDs in error handling |

## Sources

All retrieved or read on 2026-11-03 or 2026-11-04. Local paths are relative to the formats repo `E-Invoicing-Formats-and-Profiles/`.

| Artefact | Version / path / URL |
| --- | --- |
| Factur-X field table (spec target) | Factur-X 1.08 / ZUGFeRD 2.4 (2025-12-04), `Germany/Zugferd 24_EN/Documentation/1_FACTUR-X 1.08 - 2025 12 04 - EN FR - VF.xlsx`, sheets "Factur-X CII D22B EN16931" and "Business Rules" |
| Factur-X specification | `…/Documentation/0_FACTUR-X_1.08_2025_12_04_EN.pdf` (version history; §5.3, §6.1–§6.3, §7.1.6) |
| Factur-X EN 16931 XSD and schematron | `Germany/Zugferd 24_EN/Schema/3_Factur-X_1.08_EN16931/` (`FACTUR-X_EN16931.xsd` + RABIE/UDT XSDs, `FACTUR-X_EN16931.sch`) |
| EN 16931 code lists | `…/Documentation/2_EN16931 code lists values v16 - used from 2025-11-15 - Fx 1.08.xlsx` ("VATEX") |
| EN 16931 CII validation artefacts | ConnectingEurope `eInvoicing-EN16931`, tag `validation-1.3.17` (2026-10-12, commit `6e35e0a`), local clone `eInvoicing-EN16931/cii/`; BR-CO-25 removal commit `1eb862a` (2026-04-04, in 1.3.16) |
| ZUGFeRD 2.5.2 | https://www.ferd-net.de/publikationen-produkte/publikationen/detailseite/zugferd-252-english |
| ZUGFeRD 2.5 | https://www.ferd-net.de/publikationen-produkte/publikationen/detailseite/zugferd-25-english; press release https://www.ferd-net.de/fileadmin/user_upload/FeRD/f%C3%BCr_FeRD-Meldungen/20260610_ZUGFeRD_2_5_Factur-X_1_09_Press_Release_DE.pdf |
| ZUGFeRD version archive | https://www.ferd-net.de/standards/zugferd-versionsarchiv |
| Factur-X 1.09.2 | https://fnfe-mpe.org/factur-x/ |
| Regulatory baseline | `regulatory_baseline.md`, Format versions table (ZUGFeRD row verified 2026-09-15) |
| XRechnung CII schematron (BR-DE-TMP-32) | 2.6.0, `Germany/xrechnung-3.0.2-bundle-2026-08-31/xrechnung-3.0.2-schematron-2.6.0/schematron/cii/XRechnung-CII-validation.sch` |
| UStG, UStDV | `Germany/Laws/01_UStG_konsolidiert.md` (§ 14 Abs. 4, § 14a Abs. 5), `02_UStDV_konsolidiert.md` (§ 31 Abs. 4, 5) |
| UAPI v6 | `UAPI_v6_Fields.md` (`fiskaly/meta@cd19ec88cb299ee41a44302448006340dd028bf0`); API docs version 2026-06-01 (file in this repo) |
| fiskaly code | `fiskaly/meta@cd19ec8`: `src/unified/einvoicing/**`, `src/unified/api/v6/record_handler.go`, `src/unified/domain/record/**`, `src/unified/worker/e_invoicing_worker.go`, `src/unified/api/errmap/errmap.go`, `pkg/identifier/sdi/destination_code.go`, `pkg/invopop/workflow/`, `cmd/invopop-workflow-migration/workflows/` (ZUGFeRD workflow; no `DE_SEND_INVOICE_SDI_*` file) |
| GOBL | `invopop/gobl` v0.501.0 (commit `225e39d`, pinned in fiskaly `go.mod`); v0.511.0 read for `bill/rounding.go` and `bill/calculator.go`, as gobl.cii v0.81.0 requires it |
| gobl.cii | `invopop/gobl.cii` v0.81.0 (commit `3216642`, 2026-10-26); its `go.mod` requires gobl v0.511.0; latest public tag, not a confirmed production version |
| gobl.de.xinvoice | `invopop/gobl.de.xinvoice` v0.0.9 (commit `8116d89`, 2026-10-15); its `go.mod` requires gobl.cii v0.75.0 and gobl v0.510.0 |
| As-built trace | UAPI v6 → GOBL trace of 2026-11-03 (working file), spot-checked against the code |

## Verification Log

| # | Claim | How checked | Result |
| --- | --- | --- | --- |
| 1 | BT-24 is emitted as `urn:cen.eu:en16931:2017`, the value the EN 16931 profile requires | Factur-X 1.08 field table, BT-24 CIUS column; `gobl.de.xinvoice/xinvoice.go:28`, :116, :282 | CONFIRMED |
| 2 | The VAT ID wins over the tax ID; BT-32 is never emitted | `G/taxpayer.go:37-45`; `gobl.cii/party.go:140-149` | CONFIRMED |
| 3 | `VAT_REVERSE_CHARGE`, `MEAL_VOUCHER`, `DEPOSIT` are rejected | `G/entries.go:164-169`; `G/invoice.go:300-302` | CONFIRMED |
| 4 | CORRECTION → credit note → BT-3 `381`; BT-25/26 from the referenced record | `W`:196-218; `gobl/addons/eu/en16931/scenarios.go:29-36`; `gobl.cii/settlement.go:164-176` | CONFIRMED |
| 5 | `operation_date` reaches BT-72, not BT-7; the converter's BT-7 element is XSD-invalid | `G/invoice.go:152`, 433-441; `gobl.cii/settlement.go:146-152`, `invoice.go:40`; Factur-X UDT XSD `DateType` | CONFIRMED |
| 6 | `company_id` becomes a second buyer `ram:ID`, failing FX-SCH-A-000163 | `G/recipient.go:158-172`; `gobl.cii/party.go:159-162`; `FACTUR-X_EN16931.sch` | CONFIRMED |
| 7 | `instruction.text` → BT-82, `.properties` → BT-83, `.name` (BT-85) dropped; `UNKNOWN` fails BR-49 | `G/invoice.go:530-583`; `gobl.cii/settlement.go:243-309`; `gobl/addons/eu/en16931/pay.go:12-47` | CONFIRMED |
| 8 | DE `NOT_SUBJECT` conflicts with BR-O-02; `CAUSE_2` without shipping fails BR-IC-12 | Rule texts, EN 16931 CII artefacts 1.3.17; `G/german_exemption.go:26`, :56; `gobl.de.xinvoice/cii_format.go:124-136` | CONFIRMED (blocking at Invopop **UNVERIFIED**) |
| 9 | gobl.cii writes BT-11 as `ram:SpecifiedProcurringProject`, unknown to every Factur-X and CEN CII XSD | `gobl.cii/agreement.go:21`; XSD grep; link to META-3995 inferred | CONFIRMED |
| 10 | XSDs D22B-based since Factur-X 1.0.07; 2.5 updated code lists and validation artefacts, new elements only in EXTENDED; 2.5.2 renames BR-CO-27 → CII-SR-470 | Factur-X 1.08 version history; FeRD press release 2026-06-10; FeRD 2.5 and 2.5.2 pages | CONFIRMED; full 2.5.2 change list and BR-CO-25 in the 1.09.2 schematron **UNVERIFIED** |
| 11 | ZUGFeRD for a DE supplier's EMAIL recipient with `ZUGFERD_V2` or no format; `XRECHNUNG_V3` is dropped for a non-DE address, so a BE recipient gets ZUGFeRD | `RH`:3257-3268, 3331-3336, 3361-3369; `E/invopop.go:300-319` | CONFIRMED |
| 12 | An SDI block from a DE supplier is typed ZUGFeRD, but no `DE_SEND_INVOICE_SDI_*` workflow exists | `E/invopop.go:300-313`, 388-404, 834-857; workflow folder at `cd19ec8` | CONFIRMED in code; failure at Invopop **UNVERIFIED** |
| 13 | Container: PDF/A-3, `factur-x.xml`, AFRelationship `Alternative`, `fx:ConformanceLevel` `EN 16931` | Requirements in Factur-X 1.08 §5.3, §6.1–§6.3.1; Invopop `pdf` and `cii.pdf.embed` output not visible | **UNVERIFIED** (as-built) |

### Independent verification pass (2026-11-03)

Two separate checkers re-tested the decision-critical claims against code (fiskaly `cd19ec8`, gobl v0.501.0, the vendored converters) and against primary sources. Neither was given the page.

| Claim | Verdict | Evidence / correction |
| --- | --- | --- |
| `VAT_REVERSE_CHARGE`, `MEAL_VOUCHER`, `DEPOSIT` rejected in every country | CONFIRMED | `entries.go:164-169`; `invoice.go:300-301` |
| VAT ID beats tax ID; BT-32 never emitted; Steuer-ID as BT-31 rejected | CONFIRMED | `taxpayer.go:38-43`; `gobl.cii/party.go:108-115`; `gobl/regimes/de/tax_identity.go:17-27` |
| `NOT_SUBJECT` → category O while BT-31/BT-48 are present (BR-O-02) | CONFIRMED | `german_exemption.go:56-58` (the code notes the conflict for NOT_TAXABLE but leaves NOT_SUBJECT as O). In CII, BR-O-02 only checks `schemeID="VA"` |
| `buyer_id` + `company_id` → two buyer `ram:ID` (FX-SCH-A-000163) | PARTLY CONFIRMED | Two `ram:ID` are emitted (`gobl.cii/party.go:159-162`) and fail FX-SCH-A-000163 (`count(ram:ID)<=1`). The rule is a generic cardinality check, reused for ShipTo and Payee, and does not limit `ram:GlobalID` (that is FX-SCH-A-000164). It is not a strict equivalent of UBL-SR-16 |
| `operation_date` → BT-72 only; BT-7 never | CONFIRMED | `invoice.go:433-441`; `gobl.cii/settlement.go:145-151` |
| `instruction.text` → BT-82, `.properties` → BT-83 | PARTLY CONFIRMED | True for CREDIT_TRANSFER and DIRECT_DEBIT. For UNKNOWN the text goes to `Instructions.Notes`, which no converter reads (the type fails BR-49 anyway). Only the first OUTSTANDING payment's instruction is used (`invoice.go:280-283`) |
| Guideline ID `urn:cen.eu:en16931:2017` for the EN 16931 profile | CONFIRMED | Factur-X 1.08 BT-24 row; `FACTUR-X_EN16931_codedb.xml` cl id=1 |
| Factur-X XSDs D22B-based since 1.0.07 / ZUGFeRD 2.3 | CONFIRMED | Local `Schema/5. CII D22B XSD/`; FNFE-MPE/FeRD press release 2024-09-18. Compatibility runs one way only: D16B instances are valid D22B, not vice versa |
| 2.5.2 effective 2026-09-01; for EN16931 only BR-CO-27 → CII-SR-470 | PARTLY CONFIRMED | Date and rename confirmed (ferd-net.de 2.5.2 page). That nothing else changed in the EN16931 profile is UNVERIFIABLE: the detailed change log is behind a registration form |

### Validation pass (2026-11-03, after removing the endpoint, trigger and output sections)

| Check | Result |
| --- | --- |
| Every XPath, child element (`ram:LineOne` …) and attribute (`@schemeID`, `@format`, `@currencyID`) in §1–§4 vs the Factur-X 1.08 field table, sheet "Factur-X CII D22B EN16931" | CONFIRMED: all present with the stated BT. The field table labels BT-62 "Buyer tax representative name"; EN 16931 calls it the Seller tax representative name, which the page uses |
| `FX-SCH-A-000155`, `-000163`, `-000164`, `-000174`, `-000206`, `-000354` vs `FACTUR-X_EN16931.sch` (1.08) | CONFIRMED (rule texts and contexts match) |
| List of unconditionally mandatory elements in §5 | CONFIRMED against the field table cardinalities |
| The 27 VATEX codes in `G/german_exemption.go` vs EN 16931 code list v16 | CONFIRMED: all present |
| § 14 Abs. 4 Satz 1 Nr. 2 and Nr. 6, § 14a Abs. 5 UStG; § 31 Abs. 4 and 5 UStDV; Factur-X 1.08 §7.1.6 | CONFIRMED against the local consolidated texts and the specification PDF |
| fiskaly code citations in §1–§6 (`fiskaly/meta@cd19ec8`) | CONFIRMED, except for corrected line ranges (`EA`:340-377, `RH`:3332-3336, the BT-5 `EUR` default source) and two corrected triggers in §6 (the "exactly 1 business recipient" message, the CONSUMER SDI message). `could not resolve referenced invoice reference` can also fail at sync |
| GOBL converter citations in §1–§6 (gobl v0.501.0, gobl.cii v0.81.0, gobl.de.xinvoice v0.0.9) | CONFIRMED, except for line drift (`cii_format.go:124-136`, `tax_identity.go:17-31`). Added: gobl.cii requires gobl v0.511.0; `RoundToCurrency()` can add to BT-114; BR-CO-25 assertion 06 when there is no payment at all; GOBL documents `credit_transfer.name` as the bank name |

### Fix pass (2026-11-04)

| Area | Change | Evidence |
| --- | --- | --- |
| Intro | Flow-selection sentence restored: DE supplier, one BUSINESS recipient, EMAIL default, `ZUGFERD_V2` or none, silent `XRECHNUNG_V3` fallback for BE, SDI typed ZUGFeRD without a workflow. The 2.5 code-list and artefact update is named | `RH`:3257-3268, 3366-3369; `E/invopop.go:300-313`; FeRD press release 2026-06-10 |
| §6.1 Invopop row | "Friendly rewrite" claim removed: invoice faults are logged raw | `E/service.go:509`, 639-665, 860-905 |
| §1.6, §6.2 BR-CO-17 | "No tolerance" replaced: BR-CO-17 allows ±1 currency unit; BR-S-08 holds by construction | `FACTUR-X_EN16931.sch:31`, :55; `EN16931-CII-model.sch:120` |
| §6.1 BR-CO-25 | Assertion 06 dropped as unreachable; credit-note exemption stated | `G/invoice.go:262-325`; `gobl/addons/eu/en16931/bill.go:143-151`, 259-262 |
| §4 container and syntax | PDF/A-3, `factur-x.xml`, `/AFRelationship`, XMP, BT-125 and D22B/D16B rows restored; as-built marked UNVERIFIED | Factur-X 1.08 §5.3, §6.1–§6.3.1; `WF`:23-48; `gobl.de.xinvoice/xinvoice.go:61`, `cii_format.go:59` |
| §6.1 lead and rows | Sync-failure behaviour, `doc_type`/`validate`/VESID, no e-mail step; CONSUMER 400 message; 501 for an unsupported country; silent BE fallback; RECEIPT correction; SDI from a DE supplier | `SV1`:3739-3766; `WF`:23-27; `cii_format.go:27`; `E/invopop.go:860-898`; `RH`:3557-3568; `errmap.go:29`; `W`:198-201 |
| Consistency | Steuer-ID as BT-31 stated as fact; BT-9 warning cited in the worker; an upward correction credits the buyer; `operation_date` → BT-72 backed by BR-FX-EN-04 and BR-DE-TMP-32 | `gobl/regimes/de/tax_identity.go:17-31`; `E/invopop.go:709-727`; `FACTUR-X_EN16931.sch:691-692`; XRechnung CII schematron 2.6.0 :462 |
| Sources | EN 16931 validation artefacts v1.3.17 dated 2026-10-12 (was 2026-10-09) | Tag `validation-1.3.17` in the local clone, commit `6e35e0a` |
| Conventions | Plain status; API-docs BTs moved to notes (13 rows); BT names added; address groups split into per-BT rows with full XPaths; `…/` XPaths expanded; §1.2 "Buyer"; §6.1 and §6.2; "Verbatim" and ad-hoc tags replaced by code citations or the notes vocabulary; BT-119 note (FX-SCH-A-000050); one `bill.go` range; Verification Log #10 repaired, #11–#13 added | Factur-X 1.08 field table: every XPath re-checked |
