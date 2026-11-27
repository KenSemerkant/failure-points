## Application

267. Memory leak causing gradual node degradation and eventual crash
   A memory leak, from unreleased allocations, unbounded caches, or retained references, grows the process's resident set over time without bound. The node degrades as garbage-collection pressure rises and swap is consumed, eventually crashing or being OOM-killed and taking its traffic down. Detect by monitoring the memory trend and heap growth over time; mitigate with leak detection, heap profiling, and periodic restarts as a stopgap.

268. Thread or goroutine leak causing resource exhaustion over time
   Threads or goroutines that block forever on channels, locks, or network I/O accumulate because nothing reaps them, and each leaked thread consumes stack and scheduling slots. The process exhausts its thread or scheduler resources, becomes unresponsive, and can no longer accept new work. Detect by monitoring thread and goroutine counts for monotonic growth; mitigate with bounded worker pools, context cancellation, and leak detection to prevent unbounded accumulation.

269. Deadlock between two services holding locks in opposite order
   Two services each hold a lock the other needs, acquired in opposite order, so neither can release and neither can proceed. Both block forever and their requests hang until clients time out, freezing the affected operations and holding resources hostage. Detect with lock-order graph analysis and lock-acquisition timeouts; mitigate with a consistent global lock ordering and lock timeouts that break the cycle.

270. Livelock where retries keep conflicting without making progress
   Retries that keep conflicting with each other, each aborting the other's work and restarting, make no progress even though no thread is blocked. The system spins consuming CPU and resources without completing any work, so throughput collapses to zero. Detect by monitoring progress and completion rates; mitigate with randomized backoff and conflict resolution so competing retries eventually diverge and one wins.

271. Unbounded queue growth causing memory exhaustion
   A queue with no bound accepts work faster than consumers drain it, so entries accumulate indefinitely and each entry holds memory. Memory is exhausted and the process crashes or is OOM-killed, or latency grows without limit as items wait for processing. Detect by monitoring queue depth and alerting on growth; mitigate with bounded queues and backpressure so producers slow or reject when the queue fills.

272. Slow query causing connection pool exhaustion
   A slow query holds a database connection for a long time, and enough concurrent slow queries occupy every connection in the pool. The pool is exhausted, so other requests block waiting for a connection or fail outright, stalling the whole service. Detect by monitoring query latency and pool utilization; mitigate with query timeouts, pool sizing matched to workload, and indexing or rewriting slow queries.

273. Cache stampede when a hot key expires and all requests hit the backend
   When a hot cache key expires, all concurrent requests miss simultaneously and each forwards to the backend, multiplying the load by the number of waiting clients. The backend is hit with a burst of identical requests, overloading it and causing latency spikes or failure. Detect by monitoring miss storms and backend load; mitigate with request coalescing such as single-flight and staggered TTLs so misses are spread out.

274. Thundering herd of retries amplifying a brief outage
   A brief outage triggers a flood of simultaneous retries from many clients, all firing at once and multiplying the load on the recovering service. The service is overwhelmed by the retry burst, prolonging the outage instead of letting it heal. Detect by monitoring retry volume and amplification; mitigate with jittered exponential backoff and retry budgets that cap total retry load.

275. Hot partition causing uneven load distribution
   A hot key or partition concentrates load on a single node because the hash or access pattern routes most traffic there, leaving other nodes underutilized. That node saturates and becomes a bottleneck while others idle, causing uneven latency and potential failure of the hot node. Detect by monitoring per-partition load and skew; mitigate with key sharding, salting hot keys, and load-aware routing to spread traffic.

276. Backpressure failure causing unbounded in-flight requests
   Without backpressure, a slow consumer lets producers keep sending, so in-flight requests grow without bound and each one holds memory, a connection, and a thread. Memory and connections are exhausted, and the system degrades or crashes under the accumulated load, with latency climbing as work queues up. Detect by monitoring in-flight request counts and queue depth; mitigate with backpressure and flow control so producers slow or stop when the consumer falls behind.

277. Retry storm amplifying a transient failure into an outage
   Aggressive retries on a transient failure multiply the load, with each failed request spawning more retries that arrive faster than the service can recover. A brief blip is amplified into a sustained outage as the service drowns in retry traffic. Detect by monitoring retry amplification and error rates; mitigate with exponential backoff with jitter and circuit breakers that stop retries while the dependency is down.

278. Circuit breaker tripping and staying open due to misconfigured recovery
   A circuit breaker that trips and never recovers, because half-open probing is disabled or thresholds are misconfigured, keeps the service unavailable. The service stays down even after the dependency heals, causing prolonged avoidable downtime and failing requests that would otherwise succeed. Detect by monitoring breaker state transitions; mitigate with half-open probing and recovery thresholds so the breaker tests the dependency and closes when it recovers.

