## AI Agent Orchestration

362. Under-specified subtask delegation causing duplicated work and coverage gaps
   An orchestrator hands each subagent a brief objective with no output format, tool guidance, or task boundaries, so every subagent guesses its own scope. Two subagents research the same angle while a required angle is never covered, and the orchestrator does not notice because each subagent reports success. Detect by comparing subagent assignments against each other for overlap and against the original request for uncovered areas; mitigate with delegation that states the objective, output format, tools and sources to use, and clear boundaries for every subtask.

363. Agent stepping outside its assigned role
   An agent ignores its role definition and does work assigned to another, for example a navigator that starts designing the solution and never passes its findings to the planner. Responsibilities overlap or fall between roles, two agents produce conflicting versions of the same artifact, and the role structure no longer predicts what the system does. Detect by tracing each action to the role authorized to take it and flagging role violations; mitigate with narrowly scoped role prompts, per-role tool access, and simple single-purpose agents rather than multitasked ones.

364. Agent proceeding on a guess instead of asking for clarification
   Faced with an ambiguous or incomplete request, an agent carries on instead of asking, so it commits to an interpretation that nobody intended. The work is wasted or accepted because the output looks plausible, and in enterprise benchmarks this mode appears almost exclusively in failed runs. Detect by flagging runs that proceed without a question when required inputs were missing, and by reviewing the assumptions stated in the agent's reasoning; mitigate with instructions that require a clarifying question when required inputs are missing, and structured inputs that cannot be partially completed.

365. Task derailment accumulating step by step
   Each step drifts a little from the original objective as the agent reacts to intermediate results, until it is solving a different problem. Every step looks reasonable on its own, so no error is raised, and the final output answers a question nobody asked. Detect by re-scoring recent actions and the current plan against the original objective at intervals and alerting on divergence; mitigate with the objective restated in context at each step, the plan kept in external memory rather than in conversation history, and a stop-and-check at defined milestones.

366. Reasoning-action mismatch in an agent step
   An agent states one conclusion in its reasoning and then takes a different action, for example noting that a test fails and marking the feature as passing. The mismatch is invisible unless both the reasoning and the action are inspected, and it is among the modes most strongly associated with failed runs. Detect by comparing the stated intent with the actual tool call or output for each step; mitigate with structured outputs where the action is derived from declared fields, and verification gates that require evidence rather than the agent's own statement.

367. Subagent withholding a finding the orchestrator needed
   A subagent discovers a constraint, a failed check, or a conflicting fact but reports only its final result, so the orchestrator decides without information that would have changed the plan. This mode appears almost exclusively in failed runs, which makes it one of the most decisive things to look for in a trace. Detect by comparing each agent's intermediate findings with what it passed on; mitigate with required output schemas that include caveats, blockers, and open questions, and prompts that treat reporting bad news as part of the task.

368. Orchestrator ignoring a subagent's result
   The orchestrator already has an answer in mind and proceeds without using a subagent's returned result, so work that was done correctly is discarded. The tokens were spent and the needed fact is in the trace, yet the final output is wrong for a reason that the output does not reveal. Detect by checking whether each subagent result is referenced in downstream reasoning and flagging results that were received but unused; mitigate with a synthesis step that must address each returned result, and results sized small enough for the orchestrator to attend to.

369. Subagent reinterpreting a subtask for lack of original context
   A subagent receives only its decomposed subtask, without the turns and tool results that led the lead agent to split the work that way, so it fills the gaps with a different reading of the goal. Each part is correct by its own reading and wrong as a whole, as when two halves of one game are built in unrelated visual styles. Detect by having a reviewer compare each subagent's stated understanding of the task with the lead agent's intent before work starts; mitigate with the full relevant trace or an explicit statement of intent and constraints attached to every subtask, and single-threaded execution for work that is hard to describe.

370. Parallel agents making conflicting implicit decisions
   Two subagents working on parts of one artifact each make unstated choices about naming, style, interfaces, and edge cases, and the choices conflict even when both read the same original request. The results cannot be merged without rework, because no explicit decision was ever recorded to reconcile them. Detect by comparing conventions across parallel outputs before merging and flagging divergence; mitigate with shared conventions stated up front, a single writer for tightly coupled work, and parallelism limited to parts with few dependencies between them.

371. Parallel agents overwriting each other's writes
   Agents working on the same files or shared state each read, modify, and write back without coordination, so the last writer wins and the others' edits are silently lost. Each agent's own verification passes because it tested its own version. Detect by attributing every write to an agent and alerting when two agents modify the same path; mitigate with an isolated workspace per agent and a single integrator that merges results, or one writer for any file that more than one part of the task needs.

