## Messaging & Events

322. Event ordering violation after partition heal
   Events produced during a partition are delivered out of order after the partition heals, so consumers process them in the wrong sequence. State derived from ordered events becomes inconsistent, and later events may overwrite earlier ones. Detect with sequence numbers or offsets that expose gaps and reordering. Mitigate with ordered delivery guarantees where possible, or sequence-aware consumers that buffer and reorder events, and reject or defer out-of-sequence messages.

323. Duplicate event delivery causing double processing
   At-least-once delivery redelivers an event after a timeout or crash, so a non-idempotent consumer processes it twice. The result is duplicated side effects such as double charges, double inserts, or repeated notifications. Detect with deduplication keys and by monitoring for duplicate processing. Mitigate with idempotent consumers that key every operation on a stable event identifier, and broker-side or consumer-side deduplication that drops already-seen events.

324. Event handler failure causing message consumption halt
   A handler that fails without acknowledging stops the consumer, so messages accumulate and processing halts. The queue backs up, downstream systems starve, and the failure cascades as the backlog grows. Detect by monitoring consumer lag and alerting when it exceeds a threshold. Mitigate with error handling that always acknowledges or rejects a message, dead-letter routing that quarantines poison messages, and retry with backoff before giving up.

325. Consumer lag causing stale event processing
   A consumer that lags behind processes events long after they occurred, so it acts on stale state. Decisions based on outdated data produce wrong results, and the consumer may overwrite newer state with older values. Detect by monitoring consumer lag against the stream head and alerting on sustained drift. Mitigate with consumer scaling to add parallel partitions, lag alerts that page before the gap grows, and backpressure that sheds or batches work.

326. Event retention expiry causing replay loss
   Events expire from the log before a lagging consumer replays them, so the consumer cannot catch up and loses state. The consumer permanently misses events and its view diverges from the source of truth. Detect by monitoring retention against consumer lag and alerting when lag approaches the retention window. Mitigate with adequate retention that exceeds the maximum expected lag, replay windows that preserve events during maintenance, and compacted topics for state that must never expire.

327. Outbox pattern failure causing event loss on commit
   A failure in the outbox pattern loses the event that should accompany a committed transaction, so downstream consumers miss the change. The database commits but the message is never published, leaving systems out of sync. Detect by reconciling the outbox table with published events and alerting on orphaned rows. Mitigate with a transactional outbox that writes the event in the same transaction as the data change, and a relay that reliably publishes and marks outbox rows as sent.

328. Event schema evolution breaking downstream consumers
   An event schema change that is not backward-compatible breaks downstream consumers, so they fail to parse new events. Consumers crash, drop messages, or misread fields, and the failure spreads as each consumer hits the incompatible event. Detect with schema compatibility checks in CI that reject breaking changes. Mitigate with versioned, backward-compatible schemas that only add optional fields, and consumer upgrades that are staged ahead of producer changes.
