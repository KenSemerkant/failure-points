## Consensus

82. Split-brain in distributed lock managers due to network partitions
   A partition lets two lock managers grant the same lock to different clients, so both proceed with a critical section concurrently and corrupt shared state. The mutual-exclusion guarantee is silently violated, and the resulting corruption may not surface until much later. Detect with lock fencing and lease validation that reject operations from a client whose lock was superseded. Mitigate with quorum-based locking and lock TTLs so a lock cannot be held by two owners, and require a majority to grant or renew a lock.

83. Consensus failure due to network latency
   Latency delays consensus messages past their timeouts, so rounds fail and the cluster cannot reach agreement even though all nodes are healthy and connected. The system is up but cannot make progress, and clients see timeouts while the cluster repeatedly restarts rounds. Detect by measuring consensus round latency and comparing it against configured timeouts. Mitigate with latency-aware timeouts that scale with observed network conditions and with pipelining of consensus messages to reduce round-trip overhead.

84. Consensus failure due to transient network issues
   A brief network blip interrupts an in-flight consensus round, so the round aborts and must be retried, stalling progress even though the network recovers quickly. The cluster loses the work of the interrupted round and spends time re-establishing agreement. Detect by correlating round failures with network events and by tracking round retry rates. Mitigate with retry and idempotent consensus messages so a retried round is safe and does not duplicate side effects.

85. Consensus disruption due to network instability
   Ongoing network instability repeatedly interrupts consensus, so the cluster oscillates between leaders and makes no durable progress. Leadership churns as each new leader is deposed before it can commit, and the system serves little or no traffic. Detect by monitoring leadership churn and election frequency against network stability metrics. Mitigate with stable membership, pre-vote suppression to avoid disruptive elections, and backoff that prevents the cluster from thrashing during instability.

86. Leader election instability caused by concurrent elections
   Multiple nodes start elections at once, splitting votes so no candidate wins a majority and elections repeat indefinitely. The cluster is stuck in an election loop, unable to elect a leader or make progress, while clients time out. Detect by monitoring election rounds and vote-splitting patterns. Mitigate with randomized election timeouts so nodes rarely start elections simultaneously, and with pre-vote to reduce contention by checking whether a candidate can win before incrementing the term.

87. Zombie nodes attempting to participate in existing consensus
   A node that was partitioned or crashed rejoins with stale state and tries to vote or lead, disrupting the current term with messages that reflect an outdated view of the cluster. Its stale messages can confuse peers, trigger spurious elections, or cause the cluster to reject valid progress. Detect with term and epoch checks that identify messages from an old term. Mitigate by rejecting messages from stale terms and requiring the rejoining node to catch up before participating.

88. Race condition during leader handover between two nodes
   A handover where the old and new leader overlap briefly lets both issue decisions, causing conflicting state to be applied. The single-writer invariant is violated during the transition window, and the two leaders may commit incompatible updates. Detect by tracing handover boundaries and flagging any interval where two nodes both claim authority. Mitigate with fencing and a single-writer invariant enforced by leases, so the old leader is fenced off before the new leader begins issuing decisions.

89. Node death causing loss of quorum
   Enough nodes die that the cluster no longer holds a majority, so it cannot elect a leader or commit writes and becomes unavailable. The surviving nodes are healthy but cannot make progress because no quorum exists, and the system stalls until nodes are restored. Detect by monitoring quorum size and alerting when the number of live members drops below the majority threshold. Mitigate with an adequate replication factor and fast replacement of failed nodes to restore quorum quickly.

90. Node death during critical consensus steps
   A node dies mid-election, mid-commit, or partway through a multi-step agreement, leaving the round incomplete and forcing a restart of the consensus step from a checkpoint. Progress stalls until the round is retried, the partial work must be discarded or recovered, and side effects may be duplicated if steps are not idempotent. Detect by tracing step completion and checkpointed progress, flagging rounds that never reach a terminal state. Mitigate with idempotent steps and durable state so a restarted round is safe and does not repeat external effects.

91. Node crash during critical state transition and consensus
   A crash during a state transition leaves the node's persisted state inconsistent with the cluster, so it cannot safely rejoin and may serve stale or corrupt data. The node's on-disk state reflects a partial transition that does not match the committed history. Detect by validating state on restart and comparing it against the cluster's committed state. Mitigate with write-ahead logging and snapshot recovery to restore a consistent state before the node rejoins.

