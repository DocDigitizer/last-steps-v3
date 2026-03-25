# Document Type Ontology - Root Nodes

## 12 Root Nodes

| # | Root Node | Level 2 (examples) |
|---|-----------|---------------------|
| 1 | **Legal & Compliance** | Contracts, Court & Litigation, Regulatory Compliance, Intellectual Property |
| 2 | **Finance, Accounting & Tax** | Banking & Investments, Financial Statements, Bookkeeping & Ledgers, Tax Filings |
| 3 | **Insurance** | Policies, Claims, Underwriting |
| 4 | **Healthcare & Medical** | Patient Records, Clinical, Pharmaceutical, Lab & Diagnostics |
| 5 | **Government & Public Administration** | Identity Documents, Permits & Licenses, Civil Records, Public Policy |
| 6 | **Education & Research** | Academic Records, Curricula, Scientific Research, Grant Proposals |
| 7 | **Human Resources & Employment** | Recruitment, Employee Records, Benefits & Compensation, Performance |
| 8 | **Sales, Marketing & Procurement** | Sales Proposals, Marketing Campaigns, Purchasing & Vendor Management, Communications |
| 9 | **Technical & Product Documentation** | System/API Docs, Engineering Specs, User Manuals, Release Notes |
| 10 | **Operations & Corporate Governance** | SOPs, Board & Corporate Records, Strategic Planning, Internal Policies, Logistics & Supply Chain |
| 11 | **Real Estate & Property** | Deeds & Titles, Leases, Appraisals, Construction |
| 12 | **Creative & Literary Works** | Scripts & Screenplays, Novels & Poetry, Creative Briefs |

## Design Decisions

- **Fewer roots, richer subtrees**: Different companies draw domain boundaries differently. Fewer roots avoid imposing opinions on org structure. Clients can ignore subtree depth they don't need but can't easily merge roots that are split.
- **Finance + Accounting + Tax merged**: Segregation lives at level 2, where each client can emphasize what matters to them.
- **Education + Research merged**: Research outside academia (corporate R&D, think tanks) fits naturally as a subtree.
- **Technical + Product Documentation merged**: Internal-facing vs external-facing becomes a level 2 split.
- **Sales + Marketing + Procurement + Communications merged**: Covers the full commercial cycle. Communications absorbed here since its most concrete use cases are commercial in nature.
- **Insurance stays standalone**: Document types (policies, claims, endorsements) and regulatory framework are distinct enough from Finance that merging would create a confusing level 2.
- **Logistics & Supply Chain under Operations**: Often tied with operations; disambiguated as a level 2 node rather than a separate root.
- **Creative & Literary Works kept as root**: Rare for most B2B clients, but having it as a root means those rare documents get classified immediately without ambiguity.
- **No "Other" bucket**: If something doesn't fit, the taxonomy needs a new root, not a catch-all.

## Architectural Decisions

### Tree Versioning
The ontology tree must be versioned. As the system processes real documents and discovers gaps, the tree will evolve — new leaves, new branches, splits of existing nodes. Each version of the tree is a stable snapshot that existing schemas and classifications reference. This is critical for:
- Backward compatibility (old classifications remain valid against the tree version they were made with)
- Auditing (understanding why a document was classified a certain way at a point in time)
- Controlled evolution (tree changes are deliberate versioning events, not ad-hoc mutations)

### Node Types and Schemas
- **Every node in the tree can hold a schema fragment** (a set of JSON fields), not just leaves.
- **Leaf nodes** are the classification targets. Each leaf has exactly ONE public schema and may have multiple private schemas (per organization).
- **If a leaf needs multiple public schemas, the leaf is wrong** — it should be split into child nodes. This invariant keeps the ontology and schema system in sync.

### Schema Inheritance (ALL-Children Rule)
A field belongs at a node if and only if ALL of its child nodes use that field. If only some children use it, the field goes on those specific children, not the parent. This ensures:
- Parent schemas contain only universally true fields for all descendants
- No field pollution — a field at a node is a guarantee, not a suggestion
- **Design smell signal**: if a field applies to most-but-not-all children, the outlier child may be misplaced in the tree

### Classification and Fallback
- The classifier always attempts to reach a leaf node.
- If confidence is low, the classifier falls back to the nearest ancestor node whose schema it is confident about.
- Because of the ALL-children inheritance rule, the ancestor schema contains only fields that are correct regardless of which leaf the document actually belongs to. This means: less data extracted, but never wrong data.
- **On-the-fly schema generation is NOT a runtime feature.** If a document cannot be classified anywhere in the tree, that is a signal the tree needs to grow — a versioning event, not an extraction workaround.

### Confidence Scoring Thresholds
Every classification step (at every tree level) must output a confidence score. The system acts on confidence as follows:

