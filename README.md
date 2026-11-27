# Distributed Systems Failure Points

A catalog of 467 production failure modes for distributed systems, including AI agent systems. Each entry names a specific failure mode and gives a full-paragraph excerpt covering the mechanism, its impact, and how to detect and mitigate it.

## Purpose

Distributed systems fail in a finite, well-understood set of ways: partitions, clock skew, replication lag, resource exhaustion, consensus breakdowns, and the operational and security mistakes that trigger them. This catalog collects those failure modes in one place so they can be referenced during design, review, and incident response instead of being rediscovered one outage at a time.

## Structure

The catalog is an index, [`observations.md`](observations.md), pointing to one Markdown file per category in [`observations/`](observations/). Each file is named after its category in lowercase with hyphens, for example `observations/ai-agent-security.md`. The index lists every file with its entry count and number range. The 21 categories are:

| Category | Entries |
|---|---|
| Network | 81 |
| Resources | 40 |
| AI Agent Security | 38 |
| Security | 32 |
| Deployment | 32 |
| Consensus | 32 |
| AI Agent Reliability | 26 |
| Application | 24 |
| Replication | 23 |
| AI Agent Orchestration | 21 |
| Dependencies | 16 |
| Storage | 15 |
| Data Integrity | 15 |
| AI Agent Context & Memory | 15 |
| Clock | 11 |
| Operational | 10 |
| General Software Defects | 9 |
| Observability | 8 |
| Messaging & Events | 7 |
| Caching | 7 |
| Operational Process Failures | 5 |

The last two categories are quarantined: **General Software Defects** holds bugs that are not specific to distributed systems (SQL injection, off-by-one errors, integer overflow), and **Operational Process Failures** holds incident-management process failures (postmortem follow-through, escalation paths). They are kept separate so the core catalog stays focused on distributed-systems failure modes.

Four categories cover LLM-based agents and multi-agent systems: **AI Agent Orchestration**, **AI Agent Context & Memory**, **AI Agent Reliability**, and **AI Agent Security**. Agents inherit the classic failure modes above and add their own, because their components are non-deterministic, follow natural-language instructions, and work inside a finite context window.

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

Entries should name a specific, plausible failure mode and follow the excerpt format above. Avoid tautologies (the cause should not be the effect), vague catch-alls, and bugs that would fail in a single process just as readily as in a distributed system. AI agent entries follow the same rule: a mode must be specific to agents, such as context limits, non-determinism, tool use, or inter-agent communication, rather than a general LLM application bug.