92. Node failure during distributed lock acquisition process
   A node fails while acquiring a lock, so the lock is neither granted nor released cleanly and other nodes block or deadlock waiting for it. The lock remains in an indeterminate state, and contenders stall until the failure is detected. Detect by monitoring lock wait times and flagging locks that remain pending beyond a threshold. Mitigate with lock TTLs and lease-based acquisition so stale locks expire automatically and contenders can proceed.

93. Node unavailability causing disruption in distributed coordination
   A node that becomes unavailable mid-coordination leaves pending operations in limbo, so peers wait or time out waiting for a response that never arrives. The coordination protocol stalls on the missing node, and dependent operations block. Detect by tracing coordination dependencies and flagging operations stuck waiting on an unavailable peer. Mitigate with timeouts and failover so a missing node does not block progress, and with retries that route around the unavailable node.

94. Node unreachability causing cluster unbalance
   An unreachable node stops participating, so load and leadership concentrate on the remaining nodes and the cluster becomes unbalanced. The surviving nodes take on disproportionate work, risking overload and degraded performance, while the unreachable node's data may fall behind. Detect by monitoring per-node load and flagging skew after a node drops out. Mitigate with rebalancing to redistribute load and prompt replacement of the unreachable node to restore balance.

95. Node unresponsiveness causing cluster instability
   A node that is alive but unresponsive to coordination messages causes peers to repeatedly time out and retry, destabilizing the cluster. The unresponsive node holds resources and stalls operations while peers burn effort retrying, and the cluster may trigger spurious elections or failovers. Detect by distinguishing liveness from responsiveness, monitoring both heartbeats and actual response times. Mitigate with watchdog restarts and isolation of unresponsive nodes so they are removed from coordination until they recover.

96. Node startup causing resource contention on shared infrastructure
   A node starting up consumes shared CPU, disk I/O, and network bandwidth as it replays its log, rebuilds indexes, and loads state, starving co-located peers on the same host or rack. Peers then experience elevated latency and missed heartbeats, which triggers false failure detection, unnecessary elections, and eviction of healthy nodes. Detect by monitoring per-node resource usage and correlating contention spikes with startup events; mitigate with rate-limited startup, resource isolation, and staggered restarts.

97. Out-of-order message delivery violating causality
   Network reordering, retries, or parallel delivery paths cause messages to arrive in a different order than they were sent, so a node applies a later state transition before an earlier one. Causal dependencies break, producing lost updates, incorrect results, or corrupted state that propagates to downstream consumers. Detect by attaching sequence numbers and vector clocks to every message and flagging gaps or inversions; mitigate with ordered delivery channels or causal consistency protocols.

98. Pre-vote failure causing unnecessary leader elections
   A pre-vote phase that fails or is disabled lets candidates launch disruptive elections even when a healthy leader exists, because nodes no longer verify that the incumbent can still reach a quorum before campaigning. The cluster churns through repeated election rounds, disrupting service and wasting CPU and network resources. Detect by monitoring pre-vote outcomes and election frequency; mitigate by enabling pre-vote and requiring a quorum check before granting candidacy.

99. Log compaction failure causing snapshot divergence
   A failed or interrupted log compaction produces a snapshot that does not match the compacted log prefix, because entries are dropped or truncated inconsistently. Nodes that later load the corrupt snapshot diverge from the cluster, serving wrong data or failing to replicate with peers. Detect by validating snapshots against the log with checksums and index verification; mitigate with verified compaction and atomic snapshot replacement.

100. Snapshot transfer failure during new node join
   A new node fails to receive a complete snapshot because the transfer is interrupted by a network failure or corrupted in transit, so it joins with missing or corrupt state. The node then serves incorrect data or repeatedly fails to catch up, extending the window of reduced redundancy. Detect by validating transferred snapshots with integrity checks; mitigate with resumable, chunked transfer and automatic retry.

101. Membership change during active consensus causing instability
   Adding or removing members while consensus is in progress can split the quorum or strand in-flight operations between the old and new configurations, because different nodes hold different views of membership. The cluster stalls or becomes unavailable while the transition is unresolved. Detect by tracing membership transitions and correlating them with quorum loss; mitigate with joint consensus and quorum-safe membership changes.

102. Read-only node serving stale data during a partition
   A read-only replica isolated by a network partition cannot receive updates from the primary, so it keeps serving stale data to clients that route to it. Clients read outdated values, violating freshness expectations and potentially acting on data that has already changed. Detect by comparing replica lag and fencing partitioned nodes; mitigate with fencing read-only nodes or requiring quorum reads.

