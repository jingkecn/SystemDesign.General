Requirements Clarification
==========================

> [!NOTE]
>
> - How to split functional from non-functional requirements
> - What questions to ask
> - Full FR/NFR tables derived from the documents of a worked example

What
----

Before touching any architecture, we must understand what we are actually building.

Requirements split into 2 categories:

- ***Functional requirements (FR)***: what the system does - features, behaviors, user actions.
- ***Non-functional requirements (NFR)***: how the system performs - scale, latency, availability, consistency, durability.

> [!IMPORTANT]
> ~~Skipping this step~~ is the single most common ***mistake*** in system design.
> ***Assumptions made here propagate through every later decision.***

How
---

Ask clarifying questions in this order:

**Functional:**

- Who are the users? What do they do?
- What are the core user cases? (limit to 3-5 for an interview)
- What is explicitly ~~out of scope~~?
- Are there external systems to integrate? Who owns them?

**Non-functional:**

- What is the expected user volume? Concurrent users? Peak load?
- What are the latency expectations? (Real-time? Near real-time? Batch?)
- What consistency model is required? (Strong? Eventual?)
- What is the availability target? (99.9%? 99.99%?)
- What is the data retention policy?
- Are there regulatory or compliance constraints?

**Clarifying the domain:**

- What does "success" look like for a single transaction?
- What is the cost of failure? (Data loss? Financial loss? User frustration?)

Example ([Sport Bets](https://github.com/jingkecn/SystemDesign.SportBets))
--------------------------------------------------------------------------

**Given from the project description:**

- Frontend: live scores and odds, chat, betting forms
- Backend: API for publishing data, chat implementation
- External: Sports Analytics service (out of scope, capabilities unknown)

**Functional requirements:**

| #   | Requirement                                                         |
| --- | ------------------------------------------------------------------- |
| FR1 | Users can view live game scores updated in real time                |
| FR2 | Users can view live odds updated in real time                       |
| FR3 | Users can place bets on live games                                  |
| FR4 | Users can send and receive chat messages scoped to a game           |
| FR5 | Backend consumes live data from an external Sport Analytics service |
| FR6 | Chat history is available to users who join mid-game                |

**Non-functional requirements:**

| #    | Requirement                          | Assumption                               |
| ---- | ------------------------------------ | ---------------------------------------- |
| NFR1 | Low latency for score / odds updates | < 1s end-to-end                          |
| NFR2 | High concurrency                     | Thousands of simultaneous users per game |
| NFR3 | High availability                    | 99.9% uptime minimum                     |
| NFR4 | Chat message ordering                | Per-game ordering guaranteed             |
| NFR5 | Bet consistency                      | No double bets, no lost bets             |
| NFR6 | Data retention                       | Chat persisted for game duration + 7days |

**Out of scope:**
Sport Analytics internals, payment processing, user authentication, odds calculation engine.
