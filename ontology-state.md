# Document Type Ontology - Development State

## Root Node Descriptions

### 1. Legal & Compliance
Documents related to the establishment, enforcement, and adherence to laws, regulations, and contractual obligations. Includes agreements between parties, dispute resolution records, regulatory filings, and intellectual property protections.

### 2. Finance, Accounting & Tax
Documents that record, report, or facilitate the management of monetary resources. Covers investment and banking instruments, financial reporting, day-to-day bookkeeping, and tax obligations to authorities.

### 3. Insurance
Documents governing the transfer and management of risk through insurance products. Encompasses policy issuance, claims lifecycle, actuarial and underwriting assessments, and regulatory filings specific to the insurance industry.

### 4. Healthcare & Medical
Documents produced in the delivery, administration, and regulation of healthcare services. Includes patient-facing clinical records, pharmaceutical documentation, laboratory and diagnostic outputs, and healthcare compliance records.

### 5. Government & Public Administration
Documents issued by, submitted to, or maintained by government entities at any level. Covers identity and civil status records, permits and licensing, public policy instruments, and official government correspondence.

### 6. Education & Research
Documents produced within educational institutions and research environments. Spans student and academic records, curriculum design, scientific publications, research data, and funding or grant documentation.

### 7. Human Resources & Employment
Documents that support the full employee lifecycle within an organization — from recruitment and onboarding through active employment to separation. Includes talent acquisition, personnel records, compensation and benefits administration, and performance management.

### 8. Sales, Marketing & Procurement
Documents that facilitate commercial activity — selling, promoting, and purchasing goods or services. Covers sales proposals and order management, marketing strategy and campaigns, vendor and supplier management, and external business communications.

### 9. Technical & Product Documentation
Documents that describe the design, implementation, operation, or use of technical systems and products. Divided between internal engineering-facing documentation (architecture, APIs, runbooks) and external user-facing documentation (manuals, guides, release notes).

### 10. Operations & Corporate Governance
Documents that govern how an organization runs internally. Includes standard operating procedures, corporate governance records (board resolutions, bylaws), strategic planning artifacts, internal policies, and logistics and supply chain documentation.

### 11. Real Estate & Property
Documents related to the ownership, transfer, valuation, and development of real property. Covers title and deed records, lease agreements, property appraisals, and construction or development documentation.

### 12. Creative & Literary Works
Documents that are primarily artistic, narrative, or expressive in nature. Includes screenplays, novels, poetry, and creative briefs used in media, publishing, and entertainment contexts.

---

## Expansion Order

Expand the most ambiguous and cross-cutting nodes first, so that later expansions inherit clear boundaries rather than creating conflicts.

| Phase | Root | Reasoning |
|-------|------|-----------|
| 1 | Legal & Compliance | Most cross-cutting, sets boundaries for everything |
| 2 | Finance, Accounting & Tax | Heavy overlap with Legal, Insurance, Sales |
| 3 | Insurance | Now that Legal and Finance are defined, Insurance boundaries are clear |
| 4 | Operations & Corporate Governance | Internal catch-all risk — define edges against HR, Technical |
| 5 | Human Resources & Employment | Borders Operations (policies) and Legal (contracts) — both now defined |
| 6 | Sales, Marketing & Procurement | Borders Finance (payments) and Legal (contracts) — both now defined |
| 7 | Healthcare & Medical | Domain-specific but has compliance and research borders |
| 8 | Government & Public Administration | Borders Legal and Healthcare — both now defined |
| 9 | Education & Research | Borders Government and Healthcare |
| 10 | Technical & Product Documentation | Mostly self-contained, slight border with Operations |
| 11 | Real Estate & Property | Self-contained |
| 12 | Creative & Literary Works | Self-contained |

---

## Expansion Checklist (BFS - Level by Level)

### Level 2 Expansion (Root → Children)

- [x] 1. Legal & Compliance
  - **Contracts & Agreements** — Documents that formalize legally binding obligations between two or more parties. Scope: service agreements, NDAs, SLAs, MOUs, licensing agreements.
  - **Court & Litigation** — Documents produced in or related to judicial proceedings and dispute resolution. Scope: lawsuit filings, pleadings, court orders, judgments, evidence exhibits, settlement documents.
  - **Regulatory & Compliance** — Documents that demonstrate, monitor, or enforce adherence to laws, regulations, and industry standards at an organizational level. Scope: compliance reports, audit findings, regulatory filings, policy adherence records, sanctions documentation.
  - **Intellectual Property** — Documents that establish, protect, or transfer ownership of intangible creations of the mind. Scope: patents, trademarks, copyrights, trade secret documentation, IP assignment agreements.
  - **Corporate Legal** — Documents that define the legal identity, structure, and governance rights of a corporate entity. Scope: articles of incorporation, bylaws amendments, shareholder agreements, corporate resolutions (legal-facing).
  - **Notarial & Certified Documents** — Documents that have been formally authenticated, witnessed, or certified by a recognized authority to confirm their validity. Scope: notarized documents, apostilles, certified copies, legalizations.

- [x] 2. Finance, Accounting & Tax
  - **Banking & Investments** — Documents related to the management of funds through banking institutions and investment vehicles. Scope: bank statements, loan agreements, investment portfolios, fund prospectuses, credit facility documents.
  - **Financial Reporting** — Documents that summarize an organization's financial position and performance for stakeholders. Scope: balance sheets, income statements, cash flow statements, annual reports, consolidated financials.
  - **Bookkeeping & Ledgers** — Documents that record the day-to-day financial transactions of an organization in a systematic and chronological manner. Scope: general ledgers, journals, trial balances, chart of accounts, reconciliation records.
  - **Invoicing & Payments** — Documents that request, confirm, or record the exchange of monetary value for goods or services. Scope: invoices, receipts, payment confirmations, credit/debit notes, billing statements, utility bills, expense reports.
  - **Tax Filings & Records** — Documents related to the calculation, reporting, and payment of taxes to governmental authorities. Scope: tax returns, tax assessments, withholding certificates, tax correspondence, transfer pricing documentation.
  - **Audit** — Documents produced during the independent examination of financial records to verify accuracy and compliance with standards. Scope: internal/external audit reports, audit plans, management letters, findings and remediation records.

- [x] 3. Insurance
  - **Policies & Coverage** — Documents that define the terms, conditions, and scope of insurance coverage between an insurer and a policyholder. Scope: policy documents, endorsements, declarations pages, certificates of insurance, coverage summaries.
  - **Claims & Settlements** — Documents generated throughout the lifecycle of an insurance claim, from initial filing through resolution. Scope: claim forms, loss reports, settlement agreements, subrogation records, adjuster reports.
  - **Underwriting & Actuarial** — Documents used to evaluate, price, and manage insurance risk through statistical and financial analysis. Scope: risk assessments, underwriting guidelines, actuarial reports, reinsurance documents, loss ratio analyses.
  - **Insurance Regulatory** — Documents required by insurance regulators to ensure solvency, market conduct, and consumer protection within the insurance industry. Scope: insurance-specific regulatory filings, solvency reports, rate filings, market conduct reports.