| Confidence | Action |
|------------|--------|
| >= 0.90 | Proceed to next level. No review needed. |
| 0.70 - 0.89 | Proceed to next level, but flag the document for post-extraction validation. |
| 0.50 - 0.69 | Fall back to ancestor schema for extraction. Flag for human review. Optionally explore top-2 paths in parallel if cost allows. |
| < 0.50 | Fall back to ancestor schema for extraction. Route to pending review queue. This is a signal the tree may need expansion. |

When falling back to an ancestor, the system extracts using the ancestor's inherited schema (guaranteed correct fields) and logs the event as a tree quality signal.

### Ambiguity Resolution Rules
When a document could plausibly belong to more than one category, apply these rules in priority order:

1. **Explicit document title/header** — If the document self-identifies (e.g., "INVOICE", "MEDICAL RECORD"), trust it.
2. **Issuing entity** — Classify by who created it. A bank document goes to Banking, a hospital document goes to Healthcare.
3. **Primary transactional purpose** — What is the document's main function in a business process? Classify by that function.
4. **Specialized domain over generic form** — A mortgage agreement is Real Estate, not Legal, even though it is technically a contract. A utility bill is an Invoice under Finance, not a separate domain.
5. **Original purpose over current use** — Classify by what the document IS, not how it is being used. A utility bill used for KYC address verification is still an invoice.

**Common ambiguity resolutions:**

| Document | Candidate A | Candidate B | Rule Applied | Assigned To |
|----------|-------------|-------------|--------------|-------------|
| Employment contract | Legal > Contracts | HR > Employee Records | Domain over form — it is a binding contract | Legal > Contracts > Employment & Contractor Agreements |
| Utility bill | Finance > Invoicing | (no competing root) | Original purpose — it is an invoice | Finance > Invoicing & Payments > Invoices |
| Medical insurance claim | Healthcare | Insurance | Issuer rule — depends on who issued it | Healthcare if from provider, Insurance if from insurer |
| Tax invoice | Finance | (Tax is under Finance in our tree) | N/A — no ambiguity in our tree structure | Finance > Invoicing & Payments > Invoices |
| Bank loan agreement | Finance > Banking | Legal > Contracts | Domain over form — it is a banking product | Finance > Banking & Investments > Loans & Credit Facilities |
| Property lease | Real Estate > Leases | Legal > Contracts | Domain over form — property-specific | Real Estate > Leases & Rental > Lease Agreements |
| Purchase order | Sales & Procurement | Finance | Process stage — pre-payment ordering goes to Procurement | Sales, Marketing & Procurement > Procurement > Purchase Orders & Requisitions |
| Expense report | Finance > Invoicing | HR > Compensation | Primary purpose — financial accountability document | Finance > Invoicing & Payments > Billing Statements & Notices |

### Schema Lifecycle
Schemas progress through four maturity stages:

```
draft → community → staging → production → (deprecated)
```

| Stage | Description | Visibility |
|-------|-------------|------------|
| **draft** | Schema is being authored. Not used for extraction. | Owner only |
| **community** | Schema is available for use but not validated at scale. Entered when a client uploads a schema or when the system identifies a new document pattern. | Logged-in users |
| **staging** | Schema has been used for extraction with sufficient success. Under review for promotion. Requires: >= 50 successful extractions, >= 0.85 average extraction confidence. | Logged-in users |
| **production** | Schema is validated and reliable. This is the public schema at a leaf node. Only one production schema per leaf. | Everyone |
| **deprecated** | Schema is no longer recommended. Remains addressable for historical reference but hidden from selection. | Historical only |

Promotion from community to staging is automatic when thresholds are met. Promotion from staging to production requires human review. Demotion to deprecated is manual.

### Client Onboarding Flow
1. Client uploads a schema (their extraction template for a document type they process).
2. The system classifies the schema — places it at the correct leaf node in the tree.
3. The schema becomes a private schema at that leaf, owned by the client's organization, starting at **draft** maturity.
4. If no matching leaf exists, this is a candidate for tree expansion (new version).

### Country-Specific Fields (RESOLVED)
Country-specific fields are handled through **L3+ regional variants** — leaf nodes or sub-leaves scoped to a country or regulatory regime.

How it works:
- L1 and L2 are geography-neutral. They classify by domain and purpose.
- L3 leaves can be either generic (country-agnostic) or country-specific.
- A generic leaf (e.g., "Invoices") has a public schema with universally common fields (invoice_number, date, total_amount, currency).
- A country-specific leaf (e.g., "Portuguese Invoices" under Invoices, if warranted) adds locale-specific fields (nif, AT certification code).
- If a country variant doesn't have enough schema distinctness to warrant its own leaf, the country-specific fields are handled by **private schemas** at the generic leaf, owned by orgs operating in that country.

Decision criteria for when to create a country-specific leaf vs keep it in private schemas:
- **Create a leaf** when the country's regulatory framework mandates structurally different fields that many orgs in that country will need (e.g., Portuguese SAF-T invoices, Brazilian NF-e).
- **Keep in private schemas** when the country differences are minor field additions on top of the generic schema (e.g., adding a single tax ID field).
