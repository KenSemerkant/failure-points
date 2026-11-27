## Deployment

243. Configuration drift between nodes causing inconsistent behavior
   Nodes that were once identical accumulate manual or ad-hoc changes, so their settings diverge over time. The same request produces different results depending on which node serves it, causing intermittent bugs that are hard to reproduce and diagnose. Detect by comparing node configs against a baseline; mitigate with configuration management, immutable infrastructure, and automated drift detection and reconciliation.

244. Version skew during rolling upgrade causing protocol incompatibility
   During a rolling upgrade, old and new versions run simultaneously, and a change to a wire protocol or message schema makes them unable to parse each other's traffic. Requests between mixed versions fail, causing errors, dropped messages, or partial outages until the upgrade completes. Detect with compatibility testing and canary traffic; mitigate with backward-compatible protocols, versioned APIs, and staged upgrades.

245. Partial rollout leaving mixed service versions in production
   A rollout that is interrupted or fails partway leaves a subset of instances on the new version and the rest on the old. Behavior is inconsistent across the fleet, and bugs that depend on version interaction are hard to reproduce and diagnose. Detect by monitoring version distribution across instances; mitigate with atomic or canary rollouts, feature flags, and fast, tested rollback.

246. Feature flag misconfiguration enabling an unready code path
   A feature flag is flipped on before the underlying code, data migration, or dependency is ready, so requests enter a path that is incomplete or broken. Users hit errors, corrupted state, or missing functionality, and the broken path may write bad data before it is caught. Detect with flag audits and error monitoring; mitigate with flag validation, gradual rollout, and kill switches that disable the path instantly.

247. Environment variable mismatch between staging and production
   A variable such as a database URL, API key, or feature toggle differs between staging and production, so code that passed staging behaves differently when deployed. Production fails in ways staging never showed, causing outages or data corruption that are hard to trace. Detect by diffing environments and validating config at deploy time; mitigate with environment parity, config-as-code, and secret injection.

248. Schema migration failure leaving the database in an inconsistent state
   A migration that fails partway applies some statements but not others, leaving the schema half-migrated with missing columns, indexes, or constraints. Queries fail or return wrong results, and the application may write data that violates the intended schema. Detect with migration validation and post-migration checks; mitigate with transactional migrations, backward-compatible changes, and tested rollback plans.

249. Blue-green cutover failure routing traffic to an unready environment
   Traffic is switched to the green environment before its services are fully warmed, migrated, or health-checked, so requests land on an environment that cannot serve them. Users experience an immediate outage or errors until the cutover is corrected or reverted. Detect with readiness gates and synthetic checks; mitigate with health-checked cutover, warm-up traffic, and instant rollback to the blue environment.

250. Canary regression not caught before full rollout
   A regression that only manifests under real traffic or a small user subset appears in the canary but is not detected because metrics or analysis are missing. The broken version is rolled out to all users, turning a small incident into a full outage. Detect with canary metrics, error-rate comparison, and automated analysis; mitigate with gradual rollout, automatic rollback thresholds, and staged promotion.

251. Container image tag mutation breaking reproducible deployments
   A mutable image tag such as "latest" is overwritten with new code, so the same tag points to different content at different times. Deployments are not reproducible, and a redeploy can silently pick up untested or broken code, making rollback and debugging unreliable. Detect with image digest pinning and registry audit; mitigate with immutable tags, content-addressed digests, and signed images. The same failure appears in agent tooling when a tool server's definition is approved once and changed afterwards.

252. Configmap update not propagating to running pods
   A ConfigMap is updated but running pods mount a cached copy and do not reload it, so they keep serving stale configuration. The cluster runs with outdated settings, causing wrong behavior, failed integrations, or security misconfigurations until pods restart. Detect by comparing running config to the desired state; mitigate with config reload mechanisms, mounted-file watches, and rolling restarts on change.

253. Health check misconfiguration routing traffic to unhealthy nodes
   A liveness or readiness probe is misconfigured to pass even when the service is broken, so the load balancer keeps routing traffic to unhealthy nodes. Requests fail or hang, and the service appears up while actually degraded, delaying detection. Detect by validating probe behavior against real failure modes; mitigate with accurate liveness and readiness probes that exercise real dependencies and fail fast.

254. Resource limit misconfiguration causing OOM kills under load
   A memory limit is set too low for the workload, so under normal or peak load the process exceeds it and the kernel OOM-kills the container. The service restarts repeatedly, causing request failures, dropped work, and cascading load on remaining instances. Detect by monitoring OOM kill events and restart counts; mitigate with right-sized limits, load testing, and headroom for spikes.

