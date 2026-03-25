# Comparison Report: Reports (R1-R4) vs Our Ontology Work

**Date:** 2026-03-25

---

## 1. Executive Summary

The Reports (R1-R4) and our ontology work (ontology-plan.md + ontology-state.md) approach the same problem — hierarchical document classification for an IDP pipeline — but from different angles and with different priorities. The Reports are broader in scope (covering data models, APIs, LLM selection, cost analysis, and speed optimization) while our work is deeper on taxonomy design and architectural invariants (schema inheritance, fallback behavior, tree versioning).

**The two bodies of work are complementary, not contradictory.** Most differences are reconcilable. The few genuine conflicts require explicit decisions.

---

## 2. Taxonomy Structure Comparison

### 2.1 Root Level

| Aspect | Reports (R3) | Our Work |
|--------|-------------|----------|
| **L1 count** | 20 categories | 12 roots |
| **Design philosophy** | Volume-weighted, granular — high-volume doc types get dedicated roots early | Fewer roots, richer subtrees — avoid imposing org structure opinions |
| **Unknown/Other** | CAT-20 (Uncategorized/Unknown) with 5 subcategories | Explicitly rejected — unclassifiable docs are a tree versioning signal |

### 2.2 Where the Reports have more roots than us

| Reports Root | Our Treatment | Analysis |
|-------------|---------------|----------|
| **Financial & Accounting** (CAT-01) + **Banking & Financial Services** (CAT-05) | Merged into **Finance, Accounting & Tax** | Reports split by issuer (corporate accounting vs banking institutions). Our merge is simpler but may lose a useful L1 signal. |
| **Tax & Regulatory Compliance** (CAT-04) | Merged into **Finance, Accounting & Tax** | Reports separate tax/compliance as its own domain. We push it to L2. |
| **Supply Chain & Logistics** (CAT-08) | Under **Operations & Corporate Governance** as L2 | Reports give it a dedicated root due to high volume in manufacturing/logistics clients. |
| **Procurement & Purchasing** (CAT-09) | Under **Sales, Marketing & Procurement** as L2 | Reports separate buy-side from sell-side. We merged the commercial cycle. |
| **Corporate Governance** (CAT-13) | Merged into **Operations & Corporate Governance** | Reports keep governance separate from operations. We merged them. |
| **Utilities & Telecommunications** (CAT-17) | Not present as root or L2 | Reports have this as a dedicated root. We have no explicit home for utility bills, telecom statements. **This is a gap in our tree.** |
| **Travel & Expense** (CAT-18) | Not present as root or L2 | Reports have this as a dedicated root. We have no explicit home for boarding passes, hotel folios, expense reports. **This is a gap in our tree.** |
| **Correspondence & General** (CAT-19) | Absorbed into other categories (Marketing & Communications, Operations) | Reports keep it as a catch-all for generic business correspondence. We distributed it. |
| **Uncategorized / Unknown** (CAT-20) | Rejected by design | Reports use it as the chaos-mode entry point. We use ancestor fallback + tree versioning signals instead. |

### 2.3 Where our work has more structure

Our work goes deeper — we have full L3 expansion (~130 leaf nodes with descriptions and scopes) while Reports only cover L1-L2 (20 L1, 162 L2) and estimate L3 at 635-930 nodes.

### 2.4 L2 Comparison (Shared Domains)

| Domain | Reports L2 Count | Our L2 Count | Key Differences |
|--------|-----------------|--------------|-----------------|
| Financial/Accounting | 10 (CAT-01) + 9 (CAT-05) = 19 | 6 | Reports are much more granular (separate Budgets & Forecasts, AP/AR, Payroll, etc.) |
| Legal | 12 | 6 | Reports split by contract type at L2 (Service, Sales, Lease, Employment). We group all contracts together and split at L3. |
| HR | 10 | 5 | Reports add Leave & Attendance, Disciplinary & Grievance, Employee Verification as distinct L2 nodes |
| Insurance | 8 | 4 | Reports add Premium & Billing, Certificates of Insurance as distinct L2 nodes |
| Healthcare | 8 | 5 | Reports add Medical Billing & Coding, Referrals & Authorizations, Medical Certificates |
| Government | 7 | 5 | Reports add Government Benefits, Government Contracts & Grants |
| Education | 6 | 5 | Reports add Financial Aid & Scholarships |
| Real Estate | 6 | 5 | Reports add Mortgage & Financing, Zoning & Land Use |

