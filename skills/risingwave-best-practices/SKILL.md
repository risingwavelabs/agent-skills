---
name: risingwave-best-practices
description: Use when designing RisingWave schemas, writing materialized views, configuring sources or sinks, optimizing streaming SQL performance, troubleshooting slow backfills, setting up CDC, designing time-windowed aggregations, or reviewing a RisingWave streaming pipeline
license: Apache-2.0
metadata:
  author: RisingWave Labs
  version: "0.1.0"
  organization: RisingWave Labs
  date: April 2026
  abstract: >
    10 best-practice rules for RisingWave covering schema design, materialized view patterns,
    CDC setup, time-windowed aggregations, sink configuration, and performance optimization.
    Rules are organized by category and impact level (CRITICAL > HIGH > MEDIUM).
---

# RisingWave Best Practices

10 rules covering schema design, streaming SQL patterns, sink configuration, and performance.

## How to Apply

1. Read the relevant rule files listed below
2. Apply rules to the design or code under review
3. Explain violations with impact and fix

## Review Procedures

### Schema Review (new source, table, or pipeline design)
Read in order:
- [ ] [schema-source-vs-table.md](rules/schema-source-vs-table.md) — SOURCE vs TABLE selection
- [ ] [schema-append-only.md](rules/schema-append-only.md) — APPEND ONLY for immutable streams
- [ ] [schema-watermark.md](rules/schema-watermark.md) — Watermark placement

### Materialized View Review
Read in order:
- [ ] [mv-emit-on-window-close.md](rules/mv-emit-on-window-close.md) — EOWC for windowed queries
- [ ] [mv-background-ddl.md](rules/mv-background-ddl.md) — Non-blocking DDL for large tables

### Streaming SQL Review
Read in order:
- [ ] [stream-tumble-windows.md](rules/stream-tumble-windows.md) — Time window functions
- [ ] [stream-cdc-pattern.md](rules/stream-cdc-pattern.md) — Two-step CDC setup

### Sink Review
Read in order:
- [ ] [sink-snapshot-false.md](rules/sink-snapshot-false.md) — Prevent backfill duplicates
- [ ] [sink-force-compaction.md](rules/sink-force-compaction.md) — Reduce output volume

### Performance Review
Read in order:
- [ ] [perf-shared-source.md](rules/perf-shared-source.md) — Shared Kafka source
- [ ] [perf-indexes-on-mv.md](rules/perf-indexes-on-mv.md) — Indexes on materialized views

## Output Format

```markdown
## Rules Checked
- [x] schema-source-vs-table
- [x] mv-emit-on-window-close
...

## Findings

### Violations
- **[CRITICAL] schema-source-vs-table**: CDC source uses `CREATE SOURCE` instead of `CREATE TABLE`
  Fix: Use `CREATE TABLE orders FROM pg_cdc TABLE 'public.orders'`

### Compliant
- mv-background-ddl: BACKGROUND_DDL is set

## Recommendations
...
```

## Rule Categories

| Priority | Category | Impact | Prefix | Rules |
|----------|----------|--------|--------|-------|
| 1 | Schema Design | CRITICAL | `schema-` | 3 |
| 2 | Materialized Views | CRITICAL/HIGH | `mv-` | 2 |
| 3 | Streaming SQL | CRITICAL/HIGH | `stream-` | 2 |
| 4 | Sink Configuration | HIGH/MEDIUM | `sink-` | 2 |
| 5 | Performance | HIGH/MEDIUM | `perf-` | 2 |

## Quick Reference

**Schema:**
- `schema-source-vs-table` — CDC requires TABLE; SOURCE is for append-only streams
- `schema-append-only` — APPEND ONLY enables optimizations; use for immutable event data
- `schema-watermark` — Define watermarks at ingestion point (source/table), not in MVs

**Materialized Views:**
- `mv-emit-on-window-close` — Use EOWC for final window results; default emits partial updates
- `mv-background-ddl` — SET BACKGROUND_DDL=true before creating MVs over large tables

**Streaming SQL:**
- `stream-tumble-windows` — Use TUMBLE/HOP/SESSION, not date_trunc, for streaming windows
- `stream-cdc-pattern` — Two-step: CREATE SOURCE (shared) → CREATE TABLE ... FROM source

**Sinks:**
- `sink-snapshot-false` — Always use `snapshot = false` when adding sinks to existing MVs
- `sink-force-compaction` — Use `force_compaction = true` for high-cardinality upsert sinks

**Performance:**
- `perf-shared-source` — Multiple MVs from one Kafka topic should use shared source
- `perf-indexes-on-mv` — Create indexes on MV columns used in WHERE/JOIN predicates
