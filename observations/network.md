## Network

1. Network partition causing split-brain in distributed systems
   A network partition isolates a subset of nodes from the rest, and a leader lease that does not expire can do the same without one, so each side independently elects its own leader and both accept conflicting writes, causing the cluster to diverge into two conflicting histories until the partition heals. Two nodes simultaneously believe they hold leadership, which corrupts replicated state, and when the partition heals, reconciliation can lose one side's committed data or require manual intervention. Detect via fencing tokens or epoch numbers that reject stale leaders, and by alerting when two nodes claim leadership concurrently. Mitigate with majority quorum and leader leases that expire when the holder can no longer reach a majority, so only one side can hold leadership.

2. Network partition causing loss of quorum or preventing quorum formation
   When a partition leaves fewer than a majority of nodes reachable from any side, no group can assemble a quorum, so the cluster stops accepting writes and may become read-only or fully unavailable even though most nodes are still running. This is the classic availability-versus-consistency tradeoff: the system chooses consistency and refuses to make progress. Detect by monitoring quorum size against current membership and alerting when it drops below majority. Mitigate by sizing clusters for the expected failure domain and using witness or tie-breaker nodes to break ties.

3. Network partition causing stale reads
   A node isolated from the write path continues serving reads from its local replica, returning data that no longer reflects committed writes. Clients therefore observe values that have already been overwritten or deleted, which can drive incorrect decisions such as acting on a stale balance or configuration. Detect by comparing read timestamps against the primary or tracking replica lag, and alert when lag exceeds a threshold. Mitigate with read-your-writes guarantees, quorum reads, or fencing isolated nodes out of the read path.

4. Network partition causing inconsistency in replicated data
   Divergent writes accepted on each side of a partition produce replicas that disagree, and naive last-write-wins reconciliation can silently drop one side's updates. The cluster ends up with multiple conflicting versions of the same record, and the losing write is lost without any signal to the client. Detect with version vectors or logical clocks that expose conflicts at read time. Mitigate with conflict-free replicated data types or an explicit, tested conflict-resolution policy that preserves both sides' intent.

5. Network partition causing inconsistent cluster state
   Each side of a partition updates its own view of membership, configuration, or routing, so nodes disagree about who is live and where data lives. This disagreement causes misrouted requests, incorrect failover decisions, and writes sent to nodes that no longer own the data. Detect by comparing cluster metadata across nodes and flagging divergent membership views. Mitigate with a single source of truth for membership and versioned configuration that requires quorum to change.

6. Transient network partition lasting just long enough to trigger a leader reelection
   A brief partition that outlasts the election timeout causes followers to start an election and choose a new leader even though the old leader is still healthy. The cluster then churns through leadership changes and may briefly reject writes while the new leader establishes itself. Detect by correlating election events with partition duration and flagging elections that occur without a real failure. Mitigate with randomized election timeouts and pre-vote checks that suppress elections when a leader is still reachable.

7. Network partition where a node can see the gateway but not its peers
   A node retains connectivity to the load balancer or gateway but loses its peer links, so it keeps serving client traffic while being cut off from consensus and replication. It serves stale data and cannot participate in quorum, yet the gateway still routes requests to it because the process appears healthy. Detect by monitoring peer reachability independently of client reachability. Mitigate with health checks that verify quorum membership and replication lag, not just process liveness.

8. Packet loss causing missed heartbeats
   Dropped heartbeat packets make peers conclude a healthy node has failed, triggering spurious leader elections and membership churn. The cluster repeatedly evicts and re-adds nodes, wasting resources and destabilizing coordination even though no node actually failed. Detect by correlating heartbeat loss with network packet-loss counters and flagging evictions that coincide with loss windows. Mitigate with heartbeat retries and failure-detector grace periods that tolerate transient loss before declaring failure.

9. Packet loss causing consensus failure
   Lost consensus messages prevent a quorum from agreeing on a value, so commits stall and the cluster retries or re-elects repeatedly. Throughput collapses even though no node has actually failed, and clients see writes hang or time out. Detect by tracking consensus round latency and message retransmission rates, alerting when retries spike, and by correlating stalled rounds with loss and instability metrics. Mitigate with reliable transport and retransmission with backoff and bounded retries to avoid amplifying the loss into a livelock.

10. Packet loss causing delayed heartbeats
   Heartbeats that arrive late rather than never can still trip aggressive failure detectors, causing false positives and unnecessary failovers. A healthy node is marked down because its heartbeat crossed a fixed threshold by milliseconds, triggering an election and state transfer that were never needed. Detect by measuring heartbeat inter-arrival jitter and comparing it to the configured timeout. Mitigate with adaptive failure-detector thresholds that scale with observed latency rather than fixed values.