---

## 3. Architectural Differences

### 3.1 Things our work defines that Reports don't

| Concept | Our Work | Reports' Position |
|---------|----------|-------------------|
| **Schema Inheritance (ALL-children rule)** | Field at a node iff ALL children use it. Enables graceful fallback. | Not mentioned. Reports treat schemas as flat per-document-type, no inheritance concept. |
| **One public schema per leaf invariant** | If a leaf needs two public schemas, split the leaf. | Not stated. Reports allow schemas to have multiple category tags (SchemaTag entity supports it). |
| **Ancestor fallback** | Low-confidence classification falls back to ancestor node's inherited schema — less data, never wrong data. | Reports use confidence thresholds + human-in-the-loop escalation. No concept of extracting with a parent schema. |
| **On-the-fly generation rejected** | Unclassifiable docs signal tree evolution, not runtime schema generation. | Reports describe a Schema Suggestion path where the LLM proposes a new schema from a single document. |
| **Client onboarding flow** | Upload schema → system classifies to leaf → becomes private schema. | Reports describe Schema Groups and Org association but not this specific inbound classification flow. |
| **Design smell detection** | If a field applies to most-but-not-all children, the outlier may be misplaced. | Not mentioned. |

### 3.2 Things Reports define that our work doesn't

| Concept | Reports | Our Work's Position |
|---------|---------|---------------------|
| **Hybrid classification pipeline** (R1 7.1) | Embedding pre-filter + optional fine-tuned classifier + LLM CE as final tier. Critical recommendation. | Not addressed. Our work is purely taxonomy/schema architecture. |
| **Confidence scoring at every CE level** (R1 7.2) | Detailed thresholds: >0.90 proceed, 0.70-0.90 flag, 0.50-0.70 explore two paths, <0.50 escalate. | Acknowledged (fallback on low confidence) but no specific thresholds defined. |
| **Schema lifecycle stages** (R2) | community → staging → production → deprecated | We have draft → active → deprecated (from the existing system). No staging concept. |
| **Full data model** (R2) | 15 entities with complete field definitions, indexes, sample JSON | Not addressed. |
| **API design** (R2) | Full REST API specification for CRUD, CE, extraction | Not addressed. |
| **LLM model selection** (R4) | Per-step model recommendations, cascading strategies, cost analysis | Not addressed. |
| **Ambiguity resolution rules** (R3 Section 5) | Explicit rules: domain-over-form, issuer rule, original-purpose rule, etc. | We have Boundary Decisions Log but less formalized. |
| **Volume estimation** (R3) | Estimated % volume per L1 category based on IDP experience | Not addressed. |
| **Multi-label classification** (R1 7.4) | Documents can match 2-3 categories at L1 with priority rules | Our tree assumes single-path classification. |
| **Layout-aware classification** (R1 7.5) | Use multimodal LLM or LayoutLM for classification | Not addressed. |
| **Caching strategies** (R1 7.6, R4) | Document fingerprinting, org-level classification cache, prompt caching | Not addressed. |
| **Schema similarity detection** (R2) | SchemaSimilarityIndex entity with embeddings for deduplication | Not addressed. |
| **CAT-20 / Unknown handling** | Dedicated category with subcategories for composites, damaged, novel docs | Rejected. Unclassifiable = tree needs to grow. |

---

## 4. Pros and Cons

### 4.1 Our Ontology Approach — Pros

1. **Simpler top level.** 12 roots vs 20 means faster L1 classification, fewer options for the LLM, and easier client onboarding. The principle of P2 in Report 3 (balanced fan-out of 5-25) is respected.

2. **Schema inheritance is architecturally superior.** The ALL-children rule + ancestor fallback gives graceful degradation that Reports' approach doesn't have. This is our most significant architectural contribution.

3. **One public schema per leaf is a clean invariant.** It keeps tree structure and schema structure perfectly aligned. Reports' approach (schemas tagged to multiple category paths) creates ambiguity about where a schema "lives."

4. **No "Other" bucket forces better taxonomy.** CAT-20 in the Reports is pragmatic but creates a permanent escape hatch that reduces pressure to improve the tree.

5. **Deeper L3 expansion.** We have ~130 described leaf nodes. Reports only estimate L3. Our tree is more actionable today.