- [x] 4. Healthcare & Medical
  - **Patient Records** — Documents that capture the medical history, treatment, and ongoing care of an individual patient. Scope: medical histories, consultation notes, discharge summaries, consent forms, referral letters.
  - **Clinical** — Documents produced in the context of clinical trials and structured medical research on human subjects. Scope: clinical trial documents, protocols, case report forms, adverse event reports, informed consent (trials).
  - **Pharmaceutical** — Documents related to the development, approval, distribution, and monitoring of pharmaceutical products. Scope: drug approval filings, prescriptions, formularies, medication records, pharmacovigilance reports.
  - **Lab & Diagnostics** — Documents generated by laboratory testing and diagnostic imaging procedures. Scope: lab results, imaging reports, pathology reports, test requisitions, quality control records.
  - **Healthcare Administration** — Documents that support the operational and regulatory management of healthcare facilities and providers. Scope: facility licenses, accreditation documents, healthcare compliance records, credentialing, payer contracts.

- [x] 5. Government & Public Administration
  - **Identity Documents** — Official documents issued by government authorities that verify the identity of an individual. Scope: passports, national IDs, driver's licenses, social security cards, residency permits.
  - **Civil Records** — Official government records that document key life events and civil status changes of individuals. Scope: birth/marriage/death certificates, citizenship documents, name change records, adoption records.
  - **Permits & Licenses** — Documents issued by government entities granting authorization to perform specific activities or operate in regulated domains. Scope: business licenses, building permits, environmental permits, professional licenses, zoning approvals.
  - **Public Policy & Legislation** — Documents that establish, propose, or interpret the rules and laws that govern society. Scope: laws, regulations, executive orders, policy papers, government gazettes, legislative records.
  - **Government Correspondence** — Official communications issued by or directed to government entities in their administrative capacity. Scope: official letters, FOI requests/responses, public notices, government reports, inter-agency memos.

- [x] 6. Education & Research
  - **Academic Records** — Documents that formally attest to a student's enrollment, progress, and achievement within an educational institution. Scope: transcripts, diplomas, enrollment records, student evaluations, degree audits.
  - **Curriculum & Instruction** — Documents that define what is taught, how it is taught, and how learning is assessed within educational programs. Scope: syllabi, lesson plans, course catalogs, educational standards, assessment frameworks.
  - **Scientific Research** — Documents that present, support, or review original research findings and methodologies. Scope: research papers, peer reviews, lab notebooks, datasets, methodologies, preprints.
  - **Grants & Funding** — Documents related to the application, awarding, and administration of financial support for research and educational projects. Scope: grant proposals, award letters, progress reports, budget justifications, funding agreements.
  - **Institutional Administration** — Documents that govern the operation, quality assurance, and accreditation of educational and research institutions. Scope: accreditation documents, institutional policies, faculty records, program reviews.

- [x] 7. Human Resources & Employment
  - **Recruitment & Talent Acquisition** — Documents generated during the process of attracting, evaluating, and hiring candidates for employment. Scope: job postings, applications, resumes/CVs, interview evaluations, offer letters.
  - **Employee Records** — Documents that constitute the official personnel file of an individual throughout their employment lifecycle. Scope: personnel files, employment contracts, onboarding documents, termination records, organizational charts.
  - **Compensation & Benefits** — Documents related to the financial and non-financial rewards provided to employees in exchange for their work. Scope: payroll records, salary structures, benefits enrollment, stock option agreements, bonus documentation.
  - **Performance Management** — Documents that track, evaluate, and develop employee performance against organizational expectations. Scope: performance reviews, goal-setting documents, improvement plans, 360 feedback, promotion records.
  - **Training & Development** — Documents that support the planning, delivery, and tracking of employee learning and professional growth. Scope: training materials, certification records, development plans, attendance records, competency assessments.

- [x] 8. Sales, Marketing & Procurement
  - **Sales** — Documents produced during the process of selling goods or services, from opportunity identification through deal closure. Scope: proposals, quotes, sales orders, sales reports, CRM exports, deal documentation, win/loss analyses.
  - **Marketing & Communications** — Documents that plan, execute, and measure efforts to promote an organization's brand, products, or services to external audiences. Scope: campaign plans, brand guidelines, press releases, newsletters, market research, advertising materials.
  - **Procurement & Vendor Management** — Documents that govern the sourcing, purchasing, and ongoing management of external suppliers and vendors. Scope: purchase orders, RFPs/RFQs, vendor contracts, supplier evaluations, receiving reports.
  - **Customer Relations** — Documents that capture interactions with customers after initial sale, focused on satisfaction, retention, and support. Scope: customer correspondence, complaint records, satisfaction surveys, support tickets, account reviews.

- [x] 9. Technical & Product Documentation
  - **System & Architecture** — Documents that describe the high-level design, infrastructure, and structural decisions of technical systems. Scope: system design docs, architecture diagrams, infrastructure specs, network diagrams, capacity plans.
  - **API & Developer Documentation** — Documents that enable developers to understand, integrate with, and build upon software interfaces and platforms. Scope: API references, SDK docs, developer guides, integration documentation, code samples.
  - **Engineering Specs** — Documents that define the detailed technical requirements, design decisions, and validation criteria for systems or components. Scope: technical specifications, requirements documents, design reviews, test plans, technical decision records.
  - **User-Facing Documentation** — Documents written for end users to help them understand and effectively use a product or service. Scope: user manuals, how-to guides, FAQs, tutorials, release notes, knowledge base articles.
  - **Operational Runbooks** — Documents that provide step-by-step procedures for operating, maintaining, and troubleshooting production systems. Scope: incident response playbooks, deployment guides, monitoring configurations, troubleshooting guides.

- [x] 10. Operations & Corporate Governance
  - **Standard Operating Procedures** — Documents that prescribe the step-by-step instructions for carrying out routine organizational processes. Scope: SOPs, work instructions, process maps, operational guidelines, checklists.
  - **Corporate Governance** — Documents that record the decisions, structures, and oversight mechanisms of an organization's governing bodies. Scope: board resolutions, meeting minutes, shareholder communications, committee charters, annual governance reports.
  - **Strategic Planning** — Documents that articulate an organization's long-term direction, goals, and the means to achieve them. Scope: business plans, strategic roadmaps, SWOT analyses, OKRs, market positioning documents.
  - **Internal Policies** — Documents that establish organization-wide rules and standards of conduct across non-HR domains. Scope: IT policies, security policies, data governance, ethics guidelines, code of conduct. Note: HR-specific policies live under HR.
  - **Logistics & Supply Chain** — Documents that track and manage the movement, storage, and delivery of goods across the supply chain. Scope: shipping documents, customs forms, bills of lading, warehouse records, inventory management, freight documentation.

