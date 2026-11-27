## Dependencies

291. Upstream API rate limiting causing cascading failures
   An upstream API rate limit rejects requests with 429s when the caller exceeds its quota, so every request beyond the limit fails immediately and the shared quota is exhausted. Dependent services fail, and their callers fail in turn, cascading the failure through the system. Detect by monitoring upstream 429 rates; mitigate with client-side throttling, request caching, and backoff so callers stay within quota.

292. Third-party outage taking down dependent services
   A third-party outage propagates to dependent services that have no fallback path, so they fail whenever the dependency is down. The dependent services become unavailable, taking down features that rely on the third party and surfacing errors to users. Detect by monitoring dependency health and availability; mitigate with graceful degradation, cached fallbacks, and circuit breakers so the system survives the outage.

293. SDK version mismatch causing subtle behavior differences
   Different services using different SDK versions of a dependency behave subtly differently, in changed defaults, serialization, or error handling. Integrations break in ways that are hard to trace because each side sees different behavior, and failures surface only at runtime. Detect by auditing SDK versions across services; mitigate with version pinning and compatibility testing so all services use a known, consistent version.

294. Message broker failure causing event loss
   A message broker failure loses in-flight events that were not yet acknowledged or persisted, so those events never reach consumers and the broker's in-memory buffer is lost. Downstream consumers miss state changes, leaving systems out of sync and data inconsistent. Detect by monitoring broker health and message loss; mitigate with durable queues, publisher confirms, and consumer acknowledgments so events survive broker failure.

295. Queue backlog growing faster than consumers can drain
   A queue whose backlog grows faster than consumers can drain it accumulates messages, so processing falls further behind because producers outpace consumers. Messages are processed long after they are produced, causing stale or delayed downstream effects and growing end-to-end latency as the backlog compounds. Detect by monitoring queue depth and consumer lag; mitigate with consumer scaling, backpressure, and alerts when lag exceeds thresholds.

296. Dead letter queue overflow causing message loss
   A dead letter queue that overflows drops messages when its capacity is exceeded, so failed messages are lost without processing or inspection. Failures are silently discarded, losing the ability to diagnose or replay them and hiding systemic errors, so the root cause goes undiagnosed. Detect by monitoring DLQ depth and alerting on growth; mitigate with DLQ retention limits and replay so failed messages can be recovered.

297. Webhook delivery failure causing missed state updates
   A webhook that fails to deliver, due to network error, timeout, or receiver outage, causes the receiver to miss a state update. The receiver acts on stale data, producing incorrect downstream behavior and diverging from the source of truth. Detect by monitoring webhook delivery success and retries; mitigate with retries with backoff and outbox-based delivery so updates are reliably delivered in order.

298. Misbehaving client exhausting the shared database connection limit
   A misbehaving client holds connections open or opens too many, consuming the shared database connection limit, or leaks them without returning them to the pool. Other clients cannot connect, so the database becomes unavailable to the rest of the system. Detect by monitoring per-client connection counts; mitigate with per-client limits and connection pooling with timeouts to prevent one client from exhausting the pool.

299. Object storage eventual consistency causing stale reads
   Object storage that is eventually consistent returns stale data after a write, because replicas converge asynchronously, so a read immediately after a write may return the previous version. Readers see old values immediately after a write, causing incorrect decisions based on stale state, such as a deleted object still appearing. Detect by monitoring read-after-write consistency; mitigate with strong-consistency reads where available, or version objects and validate versions on read.

300. External API contract change breaking an integration
   An external API changes its contract, in field names, types, or response shape, without notice, so the integration's assumptions break. The integration fails at runtime, returning errors or misparsing responses, and the breakage is often silent until a request fails. Detect with contract tests that run against the live API; mitigate with versioned APIs and contract monitoring so changes are caught before they break. An LLM agent wrapping the integration does not fail cleanly: it reasons around the errors with guessed arguments and may still report the task as done.

301. Dependency service slow degradation causing cascading timeouts
   A dependency that degrades slowly causes its callers to time out, and those callers then time out their own callers, cascading the failure outward. A single slow dependency takes down a whole chain of services through timeout propagation. Detect by monitoring dependency latency and timeout rates; mitigate with appropriate timeouts and circuit breakers so a slow dependency is isolated before it cascades.

302. Message ordering guarantee loss after broker failover
   A broker failover loses the ordering guarantee, so messages are delivered out of order after the failover because the new leader replays messages from a different point. Consumers process messages in the wrong order, producing incorrect state and results. Detect by monitoring message ordering and sequence gaps; mitigate with sequence numbers and ordered consumers so out-of-order messages are detected and reordered.

303. External service rate limit header change breaking client throttling
   An external service changes its rate-limit headers, renaming fields or altering their semantics, so the client's throttling logic misreads the limit and either over-throttles or under-throttles. Over-throttling stalls legitimate traffic and inflates latency, while under-throttling floods the provider and triggers 429 responses or account suspension. Detect by monitoring throttle behavior and the rate of 429s against expected limits. Mitigate with robust header parsing that tolerates missing or renamed fields, versioned header contracts, and conservative fallback limits.

304. Vendor lock-in preventing failover to an alternative provider
   Tight coupling to a single vendor's proprietary APIs, SDKs, and data formats prevents failover when that vendor fails, so the service has no recovery path and remains down for the duration of the vendor's outage. Operators observe a total dependency outage with no alternative route. Detect by assessing portability through regular failover drills and dependency audits. Mitigate with abstraction layers that isolate vendor-specific code, multi-vendor support behind a common interface, and a documented exit plan.

305. Registry pull limits shared behind one egress address
   Container registry pull limits are keyed on the source address, and every node behind one NAT gateway shares the same quota. Kubelet retries add pulls, so new nodes and rescheduled pods sit in image pull back-off exactly when autoscaling needs them. Detect with pull-limit events, registry rate-limit response headers, and the count of pods in image pull back-off per node. Mitigate by mirroring images in a private registry, pinning digests, pre-pulling on node boot, and alerting on limit headers.

306. Recovery herd saturating shared startup dependencies
   After a cluster recovers, many services start at once and pull images, fetch configuration, and look up DNS records from the same shared services. The simultaneous downloads saturate resource limits on those services, and the clusters stay degraded until someone intervenes. Detect by alerting on registry, DNS, and configuration store load during recovery, and on image pull latency. Mitigate with rate limits and caches for startup resources, pre-pulled images, and staggered startup.
