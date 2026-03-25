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

### Client Onboarding Flow
1. Client uploads a schema (their extraction template for a document type they process).
2. The system classifies the schema — places it at the correct leaf node in the tree.
3. The schema becomes a private schema at that leaf, owned by the client's organization.
4. If no matching leaf exists, this is a candidate for tree expansion (new version).

### Open Decision: Country-Specific Fields
**Status: UNRESOLVED.** With doc type + country scoping deprecated, locale-specific fields (e.g., `nif` for Portugal, `VAT_registration_number` for UK) need a home. Leading candidate: public schemas are country-agnostic, private schemas add locale-specific fields. To be confirmed.
