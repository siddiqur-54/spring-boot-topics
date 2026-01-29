# Aspect Oriented Programming (AOP) in Spring Boot

## 1. Introduction

Aspect Oriented Programming (AOP) is a programming paradigm that helps you **separate cross‑cutting concerns** from your core business logic. Cross‑cutting concerns are functionalities that affect multiple parts of an application, such as:

* Logging
* Security & authorization
* Transaction management
* Auditing
* Performance monitoring
* Exception handling

Spring Boot provides **easy and powerful support for AOP** using Spring AOP, which is based on **proxy mechanisms**.

---

## 2. Why AOP is Needed

### Problem without AOP

```java
public void transferMoney() {
    log.info("Start transfer");
    checkSecurity();
    // business logic
    log.info("End transfer");
}
```

This leads to:

* Code duplication
* Poor readability
* Harder maintenance

### Solution with AOP

Business logic stays clean, and cross‑cutting logic is moved into **aspects**.

---

## 3. Core AOP Concepts

### 3.1 Aspect

A class that contains cross‑cutting logic.

```java
@Aspect
@Component
public class LoggingAspect { }
```

---

### 3.2 Join Point

A **point during program execution** where an aspect can be applied.

Examples:

* Method execution
* Method call
* Exception thrown

In Spring AOP, **only method execution join points are supported**.

---

### 3.3 Advice

The **action taken** by an aspect at a join point.

Types of advice:

* `@Before`
* `@After`
* `@AfterReturning`
* `@AfterThrowing`
* `@Around`

---

### 3.4 Pointcut

An expression that **selects join points** where advice should run.

```java
@Pointcut("execution(* com.example.service.*.*(..))")
public void serviceMethods() {}
```

---

### 3.5 Weaving

The process of **linking aspects with application code**.

Spring performs weaving at **runtime** using proxies.

---

## 4. Spring AOP Architecture

Spring AOP is:

* Proxy‑based (JDK dynamic proxy or CGLIB)
* Runtime weaving
* Method‑level only

### Proxy Types

| Proxy Type | Used When         |
| ---------- | ----------------- |
| JDK Proxy  | Interface present |
| CGLIB      | No interface      |

---

## 5. Enabling AOP in Spring Boot

Add dependency:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

AOP is auto‑enabled in Spring Boot.

---

## 6. Advice Types with Examples

### 6.1 @Before

Runs **before method execution**.

```java
@Before("execution(* com.example.service.*.*(..))")
public void beforeAdvice() {
    System.out.println("Before method call");
}
```

---

### 6.2 @After

Runs **after method execution (always)**.

```java
@After("execution(* com.example.service.*.*(..))")
public void afterAdvice() {
    System.out.println("After method call");
}
```

---

### 6.3 @AfterReturning

Runs **after successful execution**.

```java
@AfterReturning(pointcut = "execution(* com.example.service.*.*(..))", returning = "result")
public void afterReturningAdvice(Object result) {
    System.out.println("Returned: " + result);
}
```

---

### 6.4 @AfterThrowing

Runs **when exception is thrown**.

```java
@AfterThrowing(pointcut = "execution(* com.example.service.*.*(..))", throwing = "ex")
public void afterThrowingAdvice(Exception ex) {
    System.out.println("Exception: " + ex.getMessage());
}
```

---

### 6.5 @Around (Most Powerful)

Controls **entire method execution**.

```java
@Around("execution(* com.example.service.*.*(..))")
public Object aroundAdvice(ProceedingJoinPoint joinPoint) throws Throwable {
    long start = System.currentTimeMillis();
    Object result = joinPoint.proceed();
    long end = System.currentTimeMillis();
    System.out.println("Execution time: " + (end - start));
    return result;
}
```

---

## 7. Pointcut Expressions

### Common Expressions

```java
execution(* com.example..*(..))
within(com.example.service..*)
@annotation(com.example.LogExecutionTime)
```

### Execution Syntax

```
execution(modifiers return-type package.class.method(parameters))
```

---

## 8. Using Custom Annotations with AOP

### Step 1: Create Annotation

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface LogExecutionTime { }
```

### Step 2: Use in Aspect

```java
@Around("@annotation(LogExecutionTime)")
public Object logTime(ProceedingJoinPoint pjp) throws Throwable {
    return pjp.proceed();
}
```

---

## 9. Real‑World Use Cases

* Logging API requests
* Feature access control
* Auditing user actions
* Performance monitoring
* Validation checks

---

## 10. Best Practices

* Keep aspects **stateless**
* Avoid heavy logic in aspects
* Prefer annotation‑based pointcuts
* Use `@Around` sparingly
* Avoid aspects calling other aspects

---

## 11. Limitations of Spring AOP

* Only method execution join points
* Internal method calls are not intercepted
* Proxy‑based (not bytecode weaving)

For advanced needs, use **AspectJ**.

---

## 12. Spring AOP vs AspectJ

| Feature     | Spring AOP  | AspectJ             |
| ----------- | ----------- | ------------------- |
| Weaving     | Runtime     | Compile / Load‑time |
| Join Points | Method only | Many                |
| Complexity  | Simple      | Advanced            |

---

## 13. Common Pitfalls

* Calling a method within the same class
* Wrong pointcut expression
* Missing `@Component` on aspect

---

## 14. Conclusion

Spring AOP helps keep your **business logic clean and modular** by isolating cross‑cutting concerns. It is easy to use, powerful, and sufficient for most enterprise Spring Boot applications.

---

**Next steps:**

* Apply AOP to logging or authorization
* Combine AOP with custom annotations
* Explore AspectJ for advanced use cases