255. Startup ordering failure where a service starts before its dependencies
   A service begins initialization before its database, message broker, or other dependency is ready, so connection attempts fail and startup aborts. The service crash-loops or enters a broken state, delaying availability and generating error noise. Detect with startup logs and readiness checks; mitigate with dependency-aware startup ordering, retry with backoff, and readiness gates before serving traffic.

256. Rollback failure leaving the system in a mixed-version state
   A rollback that fails partway reverts some instances but not others, leaving old and new versions running together. Behavior is inconsistent, and the system may remain in the broken state the rollback was meant to fix. Detect by monitoring version distribution and error rates; mitigate with tested rollback procedures, atomic deploys, and immutable artifacts that make rollback deterministic.

257. Timezone misconfiguration causing scheduled jobs to run at the wrong time
   A scheduler or job is configured with the wrong timezone, so it fires at an unintended wall-clock time. Batch work runs off-schedule, causing missed deadlines, double processing, or data that is stamped with incorrect times. Detect by monitoring job run times against expectations; mitigate with UTC scheduling, explicit timezone configuration, and validation of cron expressions.

258. Infrastructure-as-code drift between environments
   Manual changes made directly in an environment are not captured in the infrastructure-as-code definitions, so the declared and actual state diverge. Environments behave differently, and redeploys may overwrite or conflict with the manual changes, causing outages. Detect with drift detection tools that compare declared and actual state; mitigate with immutable infrastructure, reconciliation loops, and banning manual changes.

259. Secret version mismatch between services during deployment
   A service is deployed with an old secret version while a dependency has already rotated to a new one, so the two no longer share the same credential. Authentication fails, requests are rejected, and the service cannot reach its dependency until the secret is synchronized. Detect by monitoring authentication failures and secret age; mitigate with coordinated secret rotation, versioned secrets, and overlap windows.

260. Dependency version bump introducing a breaking change
   A dependency upgrade introduces a breaking API or behavior change that was not caught in testing, so the service fails when it runs against the new version. Production breaks with errors or subtle misbehavior that only appears after deployment. Detect with compatibility tests and canary deployments; mitigate with pinned versions, changelog review, and staged upgrades with rollback. For an LLM-based system the dependency is often the model itself: an unpinned model version changes behavior with no code change at all.

261. Deployment pipeline failure leaving partial artifacts
   A pipeline that fails partway leaves some artifacts built or promoted and others missing, so the deployment is incomplete. The service starts with missing or mismatched components, causing crashes or broken functionality. Detect with pipeline status checks and artifact verification; mitigate with atomic artifact promotion, idempotent steps, and gating deployment on full pipeline success.

262. Environment-specific configuration leaking into production
   Configuration intended for staging, development, or another environment leaks into production through a shared config file, default value, or copy-paste error. Production behaves incorrectly, potentially exposing debug endpoints, wrong credentials, or test data. Detect with config validation and environment checks; mitigate with strict environment separation, config review, and rejecting non-production values in production.

263. Deployment artifact checksum mismatch causing corrupt rollout
   An artifact whose checksum does not match the expected value is corrupt, whether truncated during transfer, tampered with, or built from the wrong commit, yet the pipeline promotes it anyway. The rollout serves broken or malicious code, causing crashes, wrong behavior, or a security compromise across the fleet. Detect by verifying checksums at every promotion gate and rejecting mismatches; mitigate with signed artifacts and verified promotion so only integrity-checked builds can deploy.

264. Admission webhook failing closed and blocking its own recovery
   An admission webhook configured to fail closed rejects every request it matches when the webhook is slow or unreachable, and the API server waits up to its timeout before rejecting. If the rules match broadly, the webhook's own replacement pod cannot be created, and the kubelet cannot renew node leases or report pod status either. The result is a cluster-wide write outage that ordinary API writes cannot repair. Detect by alerting on admission latency and rejections per webhook and on a webhook service with no ready endpoints. Mitigate with narrowly scoped rules that exclude the webhook's own namespace and a documented break-glass deletion.

265. Pod termination racing endpoint removal on every rollout
   When a pod is deleted, the kubelet sends its termination signal at the same time as the controllers remove the pod from service endpoints, and nothing orders the two. Proxies, ingress controllers, and cloud load balancers learn about the removal asynchronously, so requests keep arriving after the container has closed its listener or exited. Every rollout and node drain then produces a burst of errors and resets, and long-running jobs are cut off mid-task. Detect by counting errors at the ingress during rollouts. Mitigate with a pre-stop delay sized to the propagation lag, draining of in-flight work before exit, and a grace period longer than the longest job.

