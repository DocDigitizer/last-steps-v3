# Report 3: Universal Document Category Taxonomy - First 2 Levels

**DocDigitizer Schema Engine - Category Extraction (CE) Taxonomy**
**Version:** 1.0
**Date:** 2026-03-25
**Classification:** Internal - Strategic

---

## Table of Contents

1. [Taxonomy Design Principles](#1-taxonomy-design-principles)
2. [Level 1 Categories](#2-level-1-categories)
3. [Level 2 Categories](#3-level-2-categories)
4. [CE Schema Examples](#4-ce-schema-examples)
5. [Ambiguity Resolution](#5-ambiguity-resolution)
6. [Regional and Industry Variations](#6-regional-and-industry-variations)
7. [Growth Strategy](#7-growth-strategy)
8. [Statistics](#8-statistics)

---

## 1. Taxonomy Design Principles

### 1.1 Core Philosophy

The taxonomy is designed as a **pragmatic, demand-driven hierarchy** optimized for the iterative Category Extraction (CE) mechanism. It is not an academic ontology; it is an operational classification system built around what documents IDP pipelines actually encounter at scale.

### 1.2 Design Rules

| # | Principle | Rationale |
|---|-----------|-----------|
| P1 | **Mutual Exclusivity at Each Level** | At any given level, a document should map to exactly one category with high confidence (>90%). This minimizes CE iteration ambiguity and avoids multi-path traversal. |
| P2 | **Balanced Fan-Out** | Each level should have between 5 and 25 children. Fewer than 5 wastes an iteration cycle; more than 25 degrades LLM classification accuracy and increases token cost. |
| P3 | **Volume-Weighted Design** | High-volume document types (invoices, receipts, contracts) get dedicated categories early. Rare document types aggregate into broader categories and refine at deeper levels. |
| P4 | **Domain-First, Then Purpose** | Level 1 categorizes by business domain (Financial, Legal, HR, etc.). Level 2 categorizes by document purpose within that domain (billing, reporting, compliance, etc.). |
| P5 | **Schema Complexity Alignment** | Categories at the same level should have roughly comparable schema complexity, so extraction difficulty is predictable. |
| P6 | **Extensibility by Branching** | New categories are added by branching at the appropriate level, never by restructuring existing paths. Every node has a stable ID that never changes. |
| P7 | **Geographic and Regulatory Neutrality** | L1 and L2 are universal. Regional and regulatory variations appear at L3+ (e.g., "Tax Form" is L2; "US W-2" vs "PT IRS Modelo 3" is L3+). |
| P8 | **LLM-Friendly Category Names** | Names must be unambiguous natural language labels that an LLM can distinguish without domain-specific training. Avoid jargon at L1; permit controlled jargon at L2. |
| P9 | **Confidence Thresholds** | Each CE level must return a confidence score. If no category exceeds 0.7, the system should flag for human review or trigger "unknown" path. |
| P10 | **Versioned and Immutable** | Once a category ID is assigned, it is permanent. Deprecated categories are marked inactive but never deleted or reassigned. |

### 1.3 ID Scheme

Category IDs follow a hierarchical dot-notation:

```
L1:  CAT-XX          (e.g., CAT-01)
L2:  CAT-XX.YY       (e.g., CAT-01.03)
L3+: CAT-XX.YY.ZZ    (e.g., CAT-01.03.05)
```

This allows deterministic path construction and efficient database indexing.

---

## 2. Level 1 Categories

The following 20 Level 1 categories cover the practical document landscape encountered by IDP companies. They are ordered by estimated volume in a diversified IDP pipeline.

| Cat ID | Category Name | Description | Est. Volume | Example Documents |
|--------|--------------|-------------|-------------|-------------------|
| CAT-01 | **Financial & Accounting** | Documents related to financial transactions, bookkeeping, accounting records, and financial reporting | 22% | Invoices, receipts, credit notes, financial statements, ledger entries |
| CAT-02 | **Legal & Contracts** | Legally binding documents, agreements, contracts, and legal correspondence | 12% | Contracts, NDAs, leases, terms of service, legal opinions |
| CAT-03 | **Human Resources & Employment** | Documents related to the employee lifecycle, payroll, benefits, and workforce management | 10% | Employment contracts, pay stubs, CVs/resumes, performance reviews |
| CAT-04 | **Tax & Regulatory Compliance** | Government-mandated tax filings, regulatory reports, and compliance documentation | 8% | Tax returns, VAT declarations, audit reports, regulatory filings |
| CAT-05 | **Banking & Financial Services** | Documents originating from or related to banking, lending, and financial service operations | 7% | Bank statements, loan agreements, mortgage documents, wire transfers |
| CAT-06 | **Insurance** | Documents related to insurance policies, claims, underwriting, and risk assessment | 6% | Insurance policies, claim forms, loss reports, coverage certificates |
| CAT-07 | **Healthcare & Medical** | Clinical, administrative, and regulatory documents from the healthcare sector | 5% | Medical records, prescriptions, lab results, insurance claim forms |
| CAT-08 | **Supply Chain & Logistics** | Documents governing the movement of goods, shipping, warehousing, and trade | 5% | Bills of lading, packing lists, shipping manifests, customs declarations |
| CAT-09 | **Procurement & Purchasing** | Documents in the purchase-to-pay cycle from requisition through to goods receipt | 4% | Purchase orders, RFQs, supplier evaluations, goods receipts |
| CAT-10 | **Identity & Personal Documents** | Documents that establish or verify personal identity, status, or qualifications | 4% | Passports, driver licenses, national ID cards, birth certificates |
| CAT-11 | **Real Estate & Property** | Documents related to property ownership, transactions, valuation, and management | 3% | Deeds, title reports, property appraisals, lease agreements |
| CAT-12 | **Government & Public Administration** | Official government-issued documents, permits, licenses, and public records | 3% | Permits, government notices, court filings, municipal records |
| CAT-13 | **Corporate Governance** | Documents related to corporate structure, board activities, shareholder relations, and compliance | 2% | Board minutes, articles of incorporation, shareholder resolutions, annual reports |
| CAT-14 | **Education & Academic** | Documents from educational institutions including records, certifications, and research | 2% | Transcripts, diplomas, research papers, enrollment forms |
| CAT-15 | **Technical & Engineering** | Technical documentation, specifications, manuals, and engineering drawings | 2% | Technical specs, user manuals, engineering drawings, safety data sheets |
| CAT-16 | **Marketing & Communications** | Commercial communications, marketing materials, and customer-facing documents | 1.5% | Brochures, proposals, press releases, advertisements |
| CAT-17 | **Utilities & Telecommunications** | Bills, contracts, and service documents from utility and telecom providers | 1.5% | Utility bills, service agreements, consumption reports, installation orders |
| CAT-18 | **Travel & Expense** | Documents related to business travel, expense management, and reimbursement | 1% | Expense reports, travel itineraries, hotel folios, boarding passes |
| CAT-19 | **Correspondence & General** | General business correspondence, memos, and communications not fitting other categories | 0.5% | Business letters, memos, emails (printed), fax cover sheets |
| CAT-20 | **Uncategorized / Unknown** | Documents that cannot be confidently classified into any other category; triggers human review or schema suggestion flow | 0.5% | Damaged documents, multi-type composites, novel formats |

### 2.1 Design Notes on Level 1

- **CAT-01 (Financial & Accounting)** is deliberately the highest-volume bucket. Invoices alone represent the single largest document type in most IDP pipelines. This category will have the deepest L3+ tree.
- **CAT-20 (Uncategorized / Unknown)** is a critical operational category. It is the entry point for the "Chaos" scenario and feeds the schema suggestion workflow. Documents here should be flagged for community schema creation.
- **CAT-10 (Identity)** is separated from HR because identity documents appear across many workflows (KYC, onboarding, government, etc.) and have distinct extraction patterns.
- **CAT-05 (Banking)** is separated from CAT-01 (Financial) because banking documents originate from financial institutions and have very different schemas (statements, SWIFT messages) compared to corporate accounting documents (invoices, journal entries).

---

## 3. Level 2 Categories

### CAT-01: Financial & Accounting

| Cat ID | Sub-Category | Description | Example Documents | Schema Complexity |
|--------|-------------|-------------|-------------------|-------------------|
| CAT-01.01 | **Invoices & Bills** | Commercial invoices requesting payment for goods or services | Sales invoices, proforma invoices, e-invoices, utility invoices | Medium |
| CAT-01.02 | **Receipts & Proof of Payment** | Documents confirming payment has been made | Cash register receipts, payment receipts, digital payment confirmations | Simple |
| CAT-01.03 | **Credit & Debit Notes** | Adjustments to previously issued invoices | Credit notes, debit notes, refund notices, adjustment memos | Medium |
| CAT-01.04 | **Financial Statements** | Formal records of financial activities and position | Balance sheets, income statements, cash flow statements, trial balances | Complex |
| CAT-01.05 | **Expense Reports & Reimbursements** | Internal documents for expense tracking and reimbursement | Employee expense reports, mileage claims, petty cash vouchers | Medium |
| CAT-01.06 | **Budgets & Forecasts** | Forward-looking financial planning documents | Annual budgets, departmental budgets, financial forecasts, projections | Complex |
| CAT-01.07 | **Journal Entries & Ledgers** | Accounting records of individual transactions | General ledger entries, journal vouchers, posting records | Medium |
| CAT-01.08 | **Accounts Payable / Receivable** | Documents tracking money owed to or by the organization | AP aging reports, AR aging reports, payment schedules, collection notices | Medium |
| CAT-01.09 | **Payroll Documents** | Financial records related to employee compensation processing | Payroll registers, payroll summaries, payroll tax reports | Complex |
| CAT-01.10 | **Audit & Reconciliation** | Documents related to financial verification and reconciliation | Bank reconciliation statements, audit work papers, variance reports | Complex |

### CAT-02: Legal & Contracts

| Cat ID | Sub-Category | Description | Example Documents | Schema Complexity |
|--------|-------------|-------------|-------------------|-------------------|
| CAT-02.01 | **Service Agreements** | Contracts for the provision of services | MSAs, SLAs, consulting agreements, maintenance contracts | Complex |
| CAT-02.02 | **Sales & Purchase Agreements** | Contracts for the sale or purchase of goods or assets | Sales contracts, asset purchase agreements, distribution agreements | Complex |
| CAT-02.03 | **Lease & Rental Agreements** | Contracts for the use of property or equipment | Commercial leases, equipment leases, vehicle rental agreements | Complex |
| CAT-02.04 | **Employment & Engagement Agreements** | Contracts governing work relationships | Employment contracts, freelancer agreements, contractor agreements | Complex |
| CAT-02.05 | **Non-Disclosure & Confidentiality** | Agreements protecting confidential information | NDAs, confidentiality agreements, trade secret agreements | Medium |
| CAT-02.06 | **Licensing & IP Agreements** | Agreements related to intellectual property rights | Software licenses, patent licenses, trademark assignments, royalty agreements | Complex |
| CAT-02.07 | **Legal Notices & Correspondence** | Formal legal communications | Demand letters, cease-and-desist letters, legal opinions, notice of termination | Medium |
| CAT-02.08 | **Court & Litigation Documents** | Documents related to judicial proceedings | Complaints, summons, court orders, judgments, depositions | Complex |
| CAT-02.09 | **Powers of Attorney & Authorizations** | Documents granting legal authority to act on behalf of another | POAs, proxy forms, authorization letters, delegation of authority | Medium |
| CAT-02.10 | **Terms & Policies** | Standard terms, conditions, and organizational policies | Terms of service, privacy policies, acceptable use policies, warranties | Medium |
| CAT-02.11 | **Amendments & Addenda** | Modifications to existing agreements | Contract amendments, addenda, side letters, change orders | Medium |
| CAT-02.12 | **Settlement & Release Agreements** | Documents resolving disputes or releasing claims | Settlement agreements, mutual releases, waivers of liability | Medium |

### CAT-03: Human Resources & Employment

| Cat ID | Sub-Category | Description | Example Documents | Schema Complexity |
|--------|-------------|-------------|-------------------|-------------------|
| CAT-03.01 | **Recruitment & Applications** | Documents from the hiring process | Job applications, CVs/resumes, cover letters, interview evaluation forms | Medium |
| CAT-03.02 | **Onboarding & Employment Setup** | Documents for new employee setup | Offer letters, onboarding checklists, emergency contact forms, benefit enrollment forms | Medium |
| CAT-03.03 | **Compensation & Benefits** | Documents related to pay and benefits | Pay stubs, salary letters, bonus notifications, stock option grants | Medium |
| CAT-03.04 | **Performance Management** | Documents for employee performance tracking | Performance reviews, goal-setting forms, PIPs, 360-feedback reports | Medium |
| CAT-03.05 | **Leave & Attendance** | Documents tracking time off and attendance | Leave requests, attendance records, sick notes, FMLA forms | Simple |
| CAT-03.06 | **Training & Development** | Documents related to employee learning and growth | Training certificates, course completions, development plans, skill assessments | Simple |
| CAT-03.07 | **Disciplinary & Grievance** | Documents related to workplace issues and discipline | Written warnings, disciplinary records, grievance filings, investigation reports | Medium |
| CAT-03.08 | **Termination & Offboarding** | Documents related to employment ending | Termination letters, resignation letters, exit interviews, final settlement documents | Medium |
| CAT-03.09 | **Organizational & Policies** | Internal HR policies and organizational documents | Employee handbooks, org charts, HR policies, workplace safety documents | Complex |
| CAT-03.10 | **Employee Verification** | Documents verifying employment status or history | Employment verification letters, reference letters, service certificates | Simple |

### CAT-04: Tax & Regulatory Compliance

| Cat ID | Sub-Category | Description | Example Documents | Schema Complexity |
|--------|-------------|-------------|-------------------|-------------------|
| CAT-04.01 | **Income Tax Returns & Filings** | Personal and corporate income tax submissions | Corporate tax returns, personal tax returns, estimated tax payments | Complex |
| CAT-04.02 | **VAT / Sales Tax Documents** | Value-added tax and sales tax related documents | VAT returns, sales tax filings, VAT invoices, tax exemption certificates | Medium |
| CAT-04.03 | **Withholding & Reporting Forms** | Tax withholding certificates and information returns | W-2, W-9, 1099 series, withholding tax certificates | Medium |
| CAT-04.04 | **Tax Assessments & Notices** | Government-issued tax determinations and notifications | Tax assessment notices, deficiency notices, penalty notices, refund notices | Medium |
| CAT-04.05 | **Customs & Duties** | Import/export duties and tariff documents | Customs declarations, duty payment receipts, tariff classifications | Medium |
| CAT-04.06 | **Regulatory Filings** | Mandatory filings to regulatory bodies (non-tax) | SEC filings, annual returns to company registries, environmental reports | Complex |
| CAT-04.07 | **Compliance Certificates** | Documents certifying compliance with regulations | Compliance certificates, SOC reports, ISO certifications, GDPR records | Medium |
| CAT-04.08 | **Audit Reports** | External and internal audit documentation | Tax audit reports, regulatory audit findings, compliance audit reports | Complex |
| CAT-04.09 | **Transfer Pricing Documentation** | Documents related to intercompany pricing | Transfer pricing reports, benchmarking studies, intercompany agreements | Complex |
| CAT-04.10 | **Tax Exemptions & Incentives** | Documents related to tax relief and incentives | Tax exemption applications, incentive approvals, zone certifications | Medium |

### CAT-05: Banking & Financial Services

| Cat ID | Sub-Category | Description | Example Documents | Schema Complexity |
|--------|-------------|-------------|-------------------|-------------------|
| CAT-05.01 | **Bank Statements** | Periodic account statements from financial institutions | Current account statements, savings account statements, multi-currency statements | Medium |
| CAT-05.02 | **Loan & Mortgage Documents** | Documents related to lending and mortgages | Loan agreements, mortgage deeds, amortization schedules, promissory notes | Complex |
| CAT-05.03 | **Payment & Transfer Records** | Records of fund movements | Wire transfer confirmations, SWIFT messages, payment orders, check images | Medium |
| CAT-05.04 | **Credit & Debit Card Documents** | Card-related statements and agreements | Credit card statements, card agreements, chargeback documents | Medium |
| CAT-05.05 | **Investment Documents** | Documents related to investment products | Investment statements, trade confirmations, portfolio reports, prospectuses | Complex |
| CAT-05.06 | **Account Opening & KYC** | Documents for account setup and identity verification | Account applications, KYC forms, beneficial ownership declarations | Medium |
| CAT-05.07 | **Letters of Credit & Guarantees** | Trade finance and guarantee instruments | Letters of credit, bank guarantees, standby LCs, performance bonds | Complex |
| CAT-05.08 | **Foreign Exchange Documents** | Documents related to currency exchange | FX confirmations, forward contracts, currency conversion receipts | Medium |
| CAT-05.09 | **Banking Correspondence** | General communications from/to financial institutions | Bank reference letters, account closure notices, rate change notifications | Simple |

### CAT-06: Insurance

| Cat ID | Sub-Category | Description | Example Documents | Schema Complexity |
|--------|-------------|-------------|-------------------|-------------------|
| CAT-06.01 | **Insurance Policies** | The policy documents themselves | Life insurance policies, property insurance policies, liability policies, health insurance policies | Complex |
| CAT-06.02 | **Claims & Loss Reports** | Documents related to filing and processing claims | First notice of loss, claim forms, damage reports, theft reports | Medium |
| CAT-06.03 | **Underwriting Documents** | Documents used in risk assessment and policy pricing | Underwriting applications, risk assessments, actuarial reports, medical questionnaires | Complex |
| CAT-06.04 | **Certificates of Insurance** | Proof of insurance coverage | COIs, evidence of coverage, insurance binders, declarations pages | Medium |
| CAT-06.05 | **Premium & Billing Documents** | Insurance payment-related documents | Premium invoices, payment schedules, cancellation for non-payment notices | Medium |
| CAT-06.06 | **Endorsements & Riders** | Modifications to existing policies | Policy endorsements, riders, schedule of coverage changes | Medium |
| CAT-06.07 | **Adjuster & Investigation Reports** | Post-claim investigation documents | Adjuster reports, investigation summaries, fraud investigation reports | Complex |
| CAT-06.08 | **Reinsurance Documents** | Documents between insurance and reinsurance companies | Reinsurance treaties, ceding statements, bordereaux reports | Complex |

### CAT-07: Healthcare & Medical

| Cat ID | Sub-Category | Description | Example Documents | Schema Complexity |
|--------|-------------|-------------|-------------------|-------------------|
| CAT-07.01 | **Clinical Records** | Patient medical records and notes | Progress notes, history & physical exams, consultation notes, discharge summaries | Complex |
| CAT-07.02 | **Prescriptions & Medication** | Drug-related documents | Prescriptions, medication orders, pharmacy dispensing records | Medium |
| CAT-07.03 | **Diagnostic Reports** | Test results and diagnostic findings | Lab results, radiology reports, pathology reports, ECG reports | Complex |
| CAT-07.04 | **Medical Billing & Coding** | Healthcare financial documents | Medical invoices, EOBs, superbills, UB-04 forms, CMS-1500 forms | Complex |
| CAT-07.05 | **Patient Intake & Consent** | Administrative patient documents | Registration forms, consent forms, HIPAA authorization, advance directives | Medium |
| CAT-07.06 | **Referrals & Authorizations** | Care coordination documents | Referral letters, prior authorization forms, pre-certification requests | Medium |
| CAT-07.07 | **Medical Certificates** | Health-related attestations | Fitness certificates, disability certificates, vaccination records, death certificates | Simple |
| CAT-07.08 | **Clinical Trial & Research** | Research-related medical documents | Informed consent (research), case report forms, IRB approvals, study protocols | Complex |

### CAT-08: Supply Chain & Logistics

| Cat ID | Sub-Category | Description | Example Documents | Schema Complexity |
|--------|-------------|-------------|-------------------|-------------------|
| CAT-08.01 | **Shipping & Transportation** | Documents governing the physical movement of goods | Bills of lading, airway bills, shipping instructions, delivery orders | Medium |
| CAT-08.02 | **Customs & Trade** | International trade compliance documents | Customs declarations, certificates of origin, import/export licenses | Medium |
| CAT-08.03 | **Packing & Inventory** | Documents describing goods and inventory | Packing lists, inventory reports, stock take sheets, bin cards | Medium |
| CAT-08.04 | **Warehouse & Storage** | Documents related to warehousing operations | Warehouse receipts, storage agreements, pick/pack orders, goods received notes | Medium |
| CAT-08.05 | **Delivery & Proof of Delivery** | Confirmation of goods arrival | PODs, delivery notes, signed delivery receipts, inspection reports | Simple |
| CAT-08.06 | **Freight & Carrier Documents** | Carrier-specific and freight-related documents | Freight invoices, carrier rate sheets, booking confirmations, tracking records | Medium |
| CAT-08.07 | **Dangerous Goods & Compliance** | Safety and compliance documents for regulated cargo | DG declarations, material safety data sheets, hazardous goods manifests | Complex |

### CAT-09: Procurement & Purchasing

| Cat ID | Sub-Category | Description | Example Documents | Schema Complexity |
|--------|-------------|-------------|-------------------|-------------------|
| CAT-09.01 | **Purchase Orders** | Formal orders for goods or services | Standard POs, blanket POs, framework orders, call-off orders | Medium |
| CAT-09.02 | **Quotations & Proposals** | Price offerings from suppliers | Quotes, proposals, bids, tenders, pricing schedules | Medium |
| CAT-09.03 | **Requisitions** | Internal requests to purchase | Purchase requisitions, material requisitions, service requisitions | Simple |
| CAT-09.04 | **Supplier Management** | Documents related to vendor relationships | Supplier applications, vendor evaluations, supplier scorecards, approved vendor lists | Medium |
| CAT-09.05 | **Goods Receipt & Inspection** | Documents confirming receipt and quality of goods | Goods receipt notes, inspection reports, quality certificates, rejection notices | Medium |
| CAT-09.06 | **Request for Proposals / Information** | Formal solicitation documents | RFPs, RFIs, RFQs, tender documents, evaluation criteria | Complex |
| CAT-09.07 | **Catalogs & Price Lists** | Product and pricing reference documents | Supplier catalogs, price lists, product specification sheets | Medium |

### CAT-10: Identity & Personal Documents

| Cat ID | Sub-Category | Description | Example Documents | Schema Complexity |
|--------|-------------|-------------|-------------------|-------------------|
| CAT-10.01 | **Government-Issued ID** | Primary identity documents issued by government | Passports, national ID cards, driver licenses, social security cards | Medium |
| CAT-10.02 | **Proof of Address** | Documents verifying residential address | Utility bills (as address proof), bank statements (as address proof), government letters | Simple |
| CAT-10.03 | **Civil Status Documents** | Documents recording life events | Birth certificates, marriage certificates, death certificates, divorce decrees | Medium |
| CAT-10.04 | **Immigration & Visa Documents** | Documents related to immigration status | Visas, work permits, residence permits, travel documents, refugee status documents | Medium |
| CAT-10.05 | **Professional Certifications** | Documents certifying professional qualifications | Professional licenses, trade certifications, accreditation certificates | Simple |
| CAT-10.06 | **Background & Verification** | Documents from background check processes | Criminal record checks, credit checks, reference verification reports | Medium |

### CAT-11: Real Estate & Property

| Cat ID | Sub-Category | Description | Example Documents | Schema Complexity |
|--------|-------------|-------------|-------------------|-------------------|
| CAT-11.01 | **Deeds & Title Documents** | Documents establishing property ownership | Property deeds, title reports, title insurance policies, land certificates | Complex |
| CAT-11.02 | **Property Valuation** | Documents assessing property value | Appraisals, comparative market analyses, surveyor reports, BPOs | Complex |
| CAT-11.03 | **Property Leases** | Rental agreements for real property | Residential leases, commercial leases, sublease agreements, lease renewals | Complex |
| CAT-11.04 | **Property Management** | Operational property documents | Maintenance requests, inspection reports, HOA documents, building permits | Medium |
| CAT-11.05 | **Mortgage & Financing** | Property-specific financing documents | Mortgage applications, closing disclosures, settlement statements, escrow documents | Complex |
| CAT-11.06 | **Zoning & Land Use** | Government regulatory property documents | Zoning certificates, land use permits, environmental impact reports, site plans | Medium |

### CAT-12: Government & Public Administration

| Cat ID | Sub-Category | Description | Example Documents | Schema Complexity |
|--------|-------------|-------------|-------------------|-------------------|
| CAT-12.01 | **Permits & Licenses** | Government-granted authorizations | Business licenses, building permits, operating permits, environmental permits | Medium |
| CAT-12.02 | **Government Notices & Orders** | Official communications from government bodies | Administrative orders, enforcement notices, compliance orders, penalty notices | Medium |
| CAT-12.03 | **Court & Judicial Documents** | Documents from the judicial system | Court orders, subpoenas, warrants, legal filings, case records | Complex |
| CAT-12.04 | **Government Benefits** | Documents related to public benefits and aid | Social security statements, unemployment claims, disability benefits, pension statements | Medium |
| CAT-12.05 | **Public Records & Registrations** | Official registration documents | Business registrations, vehicle registrations, voter registrations, patent registrations | Medium |
| CAT-12.06 | **Government Contracts & Grants** | Public sector agreements and funding | Government contracts, grant agreements, subsidy approvals, public tender awards | Complex |
| CAT-12.07 | **Freedom of Information & Disclosure** | Public transparency documents | FOIA requests, public disclosure records, transparency reports | Medium |

### CAT-13: Corporate Governance

| Cat ID | Sub-Category | Description | Example Documents | Schema Complexity |
|--------|-------------|-------------|-------------------|-------------------|
| CAT-13.01 | **Incorporation & Formation** | Documents establishing a legal entity | Articles of incorporation, certificates of formation, bylaws, operating agreements | Complex |
| CAT-13.02 | **Board & Shareholder Documents** | Documents from governance meetings | Board minutes, shareholder resolutions, proxy statements, voting records | Complex |
| CAT-13.03 | **Annual Reports & Filings** | Periodic corporate disclosures | Annual reports, quarterly reports, statutory filings, director reports | Complex |
| CAT-13.04 | **Corporate Changes** | Documents recording structural changes | Merger agreements, acquisition documents, name change filings, dissolution documents | Complex |
| CAT-13.05 | **Shareholder & Equity Documents** | Documents related to ownership interests | Share certificates, stock transfer documents, capitalization tables, option agreements | Medium |
| CAT-13.06 | **Corporate Policies & Charters** | Internal governance frameworks | Code of conduct, anti-bribery policies, whistleblower policies, committee charters | Medium |

### CAT-14: Education & Academic

| Cat ID | Sub-Category | Description | Example Documents | Schema Complexity |
|--------|-------------|-------------|-------------------|-------------------|
| CAT-14.01 | **Academic Records & Transcripts** | Official academic achievement records | Transcripts, grade reports, academic standing letters, GPA certificates | Medium |
| CAT-14.02 | **Diplomas & Certificates** | Qualification completion documents | Diplomas, degrees, certificates of completion, professional certifications | Simple |
| CAT-14.03 | **Enrollment & Admissions** | Documents related to entering educational programs | Application forms, admission letters, enrollment confirmations, transfer documents | Medium |
| CAT-14.04 | **Financial Aid & Scholarships** | Education funding documents | Scholarship awards, financial aid offers, student loan documents, tuition invoices | Medium |
| CAT-14.05 | **Research & Publications** | Academic research documents | Research papers, theses, dissertations, peer review reports | Complex |
| CAT-14.06 | **Accreditation & Institutional** | Institutional quality documents | Accreditation reports, institutional reviews, program assessments | Complex |

### CAT-15: Technical & Engineering

| Cat ID | Sub-Category | Description | Example Documents | Schema Complexity |
|--------|-------------|-------------|-------------------|-------------------|
| CAT-15.01 | **Technical Specifications** | Detailed technical requirement documents | Product specs, system requirements, design specifications, API documentation | Complex |
| CAT-15.02 | **Engineering Drawings & Plans** | Visual technical documents | CAD drawings, blueprints, floor plans, circuit diagrams, schematics | Complex |
| CAT-15.03 | **Manuals & Instructions** | User-facing technical documentation | User manuals, installation guides, operating procedures, maintenance manuals | Medium |
| CAT-15.04 | **Safety Data & Compliance** | Safety and regulatory technical documents | SDS/MSDS, safety certificates, CE declarations, UL listings | Medium |
| CAT-15.05 | **Test & Quality Reports** | Testing and quality assurance documents | Test reports, quality inspection reports, calibration certificates, validation reports | Medium |
| CAT-15.06 | **Project & Change Documentation** | Technical project management documents | Project plans, change requests, technical change notices, as-built documents | Medium |
| CAT-15.07 | **Patents & Technical IP** | Technical intellectual property documents | Patent applications, patent grants, technical disclosures, prior art searches | Complex |

### CAT-16: Marketing & Communications

| Cat ID | Sub-Category | Description | Example Documents | Schema Complexity |
|--------|-------------|-------------|-------------------|-------------------|
| CAT-16.01 | **Proposals & Presentations** | Business development documents | Sales proposals, pitch decks, capability statements, case studies | Medium |
| CAT-16.02 | **Brochures & Collateral** | Marketing materials | Product brochures, flyers, datasheets, catalogs | Simple |
| CAT-16.03 | **Press & Media** | Public relations documents | Press releases, media kits, news clippings, spokesperson briefs | Simple |
| CAT-16.04 | **Advertising & Campaign** | Advertising-related documents | Ad copy, media plans, campaign reports, insertion orders | Medium |
| CAT-16.05 | **Brand & Creative** | Brand management documents | Brand guidelines, style guides, logo usage documents, creative briefs | Medium |
| CAT-16.06 | **Customer Communications** | Direct customer-facing documents | Newsletters, customer letters, satisfaction surveys, loyalty program documents | Simple |

### CAT-17: Utilities & Telecommunications

| Cat ID | Sub-Category | Description | Example Documents | Schema Complexity |
|--------|-------------|-------------|-------------------|-------------------|
| CAT-17.01 | **Utility Bills & Statements** | Periodic utility billing documents | Electricity bills, water bills, gas bills, waste management bills | Medium |
| CAT-17.02 | **Telecom Bills & Statements** | Telecommunications billing | Phone bills, internet bills, mobile statements, data usage reports | Medium |
| CAT-17.03 | **Service Contracts** | Utility and telecom service agreements | Service agreements, installation contracts, SLAs, network agreements | Complex |
| CAT-17.04 | **Consumption & Usage Reports** | Usage tracking documents | Meter readings, consumption reports, bandwidth utilization reports | Simple |
| CAT-17.05 | **Installation & Maintenance** | Service delivery documents | Installation orders, maintenance schedules, service tickets, work orders | Medium |

### CAT-18: Travel & Expense

| Cat ID | Sub-Category | Description | Example Documents | Schema Complexity |
|--------|-------------|-------------|-------------------|-------------------|
| CAT-18.01 | **Travel Bookings & Itineraries** | Travel planning documents | Flight itineraries, hotel confirmations, car rental agreements, travel packages | Medium |
| CAT-18.02 | **Boarding & Transportation** | Transit documents | Boarding passes, train tickets, bus tickets, ferry tickets | Simple |
| CAT-18.03 | **Accommodation Documents** | Lodging-related documents | Hotel folios, rental agreements (short-term), hostel receipts | Medium |
| CAT-18.04 | **Expense Claims & Reports** | Expense reimbursement documents | Expense reports, mileage logs, per diem claims, corporate card reconciliations | Medium |
| CAT-18.05 | **Travel Policies & Approvals** | Travel governance documents | Travel policies, pre-trip approvals, travel authorization forms | Simple |

### CAT-19: Correspondence & General

| Cat ID | Sub-Category | Description | Example Documents | Schema Complexity |
|--------|-------------|-------------|-------------------|-------------------|
| CAT-19.01 | **Business Letters** | Formal business correspondence | Cover letters, introduction letters, follow-up letters, acknowledgment letters | Simple |
| CAT-19.02 | **Internal Memos & Notes** | Internal organizational communications | Memos, meeting notes, internal announcements, circulars | Simple |
| CAT-19.03 | **Emails (Printed/Archived)** | Email communications captured as documents | Printed emails, email threads, email attachments (standalone) | Simple |
| CAT-19.04 | **Fax & Legacy Communications** | Older communication formats | Fax cover sheets, fax transmissions, telex messages | Simple |
| CAT-19.05 | **Forms & Questionnaires** | General-purpose forms not fitting other categories | Generic forms, surveys, questionnaires, feedback forms | Medium |
| CAT-19.06 | **Meeting & Event Documents** | Documents from meetings and events | Agendas, minutes (non-board), attendance lists, event registrations | Simple |

### CAT-20: Uncategorized / Unknown

| Cat ID | Sub-Category | Description | Example Documents | Schema Complexity |
|--------|-------------|-------------|-------------------|-------------------|
| CAT-20.01 | **Low Confidence Classification** | Documents where CE confidence was below threshold | Ambiguous documents, documents with conflicting signals | N/A |
| CAT-20.02 | **Multi-Type Composites** | Documents containing multiple document types | Combined PDF packages, multi-document scans, mixed file bundles | N/A |
| CAT-20.03 | **Damaged or Illegible** | Documents that cannot be properly read | Poor quality scans, truncated documents, corrupted files | N/A |
| CAT-20.04 | **Novel Document Types** | Previously unseen document types | New regulatory forms, emerging digital document formats | N/A |
| CAT-20.05 | **Non-Document Files** | Files that are not traditional documents | Images (photos), audio transcripts, video stills, data exports | N/A |

---

## 4. CE Schema Examples

### 4.1 Level 1 Classification Schema

This is the "system schema" used for the first CE iteration. The LLM receives the document and this schema, and must extract the L1 category.

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "ce://docdigitizer.com/schemas/ce/level1/v1.0",
  "title": "CE Level 1 - Document Category Classification",
  "description": "Category Extraction schema for Level 1 classification. The LLM must analyze the document and select the single most appropriate top-level category.",
  "type": "object",
  "required": ["category_id", "category_name", "confidence"],
  "additionalProperties": false,
  "properties": {
    "category_id": {
      "type": "string",
      "description": "The Level 1 category identifier for this document",
      "enum": [
        "CAT-01", "CAT-02", "CAT-03", "CAT-04", "CAT-05",
        "CAT-06", "CAT-07", "CAT-08", "CAT-09", "CAT-10",
        "CAT-11", "CAT-12", "CAT-13", "CAT-14", "CAT-15",
        "CAT-16", "CAT-17", "CAT-18", "CAT-19", "CAT-20"
      ]
    },
    "category_name": {
      "type": "string",
      "description": "Human-readable name of the selected category",
      "enum": [
        "Financial & Accounting",
        "Legal & Contracts",
        "Human Resources & Employment",
        "Tax & Regulatory Compliance",
        "Banking & Financial Services",
        "Insurance",
        "Healthcare & Medical",
        "Supply Chain & Logistics",
        "Procurement & Purchasing",
        "Identity & Personal Documents",
        "Real Estate & Property",
        "Government & Public Administration",
        "Legal & Contracts",
        "Education & Academic",
        "Technical & Engineering",
        "Marketing & Communications",
        "Utilities & Telecommunications",
        "Travel & Expense",
        "Correspondence & General",
        "Uncategorized / Unknown"
      ]
    },
    "confidence": {
      "type": "number",
      "description": "Classification confidence score between 0.0 and 1.0",
      "minimum": 0.0,
      "maximum": 1.0
    },
    "reasoning": {
      "type": "string",
      "description": "Brief explanation of why this category was selected (max 200 chars)",
      "maxLength": 200
    },
    "alternative_category_id": {
      "type": ["string", "null"],
      "description": "Second-best category if confidence is below 0.85. Null if primary is highly confident.",
      "enum": [
        "CAT-01", "CAT-02", "CAT-03", "CAT-04", "CAT-05",
        "CAT-06", "CAT-07", "CAT-08", "CAT-09", "CAT-10",
        "CAT-11", "CAT-12", "CAT-13", "CAT-14", "CAT-15",
        "CAT-16", "CAT-17", "CAT-18", "CAT-19", "CAT-20",
        null
      ]
    }
  }
}
```

**Sample LLM Output:**

```json
{
  "category_id": "CAT-01",
  "category_name": "Financial & Accounting",
  "confidence": 0.96,
  "reasoning": "Document is a commercial invoice with line items, tax amounts, and payment terms from a supplier.",
  "alternative_category_id": null
}
```

### 4.2 Level 2 Classification Schema - Financial & Accounting (CAT-01)

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "ce://docdigitizer.com/schemas/ce/level2/CAT-01/v1.0",
  "title": "CE Level 2 - Financial & Accounting Sub-Classification",
  "description": "Category Extraction schema for Level 2 classification within Financial & Accounting. Document has been classified as CAT-01 at Level 1.",
  "type": "object",
  "required": ["category_id", "category_name", "confidence"],
  "additionalProperties": false,
  "properties": {
    "category_id": {
      "type": "string",
      "description": "The Level 2 sub-category identifier",
      "enum": [
        "CAT-01.01", "CAT-01.02", "CAT-01.03", "CAT-01.04", "CAT-01.05",
        "CAT-01.06", "CAT-01.07", "CAT-01.08", "CAT-01.09", "CAT-01.10"
      ]
    },
    "category_name": {
      "type": "string",
      "description": "Human-readable name of the selected sub-category",
      "enum": [
        "Invoices & Bills",
        "Receipts & Proof of Payment",
        "Credit & Debit Notes",
        "Financial Statements",
        "Expense Reports & Reimbursements",
        "Budgets & Forecasts",
        "Journal Entries & Ledgers",
        "Accounts Payable / Receivable",
        "Payroll Documents",
        "Audit & Reconciliation"
      ]
    },
    "confidence": {
      "type": "number",
      "description": "Classification confidence score between 0.0 and 1.0",
      "minimum": 0.0,
      "maximum": 1.0
    },
    "reasoning": {
      "type": "string",
      "description": "Brief explanation of why this sub-category was selected",
      "maxLength": 200
    },
    "alternative_category_id": {
      "type": ["string", "null"],
      "description": "Second-best sub-category if confidence is below 0.85",
      "enum": [
        "CAT-01.01", "CAT-01.02", "CAT-01.03", "CAT-01.04", "CAT-01.05",
        "CAT-01.06", "CAT-01.07", "CAT-01.08", "CAT-01.09", "CAT-01.10",
        null
      ]
    }
  }
}
```

**Sample LLM Output:**

```json
{
  "category_id": "CAT-01.01",
  "category_name": "Invoices & Bills",
  "confidence": 0.98,
  "reasoning": "Document contains invoice number, vendor details, line items with quantities and unit prices, tax calculation, and payment due date.",
  "alternative_category_id": null
}
```

### 4.3 Level 2 Classification Schema - Legal & Contracts (CAT-02)

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "ce://docdigitizer.com/schemas/ce/level2/CAT-02/v1.0",
  "title": "CE Level 2 - Legal & Contracts Sub-Classification",
  "description": "Category Extraction schema for Level 2 classification within Legal & Contracts. Document has been classified as CAT-02 at Level 1.",
  "type": "object",
  "required": ["category_id", "category_name", "confidence"],
  "additionalProperties": false,
  "properties": {
    "category_id": {
      "type": "string",
      "description": "The Level 2 sub-category identifier",
      "enum": [
        "CAT-02.01", "CAT-02.02", "CAT-02.03", "CAT-02.04", "CAT-02.05",
        "CAT-02.06", "CAT-02.07", "CAT-02.08", "CAT-02.09", "CAT-02.10",
        "CAT-02.11", "CAT-02.12"
      ]
    },
    "category_name": {
      "type": "string",
      "description": "Human-readable name of the selected sub-category",
      "enum": [
        "Service Agreements",
        "Sales & Purchase Agreements",
        "Lease & Rental Agreements",
        "Employment & Engagement Agreements",
        "Non-Disclosure & Confidentiality",
        "Licensing & IP Agreements",
        "Legal Notices & Correspondence",
        "Court & Litigation Documents",
        "Powers of Attorney & Authorizations",
        "Terms & Policies",
        "Amendments & Addenda",
        "Settlement & Release Agreements"
      ]
    },
    "confidence": {
      "type": "number",
      "minimum": 0.0,
      "maximum": 1.0
    },
    "reasoning": {
      "type": "string",
      "maxLength": 200
    },
    "alternative_category_id": {
      "type": ["string", "null"],
      "enum": [
        "CAT-02.01", "CAT-02.02", "CAT-02.03", "CAT-02.04", "CAT-02.05",
        "CAT-02.06", "CAT-02.07", "CAT-02.08", "CAT-02.09", "CAT-02.10",
        "CAT-02.11", "CAT-02.12",
        null
      ]
    }
  }
}
```

### 4.4 Level 2 Classification Schema - Healthcare & Medical (CAT-07)

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "ce://docdigitizer.com/schemas/ce/level2/CAT-07/v1.0",
  "title": "CE Level 2 - Healthcare & Medical Sub-Classification",
  "description": "Category Extraction schema for Level 2 classification within Healthcare & Medical. Document has been classified as CAT-07 at Level 1.",
  "type": "object",
  "required": ["category_id", "category_name", "confidence"],
  "additionalProperties": false,
  "properties": {
    "category_id": {
      "type": "string",
      "description": "The Level 2 sub-category identifier",
      "enum": [
        "CAT-07.01", "CAT-07.02", "CAT-07.03", "CAT-07.04",
        "CAT-07.05", "CAT-07.06", "CAT-07.07", "CAT-07.08"
      ]
    },
    "category_name": {
      "type": "string",
      "enum": [
        "Clinical Records",
        "Prescriptions & Medication",
        "Diagnostic Reports",
        "Medical Billing & Coding",
        "Patient Intake & Consent",
        "Referrals & Authorizations",
        "Medical Certificates",
        "Clinical Trial & Research"
      ]
    },
    "confidence": {
      "type": "number",
      "minimum": 0.0,
      "maximum": 1.0
    },
    "reasoning": {
      "type": "string",
      "maxLength": 200
    },
    "alternative_category_id": {
      "type": ["string", "null"],
      "enum": [
        "CAT-07.01", "CAT-07.02", "CAT-07.03", "CAT-07.04",
        "CAT-07.05", "CAT-07.06", "CAT-07.07", "CAT-07.08",
        null
      ]
    }
  }
}
```

---

## 5. Ambiguity Resolution

### 5.1 Common Ambiguity Patterns

Documents frequently straddle category boundaries. The following table identifies the most common ambiguities and the resolution rules.

| Document | Candidate A | Candidate B | Resolution Rule | Assigned To |
|----------|-------------|-------------|-----------------|-------------|
| Employment contract | CAT-02 (Legal) | CAT-03 (HR) | **Primary function rule**: If the primary use is hiring/employment management, assign to HR. If used in a legal dispute or review, assign to Legal. Default: **CAT-02.04** (Employment Agreements under Legal) because it is a binding contract. | CAT-02.04 |
| Utility bill used as proof of address | CAT-17 (Utilities) | CAT-10 (Identity) | **Original purpose rule**: Classify by what the document IS, not how it is being used. A utility bill is always a utility bill. | CAT-17.01 |
| Medical insurance claim | CAT-07 (Healthcare) | CAT-06 (Insurance) | **Issuer rule**: If issued by a healthcare provider, it goes to Healthcare. If issued by an insurance company, it goes to Insurance. | Depends on issuer |
| Tax invoice | CAT-01 (Financial) | CAT-04 (Tax) | **Transaction vs. compliance rule**: If the document is a transactional record (invoice), it goes to Financial. If it is a filing or compliance document, it goes to Tax. | CAT-01.01 |
| Bank loan agreement | CAT-05 (Banking) | CAT-02 (Legal) | **Domain-over-form rule**: Even though it is a contract, it is a banking product document. Classify by the specialized domain. | CAT-05.02 |
| Property lease | CAT-11 (Real Estate) | CAT-02 (Legal) | **Domain-over-form rule**: Property-specific contracts belong under Real Estate. | CAT-11.03 |
| Purchase order | CAT-09 (Procurement) | CAT-01 (Financial) | **Process stage rule**: Documents in the pre-payment cycle (ordering) go to Procurement. Documents in the post-delivery cycle (invoicing, payment) go to Financial. | CAT-09.01 |
| Payroll tax form | CAT-03 (HR) | CAT-04 (Tax) | **Regulatory destination rule**: If the form is filed with a tax authority, it goes to Tax. If it is an internal payroll record, it goes to HR. | Depends on use |

### 5.2 Resolution Hierarchy

When ambiguity cannot be resolved by specific rules, apply this priority cascade:

1. **Explicit document title/header** - If the document self-identifies (e.g., "INVOICE", "MEDICAL RECORD"), trust it.
2. **Issuing entity** - Classify by who created it (bank document goes to Banking, hospital document goes to Healthcare).
3. **Primary transactional purpose** - What is the document's main function in a business process?
4. **Specialized domain over generic form** - A mortgage agreement is Real Estate, not Legal, even though it is technically a contract.
5. **Original purpose over current use** - A utility bill is a utility bill, even if being used for KYC.

### 5.3 Confidence-Based Routing

```
If confidence >= 0.85:  Route to primary category, no human review
If 0.70 <= confidence < 0.85:  Route to primary category, flag for optional review
If 0.50 <= confidence < 0.70:  Route to primary category, REQUIRE human review
If confidence < 0.50:  Route to CAT-20 (Uncategorized), trigger human classification
```

### 5.4 Multi-Document Composites

When a single file contains multiple document types (e.g., a PDF with an invoice, a packing list, and a delivery note):

1. If the system supports document splitting, split first and classify each separately.
2. If splitting is not available, classify by the **first/primary document** in the file.
3. Tag the document as `composite: true` in metadata so downstream processes know it contains mixed content.
4. Route to CAT-20.02 if no primary document can be determined.

---

## 6. Regional and Industry Variations

### 6.1 Regional Variations

L1 and L2 categories are intentionally geography-neutral. Regional variations manifest at L3+ through specific document types. However, the relative volume of each category shifts significantly by region.

| Region | High-Volume Shift | Key L3+ Variations |
|--------|------------------|---------------------|
| **United States** | Higher Insurance (CAT-06), Healthcare (CAT-07) volumes due to private healthcare system | W-2, 1099, 1040, HIPAA forms, UCC filings, EOBs |
| **European Union** | Higher Tax/Regulatory (CAT-04) due to VAT complexity and GDPR | SEPA documents, EU standard invoices (EN 16931), GDPR consent forms, CE declarations |
| **United Kingdom** | Post-Brexit trade documents in Supply Chain (CAT-08) | HMRC forms, Companies House filings, UK GDPR forms |
| **Portugal** | Higher Tax (CAT-04) volume; e-invoicing mandates | SAF-T PT, IRS Modelo 3, Certidao Permanente, faturas eletronicas |
| **Brazil** | Extremely high Tax (CAT-04) due to complex tax system | NF-e, NFS-e, CT-e, SPED filings, DARF payment slips |
| **DACH (Germany/Austria/Switzerland)** | High Technical/Engineering (CAT-15) due to manufacturing sector | TUV certificates, DIN standards, ZUGFeRD invoices, Handelsregister extracts |
| **Middle East / GCC** | Higher Government (CAT-12) and Identity (CAT-10) volumes | Arabic/bilingual documents, legalized/apostilled documents, Sharia-compliant financial instruments |
| **Asia-Pacific** | Varied; high Supply Chain (CAT-08) in manufacturing hubs | Bilingual documents, specific customs forms, local tax formats (e.g., Japan's Consumption Tax) |

### 6.2 Industry-Specific Volume Profiles

Different industries will use the same L1/L2 taxonomy but with vastly different volume distributions. This affects CE optimization (caching, model fine-tuning).

| Industry | Dominant L1 Categories | Key Notes |
|----------|----------------------|-----------|
| **Banking & Finance** | CAT-05 (40%), CAT-01 (20%), CAT-02 (15%) | Very high volume of bank statements, loan docs, KYC documents. L3+ needs deep specialization. |
| **Healthcare** | CAT-07 (45%), CAT-06 (20%), CAT-01 (10%) | HIPAA/privacy concerns; medical terminology requires specialized models at L3+. |
| **Manufacturing** | CAT-01 (25%), CAT-08 (20%), CAT-09 (15%), CAT-15 (15%) | Heavy supply chain and procurement document flows. |
| **Legal Services** | CAT-02 (50%), CAT-12 (15%), CAT-13 (10%) | Contract-heavy; needs deep L3+ for contract type classification. |
| **Retail / E-Commerce** | CAT-01 (35%), CAT-08 (20%), CAT-09 (15%) | High invoice and receipt volumes; simpler schemas on average. |
| **Government / Public Sector** | CAT-12 (30%), CAT-04 (20%), CAT-10 (15%) | Form-heavy; many standardized document types. |
| **Insurance** | CAT-06 (40%), CAT-07 (15%), CAT-01 (15%) | Claims processing is the dominant use case. |
| **Real Estate** | CAT-11 (35%), CAT-05 (20%), CAT-02 (15%) | Mortgage processing drives enormous volume. |
| **Logistics & Shipping** | CAT-08 (45%), CAT-04.05 (15%), CAT-01 (15%) | International trade document complexity is high. |

### 6.3 Implications for the Schema Engine

- **Organization-level defaults**: When an organization is onboarded, their industry should set the default volume profile to optimize CE caching and model routing.
- **Regional schema variants**: The same L2 category (e.g., CAT-01.01 "Invoices") will have multiple L3+ schemas for different regional standards (e-Factura, ZUGFeRD, Peppol BIS, SAF-T).
- **Language detection**: Should occur before or in parallel with L1 CE, as it informs which regional L3+ path to follow.

---

## 7. Growth Strategy

### 7.1 Phased Rollout

| Phase | Timeline | Scope | Goal |
|-------|----------|-------|------|
| **Phase 1: Foundation** | Months 1-3 | L1 + L2 for CAT-01, CAT-02, CAT-03, CAT-05, CAT-08 | Cover ~60% of IDP volume with the top 5 most common L1 categories |
| **Phase 2: Expansion** | Months 4-6 | L2 for all remaining L1s; L3 for CAT-01 and CAT-02 | Full L2 coverage; deep L3 for invoices and contracts |
| **Phase 3: Depth** | Months 7-12 | L3 for all major L1s; L4 for CAT-01.01 (Invoices) | Support region-specific schemas; handle long-tail document types |
| **Phase 4: Community** | Ongoing | Community-contributed schemas at L3+; maturity promotion | Self-sustaining taxonomy growth through document processing feedback |

### 7.2 Taxonomy Evolution Rules

1. **Adding new L2 categories**: Requires review by the taxonomy committee (minimum 3 people). Must demonstrate that existing L2s do not adequately cover the document type and that the new category will receive meaningful volume.

2. **Adding L3+ categories**: Can be proposed by any team member or auto-suggested by the system. Requires one reviewer for "community" status, three reviewers for "production" status.

3. **Deprecating categories**: A category is deprecated (never deleted) if it has received <1% of its parent's volume for 6 consecutive months and an alternative classification path exists. Deprecated categories continue to work but are hidden from new schema creation.

4. **Splitting categories**: When a L2 category grows beyond 20 L3 children, it should be reviewed for splitting into two L2 categories. This is the only restructuring operation permitted and requires version bumping the parent L1 CE schema.

5. **Merging categories**: Extremely rare. Only permitted if two categories have >80% schema overlap and users consistently misclassify between them. Requires major version bump.

### 7.3 Feedback-Driven Growth

The system should track:

| Metric | Trigger |
|--------|---------|
| Documents routed to CAT-20 | >5% of total volume suggests missing categories |
| Low-confidence L1 classifications (<0.7) | >10% suggests ambiguous category definitions |
| Human overrides of CE classification | Pattern analysis reveals systematic misclassification |
| Schema suggestion frequency for an L2 | >20 unique schema suggestions indicates L3 refinement needed |
| Cross-category confusion matrix | Pairs with >15% confusion rate need clearer differentiation |

### 7.4 Versioning Strategy

```
CE Schema Version Format: MAJOR.MINOR

MAJOR bump: Categories added, removed, or restructured at that level
MINOR bump: Description changes, example updates, enum reordering

Examples:
  ce://docdigitizer.com/schemas/ce/level1/v1.0  (initial)
  ce://docdigitizer.com/schemas/ce/level1/v1.1  (description fixes)
  ce://docdigitizer.com/schemas/ce/level1/v2.0  (new L1 category added)
  ce://docdigitizer.com/schemas/ce/level2/CAT-01/v1.0  (initial)
  ce://docdigitizer.com/schemas/ce/level2/CAT-01/v2.0  (L2 category split)
```

All versions remain available. The system routes to the latest production version by default, but organizations can pin to a specific version.

---

## 8. Statistics

### 8.1 Taxonomy Summary

| Metric | Count |
|--------|-------|
| **Level 1 Categories** | 20 |
| **Level 2 Categories (Total)** | 162 |
| **Average L2 per L1** | 8.1 |
| **Min L2 per L1** | 5 (CAT-18: Travel & Expense, CAT-17: Utilities) |
| **Max L2 per L1** | 12 (CAT-02: Legal & Contracts) |

### 8.2 Level 2 Breakdown by L1

| L1 Category | L2 Count |
|-------------|----------|
| CAT-01 Financial & Accounting | 10 |
| CAT-02 Legal & Contracts | 12 |
| CAT-03 Human Resources & Employment | 10 |
| CAT-04 Tax & Regulatory Compliance | 10 |
| CAT-05 Banking & Financial Services | 9 |
| CAT-06 Insurance | 8 |
| CAT-07 Healthcare & Medical | 8 |
| CAT-08 Supply Chain & Logistics | 7 |
| CAT-09 Procurement & Purchasing | 7 |
| CAT-10 Identity & Personal Documents | 6 |
| CAT-11 Real Estate & Property | 6 |
| CAT-12 Government & Public Administration | 7 |
| CAT-13 Corporate Governance | 6 |
| CAT-14 Education & Academic | 6 |
| CAT-15 Technical & Engineering | 7 |
| CAT-16 Marketing & Communications | 6 |
| CAT-17 Utilities & Telecommunications | 5 |
| CAT-18 Travel & Expense | 5 |
| CAT-19 Correspondence & General | 6 |
| CAT-20 Uncategorized / Unknown | 5 |
| **TOTAL** | **162** |

### 8.3 Estimated Level 3 Requirements

| L1 Category | Est. L3 Categories | Rationale |
|-------------|-------------------|-----------|
| CAT-01 Financial & Accounting | 60-80 | Invoices alone need 15+ regional/format variants; financial statements vary by standard (IFRS, GAAP, local) |
| CAT-02 Legal & Contracts | 50-70 | Each contract sub-type has many industry-specific variants |
| CAT-03 Human Resources & Employment | 30-40 | Moderate; many forms are organization-specific rather than type-specific |
| CAT-04 Tax & Regulatory Compliance | 80-120 | Extremely region-dependent; each country has dozens of specific forms |
| CAT-05 Banking & Financial Services | 40-50 | Product-specific variations (checking, savings, investment, etc.) |
| CAT-06 Insurance | 40-60 | Line-of-business variations (auto, property, life, health, etc.) |
| CAT-07 Healthcare & Medical | 50-70 | Specialty-specific clinical documents; billing codes vary by system |
| CAT-08 Supply Chain & Logistics | 30-40 | Transport mode variations (air, sea, road, rail) and regional customs |
| CAT-09 Procurement & Purchasing | 20-30 | Relatively standardized across industries |
| CAT-10 Identity & Personal Documents | 40-60 | Every country has unique ID document formats |
| CAT-11 Real Estate & Property | 25-35 | Jurisdiction-specific property law creates variants |
| CAT-12 Government & Public Administration | 60-100 | Extremely jurisdiction-dependent |
| CAT-13 Corporate Governance | 20-30 | Relatively standardized but varies by corporate law regime |
| CAT-14 Education & Academic | 15-25 | Moderate variation by institution type and country |
| CAT-15 Technical & Engineering | 25-35 | Industry-specific standards (automotive, aerospace, pharma, etc.) |
| CAT-16 Marketing & Communications | 10-15 | Relatively few standardized types |
| CAT-17 Utilities & Telecommunications | 15-20 | Utility type variations (electric, water, gas, telecom, internet) |
| CAT-18 Travel & Expense | 10-15 | Relatively simple and standardized |
| CAT-19 Correspondence & General | 10-15 | Intentionally low; specific types graduate to proper categories |
| CAT-20 Uncategorized / Unknown | 5-10 | Structural sub-types only; content is by definition unclassified |
| **TOTAL ESTIMATED** | **635-930** | |

### 8.4 Token Cost Estimates for CE

| CE Level | Enum Size | Est. Schema Tokens | Est. Total Tokens (with document) | Latency Estimate |
|----------|-----------|--------------------|------------------------------------|------------------|
| Level 1 | 20 options | ~400 tokens | ~1,500-3,000 tokens | 0.5-1.5s |
| Level 2 | 5-12 options | ~150-300 tokens | ~1,200-2,500 tokens | 0.4-1.2s |
| Level 3 | 5-20 options | ~200-500 tokens | ~1,300-2,800 tokens | 0.4-1.5s |
| **Total CE Pipeline (3 levels)** | - | ~750-1,200 tokens | ~4,000-8,300 tokens | 1.3-4.2s |

These estimates assume a mid-sized document (~1,000 tokens for content representation) processed with a fast model (e.g., GPT-4o-mini, Claude 3.5 Haiku, or Gemini Flash). The iterative approach keeps each individual call small and fast, which is the core advantage of this architecture.

### 8.5 Maximum Category Space

The taxonomy supports polynomial growth as described in the original architecture:

```
Level 1: 20 categories
Level 2: 20 x ~8 = ~162 categories
Level 3: 162 x ~5 = ~810 estimated categories
Level 4: 810 x ~3 = ~2,430 estimated leaf schemas

Total addressable schema space: ~2,400+ unique document types
Through only 3-4 LLM calls of 5-20 options each.
```

This compares favorably to a flat classification approach which would require the LLM to choose from 2,400+ options in a single call -- an approach that would be prohibitively expensive, slow, and inaccurate.

---

## Appendix A: Quick Reference - Full L1/L2 Tree

```
CAT-01  Financial & Accounting
  CAT-01.01  Invoices & Bills
  CAT-01.02  Receipts & Proof of Payment
  CAT-01.03  Credit & Debit Notes
  CAT-01.04  Financial Statements
  CAT-01.05  Expense Reports & Reimbursements
  CAT-01.06  Budgets & Forecasts
  CAT-01.07  Journal Entries & Ledgers
  CAT-01.08  Accounts Payable / Receivable
  CAT-01.09  Payroll Documents
  CAT-01.10  Audit & Reconciliation

CAT-02  Legal & Contracts
  CAT-02.01  Service Agreements
  CAT-02.02  Sales & Purchase Agreements
  CAT-02.03  Lease & Rental Agreements
  CAT-02.04  Employment & Engagement Agreements
  CAT-02.05  Non-Disclosure & Confidentiality
  CAT-02.06  Licensing & IP Agreements
  CAT-02.07  Legal Notices & Correspondence
  CAT-02.08  Court & Litigation Documents
  CAT-02.09  Powers of Attorney & Authorizations
  CAT-02.10  Terms & Policies
  CAT-02.11  Amendments & Addenda
  CAT-02.12  Settlement & Release Agreements

CAT-03  Human Resources & Employment
  CAT-03.01  Recruitment & Applications
  CAT-03.02  Onboarding & Employment Setup
  CAT-03.03  Compensation & Benefits
  CAT-03.04  Performance Management
  CAT-03.05  Leave & Attendance
  CAT-03.06  Training & Development
  CAT-03.07  Disciplinary & Grievance
  CAT-03.08  Termination & Offboarding
  CAT-03.09  Organizational & Policies
  CAT-03.10  Employee Verification

CAT-04  Tax & Regulatory Compliance
  CAT-04.01  Income Tax Returns & Filings
  CAT-04.02  VAT / Sales Tax Documents
  CAT-04.03  Withholding & Reporting Forms
  CAT-04.04  Tax Assessments & Notices
  CAT-04.05  Customs & Duties
  CAT-04.06  Regulatory Filings
  CAT-04.07  Compliance Certificates
  CAT-04.08  Audit Reports
  CAT-04.09  Transfer Pricing Documentation
  CAT-04.10  Tax Exemptions & Incentives

CAT-05  Banking & Financial Services
  CAT-05.01  Bank Statements
  CAT-05.02  Loan & Mortgage Documents
  CAT-05.03  Payment & Transfer Records
  CAT-05.04  Credit & Debit Card Documents
  CAT-05.05  Investment Documents
  CAT-05.06  Account Opening & KYC
  CAT-05.07  Letters of Credit & Guarantees
  CAT-05.08  Foreign Exchange Documents
  CAT-05.09  Banking Correspondence

CAT-06  Insurance
  CAT-06.01  Insurance Policies
  CAT-06.02  Claims & Loss Reports
  CAT-06.03  Underwriting Documents
  CAT-06.04  Certificates of Insurance
  CAT-06.05  Premium & Billing Documents
  CAT-06.06  Endorsements & Riders
  CAT-06.07  Adjuster & Investigation Reports
  CAT-06.08  Reinsurance Documents

CAT-07  Healthcare & Medical
  CAT-07.01  Clinical Records
  CAT-07.02  Prescriptions & Medication
  CAT-07.03  Diagnostic Reports
  CAT-07.04  Medical Billing & Coding
  CAT-07.05  Patient Intake & Consent
  CAT-07.06  Referrals & Authorizations
  CAT-07.07  Medical Certificates
  CAT-07.08  Clinical Trial & Research

CAT-08  Supply Chain & Logistics
  CAT-08.01  Shipping & Transportation
  CAT-08.02  Customs & Trade
  CAT-08.03  Packing & Inventory
  CAT-08.04  Warehouse & Storage
  CAT-08.05  Delivery & Proof of Delivery
  CAT-08.06  Freight & Carrier Documents
  CAT-08.07  Dangerous Goods & Compliance

CAT-09  Procurement & Purchasing
  CAT-09.01  Purchase Orders
  CAT-09.02  Quotations & Proposals
  CAT-09.03  Requisitions
  CAT-09.04  Supplier Management
  CAT-09.05  Goods Receipt & Inspection
  CAT-09.06  Request for Proposals / Information
  CAT-09.07  Catalogs & Price Lists

CAT-10  Identity & Personal Documents
  CAT-10.01  Government-Issued ID
  CAT-10.02  Proof of Address
  CAT-10.03  Civil Status Documents
  CAT-10.04  Immigration & Visa Documents
  CAT-10.05  Professional Certifications
  CAT-10.06  Background & Verification

CAT-11  Real Estate & Property
  CAT-11.01  Deeds & Title Documents
  CAT-11.02  Property Valuation
  CAT-11.03  Property Leases
  CAT-11.04  Property Management
  CAT-11.05  Mortgage & Financing
  CAT-11.06  Zoning & Land Use

CAT-12  Government & Public Administration
  CAT-12.01  Permits & Licenses
  CAT-12.02  Government Notices & Orders
  CAT-12.03  Court & Judicial Documents
  CAT-12.04  Government Benefits
  CAT-12.05  Public Records & Registrations
  CAT-12.06  Government Contracts & Grants
  CAT-12.07  Freedom of Information & Disclosure

CAT-13  Corporate Governance
  CAT-13.01  Incorporation & Formation
  CAT-13.02  Board & Shareholder Documents
  CAT-13.03  Annual Reports & Filings
  CAT-13.04  Corporate Changes
  CAT-13.05  Shareholder & Equity Documents
  CAT-13.06  Corporate Policies & Charters

CAT-14  Education & Academic
  CAT-14.01  Academic Records & Transcripts
  CAT-14.02  Diplomas & Certificates
  CAT-14.03  Enrollment & Admissions
  CAT-14.04  Financial Aid & Scholarships
  CAT-14.05  Research & Publications
  CAT-14.06  Accreditation & Institutional

CAT-15  Technical & Engineering
  CAT-15.01  Technical Specifications
  CAT-15.02  Engineering Drawings & Plans
  CAT-15.03  Manuals & Instructions
  CAT-15.04  Safety Data & Compliance
  CAT-15.05  Test & Quality Reports
  CAT-15.06  Project & Change Documentation
  CAT-15.07  Patents & Technical IP

CAT-16  Marketing & Communications
  CAT-16.01  Proposals & Presentations
  CAT-16.02  Brochures & Collateral
  CAT-16.03  Press & Media
  CAT-16.04  Advertising & Campaign
  CAT-16.05  Brand & Creative
  CAT-16.06  Customer Communications

CAT-17  Utilities & Telecommunications
  CAT-17.01  Utility Bills & Statements
  CAT-17.02  Telecom Bills & Statements
  CAT-17.03  Service Contracts
  CAT-17.04  Consumption & Usage Reports
  CAT-17.05  Installation & Maintenance

CAT-18  Travel & Expense
  CAT-18.01  Travel Bookings & Itineraries
  CAT-18.02  Boarding & Transportation
  CAT-18.03  Accommodation Documents
  CAT-18.04  Expense Claims & Reports
  CAT-18.05  Travel Policies & Approvals

CAT-19  Correspondence & General
  CAT-19.01  Business Letters
  CAT-19.02  Internal Memos & Notes
  CAT-19.03  Emails (Printed/Archived)
  CAT-19.04  Fax & Legacy Communications
  CAT-19.05  Forms & Questionnaires
  CAT-19.06  Meeting & Event Documents

CAT-20  Uncategorized / Unknown
  CAT-20.01  Low Confidence Classification
  CAT-20.02  Multi-Type Composites
  CAT-20.03  Damaged or Illegible
  CAT-20.04  Novel Document Types
  CAT-20.05  Non-Document Files
```

---

## Appendix B: Decision Flowchart for CE Pipeline

```
Document Arrives
       |
       v
[Language Detection] -- parallel with L1 CE
       |
       v
[L1 CE Schema] --> confidence >= 0.50?
       |                    |
      YES                   NO --> Route to CAT-20, flag for human review
       |
       v
[L2 CE Schema for selected L1] --> confidence >= 0.50?
       |                                    |
      YES                                   NO --> Use L1 category only,
       |                                          flag for L2 human review
       v
[Check: Does L2 have L3 sub-categories?]
       |                    |
      YES                   NO --> Retrieve schemas tagged with L2 category
       |                          Execute extraction
       v
[L3 CE Schema] --> confidence >= 0.50?
       |                    |
      YES                   NO --> Use L2 category, retrieve broader schema set
       |
       v
[Retrieve schemas matching L3 category path]
       |
       v
[Schema count <= 20?]
       |           |
      YES          NO --> Run L4 CE or apply additional filters
       |
       v
[Execute extraction with matched schemas]
       |
       v
[Best schema match found?]
       |              |
      YES              NO --> Suggest new schema (community maturity)
       |
       v
[Return extraction result]
```

---

*End of Report 3*
*DocDigitizer Schema Engine - Category Taxonomy v1.0*
