# Memory Graph & Versioning Spec

> Draft — last updated {{DATE}}

## Purpose
Provide an append-only, provenance-rich data model so the TimeBack agent can recall every fact/event/version without "starting over."  This spec extends `analytics-spec.md`.

## Core Principles
1. **Append-only** – no hard deletes; every change is a new version row.
2. **Provenance first** – every version stores `source_id`, `checksum`, `uploaded_by`, `uploaded_at`.
3. **Latest-view simplicity** – materialized views expose “current state” so front-end / agent queries stay simple.

---

## Tables

### 1. `entity_version`
Tracks every atomic payload (student, assessment, program, etc.).

| Column | Type | Description |
|--------|------|-------------|
| `entity_version_id` | bigint pk | Surrogate key |
| `entity_type` | text | e.g. `student`, `assessment`, `program_assignment` |
| `entity_pk` | text | Business key (e.g. `student_id` or `student_id|subject|term`) |
| `json_payload` | jsonb | Full row snapshot as ingested |
| `checksum` | text | SHA-256 of raw file record |
| `source_file` | text | Original filename |
| `uploaded_by` | text | User / system token |
| `uploaded_at` | timestamptz | Upload time |
| `effective_at` | date | Date the record is valid (e.g. test_date) |
| `superseded` | bool default false | Flag set when a newer version supersedes this one |

#### Indexes
* `(entity_type, entity_pk, effective_at DESC)` – fast latest lookup.
* GIN on `json_payload` – ad-hoc JSON queries.

### 2. `file_ingest_log`
| Column | Type | Notes |
| `file_id` pk | uuid |
| `filename` | text |
| `checksum` | text |
| `row_count` | int |
| `ingest_started_at` | timestamptz |
| `ingest_completed_at` | timestamptz |
| `status` | enum (`pending`,`success`,`error`) |
| `error_message` | text |

---

## Merge Logic
1. Loader computes row checksum.
2. If `(entity_type, entity_pk, checksum)` already exists → **skip** (idempotent).
3. Otherwise insert new version row, mark previous latest row (`superseded=true`).
4. A nightly job materializes `*_current_v` views.

Pseudo-SQL:
```sql
-- inside loader tx
INSERT INTO entity_version (...)
ON CONFLICT (entity_type, entity_pk, checksum) DO NOTHING;

-- supersede previous
UPDATE entity_version
SET superseded = true
WHERE entity_type = :type
  AND entity_pk   = :pk
  AND entity_version_id <> currval('entity_version_entity_version_id_seq')
  AND superseded = false;
```

---

## Materialized Views
### `student_current_v`
```sql
CREATE MATERIALIZED VIEW student_current_v AS
SELECT (json_payload->>'student_id')::text  AS student_id,
       json_payload->>'first_name'          AS first_name,
       json_payload->>'last_name'           AS last_name,
       (json_payload->>'grade')::int        AS grade,
       uploaded_at
FROM   entity_version ev
WHERE  entity_type = 'student'
  AND  superseded = false;
```
Refresh nightly or after bulk ingest.

### `assessment_current_v`
Similar pattern keyed by `student_id|subject|term`.

---

## API Impact
* `/api/parser/parse` remains unchanged; loader writes to `entity_version` instead of flat table.
* New endpoint `/memory/timeline/:entity_type/:entity_pk` returns full version history (paginated).

---

## Open Questions
1. Do we need per-field diff storage to save space?
2. When volume grows, move cold versions to object storage + parquet/iceberg?

---

_This spec supersedes any previous versioning notes in `analytics-spec.md`.  Implementation tickets are tracked under P0 in `ax-priority-plan.md`._ 