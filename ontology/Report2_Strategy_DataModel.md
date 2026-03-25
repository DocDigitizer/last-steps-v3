# Report 2: Schema Engine / Document Ontology -- Strategy, Data Model & Approach

**DocDigitizer -- Intelligent Document Processing**
**Date:** 2026-03-25
**Classification:** Internal -- Engineering

---

## Table of Contents

1. [Strategy Overview](#1-strategy-overview)
2. [System Architecture](#2-system-architecture)
3. [Complete Data Model](#3-complete-data-model)
4. [API Design](#4-api-design)
5. [CE Pipeline Design](#5-ce-pipeline-design)
6. [Extraction Pipeline Design](#6-extraction-pipeline-design)
7. [Schema Lifecycle](#7-schema-lifecycle)
8. [Versioning Strategy](#8-versioning-strategy)
9. [Scaling Considerations](#9-scaling-considerations)
10. [Implementation Roadmap](#10-implementation-roadmap)

---

## 1. Strategy Overview

### 1.1 Core Philosophy

The Schema Engine operates on a single unifying principle: **every interaction with a document is a schema-driven extraction**. Category identification is extraction. Field extraction is extraction. Schema suggestion is extraction. By collapsing all document understanding into one primitive -- "extract structured data from a document given a schema" -- the system achieves architectural simplicity while handling all three inbound scenarios (Chaos, Semi-Chaos, Organized).

### 1.2 Design Pillars

| Pillar | Rationale |
|--------|-----------|
| **Schema-as-truth** | The schema is the contract. No extraction happens without one. Even category extraction uses schemas. |
| **Hierarchical narrowing** | Never ask an LLM to choose from more than ~20 options at once. Iterate top-down through category levels. |
| **Immutable versioning** | Schemas are append-only. A version is never mutated. Old versions remain addressable forever. |
| **Community-to-production lifecycle** | New schemas enter as "community" and graduate to "production" through validation gates. |
| **Storage is cheap, tokens are expensive** | Maintain the full long tail of schemas. Optimize ruthlessly for token economy during extraction. |
| **Organization isolation with shared commons** | Each org gets its own groups and overrides, but the system category tree and production schemas are shared global assets. |

### 1.3 Scenario Resolution

| Scenario | Resolution Path |
|----------|----------------|
| **Chaos** | Full CE pipeline: Level 1 -> Level 2 -> ... -> Level N -> candidate schemas -> extraction. If no match, trigger schema suggestion. |
| **Semi-Chaos** | Organization has a schema group. Derive the narrowest CE entry point from that group. Run CE from that point downward. |
| **Organized** | Organization sends a specific schema group ID. System selects the optimal CE entry level, runs abbreviated CE, then extracts. If the group is small enough (<= 20 schemas), skip CE entirely. |

### 1.4 Key Invariants

1. No extraction call ever receives more than 20 candidate schemas.
2. Every schema in the system is tagged with its full category path (e.g., `financial > invoice > utility_invoice`).
3. The category tree is itself a set of versioned schemas, managed alongside data schemas.
4. Every extraction job produces an auditable trace: which CE levels were traversed, which schemas were candidates, which was selected, and with what confidence.

---

## 2. System Architecture

### 2.1 Component Overview

The system is composed of five primary components, each with a clear boundary of responsibility.

```
+----------------------------------------------------------------------+
|                         API Gateway / Router                         |
+----------------------------------------------------------------------+
         |              |               |               |
         v              v               v               v
+---------------+ +--------------+ +--------------+ +----------------+
|   Schema      | |  Category    | |  Extraction  | |   Schema       |
|   Registry    | |  Extraction  | |  Pipeline    | |   Lifecycle    |
|               | |  Engine (CE) | |              | |   Manager      |
+-------+-------+ +------+-------+ +------+-------+ +-------+--------+
        |                |                |                  |
        +--------+-------+--------+-------+--------+---------+
                 |                |                |
                 v                v                v
        +----------------+ +-------------+ +----------------+
        |  Schema Store  | | Job Queue & | | Telemetry &    |
        |  (DB + Cache)  | | Workers     | | Audit Log      |
        +----------------+ +-------------+ +----------------+
```

### 2.2 Component Responsibilities

**Schema Registry**
- CRUD operations for schemas, schema groups, and organizations.
- Version management: creating new versions, resolving `latest`, pinning.
- Schema validation (JSON Schema draft 2020-12 compliance).
- Tag management: linking schemas to category paths.

**Category Extraction Engine (CE)**
- Owns the category tree (all levels, all versions).
- Executes the iterative top-down classification pipeline.
- Manages CE schemas (the special single-field schemas used at each level).
- Handles short-circuit logic when the entry point is known.

**Extraction Pipeline**
- Receives a document + a set of candidate schemas (output of CE).
- Orchestrates the LLM extraction call(s).
- Ranks results by confidence when multiple schemas are plausible.
- Returns structured extraction result with metadata.

**Schema Lifecycle Manager**
- Tracks schema maturity (community -> staging -> production).
- Manages promotion workflows, approval gates, and rollback.
- Handles schema suggestion: when CE + extraction yields low confidence, proposes a new schema.
- Monitors community schema usage patterns to surface promotion candidates.

**Schema Store**
- Primary persistence: PostgreSQL for relational data and metadata.
- Schema body storage: PostgreSQL JSONB columns (schemas are typically 1-50 KB).
- Hot cache: Redis for frequently accessed schemas, CE trees, and org-group mappings.
- Full-text and vector index: for schema similarity search during suggestion deduplication.

### 2.3 External Integration Points

| Integration | Purpose |
|-------------|---------|
| **LLM Provider(s)** | CE classification and document extraction. Abstracted behind a provider interface to support model swapping. |
| **Object Storage (S3/Blob)** | Document file storage. Documents are referenced by URI; the engine never stores raw files in the DB. |
| **Webhook / Callback** | Async job completion notifications to calling systems. |
| **Admin UI** | Schema browsing, category tree management, lifecycle approvals. |

---

## 3. Complete Data Model

### 3.1 Entity-Relationship Overview

```
Organization 1---* OrgSchemaGroupLink *---1 SchemaGroup
SchemaGroup  1---* SchemaGroupMember  *---1 Schema
Schema       1---* SchemaVersion
Schema       1---* SchemaTag          *---1 CategoryNode
CategoryNode 1---* CategoryNode (parent-child)
CategoryNode 1---1 CategorySchema (CE schema for that level)
CategorySchema 1---* CategorySchemaVersion
ExtractionJob *---1 Organization
ExtractionJob *---0..1 SchemaGroup
ExtractionJob 1---1 ExtractionTrace
ExtractionJob 1---* ExtractionResult
SchemaVersion 1---* ExtractionResult
Schema 1---1 SchemaMaturityRecord
```

### 3.2 Entity Definitions

---

#### 3.2.1 Organization

Represents a customer or tenant in the system.

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| `id` | UUID | PK | Unique identifier |
| `external_id` | VARCHAR(255) | UNIQUE, NOT NULL | Customer-facing identifier (e.g., from billing system) |
| `name` | VARCHAR(500) | NOT NULL | Display name |
| `default_schema_group_id` | UUID | FK -> SchemaGroup, NULLABLE | Default group used when no group is specified |
| `settings` | JSONB | NOT NULL DEFAULT '{}' | Org-level config: default confidence thresholds, max CE depth, etc. |
| `is_active` | BOOLEAN | NOT NULL DEFAULT true | Soft-delete / deactivation flag |
| `created_at` | TIMESTAMPTZ | NOT NULL | Creation timestamp |
| `updated_at` | TIMESTAMPTZ | NOT NULL | Last modification timestamp |

**Indexes:**
- `idx_org_external_id` on `external_id` (unique)
- `idx_org_active` on `is_active` (partial: WHERE is_active = true)

**Sample JSON:**
```json
{
  "id": "a1b2c3d4-0001-4000-a000-000000000001",
  "external_id": "cust_acme_corp",
  "name": "Acme Corporation",
  "default_schema_group_id": "b2c3d4e5-0001-4000-a000-000000000001",
  "settings": {
    "ce_confidence_threshold": 0.85,
    "max_ce_depth": 4,
    "allow_schema_suggestion": true,
    "preferred_llm_tier": "fast"
  },
  "is_active": true,
  "created_at": "2026-01-15T10:30:00Z",
  "updated_at": "2026-03-20T14:00:00Z"
}
```

---

#### 3.2.2 SchemaGroup

A named collection of schemas. An organization can be linked to multiple groups. A group can be shared across organizations.

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| `id` | UUID | PK | Unique identifier |
| `name` | VARCHAR(500) | NOT NULL | Human-readable name |
| `description` | TEXT | NULLABLE | Purpose / scope of this group |
| `owner_org_id` | UUID | FK -> Organization, NULLABLE | Org that created this group. NULL = system-owned. |
| `is_system` | BOOLEAN | NOT NULL DEFAULT false | True for system-managed groups (e.g., "all_production_schemas") |
| `metadata` | JSONB | NOT NULL DEFAULT '{}' | Arbitrary metadata |
| `created_at` | TIMESTAMPTZ | NOT NULL | |
| `updated_at` | TIMESTAMPTZ | NOT NULL | |

**Indexes:**
- `idx_sg_owner` on `owner_org_id`
- `idx_sg_system` on `is_system` (partial: WHERE is_system = true)

**Sample JSON:**
```json
{
  "id": "b2c3d4e5-0001-4000-a000-000000000001",
  "name": "Acme AP Documents",
  "description": "All document types processed by Acme's accounts payable department",
  "owner_org_id": "a1b2c3d4-0001-4000-a000-000000000001",
  "is_system": false,
  "metadata": {
    "department": "accounts_payable",
    "max_schemas": 200
  },
  "created_at": "2026-01-20T09:00:00Z",
  "updated_at": "2026-03-01T11:00:00Z"
}
```

---

#### 3.2.3 OrgSchemaGroupLink

Many-to-many relationship between organizations and schema groups, with configuration.

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| `id` | UUID | PK | |
| `org_id` | UUID | FK -> Organization, NOT NULL | |
| `schema_group_id` | UUID | FK -> SchemaGroup, NOT NULL | |
| `access_level` | VARCHAR(50) | NOT NULL DEFAULT 'read' | 'read', 'write', 'admin' |
| `is_default` | BOOLEAN | NOT NULL DEFAULT false | Whether this is the org's default group |
| `created_at` | TIMESTAMPTZ | NOT NULL | |

**Indexes:**
- `idx_osgl_unique` UNIQUE on (`org_id`, `schema_group_id`)
- `idx_osgl_org` on `org_id`
- `idx_osgl_group` on `schema_group_id`

---

#### 3.2.4 Schema

The core entity. Represents a document type's extraction contract. The schema body itself lives in SchemaVersion; this entity is the anchor.

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| `id` | UUID | PK | Stable identifier across all versions |
| `slug` | VARCHAR(255) | UNIQUE, NOT NULL | Human-readable identifier (e.g., `invoice.utility.electricity`) |
| `name` | VARCHAR(500) | NOT NULL | Display name |
| `description` | TEXT | NULLABLE | What documents this schema represents |
| `schema_type` | VARCHAR(50) | NOT NULL DEFAULT 'document' | 'document' (data extraction) or 'category' (CE) |
| `maturity` | VARCHAR(50) | NOT NULL DEFAULT 'community' | 'community', 'staging', 'production', 'deprecated' |
| `current_version_id` | UUID | FK -> SchemaVersion, NULLABLE | Points to the active/latest version |
| `origin` | VARCHAR(50) | NOT NULL DEFAULT 'manual' | 'manual', 'suggested', 'imported' |
| `suggested_by_job_id` | UUID | FK -> ExtractionJob, NULLABLE | If origin='suggested', which job triggered it |
| `created_by` | VARCHAR(255) | NULLABLE | User or system that created the schema |
| `created_at` | TIMESTAMPTZ | NOT NULL | |
| `updated_at` | TIMESTAMPTZ | NOT NULL | |

**Indexes:**
- `idx_schema_slug` UNIQUE on `slug`
- `idx_schema_type` on `schema_type`
- `idx_schema_maturity` on `maturity`
- `idx_schema_type_maturity` on (`schema_type`, `maturity`)

**Sample JSON:**
```json
{
  "id": "c3d4e5f6-0001-4000-a000-000000000001",
  "slug": "invoice.utility.electricity",
  "name": "Electricity Utility Invoice",
  "description": "Schema for extracting data from electricity utility invoices including meter readings, consumption, tariffs, and charges.",
  "schema_type": "document",
  "maturity": "production",
  "current_version_id": "d4e5f6a7-0003-4000-a000-000000000001",
  "origin": "manual",
  "suggested_by_job_id": null,
  "created_by": "admin@docdigitizer.com",
  "created_at": "2026-02-01T08:00:00Z",
  "updated_at": "2026-03-10T16:00:00Z"
}
```

---

#### 3.2.5 SchemaVersion

Immutable snapshot of a schema's body at a point in time.

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| `id` | UUID | PK | |
| `schema_id` | UUID | FK -> Schema, NOT NULL | Parent schema |
| `version_number` | INTEGER | NOT NULL | Monotonically increasing per schema. 1, 2, 3, ... |
| `body` | JSONB | NOT NULL | The actual JSON Schema document |
| `body_hash` | VARCHAR(64) | NOT NULL | SHA-256 of canonicalized body (dedup guard) |
| `change_summary` | TEXT | NULLABLE | What changed from previous version |
| `field_count` | INTEGER | NOT NULL | Number of extractable fields (precomputed for quick filtering) |
| `estimated_token_cost` | INTEGER | NOT NULL | Approx token count when serialized for LLM prompt |
| `is_active` | BOOLEAN | NOT NULL DEFAULT true | False if this version has been explicitly retracted |
| `created_by` | VARCHAR(255) | NULLABLE | |
| `created_at` | TIMESTAMPTZ | NOT NULL | |

**Indexes:**
- `idx_sv_schema_version` UNIQUE on (`schema_id`, `version_number`)
- `idx_sv_schema_active` on (`schema_id`, `is_active`) (partial: WHERE is_active = true)
- `idx_sv_body_hash` on `body_hash`

**Sample JSON (body field):**
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://schemas.docdigitizer.com/invoice/utility/electricity/v3",
  "type": "object",
  "title": "Electricity Utility Invoice",
  "required": ["vendor_name", "invoice_number", "invoice_date", "total_amount"],
  "properties": {
    "vendor_name": {
      "type": "string",
      "description": "Name of the electricity utility company"
    },
    "invoice_number": {
      "type": "string",
      "description": "Unique invoice identifier"
    },
    "invoice_date": {
      "type": "string",
      "format": "date",
      "description": "Date the invoice was issued"
    },
    "billing_period_start": {
      "type": "string",
      "format": "date"
    },
    "billing_period_end": {
      "type": "string",
      "format": "date"
    },
    "meter_number": {
      "type": "string"
    },
    "previous_reading": {
      "type": "number"
    },
    "current_reading": {
      "type": "number"
    },
    "consumption_kwh": {
      "type": "number",
      "description": "Total electricity consumed in kWh"
    },
    "unit_rate": {
      "type": "number"
    },
    "subtotal": {
      "type": "number"
    },
    "tax_amount": {
      "type": "number"
    },
    "total_amount": {
      "type": "number",
      "description": "Total amount due"
    },
    "currency": {
      "type": "string",
      "pattern": "^[A-Z]{3}$"
    },
    "due_date": {
      "type": "string",
      "format": "date"
    },
    "account_number": {
      "type": "string"
    }
  }
}
```

---

#### 3.2.6 SchemaGroupMember

Links schemas to schema groups. A schema can belong to multiple groups.

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| `id` | UUID | PK | |
| `schema_group_id` | UUID | FK -> SchemaGroup, NOT NULL | |
| `schema_id` | UUID | FK -> Schema, NOT NULL | |
| `pinned_version_id` | UUID | FK -> SchemaVersion, NULLABLE | If set, this group uses a specific version. NULL = latest. |
| `priority` | INTEGER | NOT NULL DEFAULT 0 | Higher = preferred during tie-breaking in extraction |
| `added_at` | TIMESTAMPTZ | NOT NULL | |

**Indexes:**
- `idx_sgm_unique` UNIQUE on (`schema_group_id`, `schema_id`)
- `idx_sgm_group` on `schema_group_id`
- `idx_sgm_schema` on `schema_id`

---

#### 3.2.7 CategoryNode

Represents a node in the hierarchical category tree. The tree is a directed acyclic graph (in practice, a strict tree) used by the CE pipeline.

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| `id` | UUID | PK | |
| `code` | VARCHAR(100) | UNIQUE, NOT NULL | Machine-readable code (e.g., `financial`, `invoice`, `utility_invoice`) |
| `name` | VARCHAR(500) | NOT NULL | Human-readable name |
| `description` | TEXT | NULLABLE | What this category encompasses |
| `parent_id` | UUID | FK -> CategoryNode, NULLABLE | NULL = root node |
| `level` | INTEGER | NOT NULL | 0 = root, 1 = first real level, 2, 3, ... |
| `path` | VARCHAR(2000) | NOT NULL | Materialized path: `financial.invoice.utility_invoice` |
| `tree_version_id` | UUID | FK -> CategoryTreeVersion, NOT NULL | Which version of the tree this node belongs to |
| `display_order` | INTEGER | NOT NULL DEFAULT 0 | Sort order among siblings |
| `is_leaf` | BOOLEAN | NOT NULL DEFAULT false | True if this node has no children (terminal category) |
| `schema_count` | INTEGER | NOT NULL DEFAULT 0 | Precomputed: number of schemas tagged with this category or descendants |
| `ce_schema_id` | UUID | FK -> Schema, NULLABLE | The CE schema used to classify at this node's child level |
| `metadata` | JSONB | NOT NULL DEFAULT '{}' | |
| `created_at` | TIMESTAMPTZ | NOT NULL | |
| `updated_at` | TIMESTAMPTZ | NOT NULL | |

**Indexes:**
- `idx_cn_code` UNIQUE on `code`
- `idx_cn_parent` on `parent_id`
- `idx_cn_path` on `path` (for prefix queries)
- `idx_cn_tree_version` on `tree_version_id`
- `idx_cn_level` on `level`
- `idx_cn_ce_schema` on `ce_schema_id`

**Sample JSON:**
```json
{
  "id": "e5f6a7b8-0001-4000-a000-000000000001",
  "code": "financial.invoice",
  "name": "Invoices",
  "description": "All types of invoices: commercial, utility, government, etc.",
  "parent_id": "e5f6a7b8-0000-4000-a000-000000000001",
  "level": 2,
  "path": "financial.invoice",
  "tree_version_id": "f6a7b8c9-0001-4000-a000-000000000001",
  "display_order": 1,
  "is_leaf": false,
  "schema_count": 347,
  "ce_schema_id": "c3d4e5f6-CE01-4000-a000-000000000001",
  "metadata": {
    "icon": "receipt",
    "examples": ["commercial invoice", "utility bill", "proforma invoice"]
  },
  "created_at": "2026-01-10T08:00:00Z",
  "updated_at": "2026-03-15T10:00:00Z"
}
```

---

#### 3.2.8 CategoryTreeVersion

Tracks versions of the entire category tree. When the tree is modified, a new version is created, and old nodes are preserved under the old version.

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| `id` | UUID | PK | |
| `version_number` | INTEGER | NOT NULL | Monotonically increasing |
| `description` | TEXT | NULLABLE | What changed |
| `is_active` | BOOLEAN | NOT NULL DEFAULT false | Only one version is active at a time |
| `node_count` | INTEGER | NOT NULL | Total nodes in this version |
| `max_depth` | INTEGER | NOT NULL | Deepest level in this version |
| `published_at` | TIMESTAMPTZ | NULLABLE | When this version was activated |
| `created_by` | VARCHAR(255) | NULLABLE | |
| `created_at` | TIMESTAMPTZ | NOT NULL | |

**Indexes:**
- `idx_ctv_active` UNIQUE on `is_active` (partial: WHERE is_active = true) -- enforces single active
- `idx_ctv_version` UNIQUE on `version_number`

---

#### 3.2.9 SchemaTag

Links a Schema to one or more CategoryNodes. This is how the system knows which schemas are candidates after CE narrows the category.

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| `id` | UUID | PK | |
| `schema_id` | UUID | FK -> Schema, NOT NULL | |
| `category_node_id` | UUID | FK -> CategoryNode, NOT NULL | |
| `is_primary` | BOOLEAN | NOT NULL DEFAULT false | Whether this is the schema's primary category |
| `confidence` | DECIMAL(3,2) | NOT NULL DEFAULT 1.00 | How strongly this schema belongs to this category (1.00 = definite) |
| `tagged_by` | VARCHAR(50) | NOT NULL DEFAULT 'manual' | 'manual', 'auto', 'suggested' |
| `created_at` | TIMESTAMPTZ | NOT NULL | |

**Indexes:**
- `idx_st_unique` UNIQUE on (`schema_id`, `category_node_id`)
- `idx_st_category` on `category_node_id` -- critical: used in CE result -> schema lookup
- `idx_st_schema` on `schema_id`
- `idx_st_primary` on (`schema_id`) (partial: WHERE is_primary = true)

---

#### 3.2.10 ExtractionJob

Tracks every document submission through the system end-to-end.

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| `id` | UUID | PK | |
| `org_id` | UUID | FK -> Organization, NOT NULL | |
| `schema_group_id` | UUID | FK -> SchemaGroup, NULLABLE | If provided by the caller |
| `document_uri` | VARCHAR(2000) | NOT NULL | Reference to the document in object storage |
| `document_hash` | VARCHAR(64) | NOT NULL | SHA-256 of document content (dedup & cache key) |
| `document_type_hint` | VARCHAR(255) | NULLABLE | Optional hint from the caller (e.g., "invoice") |
| `status` | VARCHAR(50) | NOT NULL DEFAULT 'pending' | 'pending', 'ce_in_progress', 'extracting', 'completed', 'failed', 'suggesting_schema' |
| `scenario` | VARCHAR(50) | NOT NULL | 'chaos', 'semi_chaos', 'organized' -- determined at intake |
| `priority` | INTEGER | NOT NULL DEFAULT 0 | Job priority for queue ordering |
| `ce_entry_level` | INTEGER | NULLABLE | The CE level at which processing started (0 = root, higher = short-circuited) |
| `callback_url` | VARCHAR(2000) | NULLABLE | Webhook to call on completion |
| `result_schema_id` | UUID | FK -> Schema, NULLABLE | The schema ultimately used for extraction |
| `result_schema_version_id` | UUID | FK -> SchemaVersion, NULLABLE | Specific version used |
| `result_confidence` | DECIMAL(5,4) | NULLABLE | 0.0000 to 1.0000 |
| `result_payload` | JSONB | NULLABLE | The extracted data |
| `error_message` | TEXT | NULLABLE | If status='failed' |
| `processing_time_ms` | INTEGER | NULLABLE | Total wall-clock time |
| `llm_calls_count` | INTEGER | NOT NULL DEFAULT 0 | Total LLM API calls made |
| `llm_tokens_used` | INTEGER | NOT NULL DEFAULT 0 | Total tokens consumed |
| `created_at` | TIMESTAMPTZ | NOT NULL | |
| `updated_at` | TIMESTAMPTZ | NOT NULL | |
| `completed_at` | TIMESTAMPTZ | NULLABLE | |

**Indexes:**
- `idx_ej_org` on `org_id`
- `idx_ej_status` on `status`
- `idx_ej_org_status` on (`org_id`, `status`)
- `idx_ej_document_hash` on `document_hash`
- `idx_ej_created` on `created_at` (descending, for recent-first queries)
- `idx_ej_result_schema` on `result_schema_id`

---

#### 3.2.11 ExtractionTrace

Detailed audit trail of the CE and extraction steps for a job. One-to-one with ExtractionJob.

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| `id` | UUID | PK | |
| `job_id` | UUID | FK -> ExtractionJob, UNIQUE, NOT NULL | |
| `ce_steps` | JSONB | NOT NULL DEFAULT '[]' | Array of CE step records (see below) |
| `candidate_schemas` | JSONB | NOT NULL DEFAULT '[]' | Schemas that were candidates after CE |
| `extraction_attempts` | JSONB | NOT NULL DEFAULT '[]' | Array of extraction attempt records |
| `schema_suggestion` | JSONB | NULLABLE | If a new schema was suggested, details here |
| `created_at` | TIMESTAMPTZ | NOT NULL | |

**`ce_steps` element structure:**
```json
{
  "level": 2,
  "category_node_id": "e5f6a7b8-0001-4000-a000-000000000001",
  "ce_schema_id": "c3d4e5f6-CE01-4000-a000-000000000001",
  "ce_schema_version_id": "d4e5f6a7-CE01-4000-a000-000000000001",
  "options_presented": ["commercial_invoice", "utility_invoice", "government_invoice", "proforma_invoice"],
  "selected_option": "utility_invoice",
  "confidence": 0.94,
  "latency_ms": 320,
  "tokens_used": 850,
  "model_used": "gpt-4o-mini"
}
```

**`extraction_attempts` element structure:**
```json
{
  "schema_id": "c3d4e5f6-0001-4000-a000-000000000001",
  "schema_version_id": "d4e5f6a7-0003-4000-a000-000000000001",
  "confidence": 0.92,
  "field_fill_rate": 0.88,
  "latency_ms": 1200,
  "tokens_used": 3500,
  "model_used": "gpt-4o",
  "was_selected": true
}
```

---

#### 3.2.12 ExtractionResult

Normalized extraction results, one per schema attempted during an extraction job. Enables per-schema analytics.

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| `id` | UUID | PK | |
| `job_id` | UUID | FK -> ExtractionJob, NOT NULL | |
| `schema_id` | UUID | FK -> Schema, NOT NULL | |
| `schema_version_id` | UUID | FK -> SchemaVersion, NOT NULL | |
| `confidence` | DECIMAL(5,4) | NOT NULL | |
| `field_fill_rate` | DECIMAL(5,4) | NOT NULL | Fraction of schema fields that were populated |
| `payload` | JSONB | NOT NULL | The extracted data for this schema |
| `validation_errors` | JSONB | NOT NULL DEFAULT '[]' | JSON Schema validation errors against the body |
| `was_selected` | BOOLEAN | NOT NULL DEFAULT false | True if this was the top result |
| `rank` | INTEGER | NOT NULL | 1 = best match, 2, 3, ... |
| `created_at` | TIMESTAMPTZ | NOT NULL | |

**Indexes:**
- `idx_er_job` on `job_id`
- `idx_er_schema` on `schema_id`
- `idx_er_job_rank` on (`job_id`, `rank`)

---

#### 3.2.13 SchemaMaturityRecord

Tracks the lifecycle transitions of a schema. One record per transition event (audit trail).

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| `id` | UUID | PK | |
| `schema_id` | UUID | FK -> Schema, NOT NULL | |
| `from_maturity` | VARCHAR(50) | NULLABLE | NULL for initial creation |
| `to_maturity` | VARCHAR(50) | NOT NULL | |
| `reason` | TEXT | NULLABLE | Why the transition happened |
| `evidence` | JSONB | NOT NULL DEFAULT '{}' | Supporting data (e.g., usage stats, validation results) |
| `approved_by` | VARCHAR(255) | NULLABLE | User who approved (NULL for auto-transitions) |
| `created_at` | TIMESTAMPTZ | NOT NULL | |

**Indexes:**
- `idx_smr_schema` on `schema_id`
- `idx_smr_schema_time` on (`schema_id`, `created_at`)

---

#### 3.2.14 SchemaSimilarityIndex

Precomputed similarity data used to detect near-duplicate schemas during the suggestion workflow.

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| `id` | UUID | PK | |
| `schema_id` | UUID | FK -> Schema, UNIQUE, NOT NULL | |
| `schema_version_id` | UUID | FK -> SchemaVersion, NOT NULL | Version this embedding was computed from |
| `field_signature` | VARCHAR(500) | NOT NULL | Sorted, hashed list of field names (quick dedup) |
| `embedding` | VECTOR(768) | NOT NULL | Embedding of schema description + field names (pgvector) |
| `updated_at` | TIMESTAMPTZ | NOT NULL | |

**Indexes:**
- `idx_ssi_field_sig` on `field_signature`
- `idx_ssi_embedding` USING ivfflat on `embedding` (for ANN search)

---

#### 3.2.15 CacheEntry (Operational)

Optional hot cache table for extraction results keyed by document hash + schema context. Enables instant re-extraction of identical documents.

| Field | Type | Constraints | Description |
|-------|------|-------------|-------------|
| `id` | UUID | PK | |
| `cache_key` | VARCHAR(200) | UNIQUE, NOT NULL | Hash of (document_hash + schema_group_id or "global" + tree_version) |
| `job_id` | UUID | FK -> ExtractionJob, NOT NULL | Original job that produced this result |
| `result_payload` | JSONB | NOT NULL | Cached extraction result |
| `expires_at` | TIMESTAMPTZ | NOT NULL | TTL-based expiry |
| `created_at` | TIMESTAMPTZ | NOT NULL | |

**Indexes:**
- `idx_cache_key` UNIQUE on `cache_key`
- `idx_cache_expires` on `expires_at` (for cleanup)

---

### 3.3 Full ER Diagram (Textual)

```
                              +-------------------+
                              |   Organization    |
                              +--------+----------+
                                       |
                          1:N via OrgSchemaGroupLink
                                       |
                              +--------v----------+
                              | OrgSchemaGroupLink |
                              +--------+----------+
                                       |
                          N:1          |
                              +--------v----------+
                              |   SchemaGroup     |
                              +--------+----------+
                                       |
                          1:N via SchemaGroupMember
                                       |
                              +--------v----------+
                              | SchemaGroupMember  |
                              +--------+----------+
                                       |
                              +--------v----------+
                              |     Schema        |<----------- SchemaMaturityRecord (1:N)
                              +--+-----+------+---+
                                 |     |      |
                    1:N          |     | 1:N  |  1:N
            SchemaVersion <------+     |      +-----> SchemaTag
                                       |                  |
                                       |             N:1  |
                                       |                  v
                              +--------v----------+ CategoryNode
                              |  ExtractionJob    | (self-referential
                              +--------+----------+  parent-child)
                                       |                  |
                              1:1      | 1:N         1:1  |
                                       |                  v
                         ExtractionTrace    ExtractionResult
                                                CategoryTreeVersion
```

---

## 4. API Design

### 4.1 Schema CRUD

All endpoints are prefixed with `/api/v1`.

---

**`POST /schemas`** -- Create a new schema

```json
// Request
{
  "slug": "invoice.utility.electricity",
  "name": "Electricity Utility Invoice",
  "description": "Schema for electricity utility invoices",
  "schema_type": "document",
  "body": { /* JSON Schema */ },
  "category_tags": ["financial.invoice.utility_invoice"],
  "initial_maturity": "community"
}

// Response 201
{
  "id": "c3d4e5f6-...",
  "slug": "invoice.utility.electricity",
  "version": {
    "id": "d4e5f6a7-...",
    "version_number": 1,
    "field_count": 16,
    "estimated_token_cost": 1200
  },
  "maturity": "community",
  "created_at": "2026-03-25T10:00:00Z"
}
```

**`GET /schemas/{id_or_slug}`** -- Get schema with current version

**`GET /schemas/{id}/versions`** -- List all versions

**`POST /schemas/{id}/versions`** -- Create a new version

```json
// Request
{
  "body": { /* updated JSON Schema */ },
  "change_summary": "Added meter_number field"
}
```

**`GET /schemas?maturity=production&category=financial.invoice&page=1&limit=50`** -- Search/filter schemas

**`DELETE /schemas/{id}`** -- Soft-delete (sets maturity to 'deprecated')

---

### 4.2 Schema Group Management

**`POST /schema-groups`** -- Create a group

```json
{
  "name": "Acme AP Documents",
  "description": "...",
  "owner_org_id": "a1b2c3d4-..."
}
```

**`POST /schema-groups/{id}/members`** -- Add schema to group

```json
{
  "schema_id": "c3d4e5f6-...",
  "pinned_version_id": null,
  "priority": 10
}
```

**`DELETE /schema-groups/{id}/members/{schema_id}`** -- Remove schema from group

**`GET /schema-groups/{id}/members?page=1&limit=100`** -- List members

**`GET /schema-groups/{id}/stats`** -- Group statistics (schema count, category distribution, maturity breakdown)

**`POST /orgs/{org_id}/schema-groups`** -- Link org to group

```json
{
  "schema_group_id": "b2c3d4e5-...",
  "access_level": "read",
  "is_default": false
}
```

---

### 4.3 Category Tree Management

**`GET /category-tree`** -- Get active tree (full or by level)

```json
// Query params: ?level=2&parent_code=financial
// Response
{
  "tree_version": 5,
  "nodes": [
    {
      "id": "e5f6a7b8-...",
      "code": "financial.invoice",
      "name": "Invoices",
      "level": 2,
      "parent_code": "financial",
      "children_count": 12,
      "schema_count": 347,
      "is_leaf": false
    }
  ]
}
```

**`POST /category-tree/versions`** -- Create a new tree version (draft)

**`POST /category-tree/versions/{version_id}/nodes`** -- Add a node to a draft tree

**`POST /category-tree/versions/{version_id}/publish`** -- Activate a tree version

**`GET /category-tree/versions`** -- List all tree versions

---

### 4.4 Document Submission & Extraction

**`POST /extract`** -- Submit a document for extraction

```json
// Request
{
  "org_id": "a1b2c3d4-...",
  "document_uri": "s3://dd-documents/acme/doc_12345.pdf",
  "schema_group_id": "b2c3d4e5-...",          // optional
  "document_type_hint": "invoice",              // optional
  "schema_id": "c3d4e5f6-...",                 // optional: skip CE entirely
  "callback_url": "https://acme.com/webhook",  // optional
  "options": {
    "include_alternatives": true,
    "max_alternatives": 3,
    "confidence_threshold": 0.80
  }
}

// Response 202
{
  "job_id": "f7a8b9c0-...",
  "status": "pending",
  "estimated_time_ms": 5000,
  "poll_url": "/api/v1/jobs/f7a8b9c0-..."
}
```

**`GET /jobs/{job_id}`** -- Poll job status

```json
// Response 200 (completed)
{
  "job_id": "f7a8b9c0-...",
  "status": "completed",
  "scenario": "semi_chaos",
  "result": {
    "schema_id": "c3d4e5f6-...",
    "schema_slug": "invoice.utility.electricity",
    "schema_version": 3,
    "confidence": 0.94,
    "field_fill_rate": 0.88,
    "data": {
      "vendor_name": "Pacific Gas & Electric",
      "invoice_number": "INV-2026-03-8847",
      "invoice_date": "2026-03-15",
      "total_amount": 247.83,
      "currency": "USD"
      // ... remaining fields
    },
    "validation_errors": []
  },
  "alternatives": [
    {
      "schema_id": "c3d4e5f6-...",
      "schema_slug": "invoice.utility.gas",
      "confidence": 0.31,
      "rank": 2
    }
  ],
  "trace": {
    "ce_steps": [ /* ... */ ],
    "total_llm_calls": 4,
    "total_tokens": 5200,
    "processing_time_ms": 3800
  }
}
```

**`GET /jobs?org_id={org_id}&status=completed&since=2026-03-01&limit=100`** -- List jobs

---

### 4.5 Schema Suggestion / Community Workflow

**`GET /schemas/suggestions?status=pending&page=1`** -- List suggested schemas awaiting review

**`POST /schemas/{id}/promote`** -- Promote schema maturity

```json
{
  "target_maturity": "staging",
  "reason": "Validated against 50 sample documents with >95% accuracy",
  "evidence": {
    "sample_count": 50,
    "avg_confidence": 0.96,
    "avg_fill_rate": 0.91
  }
}
```

**`POST /schemas/{id}/deprecate`** -- Deprecate a schema

```json
{
  "reason": "Superseded by invoice.utility.electricity_v2",
  "replacement_schema_id": "c3d4e5f6-..."
}
```

**`GET /schemas/{id}/lifecycle`** -- Get full maturity history

---

## 5. CE Pipeline Design

### 5.1 Overview

The Category Extraction (CE) pipeline is the mechanism by which the system narrows from potentially millions of schemas down to a manageable set of candidates (target: <= 20) before running the actual data extraction.

CE works by treating each level of the category hierarchy as an extraction problem. At each level, the system presents the document to an LLM with a special CE schema that has one field: the category at that level. The LLM's response selects a branch, and the system descends to the next level.

### 5.2 CE Schema Structure

Each non-leaf CategoryNode has an associated CE schema. The CE schema for a node with children is structured as:

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://schemas.docdigitizer.com/ce/financial/v2",
  "type": "object",
  "title": "Document Category Classification: Financial Subcategory",
  "description": "Classify the financial document into one of the following subcategories.",
  "required": ["category", "confidence"],
  "properties": {
    "category": {
      "type": "string",
      "enum": [
        "invoice",
        "receipt",
        "bank_statement",
        "tax_document",
        "insurance_document",
        "payment_confirmation",
        "credit_note",
        "purchase_order",
        "financial_report",
        "other_financial"
      ],
      "description": "The most specific financial subcategory that matches this document. 'invoice' = bills for goods/services, 'receipt' = proof of payment, 'bank_statement' = account activity summary, 'tax_document' = tax returns/forms/certificates, 'insurance_document' = policies/claims/certificates, 'payment_confirmation' = wire transfer/payment receipts, 'credit_note' = adjustments/refunds, 'purchase_order' = formal requests to purchase, 'financial_report' = balance sheets/P&L/audits, 'other_financial' = financial documents not matching above categories."
    },
    "confidence": {
      "type": "number",
      "minimum": 0,
      "maximum": 1,
      "description": "Your confidence in this classification from 0.0 (no confidence) to 1.0 (certain)"
    }
  }
}
```

Key design points:
- Each enum value has a clear description in the `description` field to reduce ambiguity.
- The `confidence` field is always present -- the LLM self-reports its certainty.
- An `other_*` catch-all category exists at every level to handle edge cases.

### 5.3 CE Step-by-Step Flow

```
INPUT: document, org_id, optional schema_group_id, optional hint

STEP 1: Determine Entry Point
  |
  +-- If schema_group_id provided:
  |     Compute the optimal CE entry level for this group (see 5.5)
  |
  +-- If document_type_hint provided:
  |     Map hint to closest CategoryNode. Start CE from that node's parent.
  |
  +-- Otherwise (Chaos):
        Start from root (Level 0)

STEP 2: Iterative Classification
  |
  FOR each level starting from entry_level:
  |
  |   2a. Load CE schema for current node
  |   2b. Build prompt: document content + CE schema + instructions
  |   2c. Call LLM
  |   2d. Parse response -> { category, confidence }
  |
  |   2e. DECISION:
  |       IF confidence >= threshold (default 0.85):
  |           Descend into selected child node
  |       ELIF confidence >= soft_threshold (default 0.60):
  |           Descend, but FLAG for multi-branch exploration
  |       ELSE:
  |           STOP CE. Use current node as the narrowest category.
  |
  |   2f. IF selected node is_leaf OR schema_count <= max_candidates (20):
  |           STOP CE. Sufficient narrowing achieved.
  |
  END FOR

STEP 3: Gather Candidate Schemas
  |
  Query all schemas tagged with the final category node
  (and descendants if the node is not a leaf)
  |
  Filter by:
    - maturity IN ('production', 'staging')  [community excluded by default]
    - If schema_group_id provided: also filter to group members
  |
  ORDER BY SchemaTag.confidence DESC, SchemaGroupMember.priority DESC

STEP 4: Output
  |
  Return candidate_schemas (list of schema IDs + versions)
  Return ce_trace (all steps, for audit)
```

### 5.4 Prompt Design for CE

The CE prompt is composed of four parts:

```
SYSTEM:
You are a document classification system. Your task is to classify the
provided document into the most appropriate category. Respond ONLY with
valid JSON matching the provided schema. Do not explain your reasoning.

SCHEMA:
{ce_schema_json}

CONTEXT (optional):
This document was submitted by an organization that typically processes:
{schema_group_description}
Previous classification levels determined this is a: {path_so_far}

DOCUMENT:
{document_content_or_first_N_pages}
```

Prompt design considerations:
1. **Document truncation**: For CE, only the first 2-3 pages are typically needed. This dramatically reduces token cost. Configurable per org.
2. **Model selection**: CE uses a fast, cheap model (e.g., GPT-4o-mini, Claude Haiku). Accuracy at this level is high because the categories are broad.
3. **Context injection**: When prior CE levels have been resolved, include the path. This primes the model and improves accuracy at deeper levels.
4. **No explanation requested**: Asking for JSON-only reduces output tokens and prevents the model from hedging.

### 5.5 Short-Circuit Optimization (Organized Scenario)

When a schema group is provided, the system computes the optimal entry level:

```
FUNCTION compute_ce_entry_level(schema_group_id) -> (entry_node, entry_level):

  1. Load all schemas in the group
  2. Collect all category tags for these schemas
  3. Build a set of all category paths

  4. Walk the category tree top-down:
     FOR each level starting from root:
       Count how many distinct categories at this level are represented
       IF count <= MAX_CANDIDATES (20):
         RETURN (parent_node_of_these_categories, level - 1)
     END FOR

  5. If we reach leaf level without convergence:
     RETURN (deepest common ancestor, its level)
```

**Example**: An org has a schema group with 150 schemas, but they all fall under `financial.invoice.*` (12 subcategories). The entry point is `financial.invoice` at level 2, and the first CE call classifies among those 12 invoice subtypes. One CE call instead of three.

### 5.6 Multi-Branch Exploration

When confidence is between the soft and hard thresholds, the system can explore multiple branches in parallel:

```
IF ce_confidence >= 0.60 AND ce_confidence < 0.85:
  top_categories = get_top_N_from_llm_response(N=2)

  FOR EACH candidate_category IN top_categories (PARALLEL):
    continue CE from candidate_category
  END FOR

  Merge all resulting candidate schemas
  Deduplicate
  Cap at max_candidates
```

This adds at most one extra parallel LLM call per ambiguous level. In practice, ambiguity is rare after the first level.

### 5.7 Confidence Thresholds and Fallback

| Threshold | Value (Default) | Action |
|-----------|----------------|--------|
| `hard_threshold` | 0.85 | Confident. Descend into selected category. |
| `soft_threshold` | 0.60 | Uncertain. Explore top-2 branches. |
| `abandon_threshold` | 0.40 | Very uncertain. Stop CE at current level. Use broader candidate set. |
| `suggest_threshold` | 0.30 | Likely no matching schema. Trigger schema suggestion flow. |

These thresholds are configurable per organization via `Organization.settings`.

### 5.8 CE Pipeline Pseudocode

```python
async def run_ce_pipeline(job: ExtractionJob) -> CEResult:
    """Execute the Category Extraction pipeline."""

    # Step 1: Determine entry point
    if job.schema_group_id:
        entry_node, entry_level = compute_ce_entry_level(job.schema_group_id)
    elif job.document_type_hint:
        entry_node = resolve_hint_to_node(job.document_type_hint)
        entry_level = entry_node.level
    else:
        entry_node = get_root_node()
        entry_level = 0

    # Step 2: Prepare document content (truncated for CE)
    doc_content = await load_document_content(
        job.document_uri,
        max_pages=org.settings.get("ce_max_pages", 3)
    )

    # Step 3: Iterative classification
    current_node = entry_node
    ce_steps = []
    path_so_far = current_node.path

    while True:
        # Check termination conditions
        if current_node.is_leaf:
            break
        if current_node.schema_count <= MAX_CANDIDATES:
            break

        # Load CE schema for this node
        ce_schema = load_ce_schema(current_node.ce_schema_id)

        # Build and execute LLM call
        prompt = build_ce_prompt(
            doc_content=doc_content,
            ce_schema=ce_schema,
            path_so_far=path_so_far,
            group_context=job.schema_group_id
        )

        response = await llm_client.extract(
            prompt=prompt,
            model=select_ce_model(current_node.level),
            temperature=0.0,
            max_tokens=100
        )

        result = parse_ce_response(response)

        # Record step
        ce_steps.append(CEStep(
            level=current_node.level,
            node=current_node,
            selected=result.category,
            confidence=result.confidence,
            tokens_used=response.usage.total_tokens
        ))

        # Decision logic
        if result.confidence < ABANDON_THRESHOLD:
            # Too uncertain -- stop here and use broad candidate set
            break

        if result.confidence < SOFT_THRESHOLD:
            # Explore multiple branches
            child_nodes = get_top_matching_children(current_node, result, n=2)
            # Run parallel CE from each child, merge results
            parallel_results = await asyncio.gather(*[
                run_ce_from_node(child, doc_content, ce_steps)
                for child in child_nodes
            ])
            candidate_schemas = merge_and_deduplicate(parallel_results)
            return CEResult(candidates=candidate_schemas, steps=ce_steps)

        # High confidence -- descend
        child_node = get_child_by_code(current_node, result.category)
        if child_node is None:
            # "other_*" category selected or unknown -- stop here
            break

        current_node = child_node
        path_so_far = current_node.path

    # Step 4: Gather candidate schemas from final node
    candidate_schemas = query_schemas_for_node(
        node=current_node,
        include_descendants=True,
        maturity_filter=["production", "staging"],
        schema_group_filter=job.schema_group_id,
        max_results=MAX_CANDIDATES
    )

    # Step 5: Check if we need to suggest a schema
    if len(candidate_schemas) == 0:
        return CEResult(
            candidates=[],
            steps=ce_steps,
            suggest_new_schema=True
        )

    return CEResult(candidates=candidate_schemas, steps=ce_steps)
```

---

## 6. Extraction Pipeline Design

### 6.1 Overview

After CE produces a list of candidate schemas (typically 1-20), the Extraction Pipeline selects the best schema(s) and performs the actual data extraction.

### 6.2 Schema Selection Logic

The pipeline uses a two-phase approach:

**Phase 1: Schema Ranking (cheap, fast)**

When there are more than 3 candidate schemas, run a lightweight ranking step before full extraction.

```python
async def rank_schemas(doc_content, candidates) -> list[RankedSchema]:
    """Rank candidate schemas by fit without full extraction."""

    # Build a compact representation: schema name + required fields + description
    schema_summaries = [
        {
            "schema_id": s.id,
            "name": s.name,
            "required_fields": get_required_field_names(s),
            "description": s.description
        }
        for s in candidates
    ]

    prompt = f"""Given the following document, rank these schemas by how well
    they match. Return a JSON array of schema_ids ordered by relevance,
    with confidence scores.

    Schemas: {json.dumps(schema_summaries)}

    Document (first 2 pages):
    {doc_content[:4000]}
    """

    response = await llm_client.complete(
        prompt=prompt,
        model="fast_model",  # GPT-4o-mini or Claude Haiku
        temperature=0.0
    )

    ranked = parse_ranking_response(response)
    return ranked[:TOP_N_FOR_EXTRACTION]  # Take top 3
```

**Phase 2: Full Extraction (on top-ranked schemas)**

Run full extraction on the top 1-3 schemas in parallel.

### 6.3 Extraction Execution

```python
async def extract_with_schema(doc_content, schema_version) -> ExtractionAttempt:
    """Run full extraction against a single schema."""

    prompt = build_extraction_prompt(
        doc_content=doc_content,
        schema_body=schema_version.body
    )

    response = await llm_client.extract(
        prompt=prompt,
        model=select_extraction_model(schema_version.field_count),
        temperature=0.0,
        response_format={"type": "json_object"}
    )

    extracted_data = json.loads(response.content)

    # Validate against schema
    validation_errors = validate_against_schema(extracted_data, schema_version.body)

    # Compute confidence metrics
    confidence = compute_extraction_confidence(
        extracted_data=extracted_data,
        schema=schema_version.body,
        validation_errors=validation_errors,
        llm_logprobs=response.logprobs  # if available
    )

    field_fill_rate = compute_fill_rate(extracted_data, schema_version.body)

    return ExtractionAttempt(
        schema_id=schema_version.schema_id,
        schema_version_id=schema_version.id,
        data=extracted_data,
        confidence=confidence,
        field_fill_rate=field_fill_rate,
        validation_errors=validation_errors,
        tokens_used=response.usage.total_tokens
    )
```

### 6.4 Multi-Schema Extraction and Result Ranking

```python
async def run_extraction_pipeline(job, ce_result) -> ExtractionPipelineResult:
    candidates = ce_result.candidates

    if len(candidates) == 0:
        return handle_no_candidates(job)

    if len(candidates) == 1:
        # Single candidate -- extract directly
        attempt = await extract_with_schema(doc_content, candidates[0])
        return build_result(job, [attempt])

    # Multiple candidates
    if len(candidates) > 3:
        # Phase 1: Rank
        ranked = await rank_schemas(doc_content, candidates)
        top_candidates = ranked[:3]
    else:
        top_candidates = candidates

    # Phase 2: Extract in parallel
    attempts = await asyncio.gather(*[
        extract_with_schema(doc_content, c)
        for c in top_candidates
    ])

    # Rank results
    attempts.sort(key=lambda a: (
        -a.confidence,            # Higher confidence first
        -a.field_fill_rate,       # Higher fill rate
        len(a.validation_errors)  # Fewer errors
    ))

    # Assign ranks
    for i, attempt in enumerate(attempts):
        attempt.rank = i + 1
        attempt.was_selected = (i == 0)

    return build_result(job, attempts)
```

### 6.5 Confidence Computation

Extraction confidence is a composite score:

```python
def compute_extraction_confidence(extracted_data, schema, validation_errors, llm_logprobs):
    scores = {}

    # 1. Schema validation score (0-1)
    # Full compliance = 1.0, each error deducts
    total_fields = count_all_fields(schema)
    error_count = len(validation_errors)
    scores["validation"] = max(0, 1.0 - (error_count / max(total_fields, 1)))

    # 2. Required field coverage (0-1)
    required_fields = get_required_fields(schema)
    filled_required = count_filled_required(extracted_data, required_fields)
    scores["required_coverage"] = filled_required / max(len(required_fields), 1)

    # 3. Field fill rate (0-1)
    all_fields = get_all_fields(schema)
    filled = count_filled(extracted_data, all_fields)
    scores["fill_rate"] = filled / max(len(all_fields), 1)

    # 4. LLM self-reported confidence via logprobs (0-1) -- if available
    if llm_logprobs:
        scores["logprob"] = compute_avg_token_confidence(llm_logprobs)

    # Weighted composite
    weights = {
        "validation": 0.30,
        "required_coverage": 0.35,
        "fill_rate": 0.20,
        "logprob": 0.15
    }

    total_weight = sum(weights[k] for k in scores)
    confidence = sum(scores[k] * weights[k] for k in scores) / total_weight

    return round(confidence, 4)
```

### 6.6 Schema Suggestion Flow

When extraction confidence is below the `suggest_threshold`:

```python
async def suggest_schema(job, doc_content, best_attempt):
    """LLM generates a candidate schema for an unrecognized document type."""

    prompt = f"""Analyze this document and generate a JSON Schema that would
    capture all the structured data present in it. Include:
    - Appropriate field names (snake_case)
    - Types with format hints
    - Required vs optional fields
    - A descriptive title and description

    Also suggest:
    - A category path (e.g., "financial.invoice.utility_invoice")
    - A slug (e.g., "invoice.utility.electricity")

    Document:
    {doc_content}
    """

    response = await llm_client.complete(prompt=prompt, model="capable_model")

    suggested = parse_suggestion(response)

    # Check for near-duplicates using similarity index
    similar = find_similar_schemas(
        field_signature=compute_field_signature(suggested.body),
        embedding=compute_embedding(suggested),
        threshold=0.85
    )

    if similar:
        # Near-duplicate found -- use existing schema instead
        return SuggestionResult(
            action="use_existing",
            existing_schema_id=similar[0].id,
            similarity_score=similar[0].score
        )

    # Create new community schema
    new_schema = create_schema(
        slug=suggested.slug,
        name=suggested.name,
        body=suggested.body,
        schema_type="document",
        maturity="community",
        origin="suggested",
        suggested_by_job_id=job.id,
        category_tags=suggested.category_path
    )

    return SuggestionResult(
        action="created_community",
        new_schema_id=new_schema.id
    )
```

---

## 7. Schema Lifecycle

### 7.1 Maturity States

```
                  +-----------+
                  | community |  <-- New schemas enter here
                  +-----+-----+
                        |
            Validation  |  (automated + manual review)
            Gate        |
                        v
                  +-----------+
                  |  staging  |  <-- Testing against real documents
                  +-----+-----+
                        |
            Promotion   |  (usage metrics + approval)
            Gate        |
                        v
                  +------------+
                  | production |  <-- Trusted, used by default
                  +-----+------+
                        |
            Deprecation |  (replaced or obsolete)
                        v
                  +------------+
                  | deprecated |  <-- Retained but not used for new extractions
                  +------------+
```

### 7.2 Transition Gates

**Community -> Staging**

Requirements:
- Schema has been used in at least N extraction jobs (configurable, default: 10)
- Average extraction confidence >= 0.80
- Average field fill rate >= 0.70
- No critical JSON Schema validation issues
- Manual review flag: at least one human has reviewed the schema body
- Category tags have been verified

This transition can be triggered automatically when metrics are met, or manually by an admin.

**Staging -> Production**

Requirements:
- Schema has been in staging for at least M days (configurable, default: 7)
- At least P extraction jobs during staging period (default: 50)
- Average extraction confidence >= 0.90
- Average field fill rate >= 0.80
- Zero regression: no jobs where confidence dropped below 0.50
- Admin approval required (cannot be fully automated)

**Production -> Deprecated**

Triggers:
- Manual deprecation by admin (with optional replacement schema ID)
- Zero usage in the last Q days (configurable, default: 180) triggers a deprecation suggestion (not automatic)
- Replacement schema promoted to production

Effects:
- Deprecated schemas are excluded from CE candidate results by default
- Existing schema group memberships are flagged for review
- Orgs using this schema are notified (if webhook configured)

### 7.3 Lifecycle Tracking

Every transition is recorded in `SchemaMaturityRecord` with full evidence:

```json
{
  "id": "...",
  "schema_id": "c3d4e5f6-...",
  "from_maturity": "community",
  "to_maturity": "staging",
  "reason": "Automated promotion: metrics threshold met",
  "evidence": {
    "total_jobs": 47,
    "avg_confidence": 0.87,
    "avg_fill_rate": 0.82,
    "job_ids_sample": ["j1", "j2", "j3"],
    "reviewed_by": "analyst@docdigitizer.com",
    "review_date": "2026-03-20T14:00:00Z"
  },
  "approved_by": "system:auto_promotion",
  "created_at": "2026-03-20T14:01:00Z"
}
```

---

## 8. Versioning Strategy

### 8.1 Schema Versioning

**Principle**: Schemas are versioned with monotonically increasing integers. Versions are immutable once created. The `Schema.current_version_id` pointer is updated atomically.

**Version resolution order**:
1. If a `SchemaGroupMember` has `pinned_version_id` set, use that version.
2. Otherwise, use `Schema.current_version_id` (latest active).

**When to create a new version**:
- Any change to the JSON Schema body (fields added, removed, type changes, description changes).
- The system computes `body_hash` and rejects a new version if the hash matches the current version (no-op guard).

**Backward compatibility**:
- Adding optional fields = minor (no version bump concern).
- Removing fields or changing types = breaking. The system flags this in `change_summary` and can optionally notify affected orgs.

```
Schema "invoice.utility.electricity"
  ├── v1 (created 2026-01-15) -- 12 fields
  ├── v2 (created 2026-02-10) -- 14 fields (added meter_number, consumption_kwh)
  └── v3 (created 2026-03-05) -- 16 fields (added billing_period_start/end)  <-- current
```

### 8.2 Category Tree Versioning

The category tree is versioned as a whole unit via `CategoryTreeVersion`. This is necessary because:
- Adding a new category at level 2 changes the CE schema for level 1.
- Removing a category could orphan schemas.
- CE schemas reference specific tree structures.

**Workflow for tree changes**:
1. Create a new `CategoryTreeVersion` in draft state.
2. Clone all nodes from the active version into the new version (or modify as needed).
3. Update CE schemas for any nodes whose children changed.
4. Publish the new version (set `is_active = true` on new, `false` on old).
5. Old version's nodes remain in the database for audit and replay.

**In-flight job handling**: Jobs that started CE under the old tree version continue with that version. The `ExtractionTrace.ce_steps` records which tree version was active at each step.

### 8.3 CE Schema Versioning

CE schemas are versioned just like document schemas (they share the same `Schema` + `SchemaVersion` tables with `schema_type = 'category'`). When the category tree changes and a node's children are modified, a new CE schema version is created for that node.

### 8.4 Version Garbage Collection

No versions are ever deleted. Storage cost for schema JSON is negligible (a million schema versions at 10KB each = 10 GB). Indexes and queries always filter by `is_active` or `current_version_id` to maintain performance.

---

## 9. Scaling Considerations

### 9.1 Schema Count Growth

**Target**: Support 1M+ schemas in the system without degradation.

| Concern | Mitigation |
|---------|------------|
| **Schema storage** | JSONB in PostgreSQL. At 10KB avg per version, 1M versions = 10 GB. Well within PostgreSQL capacity. |
| **Schema lookup by category** | `SchemaTag` table with index on `category_node_id`. At 1M schemas with avg 2 tags each = 2M rows. B-tree index handles this trivially. |
| **CE schema loading** | CE schemas are small (~500 bytes each). The active tree is cached entirely in Redis. At 10,000 category nodes, this is ~5 MB in cache. |
| **Schema group member queries** | Indexed join table. Even groups with 10,000 members are fast with proper indexes. |
| **Full-text / similarity search** | `SchemaSimilarityIndex` with pgvector. ANN search with IVFFlat index scales to millions of vectors. |

### 9.2 Extraction Throughput

**Target**: 100+ concurrent extraction jobs.

| Component | Scaling Strategy |
|-----------|-----------------|
| **API Gateway** | Stateless. Horizontally scaled behind a load balancer. |
| **Job Queue** | Use a durable message queue (e.g., RabbitMQ, AWS SQS, or Redis Streams). Decouple submission from processing. |
| **CE Workers** | Stateless workers consuming from the queue. Scale horizontally. Each CE call is independent. |
| **Extraction Workers** | Same as CE workers. Can be a separate pool with different scaling characteristics (extraction is heavier). |
| **LLM Rate Limits** | The primary bottleneck. Mitigations: (a) use cheaper models for CE, (b) cache CE results for identical documents, (c) maintain accounts with multiple LLM providers, (d) implement token-aware rate limiting in the worker pool. |
| **Database** | PostgreSQL with read replicas for query-heavy operations (schema lookup, job listing). Write volume is modest (one job record + one trace record per extraction). |
| **Cache** | Redis cluster for: (a) active category tree, (b) hot schemas, (c) org settings, (d) CE result cache keyed by document_hash + tree_version. |

### 9.3 Category Tree Scale

With a 4-level tree and 20 children per level:
- Level 0: 1 root
- Level 1: 20 nodes
- Level 2: 400 nodes
- Level 3: 8,000 nodes
- Level 4: 160,000 leaf nodes

This supports 160,000 distinct document categories, which is far beyond practical needs. Even at level 3 (8,000 categories), the system can address essentially any real-world document taxonomy.

CE cost per document (worst case, 4 levels): 4 LLM calls using a fast model. At ~300ms per call = ~1.2s for CE. Typically 2-3 levels suffice, bringing this to 0.6-0.9s.

### 9.4 Multi-Tenancy

The system is single-database, multi-tenant:
- All queries include `org_id` filtering where applicable.
- Schema groups provide tenant isolation for schemas.
- The category tree and production schemas are shared global assets.
- Row-Level Security (RLS) in PostgreSQL can enforce tenant boundaries at the database level.

### 9.5 Caching Strategy

| Cache Layer | Content | TTL | Invalidation |
|-------------|---------|-----|--------------|
| **L1: In-process** | Active category tree, org settings | 60s | Timer-based refresh |
| **L2: Redis** | Schema bodies, CE schemas, group memberships | 5 min | Event-driven (on schema update, publish invalidation message) |
| **L3: DB query cache** | Prepared statement plans, connection pooling | N/A | PostgreSQL managed |
| **L4: Result cache** | CE results by document_hash + tree_version | 24h | TTL-based. Invalidated when tree version changes. |

---

## 10. Implementation Roadmap

### Phase 1: Foundation (Weeks 1-4)

**Goal**: Core data model, schema CRUD, basic CE pipeline working end-to-end.

**Deliverables**:
- [ ] Database schema: Organization, SchemaGroup, Schema, SchemaVersion, OrgSchemaGroupLink, SchemaGroupMember
- [ ] Schema Registry API: full CRUD for schemas, versions, groups
- [ ] Schema validation (JSON Schema compliance check on ingest)
- [ ] Basic category tree: 2 levels, hardcoded in seed data
- [ ] CategoryNode, CategoryTreeVersion, CE schema tables
- [ ] CE pipeline: basic 2-level iteration with single-branch descent
- [ ] Simple extraction pipeline: single schema extraction (no ranking)
- [ ] ExtractionJob tracking (create, update status, store result)
- [ ] Integration with one LLM provider (e.g., OpenAI GPT-4o-mini for CE, GPT-4o for extraction)

**Test Milestone**: Submit a document with a known schema group. CE identifies the category. Extraction returns structured data.

### Phase 2: Intelligence (Weeks 5-8)

**Goal**: Multi-schema extraction, confidence scoring, schema suggestion, organized scenario optimization.

**Deliverables**:
- [ ] ExtractionTrace and ExtractionResult entities
- [ ] Multi-schema extraction: rank top 3, extract in parallel, compare results
- [ ] Confidence scoring composite (validation + fill rate + required coverage)
- [ ] Schema ranking step (pre-extraction schema selection when candidates > 3)
- [ ] Schema suggestion flow: detect low confidence, generate candidate schema, dedup check
- [ ] SchemaSimilarityIndex with pgvector for dedup
- [ ] SchemaMaturityRecord and lifecycle state machine
- [ ] Organized scenario: compute_ce_entry_level optimization
- [ ] CE soft-threshold multi-branch exploration

**Test Milestone**: Chaos scenario works end-to-end. Unknown document triggers schema suggestion. Organized scenario short-circuits CE correctly.

### Phase 3: Production Hardening (Weeks 9-12)

**Goal**: Caching, scaling, monitoring, admin tools.

**Deliverables**:
- [ ] Redis caching layer: category tree, hot schemas, CE result cache
- [ ] Job queue with durable message broker (RabbitMQ or SQS)
- [ ] Worker pool management: separate CE and extraction worker pools
- [ ] Rate limiting and token budgeting for LLM calls
- [ ] Webhook/callback delivery for async job completion
- [ ] Admin UI: schema browser, category tree editor, lifecycle approval workflow
- [ ] Telemetry: per-org usage dashboards, CE accuracy metrics, extraction quality metrics
- [ ] Category tree versioning workflow (draft -> publish)
- [ ] Document type hint resolution
- [ ] Automated community -> staging promotion based on metrics

**Test Milestone**: System handles 50 concurrent extraction jobs. Cache hit rate > 80% for repeat documents. Admin can manage schemas and category tree through UI.

### Phase 4: Scale & Optimize (Weeks 13-16)

**Goal**: Handle production volume, optimize costs, multi-model support.

**Deliverables**:
- [ ] Multi-LLM provider support (OpenAI, Anthropic, local models)
- [ ] Model routing: select model based on document complexity, org tier, cost budget
- [ ] PostgreSQL read replicas for query scaling
- [ ] Batch extraction API (submit N documents, get N results)
- [ ] Schema analytics: per-schema accuracy trends, field-level extraction quality
- [ ] Automated staging -> production promotion (with admin approval gate)
- [ ] Schema deprecation workflow with org notification
- [ ] Performance optimization: prompt compression, schema minification for token reduction
- [ ] Load testing at 1M schemas, 1000 concurrent jobs
- [ ] Row-Level Security for multi-tenant data isolation

**Test Milestone**: System operates at target scale. Average CE + extraction time < 5s. LLM cost per document < $0.05 for standard documents.

### Phase 5: Advanced Features (Weeks 17+)

**Goal**: Advanced CE, learning, and ecosystem features.

**Deliverables**:
- [ ] CE learning: track which categories are most common per org and bias CE accordingly
- [ ] Schema evolution suggestions: detect when extraction consistently populates non-schema fields, suggest schema updates
- [ ] Document clustering: group similar documents that don't match any schema to surface bulk schema creation opportunities
- [ ] Schema marketplace: orgs can publish schemas for other orgs to use
- [ ] A/B testing for CE schemas: run two CE schema versions simultaneously and compare accuracy
- [ ] Federated schema groups: orgs can subscribe to curated schema collections
- [ ] Offline/batch CE model fine-tuning on accumulated classification data

---

## Appendix A: Database DDL Summary

```sql
-- Core tables (PostgreSQL 15+, with pgvector extension)

CREATE EXTENSION IF NOT EXISTS "pgvector";
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

-- Organization
CREATE TABLE organization (
    id              UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    external_id     VARCHAR(255) NOT NULL UNIQUE,
    name            VARCHAR(500) NOT NULL,
    default_schema_group_id UUID,
    settings        JSONB NOT NULL DEFAULT '{}',
    is_active       BOOLEAN NOT NULL DEFAULT true,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- Schema Group
CREATE TABLE schema_group (
    id              UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    name            VARCHAR(500) NOT NULL,
    description     TEXT,
    owner_org_id    UUID REFERENCES organization(id),
    is_system       BOOLEAN NOT NULL DEFAULT false,
    metadata        JSONB NOT NULL DEFAULT '{}',
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- Org <-> SchemaGroup link
CREATE TABLE org_schema_group_link (
    id              UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    org_id          UUID NOT NULL REFERENCES organization(id),
    schema_group_id UUID NOT NULL REFERENCES schema_group(id),
    access_level    VARCHAR(50) NOT NULL DEFAULT 'read',
    is_default      BOOLEAN NOT NULL DEFAULT false,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    UNIQUE (org_id, schema_group_id)
);

-- Schema
CREATE TABLE schema (
    id                  UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    slug                VARCHAR(255) NOT NULL UNIQUE,
    name                VARCHAR(500) NOT NULL,
    description         TEXT,
    schema_type         VARCHAR(50) NOT NULL DEFAULT 'document',
    maturity            VARCHAR(50) NOT NULL DEFAULT 'community',
    current_version_id  UUID,
    origin              VARCHAR(50) NOT NULL DEFAULT 'manual',
    suggested_by_job_id UUID,
    created_by          VARCHAR(255),
    created_at          TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at          TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- Schema Version
CREATE TABLE schema_version (
    id                  UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    schema_id           UUID NOT NULL REFERENCES schema(id),
    version_number      INTEGER NOT NULL,
    body                JSONB NOT NULL,
    body_hash           VARCHAR(64) NOT NULL,
    change_summary      TEXT,
    field_count         INTEGER NOT NULL,
    estimated_token_cost INTEGER NOT NULL,
    is_active           BOOLEAN NOT NULL DEFAULT true,
    created_by          VARCHAR(255),
    created_at          TIMESTAMPTZ NOT NULL DEFAULT now(),
    UNIQUE (schema_id, version_number)
);

-- Add FK from schema to schema_version (deferred to avoid circular dependency)
ALTER TABLE schema
    ADD CONSTRAINT fk_schema_current_version
    FOREIGN KEY (current_version_id) REFERENCES schema_version(id);

ALTER TABLE schema
    ADD CONSTRAINT fk_schema_suggested_by_job
    FOREIGN KEY (suggested_by_job_id) REFERENCES extraction_job(id) DEFERRABLE;

-- Schema Group Member
CREATE TABLE schema_group_member (
    id                  UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    schema_group_id     UUID NOT NULL REFERENCES schema_group(id),
    schema_id           UUID NOT NULL REFERENCES schema(id),
    pinned_version_id   UUID REFERENCES schema_version(id),
    priority            INTEGER NOT NULL DEFAULT 0,
    added_at            TIMESTAMPTZ NOT NULL DEFAULT now(),
    UNIQUE (schema_group_id, schema_id)
);

-- Category Tree Version
CREATE TABLE category_tree_version (
    id              UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    version_number  INTEGER NOT NULL UNIQUE,
    description     TEXT,
    is_active       BOOLEAN NOT NULL DEFAULT false,
    node_count      INTEGER NOT NULL,
    max_depth       INTEGER NOT NULL,
    published_at    TIMESTAMPTZ,
    created_by      VARCHAR(255),
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- Category Node
CREATE TABLE category_node (
    id              UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    code            VARCHAR(100) NOT NULL UNIQUE,
    name            VARCHAR(500) NOT NULL,
    description     TEXT,
    parent_id       UUID REFERENCES category_node(id),
    level           INTEGER NOT NULL,
    path            VARCHAR(2000) NOT NULL,
    tree_version_id UUID NOT NULL REFERENCES category_tree_version(id),
    display_order   INTEGER NOT NULL DEFAULT 0,
    is_leaf         BOOLEAN NOT NULL DEFAULT false,
    schema_count    INTEGER NOT NULL DEFAULT 0,
    ce_schema_id    UUID REFERENCES schema(id),
    metadata        JSONB NOT NULL DEFAULT '{}',
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- Schema Tag (links schemas to categories)
CREATE TABLE schema_tag (
    id                  UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    schema_id           UUID NOT NULL REFERENCES schema(id),
    category_node_id    UUID NOT NULL REFERENCES category_node(id),
    is_primary          BOOLEAN NOT NULL DEFAULT false,
    confidence          DECIMAL(3,2) NOT NULL DEFAULT 1.00,
    tagged_by           VARCHAR(50) NOT NULL DEFAULT 'manual',
    created_at          TIMESTAMPTZ NOT NULL DEFAULT now(),
    UNIQUE (schema_id, category_node_id)
);

-- Extraction Job
CREATE TABLE extraction_job (
    id                      UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    org_id                  UUID NOT NULL REFERENCES organization(id),
    schema_group_id         UUID REFERENCES schema_group(id),
    document_uri            VARCHAR(2000) NOT NULL,
    document_hash           VARCHAR(64) NOT NULL,
    document_type_hint      VARCHAR(255),
    status                  VARCHAR(50) NOT NULL DEFAULT 'pending',
    scenario                VARCHAR(50) NOT NULL,
    priority                INTEGER NOT NULL DEFAULT 0,
    ce_entry_level          INTEGER,
    callback_url            VARCHAR(2000),
    result_schema_id        UUID REFERENCES schema(id),
    result_schema_version_id UUID REFERENCES schema_version(id),
    result_confidence       DECIMAL(5,4),
    result_payload          JSONB,
    error_message           TEXT,
    processing_time_ms      INTEGER,
    llm_calls_count         INTEGER NOT NULL DEFAULT 0,
    llm_tokens_used         INTEGER NOT NULL DEFAULT 0,
    created_at              TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at              TIMESTAMPTZ NOT NULL DEFAULT now(),
    completed_at            TIMESTAMPTZ
);

-- Extraction Trace
CREATE TABLE extraction_trace (
    id                  UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    job_id              UUID NOT NULL UNIQUE REFERENCES extraction_job(id),
    ce_steps            JSONB NOT NULL DEFAULT '[]',
    candidate_schemas   JSONB NOT NULL DEFAULT '[]',
    extraction_attempts JSONB NOT NULL DEFAULT '[]',
    schema_suggestion   JSONB,
    created_at          TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- Extraction Result
CREATE TABLE extraction_result (
    id                  UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    job_id              UUID NOT NULL REFERENCES extraction_job(id),
    schema_id           UUID NOT NULL REFERENCES schema(id),
    schema_version_id   UUID NOT NULL REFERENCES schema_version(id),
    confidence          DECIMAL(5,4) NOT NULL,
    field_fill_rate     DECIMAL(5,4) NOT NULL,
    payload             JSONB NOT NULL,
    validation_errors   JSONB NOT NULL DEFAULT '[]',
    was_selected        BOOLEAN NOT NULL DEFAULT false,
    rank                INTEGER NOT NULL,
    created_at          TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- Schema Maturity Record
CREATE TABLE schema_maturity_record (
    id              UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    schema_id       UUID NOT NULL REFERENCES schema(id),
    from_maturity   VARCHAR(50),
    to_maturity     VARCHAR(50) NOT NULL,
    reason          TEXT,
    evidence        JSONB NOT NULL DEFAULT '{}',
    approved_by     VARCHAR(255),
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- Schema Similarity Index
CREATE TABLE schema_similarity_index (
    id                  UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    schema_id           UUID NOT NULL UNIQUE REFERENCES schema(id),
    schema_version_id   UUID NOT NULL REFERENCES schema_version(id),
    field_signature     VARCHAR(500) NOT NULL,
    embedding           VECTOR(768) NOT NULL,
    updated_at          TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- Cache Entry
CREATE TABLE cache_entry (
    id              UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    cache_key       VARCHAR(200) NOT NULL UNIQUE,
    job_id          UUID NOT NULL REFERENCES extraction_job(id),
    result_payload  JSONB NOT NULL,
    expires_at      TIMESTAMPTZ NOT NULL,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- Key indexes (beyond those implicit in constraints)
CREATE INDEX idx_org_active ON organization(is_active) WHERE is_active = true;
CREATE INDEX idx_sg_owner ON schema_group(owner_org_id);
CREATE INDEX idx_osgl_org ON org_schema_group_link(org_id);
CREATE INDEX idx_osgl_group ON org_schema_group_link(schema_group_id);
CREATE INDEX idx_schema_type ON schema(schema_type);
CREATE INDEX idx_schema_maturity ON schema(maturity);
CREATE INDEX idx_schema_type_maturity ON schema(schema_type, maturity);
CREATE INDEX idx_sv_schema_active ON schema_version(schema_id, is_active) WHERE is_active = true;
CREATE INDEX idx_sv_body_hash ON schema_version(body_hash);
CREATE INDEX idx_sgm_group ON schema_group_member(schema_group_id);
CREATE INDEX idx_sgm_schema ON schema_group_member(schema_id);
CREATE INDEX idx_cn_parent ON category_node(parent_id);
CREATE INDEX idx_cn_path ON category_node(path);
CREATE INDEX idx_cn_tree_version ON category_node(tree_version_id);
CREATE INDEX idx_cn_level ON category_node(level);
CREATE INDEX idx_cn_ce_schema ON category_node(ce_schema_id);
CREATE INDEX idx_st_category ON schema_tag(category_node_id);
CREATE INDEX idx_st_schema ON schema_tag(schema_id);
CREATE INDEX idx_ej_org ON extraction_job(org_id);
CREATE INDEX idx_ej_status ON extraction_job(status);
CREATE INDEX idx_ej_org_status ON extraction_job(org_id, status);
CREATE INDEX idx_ej_document_hash ON extraction_job(document_hash);
CREATE INDEX idx_ej_created ON extraction_job(created_at DESC);
CREATE INDEX idx_ej_result_schema ON extraction_job(result_schema_id);
CREATE INDEX idx_er_job ON extraction_result(job_id);
CREATE INDEX idx_er_schema ON extraction_result(schema_id);
CREATE INDEX idx_er_job_rank ON extraction_result(job_id, rank);
CREATE INDEX idx_smr_schema ON schema_maturity_record(schema_id);
CREATE INDEX idx_smr_schema_time ON schema_maturity_record(schema_id, created_at);
CREATE INDEX idx_ssi_field_sig ON schema_similarity_index(field_signature);
CREATE INDEX idx_cache_expires ON cache_entry(expires_at);

-- pgvector ANN index
CREATE INDEX idx_ssi_embedding ON schema_similarity_index
    USING ivfflat (embedding vector_cosine_ops) WITH (lists = 100);
```

---

## Appendix B: Key Decision Log

| # | Decision | Rationale | Alternatives Considered |
|---|----------|-----------|------------------------|
| 1 | PostgreSQL + JSONB for schema storage | Schemas are small (< 50KB), relational queries are needed for tags/groups, JSONB enables flexible schema bodies without a separate document store | MongoDB (rejected: loses relational integrity), S3 + DynamoDB (rejected: over-engineering) |
| 2 | Materialized path (`financial.invoice.utility`) for category tree | Enables prefix queries (`path LIKE 'financial.invoice.%'`), human-readable, simple to implement | Nested sets (rejected: complex updates), closure table (rejected: storage overhead, overkill for 4-level tree) |
| 3 | CE schemas as regular schemas with `schema_type = 'category'` | Unified versioning, storage, and management. No special infrastructure needed. | Separate CE schema table (rejected: duplication) |
| 4 | Tree versioned as a whole unit | Category changes ripple across CE schemas. Whole-tree versioning ensures consistency. | Per-node versioning (rejected: consistency nightmare), no versioning (rejected: no rollback capability) |
| 5 | Soft thresholds with multi-branch exploration | Handles genuine ambiguity (e.g., a document that could be an invoice or a receipt) without sacrificing precision | Always single-branch (rejected: misses edge cases), always multi-branch (rejected: wasteful) |
| 6 | pgvector for schema similarity | Native PostgreSQL extension, no additional infrastructure. Good enough for schema dedup at scale. | Pinecone/Weaviate (rejected: external dependency), MinHash (rejected: less accurate for semantic similarity) |
| 7 | Composite confidence score | Single number is actionable for thresholds and ranking. Components are logged for diagnostics. | LLM-only confidence (rejected: unreliable self-assessment), binary match/no-match (rejected: too coarse) |
| 8 | Async job model with polling/webhook | Extraction can take 2-10s. Synchronous API would block and timeout. Async is production-standard for IDP. | Synchronous only (rejected: timeouts), WebSocket (rejected: complexity for most integrations) |

---

## Appendix C: Glossary

| Term | Definition |
|------|------------|
| **CE** | Category Extraction -- the iterative document classification process that narrows the schema candidate set |
| **Schema** | A JSON Schema document defining the structure of data to extract from a document type |
| **CE Schema** | A special schema with a single enum field used to classify a document at one level of the category hierarchy |
| **Schema Group** | A named collection of schemas, typically representing the document types an organization processes |
| **Maturity** | The lifecycle stage of a schema: community, staging, production, or deprecated |
| **Category Node** | A node in the hierarchical category tree (e.g., "financial" at level 1, "invoice" at level 2) |
| **Category Path** | The dot-separated full path from root to a category node (e.g., `financial.invoice.utility_invoice`) |
| **Field Fill Rate** | The fraction of a schema's fields that were populated during extraction |
| **Slug** | A human-readable unique identifier for a schema (e.g., `invoice.utility.electricity`) |
| **Pinned Version** | When a schema group member locks to a specific schema version rather than following "latest" |

---

*End of Report 2*