266. Pods on a failed node staying running and blocking rescheduling
   A pod's status is derived from the node that hosts it, so when a node dies without warning its pods keep reporting as running until the node is marked unreachable and a grace period has passed. A pod bound to a volume in that node or zone cannot be rescheduled elsewhere and waits as terminating or pending. Services keep routing to dead addresses and stateful members are missing for minutes. Detect with alerts on pods that are not ready while reporting running, and on pods terminating beyond their grace period. Mitigate with a shorter node monitoring grace period, zone-replicated storage, and a runbook for forcing deletion of stateless pods.

267. Node eviction rate limiting during a zone failure
   The controller that evicts pods from failed nodes slows its eviction rate when a large share of the nodes in one zone are unhealthy, to protect the control plane from a flood of requests. In a small cluster evictions stop altogether. The protection fires exactly during a zone outage, so pods stay bound to dead nodes and workloads in that zone do not recover elsewhere until operators intervene. Detect by counting unreachable nodes per zone and comparing it with eviction progress. Mitigate with workloads spread across zones, clusters above the size threshold, where eviction slows instead of stopping, and a rehearsed manual node deletion for zone loss.

268. Disruption budget that forbids every voluntary eviction
   A disruption budget set to allow zero unavailable pods permits no voluntary eviction at all, so node drains never finish and cluster upgrades stall. Operators then bypass the budget with a force flag. A related stall occurs when eviction is refused while the budget is full and the replacement pods never become ready, so the retry loops forever. Detect by alerting on budgets that allow zero disruptions and on nodes stuck in draining. Mitigate with a budget that always allows at least one disruption, one more replica than the minimum, and headroom to reschedule.

269. Missing startup probe killing slow starters
   Without a startup probe, the liveness probe runs from the moment the container starts, and its failure threshold and interval are usually shorter than a cold start that includes cache warm-up or a migration. The container is killed before it finishes booting and restarted into the same race, in a loop that never allows enough time. A fleet-wide restart turns it into an outage. Detect with restart counts where liveness failures occur in the first minute of life. Mitigate with a startup probe whose threshold and interval exceed the worst-case start time.

270. Liveness probe that checks a dependency causing a restart storm
   A liveness probe that calls a database or a downstream service fails for every replica at once when that dependency slows. The kubelet then kills and restarts all replicas together, which drops in-flight work and warm caches and sends a reconnection storm at the dependency that is already struggling. Each further failed restart doubles the restart delay, up to five minutes, so capacity returns slowly. Detect by correlating probe failures and restarts across replicas with dependency latency. Mitigate by keeping liveness checks to the process itself and reporting dependencies through readiness only.

271. Crash loop back-off delaying recovery after the cause is fixed
   A container that keeps failing is restarted with a delay that doubles up to a cap of several minutes, and the delay resets only after the container has run cleanly for a while. After the underlying fault is fixed by a configuration change, each pod still waits for its next back-off slot. Capacity therefore returns slowly and in a staggered wave after a fleet-wide crash. Detect with restart counts and the crash loop status on pods. Mitigate by deleting the affected pods once the cause is fixed and by keeping configuration that can be wrong out of the boot path.

272. Slow container runtime marking healthy nodes unreachable
   The kubelet lists containers on its node on a regular interval, and if that listing takes too long it reports itself unhealthy and the node is marked not ready although every container on it is running. The cause is usually a slow container runtime under disk or CPU pressure on the node. After a further grace period the node is tainted and its pods are evicted, which adds churn to a node that was already stressed. Detect with relist and runtime operation latency metrics. Mitigate with a cap on pods per node, disk headroom, and a runtime that supports container events together with the kubelet EventedPLEG gate.

273. Finalizer left behind blocking resource and namespace deletion
   A custom resource carries a finalizer that only the operator's reconcile loop removes. If the operator is uninstalled, crashes, or stops reconciling that resource, deletion never completes and the namespace that contains it stays terminating, which blocks tenant offboarding and cluster cleanup. Removing the finalizer by hand skips the cleanup of external resources. Detect with alerts on namespaces terminating beyond a target time and on resources with a deletion timestamp and a remaining finalizer. Mitigate with a registry of finalizer owners, best-effort finalization on a deadline, and an uninstall check for pending deletions.

274. Scale-to-zero workload that cannot wake from its own traffic
   A workload scaled to zero replicas on a request-rate signal produces no requests, so the signal stays at zero and the scaler keeps the workload at zero. Rollout commands that wait for readiness time out and pipelines roll back, while a real traffic burst is refused until a pod has been scheduled, pulled its image, and possibly waited for a node to be provisioned. Detect with alerts on the scaled object staying inactive, on rollout timeouts, and on connection failures at the service while it has no ready endpoints. Mitigate with a minimum of one replica, a paused-replicas setting during deploys, and a queue-depth trigger from outside the workload.