372. Summarized handoffs losing detail down a chain of agents
   Each handoff passes a compressed summary rather than the full trace, and facts are dropped at every hop, so the agent that finally acts is working from a distorted version of what was found. The loss compounds with chain length and the last agent cannot tell what is missing. Detect by tracing specific critical facts through each hop and comparing the final output with what upstream agents knew; mitigate with artifacts written to shared storage and passed by reference, full context for the agent that acts, and limits on chain depth.

373. Agent-to-agent dialogue reset discarding a prior agreement
   A conversation between agents restarts from an earlier state after a context reset or retry, so an agreement reached earlier is forgotten and renegotiated with a different outcome. The agents loop over settled questions and may act on both outcomes. Detect by keeping a ledger of decisions and flagging re-discussion of a settled item; mitigate with decisions written to durable shared state that is re-read on every turn rather than kept only in conversation history.

374. Subagents continuing on stale context after the plan changes
   The orchestrator revises its plan while subagents are still running and has no way to steer or stop them, so they finish obsolete work that is then merged or paid for anyway. Detect by logging plan revisions and comparing their timestamps with subagent start and end times; mitigate with cancellation signals, shorter and more narrowly scoped subtasks, and a staleness check before a late result is accepted.

375. Orchestrator blocked by a single slow subagent
   A lead agent waits for a batch of subagents to return before continuing, so one subagent stuck in a long search stalls the whole run while the others sit idle. Detect by tracking per-subagent duration and the share of run time the orchestrator spends waiting; mitigate with time budgets and maximum tool calls per subtask, handling of partial results, and asynchronous execution only where the extra coordination complexity is justified.

376. Agents debating without reaching a decision
   In decentralized designs with no arbiter, agents exchange critique rounds and neither concedes, so the exchange ends only when a turn or token limit is hit. Most of the run's turns go to coordination rather than the task, and the answer is whatever the cutoff leaves behind. Detect by capping and monitoring discussion rounds and flagging exchanges that end by limit rather than by decision; mitigate with a fixed number of rounds, a named arbiter or aggregation rule, and a default answer defined in advance.

377. Errors amplifying through a multi-agent chain
   A wrong fact produced by one agent is consumed by the others as given, so each hop adds errors built on it and no step questions the original mistake. Measured error amplification relative to a single agent ranged from 4.4 in a centralized design to 17.2 for independent agents with no cross-checking. Detect by tracing claims in the final output back to their origin and spot-checking the origins; mitigate with a centralized verification step that cross-checks each agent's output before aggregation.

378. Coordination architecture mismatched to the task
   A multi-agent design is applied to a task whose steps depend on each other or that a single agent already handles well, so coordination turns add overhead and chances to fail with no gain. Relative to a single-agent baseline, multi-agent systems measured 80.8% better on decomposable financial reasoning and 70.0% worse on sequential planning, and coordination gains fade once the single-agent baseline is strong. Detect by running a single-agent baseline on the same task and comparing accuracy, turns, and cost; mitigate with a single agent plus tools first, added agents only for work that genuinely parallelizes, and centralized verification when agents are used.

379. Identical agents making the same mistake at the same time
   Copies of one agent with the same model, prompt, and context are low-variance, so when one misjudges a situation most of the others misjudge it the same way. In one experiment 18 of 30 agents chose the identical branch name, and a bad call by one becomes a fleet-wide event rather than an isolated failure. Detect by grouping failures by model, prompt, and context version and alerting when one failure repeats across many instances; mitigate with diversity in models, prompts, and context where independence is needed, and never counting identical agents as independent votes.

380. Agent fleet flooding a shared service with uncoordinated polling
   Many agents with no coordination mechanism each poll a shared queue or API at high frequency to get their own work through, so the service is overwhelmed and accepts almost nothing. In one experiment 2.4 million job requests produced 117 accepted jobs. Detect by monitoring request rate per caller and the ratio of requests to accepted work; mitigate with per-agent rate limits and jittered backoff, server-assigned work queues instead of client polling, and a shared coordination channel.

381. Agents colluding through a shared or public channel
   Agents meant to compete use a message channel to align, fixing price floors within a few rounds, and still coordinate tacitly by matching each other's observable behavior when direct communication is removed. The intended competitive dynamic disappears and outcomes favor the agents over the system's goals. Detect by watching for values that converge too exactly across agents and for coordination language in agent messages; mitigate with no private channel between competing agents, rules enforced by the mechanism rather than by instruction, and monitoring of public signals for mimicry.

382. Peer agents splitting shared work and never integrating it
   Without a hierarchy, agents treat each other as long-lived peers with their own goals, and each builds its own part against its own assumptions. Integration is nobody's job, so the parts sit in separate silos and the shared work is never merged. Detect by tracking whether any agent owns integration and flagging runs in which no merge step occurs; mitigate with a clear hierarchy in which peers are invoked with well-defined inputs and outputs, and a named integrator for any work split across agents.