- [x] 11. Real Estate & Property
  - **Titles & Deeds** — Documents that establish, transfer, or encumber legal ownership of real property. Scope: title documents, deed transfers, title insurance, easements, liens, encumbrance records.
  - **Leases & Rental** — Documents that govern the temporary right to occupy or use real property under agreed terms. Scope: lease agreements, rental applications, tenant records, lease amendments, eviction notices.
  - **Appraisals & Valuations** — Documents that assess the monetary value of real property through professional analysis. Scope: property appraisals, comparative market analyses, valuation reports, assessment appeals.
  - **Construction & Development** — Documents produced during the planning, permitting, and execution of building or land development projects. Scope: blueprints, building plans, inspection reports, contractor agreements, permits, environmental impact assessments.
  - **Property Management** — Documents related to the ongoing administration, maintenance, and tenant relations of managed properties. Scope: maintenance records, HOA documents, property tax records, utility records, tenant communications.

- [x] 12. Creative & Literary Works
  - **Scripts & Screenplays** — Documents that provide the written blueprint for performed or produced visual, audio, or stage media. Scope: film scripts, TV scripts, stage plays, radio scripts, script treatments.
  - **Prose & Fiction** — Documents that present narrative or storytelling in written long-form or short-form prose. Scope: novels, short stories, novellas, anthologies, serialized fiction.
  - **Poetry & Verse** — Documents that express ideas, emotions, or narratives through structured or free-form verse. Scope: poems, poetry collections, spoken word scripts, lyrical compositions.
  - **Creative Briefs & Concepts** — Documents that define the vision, direction, and parameters for a creative project before production begins. Scope: creative briefs, mood boards, storyboards, treatment documents, pitch decks.
  - **Journalism & Editorial** — Documents produced through investigative, analytical, or opinion-based writing for publication. Scope: articles, op-eds, feature pieces, editorial calendars, investigative reports.

### Boundary Decisions Log
- **Contracts** anchor in Legal, even when domain-specific (employment contracts, leases). Domain trees reference back, not duplicate.
- **Corporate Legal vs Corporate Governance** — Legal holds juridical documents (articles, shareholder agreements). Operations holds governance process documents (board minutes, strategic plans).
- **Regulatory & Compliance** under Legal covers general compliance. Domain-specific compliance (healthcare, insurance, financial) stays in its domain tree.
- **Invoicing & Payments** anchors in Finance. Sales holds commercial process docs (proposals, quotes, orders).
- **Financial audits** live under Finance > Audit. Compliance audits live under Legal > Regulatory & Compliance.
- **Internal Policies** under Operations are org-wide (IT, security, ethics). HR-specific policies (leave, benefits) live under HR.
- **Journalism & Editorial** under Creative — the editorial/narrative craft is the classifying factor, not the distribution medium.

---

### Level 3 Expansion (Children → Grandchildren)

Each Level 2 node is marked as **LEAF** (ready for schema) or **BRANCH** (expanded below with Level 3 children). All Level 3 children are leaves unless noted otherwise.

#### 1. Legal & Compliance

**1.1 Contracts & Agreements** → BRANCH
  - **Service & Commercial Agreements** [LEAF] — Documents that define the terms of service delivery or commercial exchange between business parties. Scope: MSAs, SOWs, SLAs, distribution agreements.
  - **Confidentiality Agreements** [LEAF] — Documents that bind parties to protect shared confidential information from disclosure. Scope: NDAs (mutual/unilateral), confidentiality clauses, non-circumvention agreements.
  - **Employment & Contractor Agreements** [LEAF] — Documents that formalize the working relationship between an organization and an individual contributor. Scope: employment contracts, independent contractor agreements, non-compete/non-solicitation clauses.
  - **Licensing & Royalty Agreements** [LEAF] — Documents that grant rights to use intellectual property, technology, or brand assets under defined terms. Scope: software licenses, franchise agreements, royalty agreements, usage rights.
  - **Partnership & Joint Venture Agreements** [LEAF] — Documents that establish shared ownership, responsibilities, and profit-sharing between collaborating entities. Scope: partnership deeds, JV agreements, consortium agreements, co-development agreements.
  - **Letters of Intent & Preliminary Agreements** [LEAF] — Documents that express the intent to enter into a future binding agreement, outlining key terms before formalization. Scope: MOUs, LOIs, term sheets, heads of agreement.

**1.2 Court & Litigation** → BRANCH
  - **Pleadings & Motions** [LEAF] — Documents filed by parties in legal proceedings to assert claims, defenses, or procedural requests. Scope: complaints, answers, counterclaims, motions, briefs, petitions.
  - **Court Orders & Judgments** [LEAF] — Documents issued by a court that direct action, resolve disputes, or render final decisions. Scope: orders, judgments, decrees, injunctions, writs.
  - **Discovery Documents** [LEAF] — Documents produced or exchanged during the pre-trial discovery phase to establish facts and evidence. Scope: interrogatories, depositions, document production requests, subpoenas.
  - **Settlement & Resolution** [LEAF] — Documents that record the agreed resolution of a legal dispute without or after trial. Scope: settlement agreements, releases, consent decrees, mediation/arbitration records.
  - **Evidence & Supporting Documents** [LEAF] — Documents submitted to support factual claims or testimony in legal proceedings. Scope: affidavits, declarations, exhibits, expert reports.

**1.3 Regulatory & Compliance** → BRANCH
  - **Compliance Reports & Assessments** [LEAF] — Documents that evaluate an organization's adherence to regulatory requirements and internal standards. Scope: compliance assessments, gap analyses, remediation reports, self-assessments.
  - **Regulatory Filings & Registrations** [LEAF] — Documents submitted to regulatory bodies to fulfill registration, reporting, or notification obligations. Scope: registration filings, periodic regulatory reports, notification filings.
  - **Compliance Audits** [LEAF] — Documents produced during formal examinations of an organization's compliance posture by internal or external auditors. Scope: compliance audit reports, audit trails, corrective action plans, monitoring reports.
  - **Sanctions & Enforcement Actions** [LEAF] — Documents related to punitive or corrective measures imposed by regulatory authorities for non-compliance. Scope: sanction notices, enforcement actions, penalty assessments, violation records, consent orders.

**1.4 Intellectual Property** → BRANCH
  - **Patents** [LEAF] — Documents related to the application, granting, and maintenance of exclusive rights to inventions. Scope: patent applications, granted patents, patent searches, office actions, continuations.
  - **Trademarks** [LEAF] — Documents related to the registration, protection, and enforcement of distinctive brand identifiers. Scope: trademark applications, registrations, opposition filings, renewals, use declarations.
  - **Copyrights** [LEAF] — Documents related to the registration, licensing, and enforcement of rights over original creative works. Scope: copyright registrations, licensing agreements, infringement notices, DMCA takedowns.
  - **Trade Secrets & Proprietary Information** [LEAF] — Documents that identify, protect, and track access to confidential business information with competitive value. Scope: trade secret inventories, protection protocols, access logs.