11. Packet loss causing missed synchronization events
   Lost replication or synchronization messages leave replicas unaware of committed changes, producing stale state that persists until a resync. Reads from those replicas return old data indefinitely, and the divergence grows as more updates are missed. Detect by comparing replica sequence numbers against the primary and alerting when they drift. Mitigate with idempotent, acknowledged replication and periodic anti-entropy repair to converge replicas even after message loss.

12. Packet loss causing increased error rates
   Dropped packets surface as timeouts and retries at the application layer, inflating error metrics and degrading throughput even when the service is logically healthy. Operators may chase phantom application bugs because the errors appear to originate in code rather than the network. Detect by separating network-layer loss from application errors using packet-loss counters and tracing. Mitigate with retry budgets and circuit breakers so transient loss does not cascade into overload.

13. Packet loss during consensus phase causing leader election instability
   Loss concentrated during elections prevents candidates from gathering votes, so elections repeatedly time out and no stable leader emerges. The cluster is stuck in a leaderless state, rejecting writes and serving stale reads until the loss subsides. Detect by correlating election failures with loss windows and monitoring the time spent without a leader. Mitigate with larger election timeouts and pre-vote to avoid disruptive re-elections during lossy periods.

14. Packet duplication causing duplicate side-effects in non-idempotent operations
   Retransmitted or duplicated packets re-execute a request, so a non-idempotent operation such as a payment, increment, or send is applied twice. This causes double-charges, duplicate records, and side effects that cannot be undone. Detect with idempotency keys and request deduplication at the receiver, and audit for duplicate application of the same request. Mitigate by making operations idempotent or deduplicating at the receiver before applying side effects.

15. Network latency causing heartbeat loss
   Heartbeats delayed beyond the failure-detector threshold are treated as lost, so healthy nodes are marked dead and evicted. The cluster loses members and triggers failovers that were never needed, reducing capacity and causing unnecessary state transfers. Detect by comparing heartbeat latency to the configured timeout and flagging evictions that occur during latency spikes. Mitigate with latency-aware timeouts and grace periods that account for observed network delay.

16. Network latency or too-short timeouts causing leader election churn
   Slow vote and append messages stretch elections past their timeout, or a timeout set too low is shorter than the time heartbeats need to propagate under load, so leaders are deposed and candidates repeatedly restart elections. The cluster spends its time electing rather than serving requests, each leadership change interrupts in-flight work, and clients see write stalls during every election window. Detect by tracking election duration and leadership tenure against network latency, and by comparing the timeout with observed latency percentiles. Mitigate with randomized, latency-scaled election timeouts that grow with observed delay, so healthy leaders are not prematurely deposed.

17. Network latency causing consensus timeouts
   Latency in the consensus path delays quorum agreement, causing commit stalls and client timeouts even though no node has failed. Writes appear to hang, and throughput drops as requests queue behind slow consensus rounds. Detect by measuring consensus round latency separately from data-path latency and alerting when the two diverge. Mitigate with pipelining, batching, and separating the consensus path from the data path so one does not block the other.

18. Network latency causing a stale leader
   A leader that is partitioned or delayed from its followers cannot renew its lease and keeps acting as leader after the followers that still reach a majority elect a new one, so two nodes issue conflicting decisions. This is a form of split-brain driven by latency or by a lease that expires unnoticed, not only by a hard partition, and it corrupts state when both leaders accept writes. Detect with fencing tokens that reject operations from an expired leader. Mitigate by requiring the leader to renew a lease and checking that renewal before every write.

19. Network latency causing stale reads in distributed caches
   A cache node that cannot reach the source of truth serves expired entries, returning data that has since changed. Clients act on stale information, such as an outdated price or permission, and the error persists until the cache reconnects. Detect by comparing cache timestamps to the origin and monitoring cache hit age. Mitigate with TTLs, versioned cache entries, and revalidation on read so stale entries are refreshed or rejected.

20. Network latency causing database timeout
   Queries that exceed the client timeout due to network latency are aborted, but the database may still execute them, causing ambiguity about whether a write committed. Retries can then duplicate the write, producing double-application of a payment or insert. Detect by correlating timeouts with query completion and checking for duplicate rows or side effects. Mitigate with idempotent writes and query cancellation so a timed-out write is not silently re-applied.

