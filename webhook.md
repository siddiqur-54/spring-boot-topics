# 🔔 Webhook Documentation – Spring Boot

## 1. Introduction

A **Webhook** is a mechanism where one system **pushes data to another system via HTTP callbacks** when a specific event occurs.

Instead of polling an API repeatedly, the receiving system exposes an endpoint, and the sender **calls that endpoint automatically** when an event happens.

Webhooks are **event-driven**, **stateless**, and **simple to integrate**.

---

## 2. Webhook vs REST vs SSE vs WebSocket

| Feature    | REST API        | Webhook            | SSE             | WebSocket      |
| ---------- | --------------- | ------------------ | --------------- | -------------- |
| Direction  | Client → Server | Server → Server    | Server → Client | Bi-directional |
| Trigger    | Client request  | Event-based        | Event-based     | Event-based    |
| Connection | Short-lived     | Short-lived        | Long-lived      | Persistent     |
| Best for   | CRUD            | System integration | Live UI updates | Real-time apps |

---

## 3. When to Use Webhooks

Use Webhooks when:

* Integrating with third-party systems
* Notifying external systems of events
* Payroll completion callbacks
* Attendance sync with external systems
* Payment or subscription events
* Audit or compliance notifications

Do **not** use Webhooks for:

* UI real-time updates
* High-frequency streaming
* Internal synchronous communication

---

## 4. Webhook Architecture

```
System A (Producer)
   └── Event Occurs
        └── HTTP POST (Webhook)
             ↓
System B (Consumer)
        └── Webhook Endpoint
             ↓
        Process Event
```

---

## 5. Webhook Characteristics

* Uses standard HTTP (mostly POST)
* Payload is usually JSON
* Fire-and-forget by default
* Requires retries and idempotency
* Security is critical

---

## 6. Creating a Webhook Endpoint in Spring Boot

```java
@RestController
@RequestMapping("/webhooks")
public class PayrollWebhookController {

    @PostMapping("/payroll-status")
    public ResponseEntity<Void> receivePayrollStatus(
            @RequestBody PayrollWebhookPayload payload,
            @RequestHeader Map<String, String> headers) {

        // Validate
        // Process event
        return ResponseEntity.ok().build();
    }
}
```

---

## 7. Webhook Payload Design

```java
@Data
public class PayrollWebhookPayload {
    private String eventType;
    private String eventId;
    private Long companyId;
    private String status;
    private Instant occurredAt;
}
```

### Recommended Fields

| Field        | Purpose         |
| ------------ | --------------- |
| `eventType`  | Type of event   |
| `eventId`    | Idempotency key |
| `occurredAt` | Event timestamp |
| `data`       | Actual payload  |

---

## 8. Event Type Naming Convention

```
PAYROLL_GENERATED
PAYROLL_FAILED
ATTENDANCE_SYNCED
LEAVE_APPROVED
```

Use **UPPER_SNAKE_CASE** for clarity.

---

## 9. Security Best Practices

### 9.1 Shared Secret (HMAC Signature)

**Sender:**

```
X-Signature: sha256=abcdef...
```

**Receiver Validation:**

```java
public boolean isValidSignature(String payload, String signature) {
    // HMAC SHA-256 validation
    return true;
}
```

---

### 9.2 API Key / Token

```http
Authorization: Bearer <token>
```

Secure via Spring Security filters.

---

## 10. Idempotency Handling

Webhooks can be **delivered multiple times**.

### Strategy

* Use `eventId`
* Store processed event IDs
* Ignore duplicates

```java
if (eventAlreadyProcessed(eventId)) {
    return ResponseEntity.ok().build();
}
```

---

## 11. Error Handling & HTTP Status Codes

| Status | Meaning                      |
| ------ | ---------------------------- |
| 200    | Event processed successfully |
| 202    | Accepted, async processing   |
| 400    | Invalid payload              |
| 401    | Unauthorized                 |
| 500    | Retry required               |

> Producers retry on non-2xx responses.

---

## 12. Retry Strategy (Producer Side)

Recommended retry policy:

* Exponential backoff
* Max retry limit
* Dead-letter queue

Example:

```
1s → 5s → 30s → 5min
```

---

## 13. Async Webhook Processing

Always process webhooks **asynchronously**.

```java
@PostMapping("/event")
public ResponseEntity<Void> receive(@RequestBody Payload payload) {
    asyncProcessor.process(payload);
    return ResponseEntity.accepted().build();
}
```

---

## 14. Webhook Versioning

### URL Versioning

```
/webhooks/v1/payroll
/webhooks/v2/payroll
```

### Header Versioning

```
X-Webhook-Version: 1
```

---

## 15. Logging & Auditing

Log:

* Headers
* Event ID
* Event type
* Processing status

Avoid logging sensitive data.

---

## 16. Webhook Testing

### Tools

* Postman
* cURL
* RequestBin
* Webhook.site

### Local Testing

Use:

* ngrok
* cloudflared

---

## 17. HRMS Use Cases

| Module     | Webhook Usage        |
| ---------- | -------------------- |
| Payroll    | Completion callbacks |
| Attendance | External sync        |
| Payments   | Subscription events  |
| Compliance | Audit notifications  |

---

## 18. Common Pitfalls

* No signature validation
* No idempotency
* Slow synchronous processing
* No retry handling
* Logging secrets

---

## 19. Best Practices

* Always verify signature
* Respond fast (≤ 2s)
* Process async
* Design idempotent handlers
* Version your webhooks

---

## 20. Summary

Webhooks are the **best integration mechanism** for event-driven, system-to-system communication.

In enterprise systems like HRMS, webhooks are ideal for:

* External integrations
* Event notifications
* Decoupled architectures

Combine:

* **Webhooks** for external systems
* **SSE** for UI updates
* **WebSocket** for interactive features

for a clean and scalable architecture.
