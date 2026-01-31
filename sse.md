# 📢 Server-Sent Events (SSE) Documentation – Spring Boot

## 1. Introduction

Server-Sent Events (SSE) is a **server → client streaming mechanism** built on top of HTTP.

It allows the server to **push continuous updates** to the client over a **single long-lived HTTP connection**.

SSE is simpler than WebSocket and is ideal when:

* Data flows only from server to client
* Real-time updates are required
* Clients do not need to send frequent messages back

---

## 2. SSE vs WebSocket vs HTTP

| Feature    | HTTP               | SSE             | WebSocket      |
| ---------- | ------------------ | --------------- | -------------- |
| Direction  | Request → Response | Server → Client | Bi-directional |
| Connection | Stateless          | Long-lived      | Persistent     |
| Complexity | Low                | Medium          | High           |
| Best for   | CRUD APIs          | Live updates    | Full real-time |

---

## 3. When to Use SSE

Use SSE for:

* Live notifications
* Payroll / report generation progress
* Attendance live feed
* Monitoring dashboards
* Log streaming
* Status updates

Do **not** use SSE when:

* Client must frequently send data to server
* Chat or messaging systems are required

---

## 4. SSE Support in Spring Boot

Spring Boot supports SSE using:

* `SseEmitter`
* `ResponseBodyEmitter`
* Reactive Streams (WebFlux)

This document focuses on **`SseEmitter` (most common for MVC apps)**.

---

## 5. Required Dependencies

No extra dependency is required beyond:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

---

## 6. SSE Architecture

```
Client (Browser / App)
     ↓ HTTP GET
SSE Endpoint (/sse/subscribe)
     ↓
Long-lived HTTP Connection
     ↓
Server Push Events
```

---

## 7. Basic SSE Endpoint

```java
@RestController
@RequestMapping("/sse")
public class NotificationSseController {

    @GetMapping("/subscribe")
    public SseEmitter subscribe() {
        SseEmitter emitter = new SseEmitter(0L); // no timeout
        return emitter;
    }
}
```

---

## 8. Sending Events to Client

### 8.1 SSE Service

```java
@Service
public class SseNotificationService {

    private final List<SseEmitter> emitters = new CopyOnWriteArrayList<>();

    public SseEmitter subscribe() {
        SseEmitter emitter = new SseEmitter(0L);
        emitters.add(emitter);

        emitter.onCompletion(() -> emitters.remove(emitter));
        emitter.onTimeout(() -> emitters.remove(emitter));
        emitter.onError(e -> emitters.remove(emitter));

        return emitter;
    }

    public void sendNotification(Object data) {
        for (SseEmitter emitter : emitters) {
            try {
                emitter.send(SseEmitter.event()
                        .name("notification")
                        .data(data));
            } catch (Exception e) {
                emitters.remove(emitter);
            }
        }
    }
}
```

---

### 8.2 Controller Using Service

```java
@RestController
@RequestMapping("/sse")
public class NotificationSseController {

    @Autowired
    private SseNotificationService sseService;

    @GetMapping("/subscribe")
    public SseEmitter subscribe() {
        return sseService.subscribe();
    }
}
```

---

## 9. SSE Payload Format

SSE messages follow this format:

```
event: notification
data: {"message": "Salary generation started"}

```

Spring handles this automatically via `SseEmitter`.

---

## 10. Client Side (JavaScript)

```javascript
const eventSource = new EventSource('/sse/subscribe');

eventSource.onmessage = function(event) {
    console.log(JSON.parse(event.data));
};

eventSource.addEventListener('notification', function(event) {
    console.log('Notification:', event.data);
});

// Handle errors
eventSource.onerror = function() {
    console.log('SSE connection error');
};
```

---

## 11. Sending Events from Business Logic

```java
sseNotificationService.sendNotification(
    Map.of("type", "PAYROLL", "status", "IN_PROGRESS")
);
```

---

## 12. One-to-One SSE (User Specific)

### Recommended Approach

Maintain emitters per user:

```java
private final Map<Long, SseEmitter> userEmitters = new ConcurrentHashMap<>();
```

```java
userEmitters.put(userId, emitter);
```

```java
userEmitters.get(userId).send(data);
```

---

## 13. SSE Security

### 13.1 Authentication

* Use existing Spring Security filters
* SSE endpoints are standard HTTP GET APIs

```java
.antMatchers("/sse/**").authenticated();
```

JWT / session-based auth works out of the box.

---

## 14. Timeout & Reconnect Handling

### Server Side

```java
new SseEmitter(30 * 60 * 1000L); // 30 minutes
```

### Client Side

Browser automatically reconnects on disconnect.

---

## 15. Scaling SSE (Production)

### Challenges

* Each client holds one HTTP connection
* Memory usage increases with users

### Solutions

* Load balancer with sticky sessions
* Redis / Kafka for event source
* Clean up dead emitters aggressively

---

## 16. HRMS Use Cases

| Module        | SSE Usage           |
| ------------- | ------------------- |
| Payroll       | Generation progress |
| Attendance    | Live status feed    |
| Reports       | Export progress     |
| Notifications | System alerts       |

---

## 17. Common Pitfalls

* Not removing dead emitters
* Broadcasting sensitive data
* Large payloads
* Too many open connections

---

## 18. Best Practices

* Keep events small
* Send heartbeat events
* Use named events
* Secure endpoints
* Prefer SSE over WebSocket for one-way updates

---

## 19. SSE vs WebSocket – Rule of Thumb

| Requirement          | Choose    |
| -------------------- | --------- |
| One-way updates      | SSE       |
| Bi-directional       | WebSocket |
| Simple real-time     | SSE       |
| Chat / collaboration | WebSocket |

---

## 20. Summary

Server-Sent Events (SSE) provide a **simple, efficient, and scalable** solution for server-to-client real-time communication.

For enterprise systems like HRMS, SSE is often the **best choice** for:

* Progress updates
* Notifications
* Monitoring dashboards

Use WebSocket only when two-way communication is required.