21. Network latency causing timeouts in critical coordination steps
   Latency in lock acquisition, lease renewal, or transaction prepare phases causes those steps to time out, aborting otherwise valid operations. The system rejects work it could have completed, and clients see spurious failures during periods of elevated latency. Detect by tracing coordination-step latency and flagging timeouts that occur without a real failure. Mitigate with generous, latency-aware timeouts and retry-safe coordination primitives.

22. Network-induced latency causing timeouts in distributed transactions
   Latency between transaction participants delays the prepare or commit phase, so the coordinator times out and aborts a transaction that participants may have already applied. This leaves the system in an ambiguous, partially-committed state where some nodes have the write and others do not. Detect by tracing transaction phase latency and logging participant outcomes. Mitigate with idempotent commit and participant-side timeout alignment so retries converge to a single outcome.

23. Network jitter causing false failure detections in heartbeats
   Variable heartbeat latency occasionally exceeds the failure threshold, so healthy nodes are intermittently marked dead and re-added, causing membership churn. The cluster never reaches a stable membership, and coordination overhead grows as nodes repeatedly rejoin and resynchronize. Detect by measuring heartbeat jitter and flagging evictions that coincide with jitter spikes. Mitigate with jitter-tolerant thresholds and requiring multiple missed heartbeats before eviction.

24. Network jitter causing flapping nodes
   A node whose connectivity oscillates is repeatedly marked down and up, triggering repeated failovers and state transfers. Each flap forces the cluster to rebalance and resync, consuming bandwidth and CPU while degrading availability. Detect by tracking node state transitions and alerting when a node flaps more than a few times per minute. Mitigate with hysteresis and a minimum down-time before re-admission so brief flaps do not trigger failover.

25. Network jitter causing unstable membership
   Jitter causes nodes to join and leave the cluster rapidly, so membership views change constantly and coordination overhead spikes. The cluster spends resources on membership churn instead of serving requests, and gossip or consensus traffic floods the network. Detect by monitoring membership churn rate and alerting when it exceeds a threshold. Mitigate with stable membership protocols and debounced join/leave handling that ignores transient connectivity changes.

26. Network jitter causing intermittent connectivity
   Jitter produces bursts of dropped or delayed packets, so connections appear to fail and recover unpredictably. Applications see flaky behavior that is hard to reproduce, and users report intermittent errors that vanish on retry. Detect by measuring packet inter-arrival variance and correlating it with connection resets and retransmission counts. Mitigate with retries, connection pooling, and jitter buffers to smooth out variability.

27. Network jitter causing intermittent service failure or disruption
   Jitter-induced timeouts cause requests to fail sporadically, so users see intermittent errors that are hard to reproduce and diagnose. Support teams struggle to reproduce the failures because they depend on timing rather than a deterministic bug. Detect by correlating error bursts with jitter windows and packet-loss counters. Mitigate with retries with backoff and graceful degradation so transient jitter does not surface as user-visible failure.

28. Network jitter causing erratic RPC calls
   Variable RPC latency causes some calls to time out while others succeed, producing inconsistent behavior and partial failures. Callers cannot predict whether an operation completed, so they may retry and duplicate side effects or give up and lose work. Detect by tracking RPC latency percentiles and flagging high variance. Mitigate with deadline propagation and idempotent RPCs so retries are safe.

29. Network jitter causing consensus round timeouts
   Jitter pushes some consensus rounds past their timeout, so rounds are retried and progress stalls intermittently. The cluster makes uneven progress, with bursts of commits followed by stalls, and tail latency degrades for clients. Detect by measuring consensus round latency variance and alerting when the tail exceeds the timeout. Mitigate with jitter-tolerant round timeouts and pipelined consensus to keep progress moving.

30. Network jitter causing instability in distributed coordination
   Jitter disrupts the timing assumptions of coordination protocols such as leader election, locking, and membership, causing repeated re-coordination. The system never settles into a stable state, and each re-coordination event interrupts service and can drop in-flight work. Detect by monitoring coordination event frequency and flagging when elections or lock acquisitions recur rapidly. Mitigate with timing-independent coordination, randomized backoff, and jitter-tolerant timeouts.

31. Network jitter causing unstable service discovery and topology
   Variable network latency delays health-check responses and registration updates, so the service registry flips a node between healthy and unhealthy states and the routing topology oscillates. Traffic is repeatedly steered toward nodes that have already failed or away from nodes that are still serving, producing intermittent request failures and churn. Detect by tracking registry update latency and state-transition frequency; mitigate with debounced health checks that require multiple consecutive failures, plus stale-while-revalidate registries that keep serving the last known good topology during jitter.

