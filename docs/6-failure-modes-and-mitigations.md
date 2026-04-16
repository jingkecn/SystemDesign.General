Failure Modes and Mitigations
=============================

What
----

A complete design accounts for what happens when things go wrong.
Identifying failure modes demonstrates production-grade thinking and distinguishes a theoretical design from a deployable one.

How
---

For each major component, answer:

- What fails? How does it fail? (Crash, latency, data loss, split-brain?)
- Who is affected?
- What is the mitigation or recovery path?

Example
-------

| Failure                              | Impact                                                   | Mitigation                                                                                                   |
| ------------------------------------ | -------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| External Service becomes unavailable | Downstream actions                                       | Circuit Breaker opens; UI shows "data delayed" indicator; polling resumes on recovery                        |
| WebSocket Hub process crashes        | All connected users disconnected                         | Client auto-reconnect with exponential backoff; state snapshot served on reconnect                           |
| Database unavailable                 | Transaction cannot be executed; chat cannot be persisted | Primary endpoints return 503; chat degrades to in-memory broadcast without persistence; MQ writes for replay |
| Cache miss or Redis down             | Hub falls back to DB reads for snapshot; latency spikes  | Circuit Breaker on cache; fallback to DB; alert triggered                                                    |
| Chat message storm                   | Hub overwhelmed by high-frequency senders                | Per-user rate-limiting(e.g., 5 msg/s); server-side drop with client notification                             |
| ...                                  | ...                                                      | ...                                                                                                          |