279. Timeout misconfiguration causing premature request cancellation
   A timeout set too short cancels requests that would have succeeded, cutting off slow-but-valid work before it completes. The service fails legitimate requests, returning errors to users for operations that would have finished, and wastes the partial work already done. Detect by comparing configured timeouts to observed latency percentiles; mitigate with latency-aware timeouts and deadline propagation so budgets reflect real work.

280. Race condition in shared state mutation
   Concurrent mutations of shared state without synchronization interleave, so reads and writes observe inconsistent intermediate values and one update silently overwrites another. Lost updates or corrupt state result, producing wrong results that are hard to reproduce and depend on timing. Detect with race detectors and stress tests; mitigate with locking, atomic operations, or immutable state to serialize or eliminate shared mutation.

281. Retry budget exhaustion causing dropped requests
   A retry budget that is exhausted causes requests to be dropped rather than retried, so legitimate work is lost under failure. Users see failures for requests that could have succeeded, and work is silently discarded without retry. Detect by monitoring budget exhaustion and drop rates; mitigate with per-request budgets and graceful degradation so failures are surfaced rather than silently dropped.

282. Serialization of a hot path causing latency spikes
   Serialization overhead on a hot path, from encoding and decoding large or complex payloads on every request, adds CPU and latency to each call. Requests slow down and may time out, degrading throughput and user experience, and the CPU cost compounds under load. Detect by profiling the hot path to measure serialization cost; mitigate with efficient serialization formats, avoiding redundant encode/decode, and caching serialized results.

283. Distributed transaction coordinator failure leaving transactions in doubt
   A transaction coordinator that fails mid-commit leaves participants in an in-doubt state, unable to know whether to commit or roll back, and its decision log is lost. Resources stay locked and transactions cannot complete without manual intervention, blocking related work and holding locks indefinitely. Detect by monitoring in-doubt transaction counts; mitigate with coordinator failover, persistent transaction logs, and heuristic resolution to recover stuck participants.

284. Sidecar container keeping jobs from completing
   A job completes only when every container in its pod exits successfully. A log shipper or proxy declared as a regular container never exits, so jobs and scheduled jobs hang until their deadline, holding pods and compute. Marker-file workarounds and extra controllers that kill the sidecar are fragile. Detect by alerting on jobs running past their expected duration whose main container has succeeded. Mitigate by declaring helpers as init containers with a restart policy, or by running them outside the job pod.

285. Controller reconcile loop rewriting its own trigger
   An operator updates the status of a resource on every reconcile, and that write triggers the watch event for the same resource. Each pass queues another, across every custom resource, and a controller restart lists all resources at once. Work-queue depth and API server load stay high, client-side throttling slows every controller that shares the client, and recovery after a restart takes hours. Detect with work-queue depth, reconcile errors, and reconcile calls per object per minute. Mitigate by writing status only on a real change, filtering out own updates, and a long resync period.

286. Capacity signal measuring a resource that is not the bottleneck
   A service scales and sheds load on CPU while its real limit is a backing store. CPU stays low, so nothing is shed and more instances pile load onto the store. The backend is overloaded while the frontend looks idle and keeps scaling. Detect by alerting when successful throughput stays flat while instance count rises, and on backend latency or queue depth. Mitigate with shedding based on utilization reported by the backend, adaptive concurrency limits, and per-customer quotas.

287. Unprioritized load shedding starving critical traffic
   A limiter with one flat threshold rejects every request class equally. After an outage, deferred and background traffic returns at a multiple of its normal volume and crowds out user-initiated requests, so the work that matters most is the work that fails. Detect by alerting on availability per request class and on traffic share by priority. Mitigate with priority partitions or criticality levels that propagate across service hops, with a small number of fixed classes.

288. Load shedding masking the scaling signal
   If shedding starts below the autoscaler's target utilization, utilization never crosses the target and no scale-up occurs. The system stays in a degraded, shedding state indefinitely at low load per instance, and the alert that would explain it never fires. Detect by alerting on a sustained shed rate with a flat replica count and utilization below target. Mitigate by starting to shed only above the target utilization, and then shedding progressively.

289. Over-aggressive load shedding causing congestive failure
   A fixed or mistuned limiter rejects most requests once it trips, or the shedding work itself consumes the CPU it is meant to protect. Successful throughput after shedding starts falls below the level before it. Detect by alerting when successful requests per second drop after shedding begins. Mitigate with adaptive limits based on gradient or latency that accept as much work as the service objective allows.

290. One tenant's backlog starving shared workers
   Work from every tenant goes into one first-in, first-out queue served by a shared worker pool. One tenant that enqueues a burst of jobs holds the head of the queue, so every other tenant's jobs wait behind it, and workers and connection pools stay saturated long after the burst. In one public incident two pipelines caused hours of degradation for all users. Detect by tracking queue depth and the oldest-job age per tenant, not only the total backlog. Mitigate with per-tenant concurrency limits, fair or weighted queues, and quotas on how much a tenant can enqueue.