32. Network congestion causing latency spikes in RPC calls
   Congestion queues packets in switch and NIC buffers, so RPC latency spikes and tail requests blow past their deadlines while the median stays acceptable. A small fraction of requests become dramatically slow, which degrades user experience and can trigger cascading timeouts in dependent services. Detect by monitoring RPC latency percentiles against link utilization and queue depth; mitigate with load shedding, priority queues that favor latency-sensitive traffic, and backpressure so callers slow down before the tail collapses.

33. Network congestion causing RPC timeouts
   Congested links delay RPCs past their deadlines, so callers time out and immediately retry, injecting additional load into an already saturated path. Each retry deepens the congestion, creating a positive feedback loop that can drive the network into collapse. Detect by correlating timeout rates with link saturation and retry volume; mitigate with retry budgets that cap total retry load, jittered exponential backoff, and congestion-aware backoff that suppresses retries while the link is saturated.

34. Network congestion causing excessive jitter
   Congestion produces highly variable queueing delays, so packet latency becomes unpredictable even when the average remains within tolerance. Timing-sensitive protocols such as consensus heartbeats, media streaming, and clock synchronization misbehave because they depend on bounded delay rather than average delay. Detect by measuring latency variance and jitter under load, not just mean latency; mitigate with traffic shaping to smooth bursts and quality-of-service prioritization that bounds queueing delay for timing-critical flows.

35. Network congestion causing service unreachability
   Severe congestion drops or delays enough traffic that a service becomes effectively unreachable to clients, even though the service process itself is healthy and responding locally. Connection attempts time out and in-flight requests stall, so the service appears down to the rest of the system. Detect by monitoring link saturation, packet loss, and connection success rates alongside process health; mitigate with capacity planning, ingress rate limiting, and failover to alternate paths or regions that bypass the congested link.

36. Network congestion causing load balancer failures
   Congestion overwhelms the load balancer's connection table, packet processing, or CPU, so it drops or misroutes traffic even though every backend is healthy. The load balancer becomes the bottleneck and a single point of failure for all traffic behind it. Detect by monitoring load balancer saturation metrics such as connection count, packet drops, and CPU; mitigate with horizontal scaling of the load balancer tier, connection limits, and direct-server-return or consistent-hashing designs that reduce per-request state.

37. DNS caching issues causing clients to connect to decommissioned nodes
   Stale DNS cache entries keep clients resolving to addresses of nodes that have been decommissioned or repurposed, so traffic is sent to dead hosts or, worse, to a different service now bound to that address. Requests fail or reach the wrong destination, and the failure persists until the cached record expires. Detect by monitoring resolved targets against the current inventory and flagging connections to unknown addresses; mitigate with short TTLs during decommissioning and explicit cache invalidation or negative caching when a node is removed.

38. DNS failure causing service discovery failures or delays
   A DNS outage or a slow resolver prevents services from resolving peer addresses, so discovery fails or stalls and every dependent call blocks or times out. Because nearly all inter-service communication begins with a name lookup, a single DNS dependency can stall the entire service graph. Detect by monitoring DNS resolution latency and failure rate per resolver; mitigate with local DNS caching, multiple redundant resolvers, and static fallback address lists so services can keep routing during a resolver outage.

39. DNS failure causing service-to-service disconnection
   When DNS resolution fails, services cannot establish new connections to peers, so inter-service calls fail even though the peer processes are healthy and running. Existing connections may survive, but any reconnect or new call path breaks. Detect by correlating connection failures with DNS error codes and resolution timeouts; mitigate with connection reuse and pooling, plus cached peer addresses so a transient DNS failure does not break established or re-established connections.

40. DNS resolution failures causing service unavailability
   Persistent DNS failures make a service unreachable by name, so clients cannot route to it at all regardless of its health. The service is effectively down even though its processes are running and its ports are open. Detect by monitoring DNS failure rates per service name and alerting when resolution success drops; mitigate with redundant DNS providers, health-checked failover records, and static or anycast addresses that keep the name resolvable during provider outages.

41. DNS resolution issues causing endpoint unreachability
   A specific endpoint fails to resolve because its record is missing, mistyped, or misconfigured, so only that endpoint becomes unreachable while the rest of the service continues to work normally. This partial failure is easy to miss because aggregate health metrics stay green. Detect by probing per-endpoint resolution and alerting on individual record failures; mitigate with record validation at deploy time and synthetic monitoring that continuously resolves and connects to every critical endpoint.

