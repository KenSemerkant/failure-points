## Data Integrity

315. Schema drift between producer and consumer of a data stream
   A producer and consumer that drift in schema evolve independently, so the producer emits fields the consumer does not expect or the consumer expects fields the producer no longer sends. Messages then fail to parse or are silently misinterpreted, dropping or corrupting data. Detect with a schema registry that validates every message against its declared schema. Mitigate with versioned schemas, explicit compatibility checks on every change, and consumer-side tolerant readers that ignore unknown fields.

316. Encoding mismatch causing data corruption across services
   Services that encode and decode data with different encodings, such as UTF-8 versus Latin-1 or JSON versus protobuf, misinterpret bytes in transit and corrupt the data. The result is garbled text, wrong numeric values, or parse failures that surface only downstream. Detect with encoding validation at every boundary and round-trip tests. Mitigate with a single canonical encoding declared in the contract, explicit content-negotiation headers, and rejection of undeclared encodings.

317. Idempotency key collision causing duplicate processing
   Two distinct requests that share an idempotency key are treated as one, so one request is dropped or both are processed incorrectly. This causes lost work when a legitimate request is suppressed, or duplicated side effects when the key is reused across unrelated operations. Detect by monitoring idempotency key collisions and unexpected deduplication. Mitigate with unique, namespaced idempotency keys that include the operation and tenant, and by scoping keys to a single logical operation.

318. TTL misconfiguration causing premature data deletion
   A TTL set too short deletes data before it is no longer needed, so reads return missing data and force recomputation or fail outright. Operators observe intermittent missing records and elevated cache or database misses. Detect by monitoring TTL expirations and the rate of unexpected misses. Mitigate with TTL validation at write time, grace periods that extend access to expiring data, and separate TTLs for hot versus cold data.

319. Backup failure leaving no recovery path
   A backup that fails silently, because the job error is swallowed or the output is never verified, leaves no recovery path, so data loss is unrecoverable. Operators discover the gap only after an incident, when the restore attempt finds no valid snapshot. Detect by monitoring backup success and alerting on any failed or skipped job. Mitigate with verified backups that are checksummed and test-restored, plus regular restore drills that prove recoverability.

320. Restore failure during disaster recovery
   A restore that fails during disaster recovery, due to an untested procedure, missing dependencies, or a corrupt backup, leaves the system down and misses the recovery objective. Operators face extended downtime while debugging the restore itself. Detect with scheduled restore testing that exercises the full procedure against production-like data. Mitigate with tested, automated restore procedures, dependency pinning, and runbooks that are rehearsed before they are needed.

321. Partial backup restore causing data inconsistency
   A restore that completes only partially, because it was interrupted or some shards were missing, leaves the data inconsistent, so the system serves a mix of old and new state. Queries return contradictory results and downstream logic acts on partial truth. Detect by validating restore completeness against the expected record count and checksums. Mitigate with atomic restore that swaps in a fully verified snapshot, and per-shard checksums that confirm every partition restored.

322. Data retention policy violation causing compliance issues
   Data retained beyond or deleted before the policy window causes compliance violations, exposing the organization to fines, legal action, and audit findings. Operators may not notice until an auditor or regulator flags the discrepancy. Detect with retention audits that compare actual data age against policy. Mitigate with automated retention enforcement that expires data on schedule, legal holds that override deletion, and immutable audit logs of every retention action.

323. Checksum mismatch silently ignored during data transfer
   A checksum mismatch that is silently ignored, logged but not acted on, lets corrupt data through, so the receiver stores invalid data. The corruption propagates downstream and surfaces later as wrong results or crashes. Detect by enforcing checksum validation and alerting on every mismatch. Mitigate with end-to-end checksums computed at the source and verified at the destination, plus automatic retry or quarantine on mismatch so corrupt data never enters storage.

324. Timestamp precision loss during serialization
   Serialization that truncates timestamp precision, such as converting nanoseconds to seconds, loses ordering information, so events that were distinct become indistinguishable. Consumers that rely on timestamp ordering then process events in the wrong sequence or drop duplicates. Detect by comparing timestamps before and after serialization in round-trip tests. Mitigate with high-precision timestamp types that preserve full resolution, and by using monotonic sequence numbers rather than wall-clock time for ordering.

325. Truncated write appearing as valid data
   A write that is truncated, by a full disk, a dropped connection, or a partial flush, but not detected appears valid, so the stored data is incomplete and corrupt. Readers then parse a malformed record or silently return truncated content. Detect with length and checksum validation on every read and write. Mitigate with atomic writes that only publish a record after it is fully written, and write verification that confirms the full payload before acknowledging success.

326. Data sharding key change causing cross-shard query failures
   Changing a sharding key breaks the mapping between keys and shards, so queries route to the wrong shard and fail or return wrong data. Existing records become unreachable under the new key, and cross-shard queries span the wrong partitions. Detect by validating shard routing against the key mapping and testing lookups after any change. Mitigate with stable sharding keys that never change, and migration tooling that rehashes and relocates records atomically when a key must change.

327. Row-level policy recreated in separate transactions
   A migration that must change a column used in a row-level security policy drops the policy, alters the column, and recreates the policy. If the migration tool runs those statements in separate transactions, there is a window in which the table has no policy, and a table with no policy denies every row by default. Every tenant sees an empty product and inserts fail until the policy returns. Detect by asserting the policy count on each tenant table after every migration step. Mitigate with one transaction for the drop, change, and recreate, and tooling that groups them.

328. Schema rewrite locking every tenant out
   One schema change on a table shared by all tenants takes an exclusive lock for the duration of a table rewrite, and every tenant's queries wait behind it. Connection pools fill behind the lock, so a planned maintenance step becomes a service-wide outage. Detect by alerting on lock waits and query age during migrations, and by timing the statement on a copy first. Mitigate with a lock timeout and retry, additive changes that avoid a rewrite, and a staged plan for any per-tenant key change.

329. Usage events lost or late around invoice finalization
   Metering events that drive billing arrive late, more than once, or not at all, and usage totals are reported to billing before the late-arrival window has passed. Customers are under- or over-charged, and usage that arrives after an invoice is finalized does not change it. Detect by reconciling raw events to billed totals per tenant and per period. Mitigate with idempotent ingestion, a late-arrival window with recomputation, and reporting each period once.