**1.5 Corporate Legal** → BRANCH
  - **Formation & Entity Documents** [LEAF] — Documents that legally establish a corporate entity and define its foundational structure. Scope: articles of incorporation/organization, certificates of formation, operating agreements, certificates of good standing.
  - **Bylaws & Governing Documents** [LEAF] — Documents that set out the internal rules governing the management and operation of a corporate entity. Scope: corporate bylaws, amendments, restated bylaws.
  - **Shareholder & Equity Documents** [LEAF] — Documents that record ownership interests, equity transactions, and shareholder rights within a corporate entity. Scope: shareholder agreements, stock certificates, proxy statements, cap tables, equity transfer records.
  - **Corporate Resolutions & Authorizations** [LEAF] — Documents that formally record decisions made by corporate governing bodies and delegate authority. Scope: board resolutions, written consents, corporate authorizations, powers of attorney.

**1.6 Notarial & Certified Documents** → **LEAF** — Documents sharing the common structure of certifying authority, authentication method, date, and certified content.

---

#### 2. Finance, Accounting & Tax

**2.1 Banking & Investments** → BRANCH
  - **Bank Accounts & Statements** [LEAF] — Documents that record banking relationships and the movement of funds through bank accounts. Scope: bank statements, account opening documents, bank confirmations, wire transfer records.
  - **Loans & Credit Facilities** [LEAF] — Documents that establish and manage the terms of borrowed capital between lenders and borrowers. Scope: loan agreements, credit facility documents, promissory notes, amortization schedules.
  - **Investment Records** [LEAF] — Documents that track the acquisition, holding, and disposition of investment assets. Scope: portfolio statements, trade confirmations, custody agreements, investment policy statements.
  - **Fund Documentation** [LEAF] — Documents that describe, market, or govern collective investment vehicles. Scope: prospectuses, offering memoranda, fund fact sheets, subscription agreements.

**2.2 Financial Reporting** → BRANCH
  - **Financial Statements** [LEAF] — Core financial documents that present an organization's financial position, performance, and cash flows at a point in time. Scope: balance sheets, income statements, cash flow statements, statements of equity, notes to financials.
  - **Periodic & Annual Reports** [LEAF] — Documents that provide comprehensive financial and operational summaries to stakeholders on a recurring schedule. Scope: annual reports, quarterly reports, management discussion & analysis, earnings releases.
  - **Consolidated & Segment Reporting** [LEAF] — Documents that aggregate financial results across subsidiaries, divisions, or business segments. Scope: consolidated financials, intercompany reconciliations, segment reports, elimination entries.

**2.3 Bookkeeping & Ledgers** → **LEAF** — Transactional accounting records sharing a common structure of dates, accounts, debits/credits, and descriptions.

**2.4 Invoicing & Payments** → BRANCH
  - **Invoices** [LEAF] — Documents that request payment from a buyer for goods or services delivered or to be delivered. Scope: sales invoices, proforma invoices, recurring invoices, self-billing invoices.
  - **Receipts & Payment Confirmations** [LEAF] — Documents that acknowledge the receipt of payment or transfer of funds. Scope: payment receipts, remittance advices, wire confirmations.
  - **Credit & Debit Notes** [LEAF] — Documents that adjust the amount owed between parties due to returns, errors, or agreed changes. Scope: credit notes, debit notes, adjustments, refund authorizations.
  - **Billing Statements & Notices** [LEAF] — Documents that summarize outstanding balances and payment obligations over a period. Scope: account statements, billing summaries, past-due notices, collection letters.
  - **Utility Bills & Statements** [LEAF] — Documents issued by utility or telecommunications providers that request payment for metered services, with fields distinct from standard commercial invoices (meter readings, consumption units, tariff tiers). Scope: electricity bills, water bills, gas bills, telecom bills, internet bills, waste management bills.
  - **Expense Reports & Reimbursements** [LEAF] — Documents that itemize business expenditures incurred by an individual and request organizational reimbursement, typically aggregating multiple receipts into a single accountability record. Scope: employee expense reports, mileage claims, per diem claims, petty cash vouchers, corporate card reconciliations.

**2.5 Tax Filings & Records** → BRANCH
  - **Tax Returns & Declarations** [LEAF] — Documents submitted to tax authorities declaring income, liabilities, and tax owed for a given period. Scope: corporate income tax returns, VAT/GST returns, payroll tax filings, excise tax returns.
  - **Tax Assessments & Notices** [LEAF] — Documents issued by tax authorities communicating assessed obligations, discrepancies, or required actions. Scope: assessment notices, deficiency notices, audit letters, appeals.
  - **Withholding & Information Reporting** [LEAF] — Documents that report or certify tax withholding obligations and third-party payments to tax authorities. Scope: withholding certificates, W-2s, 1099s, tax information reporting forms.
  - **Transfer Pricing Documentation** [LEAF] — Documents that justify the pricing of transactions between related entities across jurisdictions. Scope: transfer pricing studies, benchmarking analyses, advance pricing agreements, country-by-country reports.

**2.6 Audit** → BRANCH
  - **Audit Reports** [LEAF] — Documents that present the findings and opinion of an independent examination of financial records. Scope: internal audit reports, external audit opinions, special examination reports, qualified/unqualified opinions.
  - **Audit Planning & Engagement** [LEAF] — Documents that define the scope, objectives, and terms of an audit engagement before fieldwork begins. Scope: audit plans, risk assessments, engagement letters, materiality assessments.
  - **Management Letters & Remediation** [LEAF] — Documents that communicate audit findings to management and track the resolution of identified issues. Scope: management letters, management responses, remediation tracking, follow-up reports.

---

#### 3. Insurance

**3.1 Policies & Coverage** → BRANCH
  - **Policy Documents** [LEAF] — The core documents that establish the insurance contract, defining coverage terms, conditions, and exclusions. Scope: master policies, policy schedules, renewal notices, cancellation notices.
  - **Endorsements & Riders** [LEAF] — Documents that modify, add to, or restrict the terms of an existing insurance policy. Scope: policy endorsements, riders, amendments, exclusion modifications.
  - **Certificates & Summaries** [LEAF] — Documents that provide proof or a simplified overview of insurance coverage to third parties. Scope: certificates of insurance, coverage summaries, declarations pages, evidence of coverage.

**3.2 Claims & Settlements** → BRANCH
  - **Claim Filings** [LEAF] — Documents that initiate an insurance claim by notifying the insurer of a loss or event. Scope: first notice of loss, claim forms, supporting documentation submissions.
  - **Adjuster & Investigation Reports** [LEAF] — Documents that record the findings of an investigation into the circumstances and extent of a claimed loss. Scope: adjuster reports, investigation findings, damage assessments, cause-of-loss determinations.
  - **Settlement & Resolution Documents** [LEAF] — Documents that record the outcome of a claim, whether paid, denied, or otherwise resolved. Scope: settlement offers, releases, subrogation demands, payment schedules, denial letters.