42. MTU mismatch causing large packets to be dropped silently
   A path segment with a smaller MTU than the sender assumes drops oversized packets without returning ICMP feedback, so large transfers stall or fail while small ones succeed. The result is mysterious, size-dependent failures that are hard to reproduce and diagnose. Detect by testing path MTU with progressively larger probes and watching for silent drops; mitigate with path MTU discovery, and by clamping the TCP MSS to the smallest segment on the path so no packet ever exceeds the limit.

43. MTU mismatch leading to truncated messages
   Fragmentation or truncation at a mismatched MTU boundary corrupts or cuts off messages, so receivers get malformed payloads, parse errors, or silently incomplete data. Applications that assume a message arrives whole misbehave in ways that are hard to trace to the network. Detect by validating message integrity and length at the application layer; mitigate with consistent MTU configuration across the path and jumbo-frame alignment so frames are never fragmented or truncated in transit.

44. Firewall configuration causing partition in cluster
   A firewall rule change blocks inter-node traffic, partitioning the cluster even though the physical network is intact. Nodes can no longer reach each other for consensus, replication, or heartbeats, so each side may elect its own leader and diverge. Detect by monitoring inter-node reachability immediately after configuration changes and alerting on consensus stalls; mitigate with change review gates and automated connectivity tests that run before and after every firewall change to catch blocked ports.

45. Firewalls dropping legitimate inter-node communication after config change
   A firewall update drops traffic on ports used for replication or consensus, silently breaking coordination between nodes. The cluster degrades without any obvious error because the drop is silent and the affected services keep running locally. Detect by correlating firewall config changes with connectivity loss and consensus latency; mitigate with allow-list validation that confirms required ports remain open, and post-change smoke tests that exercise inter-node traffic before the change is considered complete.

46. TCP connection exhaustion on a single node under heavy load
   A node exhausts its connection table or ephemeral port range, so new connections fail even though the node has spare CPU and memory. The node becomes unreachable to new clients while existing connections continue to work, producing a confusing partial outage. Detect by monitoring connection count, ephemeral port usage, and accept failures; mitigate with connection pooling, per-client connection limits, and load balancing across nodes so no single node accumulates all connections.

47. Bandwidth saturation on the backplane causing delayed consensus votes
   Saturated inter-node bandwidth delays consensus messages, so votes, appends, and heartbeats arrive late and elections or commits stall. The data plane starves the control plane, and the cluster can lose quorum even though data traffic is flowing normally. Detect by monitoring backplane utilization against consensus latency and election frequency; mitigate with traffic isolation that separates data and control traffic, and bandwidth reservation or quality-of-service rules that prioritize consensus messages.

48. Network interface saturation preventing control plane traffic
   Data-plane traffic saturates the network interface, starving control-plane messages such as heartbeats, elections, and membership updates. The cluster loses coordination and may split or stall even though data continues to flow, because the control messages are delayed or dropped. Detect by monitoring control-plane latency and heartbeat loss under load; mitigate with quality-of-service prioritization that gives control traffic strict priority over data traffic on the same interface.

49. Network topology changes causing routing loops in service mesh
   A topology change creates a routing loop, so packets circulate between nodes until their TTL expires, causing latency spikes, dropped traffic, and wasted bandwidth. The loop persists until the control plane converges on a correct topology. Detect by monitoring route convergence time and loop counters or TTL-expiry metrics; mitigate with loop prevention mechanisms in the mesh control plane and fast convergence so transient loops are resolved before they cause sustained damage.

50. Asymmetric network connectivity where node A can talk to B but B cannot talk to A
   One-way connectivity lets node A send to B while B's replies or requests fail, so each side sees a different view of the other and state diverges. Coordination breaks in ways that are hard to diagnose because A believes B is healthy while B believes A is unreachable. Detect with bidirectional reachability probes that test both directions independently; mitigate with symmetric routing policies and health checks that verify connectivity in both directions before declaring a peer reachable.

51. Transient network failure during a sensitive multi-step operation
   A brief network failure mid-operation leaves a multi-step workflow partially applied, so state is inconsistent and retries may duplicate steps that already completed. The system cannot tell which steps succeeded and which did not, so recovery is ambiguous. Detect by tracing step boundaries and recording which steps completed; mitigate with idempotent steps that are safe to retry, and compensating transactions that unwind partial work when a workflow cannot be completed.

52. IPv6 misconfiguration causing connectivity loss after migration
   An incomplete IPv6 migration leaves some paths IPv4-only and others IPv6-only, so nodes that resolve to the wrong address family cannot connect. Connectivity breaks selectively, depending on which family a client resolves and which path it takes. Detect by probing both address families and comparing reachability; mitigate with dual-stack deployment so every service listens on both families, and family-aware health checks that catch one-family failures before they affect traffic.

