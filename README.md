# Distributed Systems Failure Points

A catalog of 300 production failure modes for distributed systems. Each entry names a specific failure mode and gives a full-paragraph excerpt covering the mechanism, its impact, and how to detect and mitigate it.

## Purpose

Distributed systems fail in a finite, well-understood set of ways: partitions, clock skew, replication lag, resource exhaustion, consensus breakdowns, and the operational and security mistakes that trigger them. This catalog collects those failure modes in one place so they can be referenced during design, review, and incident response instead of being rediscovered one outage at a time.

## Structure

The catalog is a single Markdown file, [`observations.md`](observations.md), organized into 17 categories:

| Category | Entries |
|---|---|
| Network | 72 |
| Consensus | 38 |
| Replication | 24 |
| Deployment | 21 |
| Security | 18 |
| Application | 17 |
| Dependencies | 14 |
| Storage | 13 |
| Resources | 13 |
| Clock | 12 |
| Data Integrity | 12 |
| Operational | 10 |
| General Software Defects | 9 |
| Observability | 8 |
| Messaging & Events | 7 |
| Caching | 7 |
| Operational Process Failures | 5 |

The last two categories are quarantined: **General Software Defects** holds bugs that are not specific to distributed systems (SQL injection, off-by-one errors, integer overflow), and **Operational Process Failures** holds incident-management process failures (postmortem follow-through, escalation paths). They are kept separate so the core catalog stays focused on distributed-systems failure modes.

## Entry format

Each entry is a numbered title followed by a full-paragraph excerpt in a fixed order:

1. **Mechanism** — what happens and why.
2. **Impact** — what breaks, what the operator or user observes.
3. **Detection and mitigation** — how to catch it and how to prevent it.

Example:

> **1. Network partition causing split-brain in distributed systems**
>
> A network partition isolates a subset of nodes from the rest, so each side independently elects its own leader and both accept conflicting writes, causing the cluster to diverge until the partition heals. Two nodes simultaneously believe they hold leadership, which corrupts replicated state and can lose committed data when the partition resolves and one side's writes are discarded. Detect via fencing tokens or epoch numbers that reject stale leaders. Mitigate with majority quorum and leader leases so only one side can ever hold leadership.

## Use cases

- **Design review** — check a new system or protocol against the relevant categories before it ships.
- **Incident postmortem** — map an outage to its failure mode and confirm the mitigation is in place.
- **Chaos engineering** — pick failure modes to inject and verify the system degrades gracefully.
- **Failure-mode analysis** — enumerate what can go wrong in a component and prioritize mitigations.

## Contributing

Entries should name a specific, plausible failure mode and follow the excerpt format above. Avoid tautologies (the cause should not be the effect), vague catch-alls, and bugs that would fail in a single process just as readily as in a distributed system.
