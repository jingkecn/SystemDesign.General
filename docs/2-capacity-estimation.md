Capacity Estimation
===================

> [!NOTE]
> The 4 dimensions(**traffic**, **storage**, **bandwidth**, **memory**), with all calculations worked through for the system.

What
----

Capacity estimation **translates** requirements into concrete numbers.
It ***grounds the architecture*** in reality and ***drives decisions*** about storage technology, caching strategy, and infrastructure sizing.
~~Precise numbers~~ are not the goal - ***order-of-magnitude reasoning*** is.

How
---

Work through the 4 dimensions in sequence.

***Traffic*** tells us whether our problem is fundamentally a *read* problem or a *write* problem, therefore where to invest: write throughput? broadcast efficiency? caching?...

```text
DAU = Daily Active Users
RPS = Requests Per Second (read)
WPS = Writes Per Second
Peak = assume 2-5x average for burst-y system
```

***Storage*** tells us what dominates and therefore what needs a lifecycle strategy.
Tiny but precious records? Keep them forever.
Voluminous and transient records? TTL and cleanup.
Without the math we might design both with the same storage approach and either waste money or lose data.

```text
Record size (bytes) x writes/day x retention period = storage required
Add replication factor (typically 3x for distributed systems)
```

***Bandwidth*** tells you whether the problem is even solvable on a single machine, or whether distribution is mandatory from day one.
For example, 20 MB/s outbound is comfortably within a single server's capacity - so horizontal scaling is a ~~*future* concern~~, not an immediate one.
That's a design simplification we can defend.

```text
Payload size (bytes) x RPS = bytes/second in or out
Convert: 1 MB/s = 8 Mbps
```

**Memory / cache estimation:**

```text
Working set = data accessed in a rolling time window
Cache hit ratio target drives cache size
```

> [!NOTE]
>
> - Use round numbers
> - State your assumption explicitly

---

> [!IMPORTANT]
> ***The dominant flow*** is the real output of the whole exercise.
> It's the one number that should shape our architecture.
> For example, in a system where the real-time fan-out dominates, once we see this dominant flow, the argument for server-push over client-polling becomes obvious from math, not just taste.

Example ([Sport Bets](https://github.com/jingkecn/SystemDesign.SportBets))
--------------------------------------------------------------------------

**Assumptions:**

- 100K DAU
- 10 concurrent games at peak
- Average 5K users per game at peak
- Score / odds push: very 5 seconds per game
- Chat: average 2 messages/min/user during a game (burst-y)
- Bet placement: average 3 bets/session

**Traffic:**

| Flow                        | Calculation                          | Result                                                    |
| --------------------------- | ------------------------------------ | --------------------------------------------------------- |
| Score / odds pushes (write) | 10 games x 1 update/5s               | 2 writes/s to cache                                       |
| Score / odds fan-out (read) | 10 games x 5K users/game x 1 push/5s | 10K pushes/s                                              |
| Chat messages (write)       | 10 games x 5K users/game x 2 msg/60s | ~1667 write/s                                             |
| Chat fan-out (read)         | 1667 msg/s x avg 5K recipients/game  | 8.3M deliveries/s/game (via groups, not individual sends) |
| Bet placement               | 100K users/day(=24x60x60s) x 3 bets  | ~3.5 writes/s (low, spike at game end)                    |

> [!NOTE]
> **Key insight:**
> Chat delivery is the highest-volume flow, but it is handled by broadcast groups - not individual message routing - which dramatically reduces per-connection work.

**Storage:**

| Data          | Record | Volume                                       | Retention         | Total        |
| ------------- | ------ | -------------------------------------------- | ----------------- | ------------ |
| Chat messages | 500B   | 1667 writes/s x (24x60x60)s = ~144M msg/day  | 7 days            | ~504 GB/week |
| Bets          | 1KB    | ~300K bets/day                               | Indefinite        | ~300 MB/day  |
| Scores/odds   | 2KB    | 2 writes/s x (24x60x60)s = ~173K records/day | Current game only | Ephemeral    |

> [!NOTE]
> **Key insight:**
> Chat storage dominates.
> An archival or TTL-based cleanup strategy is required.

**Bandwidth:**

| Flow                       | Payload | RPS  | Bandwidth         |
| -------------------------- | ------- | ---- | ----------------- |
| Score / odds push to users | 2KB     | 10K  | ~20 MB/s outbound |
| Chat message injection     | 500B    | 1667 | ~0.8 MB/s inbound |

> [!NOTE]
> **Key insight:**
> 20 MB/s outbound is manageable on a single server but warrants horizontal scaling planning.