**3.3 Underwriting & Actuarial** → BRANCH
  - **Underwriting Documents** [LEAF] — Documents used to evaluate applications for insurance and decide whether and on what terms to accept risk. Scope: applications, risk assessments, underwriting worksheets, binding agreements, declination letters.
  - **Actuarial Reports & Studies** [LEAF] — Documents that apply statistical and financial methods to quantify insurance risk, price products, and estimate reserves. Scope: actuarial valuations, loss reserve analyses, rate adequacy studies, experience studies.
  - **Reinsurance Documents** [LEAF] — Documents that govern the transfer of risk from a primary insurer to a reinsurer. Scope: reinsurance treaties, cession statements, retrocession agreements, bordereaux reports.

**3.4 Insurance Regulatory** → **LEAF** — Documents required by insurance regulators, sharing the common purpose of demonstrating solvency, market conduct, and consumer protection compliance.

---

#### 4. Healthcare & Medical

**4.1 Patient Records** → BRANCH
  - **Medical History & Demographics** [LEAF] — Documents that capture a patient's baseline health information and personal demographics at intake. Scope: patient intake forms, medical history questionnaires, demographic records, insurance information.
  - **Clinical Notes** [LEAF] — Documents written by healthcare providers that record observations, assessments, and plans during patient encounters. Scope: consultation notes, progress notes, nursing notes, specialist referral notes.
  - **Discharge & Transfer Documents** [LEAF] — Documents that summarize a patient's care episode and facilitate continuity when leaving a facility or transferring. Scope: discharge summaries, transfer letters, care transition plans, follow-up instructions.
  - **Consent & Authorization** [LEAF] — Documents in which patients grant informed permission for treatment, procedures, or release of information. Scope: informed consent forms, treatment authorizations, release of information, advance directives.

**4.2 Clinical** → BRANCH
  - **Clinical Trial Protocols** [LEAF] — Documents that define the objectives, design, methodology, and organization of a clinical study. Scope: study protocols, protocol amendments, investigator brochures, site selection documents.
  - **Case Report Forms & Data** [LEAF] — Documents used to collect and verify individual subject data during clinical trials. Scope: CRFs, electronic data capture forms, data query forms, monitoring reports.
  - **Adverse Event Reports** [LEAF] — Documents that record and report harmful or unintended outcomes observed during clinical research. Scope: adverse event forms, serious adverse event reports, safety narratives, DSMB reports.
  - **Regulatory Submissions (Clinical)** [LEAF] — Documents submitted to regulatory authorities to obtain or maintain approval for clinical research activities. Scope: IND/CTA applications, clinical study reports, ethics committee submissions, informed consent (trials).

**4.3 Pharmaceutical** → BRANCH
  - **Drug Approval & Registration** [LEAF] — Documents submitted to regulatory agencies to obtain marketing authorization for pharmaceutical products. Scope: NDA/MAA submissions, drug registration dossiers, labeling approvals, post-market commitments.
  - **Prescriptions & Medication Records** [LEAF] — Documents that authorize and track the dispensing and administration of medications to patients. Scope: prescriptions, medication administration records, dispensing logs, e-prescriptions.
  - **Formulary & Drug Information** [LEAF] — Documents that catalog approved medications and provide standardized drug reference information. Scope: formulary listings, drug monographs, therapeutic guidelines, medication guides.
  - **Pharmacovigilance** [LEAF] — Documents that monitor, detect, and report adverse effects of pharmaceutical products after market release. Scope: adverse drug reaction reports, periodic safety update reports, risk management plans, signal detection reports.

**4.4 Lab & Diagnostics** → BRANCH
  - **Laboratory Results** [LEAF] — Documents that report the findings of chemical, biological, or hematological analyses performed on patient specimens. Scope: blood work results, urinalysis, microbiology reports, chemistry panels, reference ranges.
  - **Diagnostic Imaging** [LEAF] — Documents that report the findings and interpretations of medical imaging procedures. Scope: radiology reports, MRI/CT findings, ultrasound reports, imaging requisitions.
  - **Pathology Reports** [LEAF] — Documents that report the microscopic and macroscopic examination of tissue, cell, or fluid samples. Scope: histopathology reports, cytology reports, biopsy results, autopsy reports.
  - **Quality Control & Accreditation** [LEAF] — Documents that ensure laboratory processes meet established quality and regulatory standards. Scope: QC logs, proficiency testing results, lab accreditation records, calibration records.

**4.5 Healthcare Administration** → BRANCH
  - **Facility Licensing & Accreditation** [LEAF] — Documents that authorize healthcare facilities to operate and certify they meet quality standards. Scope: facility licenses, accreditation certificates, survey reports, corrective action plans.
  - **Provider Credentialing** [LEAF] — Documents that verify the qualifications, experience, and competency of healthcare professionals. Scope: credentialing applications, privilege delineation, peer review records, re-credentialing documents.
  - **Payer & Reimbursement Documents** [LEAF] — Documents that govern the financial relationships between healthcare providers and insurance payers. Scope: payer contracts, reimbursement schedules, claims submissions, explanation of benefits, denial appeals.

---

#### 5. Government & Public Administration

**5.1 Identity Documents** → BRANCH
  - **Passports & Travel Documents** [LEAF] — Government-issued documents that certify identity and citizenship for international travel. Scope: passports, travel documents, visa stamps, travel permits.
  - **National & Government-Issued IDs** [LEAF] — Documents issued by government authorities as primary proof of identity within a jurisdiction. Scope: national identity cards, social security cards, taxpayer identification, voter registration cards.
  - **Driver's Licenses & Vehicle Documents** [LEAF] — Government-issued documents that authorize individuals to operate vehicles and register vehicle ownership. Scope: driver's licenses, vehicle registrations, driving permits, vehicle inspection certificates.
  - **Residency & Immigration Documents** [LEAF] — Documents that authorize and record an individual's legal right to reside or work in a foreign jurisdiction. Scope: residency permits, work permits, immigration visas, naturalization certificates.

**5.2 Civil Records** → BRANCH
  - **Birth & Death Records** [LEAF] — Official government records that document the vital events of birth and death. Scope: birth certificates, death certificates, stillbirth records, fetal death reports.
  - **Marriage & Domestic Partnership** [LEAF] — Official records that document the legal formation or dissolution of marital or partnership unions. Scope: marriage certificates, marriage licenses, divorce decrees, domestic partnership registrations.
  - **Citizenship & Nationality** [LEAF] — Documents that establish, transfer, or renounce an individual's legal relationship with a sovereign state. Scope: citizenship certificates, nationality declarations, renunciation documents, dual citizenship records.
  - **Name & Status Changes** [LEAF] — Legal records that document court-ordered or administrative changes to an individual's personal identity or family status. Scope: legal name change orders, gender marker changes, adoption decrees, guardianship orders.

