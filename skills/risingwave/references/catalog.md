# RisingWave System Catalog (rw_catalog)

## Core Object Metadata

| Table | Key Columns | Purpose |
|-------|------------|---------|
| `rw_tables` | `id, name, schema_id, owner, definition, append_only, created_at` | All tables |
| `rw_materialized_views` | `id, name, schema_id, owner, definition, append_only` | All MVs |
| `rw_sources` | `id, name, schema_id, owner, connector, columns, format` | All sources |
| `rw_sinks` | `id, name, sink_type, connection_params, status` | All sinks |
| `rw_views` | `id, name, definition` | Non-materialized views |
| `rw_indexes` | `id, name, primary_table_id, index_table_id` | Index definitions |
| `rw_schemas` | `id, name, owner` | Schema catalog |
| `rw_databases` | `id, name, owner` | Database catalog |
| `rw_columns` | `relation_id, name, position, is_nullable, data_type` | Column-level info |
| `rw_relations` | Union of all relation types | |

## Streaming Operations

| Table | Key Columns | Purpose |
|-------|------------|---------|
| `rw_streaming_jobs` | `id, name, parallelism, max_parallelism` | Active streaming jobs |
| `rw_fragments` | `fragment_id, job_id, fragment_type, distribution_type` | Fragment distribution |
| `rw_actors` | `actor_id, fragment_id, worker_node` | Parallelism unit details |
| `rw_worker_nodes` | `id, host, type, state` | Cluster node status |
| `rw_ddl_progress` | `ddl_id, ddl_statement, progress` | DDL backfill progress |
| `rw_fragment_backfill_progress` | `fragment_id, upstream_mv_count, finished_mv_count` | Per-fragment backfill |
| `rw_cdc_progress` | `source_id, source_name, progress` | CDC replication lag |
| `rw_kafka_job_lag` | `source_id, topic, partition, consumer_lag` | Kafka consumer lag |

## Storage (Hummock)

| Table | Purpose |
|-------|---------|
| `rw_hummock_current_version` | Current storage version ID |
| `rw_hummock_sstables` | SST file metadata (level, size, key range) |
| `rw_hummock_compact_task_progress` | Compaction task status |
| `rw_meta_snapshot` | Snapshot metadata |

## Access Control

| Table | Purpose |
|-------|---------|
| `rw_users` | User accounts and roles |
| `rw_secrets` | Encrypted credential references |
| `rw_user_secrets` | User-to-secret mapping |

## Events & Recovery

| Table | Purpose |
|-------|---------|
| `rw_event_logs` | System event history |
| `rw_recovery_info` | Recovery state and barrier progress |

---

## Common Diagnostic Queries

```sql
-- Check MV creation / backfill progress
SELECT ddl_id, ddl_statement, progress
FROM rw_catalog.rw_ddl_progress;

-- Kafka consumer lag per partition
SELECT source_id, topic, partition, consumer_lag
FROM rw_catalog.rw_kafka_job_lag
ORDER BY consumer_lag DESC;

-- CDC replication progress
SELECT source_name, progress
FROM rw_catalog.rw_cdc_progress;

-- All streaming jobs with parallelism
SELECT name, parallelism, max_parallelism
FROM rw_catalog.rw_streaming_jobs;

-- Fragment distribution across worker nodes
SELECT f.fragment_id, f.job_id, a.worker_node
FROM rw_catalog.rw_fragments f
JOIN rw_catalog.rw_actors a USING (fragment_id);

-- Find all MVs that read from a given source
SELECT mv.name, mv.definition
FROM rw_catalog.rw_materialized_views mv
WHERE mv.definition ILIKE '%source_name%';
```
