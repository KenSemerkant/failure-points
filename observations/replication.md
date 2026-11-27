## Replication

120. Replication lag causing reads from stale secondaries
   A secondary that lags the primary serves reads from an older state, because it has not yet applied recent writes. Clients see data that has already changed, and a stale read can cause a client to base a new write on an outdated value, overwriting newer data. Detect by monitoring replica lag and alerting when it exceeds a threshold; mitigate with read-your-writes routing and lag-aware read policies.

121. Replication lag causing read-after-write inconsistencies
   A client writes to the primary then reads from a lagging replica, so it does not see its own write, because the replica has not yet applied the change. This read-after-write inconsistency confuses users, who believe the write failed and retry it, causing duplicate submissions. Detect by tracking write-to-read consistency; mitigate with primary reads for the writing session or causal consistency.

122. Replication lag causing inconsistent reads
   Different replicas at different lag points return different values for the same key, because each has applied a different prefix of the write log. Clients observe inconsistency across requests, and a client may read a value then read an older value on the next request, seeing the data flip back and forth. Detect by comparing replica states and alerting on divergence; mitigate with monotonic reads and quorum reads.

123. Replication lag causing staleness in distributed caches
   A cache fed by a lagging replica serves stale values, because the replica has not yet received the latest write. Cached data diverges from the source of truth, so clients read outdated information and may act on it, and the stale entry persists until its TTL expires or it is evicted. Detect by comparing cache entries to the primary and alerting on version drift; mitigate with versioned cache entries and revalidation.

124. Replication lag causing distributed state divergence
   Sustained lag lets replicas accumulate different states, because each applies writes at a different rate, and divergence grows with every write the lagging replica misses. They diverge, and reconciliation becomes expensive or lossy, requiring full resync or risking data loss. Left unchecked, the replicas can no longer be merged automatically. Detect by monitoring divergence and alerting on growing gaps; mitigate with anti-entropy repair and bounded lag.

125. Partial writes during node crash before commit completion
   A node crashes after writing part of a transaction but before committing, leaving partial data that violates atomicity, because the write was not made durable atomically. On recovery, the partial state is visible or corrupts subsequent operations, and the transaction appears half-applied, breaking invariants. Detect with write-ahead logging, which records the transaction's intent before any data is modified; mitigate with atomic commit and crash recovery.

126. Partial failure of multi-node transaction causing inconsistent state
   A distributed transaction that commits on some participants but not others leaves the system in a mixed state, because the commit decision is not atomic across nodes. Some participants hold the new value while others hold the old, so reads disagree, and the coordinator must track which participants committed and which did not. Detect by tracing transaction outcomes; mitigate with two-phase commit and compensating transactions.

127. Partial state sync causing data inconsistency
   A sync that completes only partially leaves a replica with a mix of old and new state, because the transfer was interrupted before finishing. The replica serves inconsistent data, returning values that do not match any single point in time, and it cannot be trusted until the sync is verified complete. Detect by validating sync completion; mitigate with resumable, checksummed sync and retry from the last checkpoint.

128. Partial state update leading to data divergence
   An update applied to only some replicas causes them to diverge, because the write did not reach every replica, yet the write is considered successful even though some replicas never received it. Subsequent reads disagree, returning different values depending on which replica serves them. Detect by comparing replica versions; mitigate with atomic broadcast and quorum writes, so a majority must acknowledge before the write is visible.

129. Inconsistent state due to partial updates across multiple nodes
   A multi-node update that succeeds on a subset of nodes leaves the cluster with inconsistent state that is hard to reconcile, because the write was not applied atomically across nodes. Some nodes hold the new value while others hold the old, and no single source of truth remains. Detect by tracking per-node update status; mitigate with idempotent updates and reconciliation.

130. Incomplete data propagation across distributed nodes
   When a write is acknowledged before every replica receives it, or a replication link drops messages silently, some nodes never learn of the committed change and keep serving the previous value. Reads routed to those lagging nodes return stale or missing data, and a failover can promote a replica that lacks the latest writes. Detect by tracking per-replica propagation completeness and comparing applied offsets against the primary. Mitigate with acknowledged replication that confirms a write only after a quorum persists it, plus anti-entropy repair that periodically reconciles divergent replicas.

131. Data corruption during serialization/deserialization process
   A serializer bug or a version mismatch between producer and consumer corrupts the byte stream, so the receiver deserializes wrong values, truncated fields, or throws an exception. The corruption is silent when the bytes still parse, producing logically incorrect records that propagate downstream. Detect by validating schemas on both ends and attaching checksums to every serialized payload so tampering or truncation is caught at read time. Mitigate with versioned schemas, forward and backward compatibility checks, and rejecting any message whose checksum or schema fingerprint does not match the expected contract.

132. Data corruption due to partial node failure or crash
   A node that crashes mid-write can leave a record half-written, with the value torn between old and new bytes, so the stored data is invalid and later reads return garbage or fail validation. The corruption may go unnoticed until the record is read, at which point it poisons queries and downstream consumers. Detect with checksums computed at write time and verified on every read. Mitigate with atomic writes that commit a full value or nothing, and write-ahead logging that replays or rolls back partial writes after a crash.