**5.3 Permits & Licenses** → BRANCH
  - **Business & Commercial Licenses** [LEAF] — Government authorizations permitting entities to conduct specific commercial activities within a jurisdiction. Scope: business licenses, commercial permits, operating licenses, liquor licenses.
  - **Building & Construction Permits** [LEAF] — Government approvals required before commencing construction, renovation, or demolition of structures. Scope: building permits, demolition permits, occupancy certificates, variance approvals.
  - **Environmental Permits** [LEAF] — Government authorizations that regulate activities with potential environmental impact. Scope: environmental impact permits, discharge permits, emissions permits, waste handling licenses.
  - **Professional Licenses** [LEAF] — Government or regulatory body authorizations permitting individuals to practice regulated professions. Scope: professional licenses, certifications, practice permits, license renewals, disciplinary records.

**5.4 Public Policy & Legislation** → BRANCH
  - **Laws & Statutes** [LEAF] — Formal rules enacted by legislative bodies that govern conduct within a jurisdiction. Scope: enacted laws, statutes, codes, amendments, codifications.
  - **Regulations & Administrative Rules** [LEAF] — Rules issued by executive agencies to implement and enforce legislative mandates. Scope: regulations, administrative rules, regulatory guidance, compliance directives.
  - **Executive & Administrative Orders** [LEAF] — Directives issued by heads of state or government executives exercising executive authority. Scope: executive orders, presidential/ministerial directives, proclamations, emergency declarations.
  - **Policy Papers & Proposals** [LEAF] — Documents that analyze issues and propose policy directions for consideration by decision-makers. Scope: white papers, green papers, policy briefs, legislative proposals, consultation documents.

**5.5 Government Correspondence** → BRANCH
  - **Official Communications** [LEAF] — Formal correspondence between government entities or from government to individuals/organizations in an official capacity. Scope: government letters, inter-agency memos, diplomatic notes, official circulars.
  - **FOI & Transparency Documents** [LEAF] — Documents produced under freedom-of-information or open-government obligations. Scope: FOI requests, FOI responses, transparency reports, disclosure logs, open data releases.
  - **Public Notices & Announcements** [LEAF] — Documents published by government entities to inform the public of decisions, opportunities, or requirements. Scope: public notices, government gazette entries, tender notices, public consultations, regulatory announcements.

---

#### 6. Education & Research

**6.1 Academic Records** → BRANCH
  - **Transcripts & Grade Records** [LEAF] — Official institutional documents that record a student's courses, grades, and academic standing. Scope: official transcripts, grade reports, GPA calculations, academic standings.
  - **Diplomas & Certificates** [LEAF] — Documents that formally attest to the completion of an academic program or achievement of a credential. Scope: diplomas, degree certificates, honor recognitions, professional certificates.
  - **Enrollment & Registration** [LEAF] — Documents that record a student's admission, course selection, and enrollment status at an institution. Scope: enrollment records, registration forms, course selections, transfer credits, withdrawal records.

**6.2 Curriculum & Instruction** → BRANCH
  - **Syllabi & Course Materials** [LEAF] — Documents that outline course content, objectives, schedule, and resources for a specific academic offering. Scope: syllabi, reading lists, lecture notes, course outlines, supplementary materials.
  - **Lesson Plans & Teaching Guides** [LEAF] — Documents that structure the delivery of instruction for individual lessons or units. Scope: lesson plans, teaching guides, unit plans, instructional strategies.
  - **Assessment & Evaluation Instruments** [LEAF] — Documents that define how student learning is measured and evaluated against defined outcomes. Scope: exam papers, rubrics, grading criteria, standardized test frameworks, learning outcome assessments.

**6.3 Scientific Research** → BRANCH
  - **Research Papers & Publications** [LEAF] — Documents that present original research findings for dissemination to the scholarly community. Scope: journal articles, conference papers, preprints, working papers, review articles.
  - **Research Data & Methodology** [LEAF] — Documents that describe experimental designs, data collection methods, and the resulting datasets. Scope: datasets, data dictionaries, methodology descriptions, experimental protocols, statistical analyses.
  - **Peer Review & Editorial** [LEAF] — Documents generated during the evaluation of submitted research by subject-matter experts and journal editors. Scope: peer review reports, editorial decisions, revision requests, acceptance/rejection letters.

**6.4 Grants & Funding** → BRANCH
  - **Grant Proposals & Applications** [LEAF] — Documents that request financial support for research or educational projects from funding bodies. Scope: grant proposals, funding applications, project narratives, letters of support.
  - **Award & Agreement Documents** [LEAF] — Documents that formalize the terms and conditions under which funding is granted. Scope: award letters, grant agreements, funding terms, cooperative agreements.
  - **Grant Reporting & Administration** [LEAF] — Documents that track progress, expenditures, and compliance throughout the funded project lifecycle. Scope: progress reports, final reports, financial reports, no-cost extension requests, budget modifications.

**6.5 Institutional Administration** → BRANCH
  - **Accreditation Documents** [LEAF] — Documents produced during the process of evaluating whether an institution meets established quality standards. Scope: self-study reports, accreditation applications, site visit reports, accreditation decisions.
  - **Faculty & Staff Records** [LEAF] — Documents that record the appointments, qualifications, and administrative status of academic personnel. Scope: faculty appointments, tenure records, sabbatical requests, workload assignments.
  - **Program & Institutional Reviews** [LEAF] — Documents that assess the effectiveness and quality of academic programs and institutional operations. Scope: program review reports, institutional effectiveness reports, strategic plan assessments, benchmark comparisons.

---

#### 7. Human Resources & Employment

**7.1 Recruitment & Talent Acquisition** → BRANCH
  - **Job Postings & Descriptions** [LEAF] — Documents that define and advertise open positions within an organization. Scope: job advertisements, role descriptions, qualification requirements, team structure documents.
  - **Applications & Candidate Documents** [LEAF] — Documents submitted by or collected about candidates during the application process. Scope: resumes/CVs, cover letters, applications, portfolios, candidate assessments.
  - **Interview & Evaluation Records** [LEAF] — Documents that capture the structured assessment of candidates during the selection process. Scope: interview scorecards, evaluation forms, reference checks, background check reports.
  - **Offer & Onboarding Documents** [LEAF] — Documents that formalize the hiring decision and guide a new employee's integration into the organization. Scope: offer letters, onboarding checklists, welcome packages, pre-employment agreements.

**7.2 Employee Records** → BRANCH
  - **Personnel Files** [LEAF] — Documents that constitute the core administrative record of an individual's employment. Scope: employee information forms, emergency contacts, personal data records, photo IDs.
  - **Employment Contracts & Amendments** [LEAF] — Documents that define or modify the formal terms of an individual's employment relationship. Scope: employment agreements, contract amendments, role change letters, secondment agreements.
  - **Separation Documents** [LEAF] — Documents produced when an employee's relationship with the organization ends, voluntarily or involuntarily. Scope: resignation letters, termination notices, exit interviews, separation agreements, clearance forms.

