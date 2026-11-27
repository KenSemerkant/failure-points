## Caching

329. Cache invalidation failure serving stale data
   A cache that is not invalidated on write serves stale data, so clients see old values after the source has changed. Users observe outdated content, and decisions made on stale reads produce wrong results. Detect by comparing cache contents to the source of truth and monitoring for stale reads. Mitigate with write-through invalidation that removes or updates the cache on every write, versioned keys that change on update, and short TTLs as a backstop.

330. Cache eviction policy causing hot data thrashing
   An eviction policy that evicts hot data causes constant misses and reloads, so the cache thrashes and provides little benefit. The hit rate collapses, latency rises, and the backend absorbs the load the cache was meant to offload. Detect by monitoring hit rate and eviction counts per key. Mitigate with an appropriate eviction policy such as LRU or LFU matched to the access pattern, and adequate cache sizing so hot data stays resident.

331. Distributed cache node failure causing cache miss storms
   A cache node failure shifts its keys to the backend, so a miss storm overloads the origin. The backend receives a sudden flood of requests for the lost keys and may degrade or fail under the load. Detect by monitoring miss rate after node loss and alerting on the spike. Mitigate with replication so keys survive node failure, request coalescing that collapses concurrent misses into one backend call, and circuit breakers that shed load when the origin is overwhelmed.

332. Cache key collision causing cross-tenant data leakage
   A cache key collision lets one tenant read another tenant's cached data, because keys are not scoped to the tenant. This is a cross-tenant data leakage and a security breach, exposing confidential data to unauthorized users. Detect with key namespacing and by auditing cache keys for tenant scope. Mitigate with tenant-scoped keys that include the tenant identifier, collision checks that reject ambiguous keys, and separate cache namespaces per tenant.

333. Negative caching causing prolonged error responses
   Caching a negative result such as an error or not-found for too long serves errors after the underlying issue is fixed. Users continue to see failures or missing data even though the source has recovered, prolonging the incident. Detect by monitoring negative cache hits and their age. Mitigate with short negative TTLs that expire quickly, invalidation on recovery that clears negative entries, and distinguishing transient errors from permanent not-found results.

334. Cache warmup failure causing cold-start latency spikes
   A cache that starts cold serves misses for all requests, so latency spikes until it warms. The first requests after a deploy or restart hit the backend directly, and a large cache population can overload the origin. Detect by monitoring cold-start latency and miss rate after restarts. Mitigate with pre-warming that loads hot keys before serving traffic, gradual traffic ramp that limits initial load, and persistent caches that survive restarts.

335. Cache stampede protection failure causing backend overload
   When stampede protection such as request coalescing fails, a hot-key expiry floods the backend with concurrent misses. Many clients request the same expired key at once, and each triggers a separate backend call, overloading the origin. Detect by monitoring miss storms and the ratio of concurrent misses to unique keys. Mitigate with locking or single-flight requests that allow only one caller to repopulate a key, and jittered TTLs that spread expirations.
