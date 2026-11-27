# Distributed Systems Failure Points

A catalog of production failure modes for distributed systems, including LLM-based agent systems. Each category lives in its own file, so a review can read only the categories a change touches. Each entry names a failure mode and gives a full-paragraph excerpt covering the mechanism, its impact, and how to detect and mitigate it.

Entries are numbered continuously across categories, so the range column shows where each file starts and ends. The last column says when a change calls for that file.

| Category | File | Entries | Numbers | Read it when a change touches |
|---|---|---|---|---|
| Network | [observations/network.md](observations/network.md) | 81 | 1 to 81 | Partitions, packet loss, latency, DNS, firewalls, load balancing, cloud networking |
| Consensus | [observations/consensus.md](observations/consensus.md) | 32 | 82 to 113 | Leader election, quorum, split-brain, membership changes, log and snapshot handling |
| Replication | [observations/replication.md](observations/replication.md) | 23 | 114 to 136 | Replica lag, stale reads, failover, partial replication, cross-region writes |
| Clock | [observations/clock.md](observations/clock.md) | 11 | 137 to 147 | Clock skew and drift, time sync, leap seconds, ordering by timestamp |
| Storage | [observations/storage.md](observations/storage.md) | 15 | 148 to 162 | Disks, write-ahead logs, filesystem corruption, volumes, the cluster's backing store |
| Resources | [observations/resources.md](observations/resources.md) | 40 | 163 to 202 | CPU, memory, connection pools, autoscaling, load shedding, tenancy quotas |
| Security | [observations/security.md](observations/security.md) | 32 | 203 to 234 | Credentials, tokens, certificates, workload identity, encryption, tenant isolation |
| Deployment | [observations/deployment.md](observations/deployment.md) | 32 | 235 to 266 | Rollouts, config drift, feature flags, migrations, probes, graceful shutdown, webhooks, operators |
| Application | [observations/application.md](observations/application.md) | 24 | 267 to 290 | Service logic under load: leaks, deadlocks, queues, retries, thundering herds, transaction coordinators |
| Dependencies | [observations/dependencies.md](observations/dependencies.md) | 16 | 291 to 306 | Upstream APIs and third parties, SDKs, brokers, webhooks, registries, object storage |
| Data Integrity | [observations/data-integrity.md](observations/data-integrity.md) | 15 | 307 to 321 | Schema and encoding mismatches, idempotency, backups and restores, retention, metering |
| Messaging & Events | [observations/messaging-and-events.md](observations/messaging-and-events.md) | 7 | 322 to 328 | Event ordering, duplicate delivery, consumer lag, retention, outbox, schema evolution |
| Caching | [observations/caching.md](observations/caching.md) | 7 | 329 to 335 | Eviction, cache keys, negative caching, warmup, stampedes |
| Observability | [observations/observability.md](observations/observability.md) | 8 | 336 to 343 | Metrics, logs, traces, dashboards, alerting |
| Operational | [observations/operational.md](observations/operational.md) | 10 | 344 to 353 | Operator actions, runbooks, on-call handoff, incident response, change windows |
| AI Agent Orchestration | [observations/ai-agent-orchestration.md](observations/ai-agent-orchestration.md) | 21 | 354 to 374 | Splitting work across agents, handoffs, parallel agents, coordination, convergence |
| AI Agent Context & Memory | [observations/ai-agent-context-and-memory.md](observations/ai-agent-context-and-memory.md) | 15 | 375 to 389 | What an agent sees: context window, compaction, retrieval, memory |
| AI Agent Reliability | [observations/ai-agent-reliability.md](observations/ai-agent-reliability.md) | 26 | 390 to 415 | Whether a run completes correctly: tool calls, loops, retries, model changes, provider limits, cost, evals |
| AI Agent Security | [observations/ai-agent-security.md](observations/ai-agent-security.md) | 38 | 416 to 453 | Misuse of an agent: injection, tools, credentials, approvals, memory poisoning, LLM security review |
| General Software Defects | [observations/general-software-defects.md](observations/general-software-defects.md) | 9 | 454 to 462 | Ordinary bugs that also fail in a single process; check last |
| Operational Process Failures | [observations/operational-process-failures.md](observations/operational-process-failures.md) | 5 | 463 to 467 | Postmortems, incident communication and escalation, audit trails |