53. Network policy change breaking cross-namespace communication
   A network policy update blocks traffic between namespaces or services, so previously working calls fail. The change is often unrelated to the affected services, making the outage hard to attribute to its cause. Detect by correlating policy changes with connectivity loss and alerting on sudden drops in cross-namespace traffic; mitigate with policy validation that simulates the effect of a change, and staged rollout so a bad policy affects only a small subset of traffic first.

54. Proxy misconfiguration causing request interception failures
   A misconfigured proxy drops, rewrites, or misroutes requests, so clients receive errors or responses from the wrong backend. The proxy becomes a silent failure point because it sits in the request path and its misbehavior is not visible to the services behind it. Detect by tracing requests through the proxy and comparing intended versus actual routing; mitigate with proxy config validation and canary testing that routes a small fraction of traffic through the new config before full rollout.

55. Connection reuse causing requests to hit a dead peer
   A pooled connection to a peer that has since failed is reused, so requests fail until the pool evicts the stale connection. Clients see intermittent errors that are hard to reproduce because they depend on which pooled connection happens to be selected. Detect by monitoring connection reuse failures and correlating them with peer health; mitigate with health-checked connection pools that validate connections before reuse, and connection TTLs that force periodic re-establishment.

56. Half-open TCP connections accumulating on a node
   Connections whose peer has silently died remain half-open, consuming file descriptors and memory until the node exhausts its resources. The node degrades gradually and eventually fails to accept new connections, even though the peer is long gone. Detect by monitoring half-open connection counts and file descriptor usage; mitigate with TCP keepalive probes that detect dead peers, and idle connection timeouts that close connections that have seen no traffic for a bounded period.

57. Packet reordering causing out-of-order processing
   Reordered packets deliver messages out of sequence, so a receiver that assumes ordering processes them incorrectly and applies state in the wrong order. The result is corrupted state or incorrect results that are hard to trace to the network. Detect by validating sequence numbers and flagging out-of-order delivery; mitigate with sequence-aware reassembly that buffers and reorders messages, and ordering guarantees at the transport or application layer where strict ordering is required.

58. Network buffer exhaustion causing dropped packets under burst
   A burst of traffic overflows switch or NIC buffers, so packets are dropped and flows stall or retransmit. Bursty workloads see disproportionate loss because buffers fill faster than they drain, even when average utilization is low. Detect by monitoring buffer drops and retransmission rates during bursts; mitigate with larger buffers, traffic pacing that smooths bursts at the source, and burst-tolerant protocols that recover from loss without collapsing throughput.

59. Link flapping causing route instability
   A link that repeatedly goes up and down causes routes to flap, so traffic is rerouted constantly and connections drop each time the topology changes. The network never converges, and applications see persistent instability. Detect by monitoring link state transitions and route churn; mitigate with link stabilization that suppresses flapping interfaces, and route dampening that delays route withdrawal until a link has been down for a sustained period.

60. BGP route withdrawal causing traffic blackholing
   A withdrawn BGP route leaves no path to a prefix, so traffic is dropped at the network edge before it can reach the service. The service becomes unreachable from outside even though it is healthy, and the failure is invisible to the service itself. Detect by monitoring route tables and external reachability for each advertised prefix; mitigate with redundant paths and multiple upstream providers, plus prefix monitoring that alerts when a route disappears.

61. VLAN misconfiguration isolating a subset of nodes
   A VLAN change moves a subset of nodes onto a different broadcast domain, so layer-2 frames no longer reach their peers and the cluster partitions even though IP routing may still appear intact. The isolated nodes keep running but cannot exchange heartbeats, replicate data, or serve traffic, so they diverge from the rest and may be marked down. Detect by monitoring layer-2 reachability and MAC-address-table changes against the expected topology. Mitigate with VLAN validation in change pipelines and automated connectivity tests that run after every network change.

62. Network switch failure isolating a rack of nodes
   A failed top-of-rack switch drops every link in that rack at once, so all nodes behind it lose connectivity to the cluster in a single hardware event rather than one at a time. The blast radius is large: an entire rack stops serving, and if replicas were co-located there, quorum or data availability is lost. Detect by monitoring switch health, port status, and rack-level reachability so a switch failure is visible immediately. Mitigate with redundant switches, multi-rack replica placement, and rack-aware scheduling that keeps quorum members on separate switches.