133. Data inconsistency due to divergent replication streams
   When replicas receive writes through different paths or in different orders, their replication streams diverge and each replica accumulates a distinct history, so the cluster can no longer agree on the current state. Reads against different replicas return conflicting values, and a merge becomes ambiguous because no single history is authoritative. Detect by comparing stream positions and sequence numbers across replicas to surface divergence early. Mitigate with single-writer replication so one node defines the canonical order, and deterministic conflict resolution such as last-write-wins or version vectors when divergence is unavoidable.

134. Data inconsistency from partial replication
   Replication that stops partway through a batch leaves replicas holding different subsets of the data, so reads routed across the cluster return inconsistent results depending on which node answers. The inconsistency is intermittent and hard to reproduce, since it depends on which replica serves each request. Detect by monitoring replication progress and comparing applied offsets or row counts between primary and replicas. Mitigate with resumable replication that continues from the last applied position after interruption, and periodic consistency checks that identify and repair replicas that fell behind.

135. Metadata inconsistency between primary and replica nodes
   Schema, index definitions, or configuration can diverge between the primary and its replicas when changes are applied out of order or to only some nodes, so the same query behaves differently depending on which node executes it. A replica with a stale schema may reject valid writes or return wrong columns. Detect by comparing metadata versions and hashes across nodes on a schedule. Mitigate with versioned metadata and synchronized schema changes that apply atomically to the primary and all replicas before the new version is served.

136. Index corruption leading to incorrect query results
   A corrupted index structure returns wrong or missing rows even though the underlying data is intact, so queries silently produce incorrect results reaching users and reports. The data itself is fine, so integrity checks that only scan base tables miss the failure. Detect with index validation routines that compare index entries against the source data, and query result sampling that cross-checks indexed lookups against full scans. Mitigate with index rebuilds on detection, checksums on index pages, and write-ahead logging so a crash cannot leave the index half-updated.

137. Replication stream stall causing unbounded lag growth
   A replication stream that stalls, from a blocked thread, saturated network, or dead consumer, lets lag grow without bound, so replicas fall further behind the primary and recovery grows costly. Reads from lagging replicas return progressively older data, and failover to it loses a large window of committed writes. Detect by monitoring the lag trend, not just the current value, so a growing gap is caught before it becomes unrecoverable. Mitigate with stall detection that alerts or restarts the stream, and automatic resync once lag exceeds a threshold.

138. Conflict resolution failure during multi-master merge
   When multiple masters accept concurrent writes to the same key, the merge step must reconcile them, and if no automatic rule applies, the merge fails or silently drops a write. The result is lost data or a stuck merge that blocks further updates to the affected key. Detect by monitoring conflict counts and merge failures per key so unresolved conflicts surface immediately. Mitigate with conflict-free replicated data types that merge deterministically without loss, and explicit resolution policies such as last-write-wins or application-defined merge functions where automatic resolution is unsafe.

139. Replica promotion failure during failover
   When the primary fails, a replica must be promoted to take its place, and if promotion fails because the replica is not caught up or the election logic is broken, the system is left without a primary. Writes fail until an operator intervenes manually, extending the outage. Detect by monitoring promotion attempts and alerting when a failover does not complete within a bounded time. Mitigate with tested, automated failover exercised regularly, and pre-flight checks that verify a candidate replica is caught up before promotion.

140. Write amplification from replication causing disk pressure
   Replication multiplies every logical write by the replication factor, so disk I/O and space consumption grow several times faster than the application's write rate, and nodes degrade as storage fills. Write-ahead logging and compaction compound it by writing the same data multiple times. Detect by monitoring write amplification, the ratio of physical bytes written to logical bytes, alongside disk utilization and I/O latency. Mitigate with batching that groups small writes into larger sequential ones, and compression that reduces the physical footprint of replicated data.

141. Replica resync causing read disruption during catch-up
   A replica that resyncs from scratch cannot serve current data during catch-up, so reads routed to it either fail or return stale values until the resync completes. If it stays in the read path, users see intermittent errors or old data for the duration, which can last hours for large datasets. Detect by monitoring resync progress and the read error rate on the resyncing node. Mitigate with incremental catch-up that transfers only the missing delta, and read draining that removes the replica from the read path until fully caught up.

142. Replication factor reduction causing data loss on node failure
   Lowering the replication factor reduces the number of copies of each piece of data, so a single node failure can destroy the only remaining copy and cause permanent data loss. The risk peaks when the factor drops below the level needed to survive expected simultaneous failures. Detect by monitoring the effective replication factor against policy and alerting when any data is under-replicated. Mitigate with minimum-factor enforcement that refuses to drop below a safe threshold, and re-replication that restores the target factor after any node loss.

143. Cross-region replication latency causing eventual consistency violations
   Replication across regions introduces network latency, so a client reading from one region sees stale data that was written in another region and has not yet propagated, violating the consistency the application expects. The violation is most visible in read-after-write patterns where a user writes in one region and immediately reads from another. Detect by measuring cross-region replication lag and exposing it to the application. Mitigate with region-aware consistency that routes reads to the writer's region when freshness is required, and conflict resolution for concurrent cross-region writes.
