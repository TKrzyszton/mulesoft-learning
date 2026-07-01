# Lesson 3 — Point-to-Point vs Pub/Sub

**Phase:** 1 — Integration Architecture Fundamentals  
**Week:** 1  
**Day:** Wednesday  

---

## Key Concepts

### Point-to-Point Channel

One sender, one receiver. Message delivered exactly once to exactly one consumer.  
Even if multiple consumers listen on the same channel — only one gets the message (**Competing Consumers** pattern).

**Use when:** you have a specific instruction for a specific system. Command Messages almost always go P2P.

**Key property:** message waits in the queue if the receiver is unavailable. When it comes back — it processes the message. Nothing is lost.

### Publish-Subscribe Channel

One sender, many receivers. Every subscriber gets their own copy of the message.  
The sender does not know and does not care who is listening — it just publishes the event.

**Use when:** something happened and multiple systems may be interested. Event Messages almost always go Pub/Sub.

**Key property:** if a subscriber is offline when the message is published — depending on implementation, the message may be lost. Requires careful design (→ Lesson 4: Dead Letter Queue).

### Comparison

| | Point-to-Point | Pub/Sub |
|---|---|---|
| Receivers | Exactly 1 | Many |
| Sender knows receiver? | Yes (indirectly) | No |
| Coupling | Higher | Lower |
| Typical message type | Command | Event |
| Receiver unavailable? | Message waits in queue | Message may be lost |
| Debugging | Simpler | Harder — who got it, who didn't? |
| Message ordering | Easier to preserve | Harder |

### The Golden Rule

> If you care about **what happened** → Pub/Sub (Event)  
> If you care about **something getting done** → P2P (Command)

### Coupling and Pub/Sub

Pub/Sub produces **lower coupling** because systems don't need to know about each other — they only interact with the channel. Adding a new subscriber requires zero changes to the publisher.

However, lower coupling comes at a cost:
- Harder to guarantee delivery
- Harder to preserve message order
- Harder to debug ("who received it?")
- Getting a response back is complex (async request-reply pattern)

**Pub/Sub is not "better" — it's appropriate for different scenarios.** An architect chooses consciously, not by default.

### Competing Consumers

If 3 consumers listen on the same **Point-to-Point** channel — exactly **1** of them gets the message.  
This is the **Competing Consumers** pattern — used for scaling: multiple instances of the same service process messages in parallel from one queue, each getting a different message.

---

## Diagrams

![Point-to-Point vs Pub/Sub](diagrams/p2p-vs-pubsub-diagram.png)

**Diagram 1 — Pub/Sub:**  
Synerise publishes `customer.registered` → Pub/Sub channel → CRM + Email Platform + Analytics.  
Synerise does not know who subscribes.

**Diagram 2 — Point-to-Point:**  
Synerise sends Command `update.customer.profile` → P2P Queue → exactly one receiver: CRM.

---

## My Environment — Synerise Examples

| Scenario | Pattern | Reasoning |
|---|---|---|
| Synerise sends purchase data to loyalty system | **P2P** | One specific recipient, Command "update points" |
| Customer changes marketing consent | **Pub/Sub** | Event that multiple systems care about (CRM, email platform, data warehouse) |
| Synerise triggers a specific transactional email | **P2P** | One specific email platform, one recipient |
| New customer profile from POS terminal | **P2P → consider Pub/Sub** | If only Synerise receives it — P2P. But typically a new customer event interests CRM, loyalty, data warehouse too — making it a Pub/Sub `customer.created` event where Synerise is one of the subscribers |
| Synerise exports segment to Google Ads + Facebook Ads | **Pub/Sub** | Two subscribers of the same `segment.exported` event — Synerise doesn't need to know how many recipients |

### Key insight from POS example
Always ask: **"Is only one system interested in this message?"**  
If the answer might be "no" — consider Pub/Sub even if today there's only one consumer.  
Adding a second consumer to P2P requires changing the publisher. Adding a second subscriber to Pub/Sub requires zero changes.

---

## Architectural Observations

### 1. Competing Consumers = scaling pattern
3 consumers on the same P2P queue → only 1 gets each message.  
Used to scale processing: 3 instances of the same service process the queue in parallel.  
Each message is processed exactly once — no duplicates.

### 2. Pub/Sub subscriber offline = potential data loss
Unlike P2P (where the queue holds the message), Pub/Sub has no built-in "wait for subscriber".  
Solution: **Dead Letter Queue** — covered in Lesson 4.

### 3. Low coupling ≠ always better
Pub/Sub reduces coupling but increases complexity in debugging, ordering, and delivery guarantees.  
Choose based on requirements, not instinct.

---

## Open Questions
- What happens to a Pub/Sub message when a subscriber is offline? → **Lesson 4 (Dead Letter Queue)**
- How does MuleSoft implement Pub/Sub? (Anypoint MQ topics) → **Phase 2, Week 12**
- Carried over: How to design retry for webhooks that don't support it? → **Lesson 4**

---

## Anki Cards

| Front | Back |
|---|---|
| Point-to-Point channel — how many consumers receive the message? | Exactly 1 — even if multiple consumers are listening |
| What is the Competing Consumers pattern? | Multiple consumers listen on the same P2P queue — each message goes to exactly one. Used for scaling. |
| Pub/Sub — how many consumers receive the message? | All subscribers — each gets their own copy |
| Which pattern produces lower coupling and why? | Pub/Sub — sender doesn't know who's listening, adding a subscriber requires zero changes to the publisher |
| Command Message → which channel type? | Point-to-Point |
| Event Message → which channel type? | Pub/Sub |
| What happens to a P2P message if the receiver is unavailable? | It waits in the queue until the receiver comes back |
| What is the risk with Pub/Sub when a subscriber is offline? | The message may be lost — solved by Dead Letter Queue (Lesson 4) |
| The golden rule: "if you care about something getting done" → ? | Point-to-Point (Command) |
| The golden rule: "if you care about what happened" → ? | Pub/Sub (Event) |

---

## Resources
- [EIP — Point-to-Point Channel](https://www.enterpriseintegrationpatterns.com/patterns/messaging/PointToPointChannel.html)
- [EIP — Publish-Subscribe Channel](https://www.enterpriseintegrationpatterns.com/patterns/messaging/PublishSubscribeChannel.html)
- [EIP — Competing Consumers](https://www.enterpriseintegrationpatterns.com/patterns/messaging/CompetingConsumers.html)

---

*Next lesson: **Lesson 4 — Dead Letter Channel***  
*You will learn: what happens when a message can't be delivered — and how to design retry and DLQ for Synerise webhooks that currently have no retry.*
