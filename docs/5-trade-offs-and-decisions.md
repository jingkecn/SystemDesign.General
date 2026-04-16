Trade-Offs and Decisions
========================

What
----

Every design decision is a trade-off.
This step makes those trade-offs explicit defends the choice made.
In an interview, this is where senior engineers distinguish themselves.

Use the **Decision Record** format for each significant choice.

How
---

For each decision:

- **Context** - what problem does this decision solve?
- **Options considered** - at least 2 alternatives
- **Decision** - what was chosen?
- **Rationale** - why this option over the others?
- **Consequences** - what does this decision make harder or impossible?

Example: Decision Records
-------------------------

**DR-01 - Frontend-to-backend real-time protocol:**

| Field        | Content                                                                                                                                                                                                                                                                                      |
| ------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Context      | Chat requires low-latency delivery to thousands of concurrent users                                                                                                                                                                                                                          |
| Options      | (A) WebSocket, (B) Server-Sent Events + separate chat API, (C) HTTP long polling                                                                                                                                                                                                             |
| Decision     | WebSocket (SignalR)                                                                                                                                                                                                                                                                          |
| Rationale    | Chat is bidirectional - SSE cannot carry client-to-server messages without a separate HTTP channel, adding complexity. WebSocket handles both direction in a single connection. SignalR adds connection management, group broadcasting, and automatic transport fallback over raw WebSocket. |
| Consequences | Stateful connections require careful scaling (sticky sessions or a distributed back-plane). Server resources usage is higher than SSE for read-only flows.                                                                                                                                   |

**DR-02 - Backend-to-external-service protocol:**

| Field        | Content                                                                                                                                                                                                                                                              |
| ------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Context      | External service capabilities are unknown; reliability is not guaranteed                                                                                                                                                                                             |
| Options      | (A) WebHook preferred with REST polling fallback, (B) WebSocket stream, (C) REST polling only                                                                                                                                                                        |
| Decision     | Adaptive: WebHook preferred, REST polling fallback                                                                                                                                                                                                                   |
| Rationale    | WebHook minimizes latency when supported. REST polling provides a reliable fallback without depending on the external service maintaining a long-lived connection. Locking into WebSocket streaming introduces a hard dependency on a service whose SLA is not known |
| Consequences | Polling introduces a latency floor equal to the poll interval. Adaptive logic adds implementation complexity (capability probe, automatic fallback, etc.)                                                                                                            |

**DR-03 - Chat scaling strategy:**

| Field        | Content                                                                                                                                                                                                                                                                                          |
| ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Context      | Chat must scale to thousands of users per group; deployment model is single server initially                                                                                                                                                                                                     |
| Options      | (A) Single server with in-process SignalR groups, (B) Separate chat microservice, (C) Redis back-plane from day one                                                                                                                                                                              |
| Decision     | (A) Single server with in-process groups + (C) Redis back-plane as a planned upgrade path                                                                                                                                                                                                        |
| Rationale    | A single server with SignalR groups handles tens of thousands of connections without additional infrastructure. Introducing Redis adds operational complexity before it's needed. The architecture isolates the Chat Service so the back-plane can be added without redesigning other components |
| Consequences | Horizontal scaling requires adding a Redis back-plane. Single server is a single point of failure for WebSocket connections (mitigation: connection recovery with state snapshot)                                                                                                                |
