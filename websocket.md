# 📡 WebSocket Documentation – Spring Boot

## 1. Introduction

WebSocket is a **full-duplex, persistent communication protocol** that allows real-time data exchange between client and server over a single TCP connection.

Unlike HTTP (request-response), WebSocket supports **bi-directional communication**, making it ideal for real-time systems.

---

## 2. When to Use WebSocket

Use WebSocket when your application requires:

* Real-time notifications
* Live dashboards
* Chat or messaging systems
* Background job progress tracking
* Attendance / punch-in live updates
* Payroll processing status updates

Do **not** use WebSocket for standard CRUD operations.

---

## 3. WebSocket vs HTTP

| Feature    | HTTP      | WebSocket         |
| ---------- | --------- | ----------------- |
| Connection | Stateless | Persistent        |
| Direction  | One-way   | Bi-directional    |
| Latency    | Higher    | Low               |
| Best for   | REST APIs | Real-time updates |

---

## 4. WebSocket Support in Spring Boot

Spring Boot supports WebSocket via:

* Raw WebSocket
* **STOMP over WebSocket (Recommended)**

STOMP provides:

* Message routing
* Subscriptions
* Loose coupling
* Better scalability

---

## 5. Required Dependencies

### Maven

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-websocket</artifactId>
</dependency>
```

(Optional – for security)

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

---

## 6. WebSocket Architecture (STOMP)

```
Client (Browser / App)
     ↓
WebSocket Endpoint (/ws)
     ↓
Message Broker
     ↓
@MessageMapping Controllers
     ↓
Subscribed Clients
```

---

## 7. WebSocket Configuration

### 7.1 Enable WebSocket and Message Broker

```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/ws")
                .setAllowedOriginPatterns("*")
                .withSockJS();
    }

    @Override
    public void configureMessageBroker(MessageBrokerRegistry registry) {
        registry.setApplicationDestinationPrefixes("/app");
        registry.enableSimpleBroker("/topic", "/queue");
    }
}
```

### Explanation

| Element  | Description                  |
| -------- | ---------------------------- |
| `/ws`    | WebSocket handshake endpoint |
| `/app`   | Client → Server destination  |
| `/topic` | Broadcast destination        |
| `/queue` | One-to-one destination       |
| SockJS   | Fallback for older browsers  |

---

## 8. Message Routing Convention

| Direction                   | Prefix   | Example                  |
| --------------------------- | -------- | ------------------------ |
| Client → Server             | `/app`   | `/app/attendance/mark`   |
| Server → Client (broadcast) | `/topic` | `/topic/attendance/live` |
| Server → Client (private)   | `/queue` | `/queue/notifications`   |

---

## 9. WebSocket Controller

```java
@Controller
public class NotificationSocketController {

    @MessageMapping("/notify")
    @SendTo("/topic/notifications")
    public NotificationMessage send(NotificationMessage message) {
        return message;
    }
}
```

### Flow

```
Client → /app/notify
Server → /topic/notifications
```

---

## 10. Payload / DTO Design

```java
@Data
public class NotificationMessage {
    private String type;
    private String message;
    private Long employeeId;
}
```

**Best Practice:**

* Keep WebSocket DTOs separate from REST DTOs
* Keep payloads small

---

## 11. Sending Messages from Service Layer

```java
@Service
public class WebSocketNotificationService {

    @Autowired
    private SimpMessagingTemplate messagingTemplate;

    public void sendGlobalNotification(NotificationMessage msg) {
        messagingTemplate.convertAndSend("/topic/notifications", msg);
    }

    public void sendPrivateNotification(Long userId, NotificationMessage msg) {
        messagingTemplate.convertAndSendToUser(
                userId.toString(),
                "/queue/notifications",
                msg
        );
    }
}
```

---

## 12. One-to-One Messaging

### Client Subscription

```
/user/queue/notifications
```

### Server Send

```java
convertAndSendToUser(userId, "/queue/notifications", payload);
```

> Requires authenticated `Principal`

---

## 13. WebSocket Security

### 13.1 Destination Security

```java
@Configuration
@EnableWebSocketSecurity
public class WebSocketSecurityConfig {

    @Bean
    AuthorizationManager<Message<?>> messageAuthorizationManager() {
        return AuthorizationManagerBuilder
                .message()
                .simpDestMatchers("/app/**").authenticated()
                .simpSubscribeDestMatchers("/topic/**").authenticated()
                .anyMessage().denyAll()
                .build();
    }
}
```

---

### 13.2 JWT Handshake Interceptor

```java
@Component
public class JwtHandshakeInterceptor implements HandshakeInterceptor {

    @Override
    public boolean beforeHandshake(ServerHttpRequest request,
                                   ServerHttpResponse response,
                                   WebSocketHandler wsHandler,
                                   Map<String, Object> attributes) {
        // Extract and validate JWT
        return true;
    }

    @Override
    public void afterHandshake(...) {}
}
```

---

## 14. Client Side (JavaScript)

### Connect

```javascript
const socket = new SockJS('/ws');
const stompClient = Stomp.over(socket);

stompClient.connect({}, function () {
    console.log("Connected");
});
```

### Subscribe

```javascript
stompClient.subscribe('/topic/notifications', function (message) {
    console.log(JSON.parse(message.body));
});
```

### Send Message

```javascript
stompClient.send(
    "/app/notify",
    {},
    JSON.stringify({ type: "INFO", message: "Hello" })
);
```

---

## 15. Error Handling

```java
@ControllerAdvice
public class WebSocketExceptionHandler {

    @MessageExceptionHandler
    public void handleException(Throwable ex) {
        log.error("WebSocket error", ex);
    }
}
```

---

## 16. Scaling WebSocket (Production)

### Default Simple Broker

* Works only for single instance

### Recommended for Scale

* Redis Pub/Sub
* RabbitMQ
* ActiveMQ

```java
registry.enableStompBrokerRelay("/topic", "/queue")
        .setRelayHost("localhost")
        .setRelayPort(61613);
```

---

## 17. HRMS Use Cases

| Module        | Usage                      |
| ------------- | -------------------------- |
| Attendance    | Live punch-in/out          |
| Payroll       | Salary generation progress |
| Leave         | Approval notifications     |
| Recruitment   | Interview updates          |
| Notifications | Company-wide alerts        |

---

## 18. REST vs WebSocket

| Use REST       | Use WebSocket     |
| -------------- | ----------------- |
| CRUD APIs      | Real-time updates |
| Stateless      | Stateful          |
| Request-driven | Event-driven      |

---

## 19. Common Pitfalls

* Using WebSocket for CRUD
* Broadcasting sensitive data
* No authentication on `/ws`
* Large payloads
* No reconnect handling

---

## 20. Recommended Package Structure

```
websocket/
 ├── config
 ├── controller
 ├── dto
 ├── service
 └── security
```

---

## 21. Best Practices

* Version message types
* Enable heartbeats
* Log connect/disconnect
* Secure subscriptions
* Use message broker for scale

---

## 22. Summary

Spring Boot WebSocket works best with:

* STOMP protocol
* Proper destination mapping
* Strong security
* External broker for scalability

This setup enables reliable, real-time communication for enterprise applications like HRMS.
d