63. Duplicate IP address assignment causing routing conflicts
   Two nodes are assigned the same IP address, so switches and hosts cannot determine which MAC owns the address and traffic is delivered unpredictably, alternating between the two nodes. Connections intermittently fail, reach the wrong node, or flap as ARP entries are overwritten, producing confusing application errors and data sent to the wrong destination. Detect by monitoring ARP conflicts and duplicate-address detection events on the network. Mitigate with DHCP or IPAM validation that rejects duplicate assignments and with duplicate-address detection enabled on all interfaces.

64. Network cable or transceiver failure causing intermittent link loss
   A degrading cable or transceiver drops the link for brief intervals, so connectivity flaps and packets are lost intermittently rather than failing cleanly. The node appears up but suffers retransmissions, timeouts, and slow responses, and the failure is hard to localize because it is transient and often load-dependent. Detect by monitoring link error counters, CRC errors, and interface flap counts, which rise before a hard failure. Mitigate with redundant links and proactive replacement of hardware whose error counters trend upward.

65. Network congestion control misconfiguration causing throughput collapse
   Misconfigured congestion control, such as an overly aggressive window or disabled ECN, causes senders to overdrive the link and then collapse throughput when buffers fill and packets drop. The link is underutilized while latency spikes and retransmissions dominate, so effective throughput falls far below capacity. Detect by monitoring achieved throughput against link capacity and tracking retransmission and queueing latency. Mitigate with tuned congestion-control parameters, ECN enabled end to end, and buffer sizing matched to the bandwidth-delay product.

66. Network segmentation change breaking service discovery
   A segmentation change moves services into different network segments, so discovery lookups still return the old addresses but those addresses are no longer reachable from the caller. Services resolve peers they cannot connect to, and calls fail with timeouts or connection-refused errors even though the services themselves are healthy. Detect by correlating segmentation changes with discovery failures and by validating that resolved addresses are reachable. Mitigate with segment-aware discovery that returns topology-consistent endpoints and with post-change connectivity validation.

67. ARP cache poisoning causing traffic misdirection
   A poisoned ARP cache maps an IP address to the wrong MAC address, so frames are delivered to an attacker or an unintended node instead of the legitimate host. This enables man-in-the-middle interception or denial of service, and traffic silently flows to the wrong destination without the sender noticing. Detect by monitoring ARP table changes and flagging unexpected MAC-to-IP bindings. Mitigate with static ARP entries for critical hosts, switch port security, and dynamic ARP inspection to drop spoofed replies.

68. Network interface teaming misconfiguration causing failover failures
   Misconfigured interface teaming does not actually fail over when a link dies, so the node loses connectivity despite having redundant physical links. The redundancy is illusory: the teaming driver keeps using the dead link or fails to promote the standby, and the node drops off the network. Detect by testing failover explicitly, pulling links and confirming traffic continues. Mitigate with validated teaming configuration, matching teaming modes on both ends, and periodic failover drills that exercise the standby path.

69. Network path change causing asymmetric latency between nodes
   A routing change makes the forward and reverse paths differ, so one-way latency becomes asymmetric and no longer equals half the round-trip time. Timing-sensitive protocols that assume symmetric delay, such as clock synchronization and some consensus implementations, then misbehave and produce skewed estimates. Detect by measuring round-trip and one-way latency separately and flagging divergence. Mitigate with symmetric routing where possible and latency-aware timeouts that tolerate asymmetric paths.

70. Network address translation timeout breaking long-lived connections
   A NAT device expires the mapping for an idle long-lived connection, so subsequent traffic from the server is dropped because the translation no longer exists, and the connection silently dies. Long-lived connections such as streaming, WebSockets, and database links break without either endpoint receiving a clean close. Detect by monitoring connection liveness and idle time against NAT timeout settings. Mitigate with application-level keepalives sent more frequently than the NAT timeout and NAT-aware timeout configuration.

71. Load balancer health check false-negative causing node removal
   A health check that is too strict or misconfigured marks healthy nodes as down, so the load balancer removes them from rotation and concentrates traffic on the remaining nodes. The surviving nodes then fail under the extra load, cascading the outage even though the removed nodes were fine. Detect by comparing health-check results with actual node health and alerting when removal is not corroborated by node metrics. Mitigate with tolerant checks, multiple probes before removal, and staggered health-check intervals.

72. Egress rule change blocking outbound API calls
   A change to egress firewall or security-group rules blocks outbound calls to external APIs, so integrations fail with connection timeouts or refused errors. The change is often unrelated to the affected service and applied globally, so the failure appears without any code or configuration change in the service itself. Detect by correlating egress rule changes with integration failures and by monitoring outbound connection success rates. Mitigate with egress allow-list validation in change pipelines and monitoring that alerts on sudden drops in outbound connectivity.