**7.3 Compensation & Benefits** → BRANCH
  - **Payroll Records** [LEAF] — Documents that record the calculation and disbursement of employee wages and deductions. Scope: payslips, payroll registers, deduction records, tax withholding forms.
  - **Benefits Enrollment & Administration** [LEAF] — Documents related to employees' selection and management of employer-provided benefit programs. Scope: benefits enrollment forms, plan summaries, beneficiary designations, COBRA notices.
  - **Equity & Incentive Plans** [LEAF] — Documents that define and administer long-term compensation tied to company performance or equity ownership. Scope: stock option grants, RSU agreements, bonus plans, profit-sharing documentation, vesting schedules.

**7.4 Performance Management** → BRANCH
  - **Performance Reviews & Appraisals** [LEAF] — Documents that formally evaluate an employee's work performance against established criteria over a review period. Scope: annual reviews, mid-year reviews, probation assessments, self-assessments.
  - **Goal Setting & Objectives** [LEAF] — Documents that define individual or team performance targets aligned with organizational strategy. Scope: individual goals, team objectives, development goals, competency frameworks.
  - **Performance Improvement** [LEAF] — Documents that address underperformance through structured corrective action and monitoring. Scope: PIPs, warning letters, counseling records, follow-up evaluations.

**7.5 Training & Development** → BRANCH
  - **Training Programs & Materials** [LEAF] — Documents that design, deliver, or support structured learning activities for employees. Scope: training curricula, course materials, e-learning content, workshop outlines.
  - **Certification & Compliance Training** [LEAF] — Documents that record the completion of mandatory or professional certifications and required training. Scope: certification records, mandatory training completions, license renewals, CPD logs.
  - **Development Plans** [LEAF] — Documents that chart an employee's professional growth trajectory and the actions needed to achieve it. Scope: individual development plans, mentorship agreements, succession planning documents, career path frameworks.

---

#### 8. Sales, Marketing & Procurement

**8.1 Sales** → BRANCH
  - **Proposals & Quotes** [LEAF] — Documents that present a prospective offer of goods or services with pricing and terms to a potential buyer. Scope: sales proposals, price quotes, RFP responses, bid submissions.
  - **Sales Orders & Contracts** [LEAF] — Documents that formalize a buyer's commitment to purchase and the seller's commitment to deliver. Scope: purchase orders (sell-side), order confirmations, sales contracts, order amendments.
  - **Sales Reporting & Analytics** [LEAF] — Documents that aggregate and analyze sales activity, pipeline health, and revenue performance. Scope: sales reports, pipeline reports, forecasts, win/loss analyses, CRM exports.

**8.2 Marketing & Communications** → BRANCH
  - **Campaign & Strategy Documents** [LEAF] — Documents that plan and coordinate marketing initiatives across channels and audiences. Scope: campaign plans, marketing strategies, content calendars, channel plans.
  - **Brand & Creative Assets** [LEAF] — Documents that define and govern the visual and verbal identity of an organization's brand. Scope: brand guidelines, style guides, logo usage docs, messaging frameworks.
  - **Market Research & Analysis** [LEAF] — Documents that gather and interpret data about markets, customers, and competitors to inform business decisions. Scope: market research reports, survey results, focus group findings, competitive analyses.
  - **External Communications** [LEAF] — Documents crafted for distribution to external audiences to inform, promote, or manage public perception. Scope: press releases, newsletters, public announcements, media kits, spokesperson briefings.

**8.3 Procurement & Vendor Management** → BRANCH
  - **Purchase Orders & Requisitions** [LEAF] — Documents that formally authorize and request the acquisition of goods or services from a supplier. Scope: purchase orders, purchase requisitions, blanket orders, order acknowledgments.
  - **RFPs & Sourcing** [LEAF] — Documents used to solicit, evaluate, and select suppliers through a structured procurement process. Scope: requests for proposal, requests for quotation, requests for information, bid evaluations, sourcing strategies.
  - **Vendor Records & Evaluations** [LEAF] — Documents that track supplier relationships and assess vendor performance against contractual expectations. Scope: vendor onboarding forms, supplier scorecards, performance evaluations, approved vendor lists.

**8.4 Customer Relations** → BRANCH
  - **Customer Correspondence** [LEAF] — Formal written communications between an organization and its customers regarding account matters. Scope: formal letters, account communications, relationship summaries, meeting notes.
  - **Complaints & Escalations** [LEAF] — Documents that record customer dissatisfaction and the process of investigating and resolving issues. Scope: complaint forms, escalation records, root cause analyses, resolution documentation.
  - **Surveys & Feedback** [LEAF] — Documents that systematically collect and analyze customer opinions to measure satisfaction and inform improvements. Scope: satisfaction surveys, NPS reports, feedback compilations, voice-of-customer reports.

---

#### 9. Technical & Product Documentation

**9.1 System & Architecture** → BRANCH
  - **System Design Documents** [LEAF] — Documents that describe the structural design of a system at varying levels of abstraction. Scope: high-level design docs, low-level design docs, system context diagrams, component specifications.
  - **Architecture Decision Records** [LEAF] — Documents that capture the rationale behind significant architectural choices and their trade-offs. Scope: ADRs, technology selection rationale, architectural patterns, trade-off analyses.
  - **Infrastructure Documentation** [LEAF] — Documents that describe the physical and virtual infrastructure supporting technical systems. Scope: infrastructure diagrams, network topology, capacity plans, disaster recovery plans, environment specifications.

**9.2 API & Developer Documentation** → BRANCH
  - **API References** [LEAF] — Documents that provide the complete technical specification of a software interface for developers to consume. Scope: API specifications (OpenAPI/Swagger), endpoint documentation, request/response schemas, authentication guides.
  - **SDK & Library Documentation** [LEAF] — Documents that guide developers in using software development kits and code libraries. Scope: SDK guides, library references, code samples, quickstart guides, migration guides.
  - **Integration Documentation** [LEAF] — Documents that describe how to connect systems, services, or platforms through defined interfaces. Scope: integration guides, webhook documentation, third-party connector docs, data mapping specifications.

**9.3 Engineering Specs** → BRANCH
  - **Requirements Documents** [LEAF] — Documents that define what a system or component must do and the constraints it must satisfy. Scope: functional requirements, non-functional requirements, user stories, acceptance criteria, specifications.
  - **Design Reviews & Proposals** [LEAF] — Documents that propose technical approaches and subject them to structured evaluation before implementation. Scope: design proposals, RFCs, technical design reviews, feasibility assessments, proof-of-concept results.
  - **Test Documentation** [LEAF] — Documents that plan, execute, and report on the verification and validation of technical systems. Scope: test plans, test cases, test reports, QA checklists, regression test suites, bug reports.