6. **Architectural decisions are explicitly recorded** with reasoning and invariants. Reports are more descriptive; our work is more prescriptive.

### 4.2 Our Ontology Approach — Cons

1. **Missing document types.** No home for utility bills, telecom statements, travel documents, expense reports, boarding passes. These are common in IDP pipelines. Report 3 estimates Utilities at 1.5% and Travel at 1% of volume — small but real.

2. **Over-consolidation risk.** Merging Financial + Banking + Tax into one root creates a very broad L1 category. Report 3 estimates Financial (22%) + Banking (7%) + Tax (8%) = 37% of all documents hitting one root. That's unbalanced.

3. **No volume-weighted design.** Our expansion order was driven by cross-cutting ambiguity (a good ontology principle) but ignores practical IDP volume. Invoices alone may represent 15-20% of all documents — our tree doesn't prioritize this.

4. **No confidence scoring specification.** We acknowledge fallback on low confidence but haven't defined thresholds. Reports have concrete thresholds that are operationally necessary.

5. **No hybrid classification strategy.** Reports' recommendation to use embeddings + fine-tuned classifiers for early levels is compelling and we haven't addressed it.

6. **No ambiguity resolution rules.** Reports have explicit rules (domain-over-form, issuer rule, etc.) that solve real classification conflicts. Our Boundary Decisions Log is less systematic.

7. **Country-specific fields still unresolved.** Reports handle this through L3+ regional variants (e.g., CAT-01.01 Invoices → L3: Portuguese Invoice, UK Invoice). This is cleaner than our open question.

### 4.3 Reports Approach — Pros

1. **Comprehensive scope.** Covers taxonomy, data model, API, LLM selection, cost, speed — everything needed to build the system.

2. **Volume-weighted pragmatism.** Categories are ordered and sized by real IDP volume, not theoretical completeness.

3. **Explicit ambiguity resolution.** Five priority rules for handling cross-category documents.

4. **Hybrid classification pipeline.** The embedding + classifier + LLM cascade is a strong engineering recommendation.

5. **Detailed cost/speed analysis.** Report 4 gives concrete per-document cost and latency estimates across model tiers.

6. **CAT-20 as operational safety net.** Pragmatic for v1 — catches what the tree misses without blocking the pipeline.

7. **Regional/industry variation analysis.** Report 3 Section 6 maps volume shifts by region and industry, which is operationally valuable.

8. **Schema lifecycle with staging.** community → staging → production is more careful than a direct promotion.

### 4.4 Reports Approach — Cons

1. **No schema inheritance.** Every leaf schema is self-contained. Shared fields (dates, parties, amounts) are duplicated across hundreds of schemas.

2. **No ancestor fallback.** Low-confidence classification goes to human review, not to a partial extraction. This means the pipeline stalls on uncertain documents.

3. **20 L1 categories may be too many.** Report 3's own design rule (P2: balanced fan-out of 5-25) is respected but at the upper bound. 20 options makes LLM classification harder than 12.

4. **CAT-20 is an escape hatch.** It reduces pressure to improve the taxonomy. Report 3 acknowledges this risk but keeps it anyway.

5. **Schema-to-category mapping is many-to-many.** A schema can be tagged to multiple category paths (SchemaTag). This creates ambiguity about canonical ownership and complicates the "where does this document type live?" question.

6. **No tree structure invariants.** Reports describe the tree pragmatically but don't define invariants like "one public schema per leaf" that keep the system coherent as it scales.

7. **L2 only — no L3 content.** Reports estimate L3 at 635-930 categories but don't define any. Our ~130 L3 leaves with descriptions are more actionable.

---

## 5. Recommendations for Combining

### 5.1 Take from our work

- **Schema inheritance (ALL-children rule)** — this is the strongest architectural contribution and should be the foundation.
- **Ancestor fallback on low confidence** — extracts partial data instead of stalling. Reports' human-in-the-loop can be layered on top.
- **One public schema per leaf invariant** — keeps tree and schema aligned.
- **No CAT-20** — unclassifiable docs signal tree evolution. But: add an operational "pending review" queue (not a tree node) for runtime handling.
- **Tree versioning** — explicitly version the tree as a first-class concept.

### 5.2 Take from the Reports