73. Conntrack race dropping parallel A and AAAA queries
   A resolver sends the A and AAAA lookups for one name from the same UDP socket, so both share one conntrack entry; the reply to the first can tear that entry down, and the kernel then drops the second reply. The client waits out its full retry timeout for the dropped reply, which produces multi-second stalls in name resolution with nothing wrong at the DNS server itself. Detect by comparing resolver timeout histograms on pods with CoreDNS latency, which stays flat, and by packet capture at the pod. Mitigate with a node-local DNS cache that skips conntrack, a kernel with the race fixed, or TCP fallback for lookups.

74. Conntrack table exhaustion on a node
   Every connection and every UDP flow through a node occupies a conntrack entry, and entries for UDP, including DNS queries, are held until their timeout. Short-lived flows from many pods fill the table, and once it reaches its maximum the kernel drops every new flow on that node. Pods see random connection failures and DNS failures first. Detect by tracking the conntrack entry count against its maximum, the kernel drop counters, and the table-full message in the kernel log. Mitigate by raising the table size, shortening the UDP timeout, and keeping DNS lookups off the node firewall.

75. Cloud resolver packet limit throttling DNS silently
   One cloud resolver enforces a packet rate per network interface, 1024 packets per second on AWS, so one busy node drops queries at the interface before they reach the resolver. The drops are invisible in flow logs and query logs because no query is recorded. CoreDNS forwarders and pods that call the resolver directly show sporadic lookup timeouts that vary with the load on each node. Detect with the interface metric for exceeded link-local allowance and with packet captures at the node. Mitigate by caching lookups on the node, cutting the query rate per lookup, and spreading resolver load across interfaces.

76. Search domain expansion multiplying DNS queries
   The pod resolver configuration lists several search domains and a high dot threshold, so any name with fewer dots is tried against every search domain before the absolute name. One lookup of an external hostname becomes four or more queries to the cluster DNS service, and the amplification grows with call volume. The cluster DNS service saturates, lookups hit their timeout, and latency rises for every workload. Detect by comparing cluster DNS query rates with application call rates and by watching DNS latency percentiles. Mitigate with a lower dot threshold, absolute names for external hosts, and a node-local cache.

77. Single zonal NAT gateway serving every availability zone
   A NAT gateway lives in one availability zone, and route tables in the other zones point at it to avoid the cost of one gateway per zone. All outbound traffic from every private subnet then depends on one zone. When that zone is impaired, workloads in the healthy zones keep running but lose all outbound access, so calls to object storage, external APIs, and image registries fail. Detect with outbound probes per zone and a review of route tables for cross-zone targets. Mitigate with one gateway per zone and same-zone routes.

78. Node-local external traffic concentrating load on few nodes
   A service exposed through a load balancer can be set to route only to nodes that run a ready pod, which preserves the client address but removes cross-node traffic distribution. When pods sit on a few nodes, because of a rollout, a scale-down, or uneven scheduling, those nodes carry most or all of the traffic while the others receive none. Latency and saturation rise although total capacity looks sufficient. Detect by tracking the request rate per pod and the count of healthy load balancer targets per service. Mitigate with topology spread constraints and enough replicas to cover the nodes.

79. Outbound port exhaustion at a shared NAT device
   Many private addresses share one or a few public addresses on a NAT gateway or load balancer, and each public address offers a fixed number of source ports. Connection churn and bursts of outbound calls use the ports up, and new connections fail while existing ones and inbound traffic keep working. Failures appear intermittently and grow after scale-out. Detect with the provider's port allocation error and connection metrics for the NAT device. Mitigate with connection pooling that keeps connections open, more public addresses, and a limit on outbound connection churn per instance.

80. Cluster address exhaustion stranding pods in creation
   Pods take an address from the node's attached network interfaces or from the subnet, and either can run out while CPU and memory remain free. The network plugin cannot assign an address, so the pod stays in creation, rollouts stall, and new capacity cannot come up during scale-out. Detect by tracking free addresses per subnet, the interface attachment limit per node, and sandbox creation failures in events. Mitigate with larger subnets, prefix delegation, and a secondary address range for pods.

81. Security group rule limit blocking new ingress
   A security group holds a fixed maximum number of rules, and controllers that manage ingress often write every managed rule into one group. Once the group is full, any controller that tries to add a rule to it fails, so new load balancers, listeners, and ingress resources fail to create and the cluster stops accepting new ingress. Detect by tracking rule counts per group against the quota and by controller errors about rule limits. Mitigate by distributing rules across groups, consolidating rules into wider address ranges and port ranges, and tracking the rule count per group against the quota.
