# Lesson 2 — Message as the Building Block of Integration

**Phase:** 1 — Integration Architecture Fundamentals  
**Week:** 1  
**Day:** Tuesday  

---

## Key Concepts

### Message Structure

Every integration, regardless of the tool, transmits a **Message** with three core parts:

- **Header** — metadata about the message: who sent it, when, what type, what format. Not the content itself, just information *about* the message.
- **Body** — the actual payload. What the target system needs to process.
- **Correlation ID** — a unique identifier linking together all messages belonging to the same business process execution. Critical for debugging "what happened to order #1234" across a system made of 5 microservices.

> ⚠️ **Important distinction:** EIP "Header" (business message metadata) is *not* the same as HTTP protocol headers (`Content-Type`, `Authorization`, `Accept`). HTTP headers describe *how to transport* the message. EIP headers describe *what the business message is* — and in practice they often live inside the request body as a separate object, not in HTTP headers.

### Message Types (EIP)

| Type | Meaning | Example | Typical channel |
|---|---|---|---|
| **Command Message** | "Do something" | "Update customer profile" | Point-to-Point (one recipient) |
| **Event Message** | "Something happened" | "Customer registered" | Pub/Sub (can have many subscribers) |
| **Document Message** | Pure data transfer, no action intent | Data export | Either |

### Correlation ID vs Business Entity ID

A common beginner mistake: confusing `correlationId` (identifies **one process execution**) with `customerId` / `orderId` (identifies a **business entity**).

If the same customer registers twice (e.g. once via web, once via mobile app), these are **two separate processes** — each gets its own Correlation ID, even though `customerId` is identical.

---

## My Environment — Synerise Automation Webhooks

### Mapping Synerise concepts to EIP

| Synerise concept | EIP equivalent | Notes |
|---|---|---|
| `diagramId` | **Not** a Correlation ID | Identifies the automation **definition** (the "class"), not a specific execution. Same value across thousands of runs. |
| `journeyId` | **Correlation ID** ✅ | Identifies one specific automation **execution** (the "instance"). Unique per run — this is the real correlation identifier. |
| HTTP `Content-Type`, `Accept`, `Authorization` | Transport-layer headers, not EIP Header | Describe *how* to send the request, not *what* the business message is. |
| Update Customer API call | **Command Message** | "Update customer data from mobile app" — explicit instruction, one recipient. |

### Real example from Synerise

A webhook triggered by an Automation diagram to update a customer record:

```json
{
  "header": {
    "correlationId": "journeyId-here",   // unique per execution — the real correlation ID
    "diagramId": "diagram-definition-id", // identifies the automation flow, NOT the execution
    "timestamp": "2026-06-30T21:37:00Z",
    "sourceSystem": "Mobile App",
    "messageType": "Command"
  },
  "body": {
    "customerId": "1457909",
    "email": "jan.kowalski@gmail.com",
    "updatedAt": "2026-06-30T21:37:00Z"
  }
}
```

---

## Architectural Observations

### 1. diagramId ≠ Correlation ID
`diagramId` is closer to a **class definition** — same value for every execution of that automation.  
`journeyId` is the **instance** — unique per execution. Only `journeyId` qualifies as a true Correlation ID.

### 2. HTTP headers ≠ EIP Header
Don't conflate transport metadata (`Content-Type`, `Authorization`) with business message metadata (timestamp, source system, correlation id, message type). The latter usually lives in the body as a structured object.

### 3. Command vs Event shapes the channel
- **Command** → typically Point-to-Point (one specific system must act)
- **Event** → typically Pub/Sub (multiple systems may care that something happened)

This decision affects how you design the integration topology — covered in **Lesson 3**.

---

## Open Questions
- How do diagramId and journeyId behave when an automation has branching logic / multiple webhook calls in one journey? → investigate further
- Carried over from Lesson 1: How to design retry for webhooks that don't support it? → **Lesson 4 (Dead Letter Queue)**

---

## Anki Cards

| Front | Back |
|---|---|
| What are the 3 core parts of an EIP Message? | Header, Body, Correlation ID |
| What does Correlation ID identify — a business entity or a process execution? | A process execution (not customerId/orderId) |
| Two registrations of the same customer on the same day — same or different Correlation ID? | Different — each is a separate process execution |
| Difference between Command Message and Event Message? | Command = "do something" (one recipient, P2P). Event = "something happened" (can have many subscribers, Pub/Sub) |
| Is HTTP `Content-Type` an EIP Header? | No — it's a transport-layer header. EIP Header is business message metadata, often inside the body |
| In Synerise Automation, which ID is the true Correlation ID: diagramId or journeyId? | journeyId — it's unique per execution. diagramId identifies the automation definition, not the run |

---

## Resources
- [Enterprise Integration Patterns — Message](https://www.enterpriseintegrationpatterns.com/patterns/messaging/Message.html)
- [Enterprise Integration Patterns — Correlation Identifier](https://www.enterpriseintegrationpatterns.com/patterns/messaging/CorrelationIdentifier.html)

---

*Next lesson: **Lesson 3 — Point-to-Point vs Pub/Sub***  
*You will learn: when to route a message to exactly one recipient vs broadcast it to many — and map this onto Synerise event types.*