- **Adopt the ambiguity resolution rules** from Report 3 Section 5 — domain-over-form, issuer rule, original-purpose rule. Formalize these in ontology-plan.md.
- **Adopt confidence scoring thresholds** from Report 1 Section 7.2. These are operationally necessary.
- **Adopt the hybrid classification pipeline recommendation** from Report 1 Section 7.1 — embeddings for pre-filter, LLM for final classification.
- **Adopt the data model concepts** from Report 2 — CategoryNode, CategoryTreeVersion, SchemaTag, ExtractionTrace. These align well with our architectural decisions.
- **Adopt the regional/industry variation analysis** from Report 3 Section 6 — country-specific fields resolve to L3+ regional variants, which also answers our open question.
- **Adopt the schema lifecycle with staging** — community → staging → production is more careful than direct promotion.
- **Adopt the volume-weighted estimates** for prioritizing which leaf schemas to build first.
- **Adopt the LLM model selection and cost analysis** from Report 4 — this is operationally necessary and our work doesn't address it.

### 5.3 Resolve conflicts

| Conflict | Reports' Position | Our Position | Recommended Resolution |
|----------|------------------|--------------|----------------------|
| **Unknown/Other bucket** | CAT-20 exists | Rejected | **No tree node for "unknown."** Use an operational queue outside the tree (aligns with our architecture) but adopt Reports' subcategories (composites, damaged, novel) as queue routing labels. |
| **Schema-to-category mapping** | Many-to-many (SchemaTag) | One-to-one (one public schema per leaf) | **Keep our invariant for public schemas.** Private schemas can be tagged to one leaf. If a schema legitimately fits two leaves, that's a signal the tree needs an ancestor node. |
| **Low-confidence handling** | Human-in-the-loop escalation | Ancestor schema fallback | **Both.** Fall back to ancestor schema for immediate partial extraction, AND flag for human review. They're not mutually exclusive. |
| **Schema lifecycle** | community → staging → production | draft → active → deprecated | **Adopt Reports' model.** Adding a staging step before production is safer. |
| **Country handling** | L3+ regional variants | Unresolved (private schemas?) | **Adopt Reports' approach.** Regional variants at L3+ is cleaner and resolves our open question. Public schemas at L3+ can be country-specific. |

### 5.4 Gaps assessed — no structural changes needed

The following were initially flagged as gaps but after analysis are **not issues**:

- **Utilities & Telecommunications**: Utility bills are invoices (Finance > Invoicing & Payments). Telecom contracts are service contracts (Legal > Contracts). These documents have homes in the existing tree. Specific leaf nodes can be added if schema distinctness warrants it (e.g., "Utility Bills" leaf with meter/consumption fields), but no new root is needed. Classify by what the document IS, not its industry of origin.
- **Travel & Expense**: Expense reports are financial accountability docs (Finance or HR). Hotel folios and flight invoices are invoices. Boarding passes and itineraries are niche and rare in B2B IDP. No new root needed.
- **Finance root volume concentration (~37%)**: An imbalanced tree is fine for classification — "Finance" is highly recognizable. The L2 fan-out (6 nodes) is well within manageable range. Schema inheritance at the Finance root still works for universal fields (currency, date, organization_name). Not a problem.

---

## 6. Summary

**No critical structural issues were found.** The 12-root tree holds up. The initially flagged gaps (Utilities, Travel, Finance volume) resolved on closer analysis — existing tree structure accommodates these document types without new roots.

The two approaches are complementary. Our work contributes the stronger **architectural invariants** (inheritance, fallback, one-public-per-leaf). The Reports contribute the stronger **operational design** (hybrid classification, confidence scoring, data model, cost analysis, ambiguity rules, volume weighting).

The combined system would be:
- **Our tree structure** (12 consolidated roots, ~130 L3 leaves) — unchanged
- **Our schema architecture** (inheritance, ancestor fallback, one public per leaf)
- **Reports' operational layer** (hybrid classification, confidence scoring, data model, schema lifecycle)
- **Reports' ambiguity resolution rules** formalized into the tree
- **Reports' regional variant strategy** (L3+ country-specific) resolving our open country question

Next actions:
1. Formalize ambiguity resolution rules into ontology-plan.md
2. Adopt confidence scoring thresholds
3. Mark country-specific fields as resolved: L3+ regional variants
4. Adopt schema lifecycle with staging step
5. Add specific leaf nodes for common document types (utility bills, expense reports) where schema distinctness warrants it