**9.4 User-Facing Documentation** → BRANCH
  - **User Manuals & Guides** [LEAF] — Comprehensive documents that instruct end users on how to operate and configure a product. Scope: user manuals, getting started guides, administration guides, configuration guides.
  - **Tutorials & How-To Articles** [LEAF] — Task-oriented documents that walk users through specific procedures or workflows step by step. Scope: step-by-step tutorials, how-to articles, walkthroughs, video script documentation.
  - **Reference & Knowledge Base** [LEAF] — Lookup-oriented documents that provide quick answers, definitions, and solutions to common issues. Scope: FAQs, knowledge base articles, glossaries, troubleshooting guides, release notes.

**9.5 Operational Runbooks** → **LEAF** — Procedural documents sharing a common structure of trigger/condition, step-by-step actions, escalation paths, and rollback procedures.

---

#### 10. Operations & Corporate Governance

**10.1 Standard Operating Procedures** → **LEAF** — Prescriptive procedural documents sharing a common structure of purpose, scope, step-by-step instructions, and responsibilities.

**10.2 Corporate Governance** → BRANCH
  - **Board Documents** [LEAF] — Documents that support the decision-making and oversight functions of a corporate board of directors. Scope: board resolutions, board meeting agendas, board packs/materials, director appointment records.
  - **Meeting Records** [LEAF] — Documents that capture the proceedings, decisions, and action items from formal organizational meetings. Scope: meeting minutes (board, committee, shareholder), attendance records, action item trackers.
  - **Shareholder & Investor Communications** [LEAF] — Documents directed at shareholders and investors to inform them of corporate activities and decisions. Scope: shareholder letters, annual meeting notices, proxy materials, dividend declarations.
  - **Committee & Charter Documents** [LEAF] — Documents that establish the mandate, composition, and operating rules of governance committees. Scope: committee charters, terms of reference, committee reports, governance frameworks.

**10.3 Strategic Planning** → BRANCH
  - **Business Plans** [LEAF] — Documents that articulate a comprehensive strategy for launching, growing, or transforming a business or initiative. Scope: business plans, business cases, feasibility studies, market entry strategies.
  - **Strategic Roadmaps & Objectives** [LEAF] — Documents that translate strategic vision into structured goals, milestones, and measurable outcomes. Scope: strategic roadmaps, OKRs, KPIs, balanced scorecards, goal-setting frameworks.
  - **Market & Competitive Analysis** [LEAF] — Documents that assess the external business environment to inform strategic positioning and decision-making. Scope: SWOT analyses, competitive landscapes, market sizing, industry benchmarks, positioning papers.

**10.4 Internal Policies** → **LEAF** — Organization-wide policy documents sharing a common structure of policy statement, scope, applicability, compliance requirements, and enforcement.

**10.5 Logistics & Supply Chain** → BRANCH
  - **Shipping & Freight Documents** [LEAF] — Documents that accompany and authorize the physical movement of goods from origin to destination. Scope: bills of lading, shipping manifests, freight invoices, delivery receipts, packing lists.
  - **Customs & Trade Compliance** [LEAF] — Documents required for the lawful import and export of goods across international borders. Scope: customs declarations, import/export licenses, certificates of origin, tariff classifications.
  - **Warehouse & Inventory Records** [LEAF] — Documents that track the storage, quantity, and movement of goods within warehousing and inventory systems. Scope: inventory reports, warehouse receipts, stock counts, movement logs, bin cards.

---

#### 11. Real Estate & Property

**11.1 Titles & Deeds** → BRANCH
  - **Deeds & Transfers** [LEAF] — Documents that legally convey ownership of real property from one party to another. Scope: warranty deeds, quitclaim deeds, grant deeds, deeds of trust, title transfers.
  - **Title Records & Insurance** [LEAF] — Documents that establish the history of ownership and protect against defects in title. Scope: title searches, title insurance policies, title commitments, chain of title reports.
  - **Easements & Encumbrances** [LEAF] — Documents that grant limited rights to use property or record claims and restrictions against it. Scope: easement agreements, right-of-way documents, liens, encumbrance records, covenant declarations.

**11.2 Leases & Rental** → BRANCH
  - **Lease Agreements** [LEAF] — Documents that establish the terms under which a tenant may occupy and use real property for a defined period. Scope: commercial leases, residential leases, ground leases, sublease agreements, lease renewals.
  - **Tenant Management Documents** [LEAF] — Documents used to screen, onboard, and manage tenants throughout a lease term. Scope: rental applications, tenant screening reports, move-in/move-out checklists, tenant correspondence.
  - **Lease Modifications & Terminations** [LEAF] — Documents that alter, renew, or end an existing lease relationship. Scope: lease amendments, rent adjustments, termination notices, eviction filings, surrender agreements.

**11.3 Appraisals & Valuations** → **LEAF** — Documents sharing the common structure of property identification, valuation methodology, comparable analysis, and value determination.

**11.4 Construction & Development** → BRANCH
  - **Plans & Blueprints** [LEAF] — Technical drawings that specify the design, dimensions, and spatial layout of a building or development project. Scope: architectural drawings, engineering blueprints, site plans, floor plans, elevation drawings.
  - **Permits & Approvals** [LEAF] — Government authorizations required before and during construction or land development activities. Scope: building permits, zoning approvals, environmental clearances, utility connection permits.
  - **Inspection & Compliance** [LEAF] — Documents that verify construction work meets applicable building codes, safety standards, and design specifications. Scope: inspection reports, code compliance certificates, occupancy permits, punch lists.
  - **Contractor & Project Documents** [LEAF] — Documents that manage the contractual and financial relationship between property owners and construction service providers. Scope: contractor agreements, change orders, progress reports, payment applications, lien waivers.

**11.5 Property Management** → BRANCH
  - **Maintenance & Repair Records** [LEAF] — Documents that track the upkeep, repair, and preventive maintenance of managed properties. Scope: work orders, maintenance logs, repair invoices, warranty claims, preventive maintenance schedules.
  - **Association & Community Documents** [LEAF] — Documents that govern shared property interests and community living arrangements. Scope: HOA bylaws, CC&Rs, association meeting minutes, assessment notices, community rules.
  - **Property Financial Records** [LEAF] — Documents that track the financial performance and obligations associated with property ownership and management. Scope: property tax records, utility records, operating budgets, capital expenditure plans, rent rolls.

---

#### 12. Creative & Literary Works

**12.1 Scripts & Screenplays** → **LEAF** — Performance-oriented written works sharing the common format of scene headings, action lines, dialogue, and transitions.

**12.2 Prose & Fiction** → **LEAF** — Narrative written works structured through chapters, scenes, and storytelling conventions.

**12.3 Poetry & Verse** → **LEAF** — Verse-based written works expressed through stanzas, meter, rhythm, and/or free-form structure.

**12.4 Creative Briefs & Concepts** → **LEAF** — Pre-production direction documents structured around objectives, audience, tone, deliverables, and constraints.

**12.5 Journalism & Editorial** → **LEAF** — Published or publishable written works of reporting, analysis, or opinion, structured around leads, evidence, and editorial voice.