103. Witness node failure causing quorum loss in an edge cluster
   A witness or tie-breaker node that fails removes the deciding vote in an even-sized edge cluster, because the remaining members can no longer form a majority. The cluster loses quorum and becomes unavailable even though the data-bearing nodes are healthy, so all writes and reads stall. Detect by monitoring witness health and alerting on its loss; mitigate with redundant witnesses and automatic replacement.

104. Heartbeat interval misconfiguration causing false timeouts
   A heartbeat interval set too long or a timeout set too short causes healthy nodes to be marked failed, because the gap between heartbeats exceeds the failure threshold. False failure detection triggers unnecessary elections and evictions, and nodes are repeatedly removed and re-added, churning membership and destabilizing the cluster. Detect by comparing heartbeat cadence to the timeout and alerting on mismatches; mitigate with aligned intervals and grace periods.

105. Commit index lag causing stale reads
   A follower whose commit index lags the leader serves reads from an older point in the log, because it has not yet applied entries the leader has committed. Clients read data that has since been overwritten, violating linearizability, and may act on a value that no longer exists. Detect by comparing follower commit index to the leader and alerting on lag; mitigate with quorum reads or read-index barriers.

106. Joint consensus failure during configuration change
   A joint-consensus transition that fails midway leaves the cluster with two overlapping configurations, because the old and new configurations are both partially active. Quorum becomes ambiguous and progress stalls, since no single configuration can commit, and writes and membership changes block until the transition is resolved. Detect by tracing configuration transitions and detecting stuck states; mitigate with durable, resumable joint consensus.

107. Leader stepping down due to transient network blip causing service disruption
   A leader that steps down on a brief network blip forces an unnecessary election, because it interprets the transient loss of connectivity as a loss of quorum. The cluster pauses service during the transition, and clients experience latency or errors while the new leader rebuilds its state before serving writes. Detect by correlating step-downs with network blips; mitigate with grace periods before stepping down.

108. Follower falling too far behind and being removed from the cluster
   A follower that lags beyond the retention window is removed, because its log has been compacted away and it can no longer catch up incrementally. It must rejoin via a full snapshot, and the cluster temporarily loses a replica, so the remaining replicas bear the full read and write load. Detect by monitoring follower lag and alerting before the retention boundary; mitigate with adequate log retention and catch-up.

109. Snapshot restore failure after node recovery
   A recovered node fails to restore its snapshot, because the snapshot is corrupt, missing, or incompatible with the current code. The node cannot rejoin, and the cluster runs with reduced redundancy until the restore succeeds; a failed restore may also leave the node half-initialized and require manual intervention. Detect by validating restore and alerting on failure; mitigate with tested, checksummed snapshots and regular restore drills.

110. Membership change leaving a learner node with stale state
   A learner added during a membership change that does not finish catching up serves stale state or blocks the change, because promotion is attempted before the learner has replicated the full log. The learner serves wrong data or the membership change stalls indefinitely, and the cluster cannot complete the reconfiguration until the learner catches up. Detect by tracking learner progress and gating promotion; mitigate with gated promotion of learners.

111. Configuration change rejected due to quorum loss
   A configuration change cannot be committed because the cluster has lost quorum, so the change is rejected and the cluster cannot adapt to failures or scale. The cluster remains stuck in its current configuration, unable to replace failed nodes or add capacity, prolonging the outage; operators must first restore quorum before retrying the change. Detect by monitoring quorum before initiating changes; mitigate with quorum-safe change sequencing.

112. Leader election tie causing repeated election rounds
   Two candidates receive equal votes, so neither wins and elections repeat until randomization breaks the tie. The cluster churns through election rounds, and each round consumes a full timeout period, extending the leaderless window while no leader can commit writes. Detect by monitoring election round counts and alerting on repeated ties; mitigate with randomized timeouts and ranked voting to break ties deterministically.

113. Consensus log divergence after a node rejoins with a stale snapshot
   A node that rejoins with an outdated snapshot has a log that diverges from the cluster, because its snapshot predates entries the cluster has since committed. It must truncate and resync, discarding local entries that conflict with the cluster's history and risking data loss if it had uncommitted entries. Detect by comparing log tails and snapshot indexes; mitigate with verified snapshot and log reconciliation.
