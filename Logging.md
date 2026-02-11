# Logging in Spring Boot Microservices (Real-World & Interview Notes)

🚧 **Status: Implemented (Phase – Logging Foundation Completed)**

This document explains **why logging is critical in microservices**, how it is implemented in this project, and how it scales in real production systems.  
These notes are written with **SDE-1 / SDE-2 backend interviews** in mind.

---

## 0. What is SLF4J?

**SLF4J (Simple Logging Facade for Java)**  
- It is **NOT a logging framework**
- It is an **abstraction (API)**

SLF4J allows switching logging implementations without changing code.

```
Application Code
↓
SLF4J (API)
↓
Logback / Log4j2 (Implementation)
```


In Spring Boot:
- **Logback is the default implementation**
- No extra dependency required

---

## 1. Why Logging Is Critical in Microservices

**Simple definition**
> Logging explains **WHY** something happened, metrics explain **WHAT** happened.

In microservices:
- One request flows through **multiple services**
- Failures may be hidden by retries or circuit breakers
- Bugs rarely reproduce locally

Without logging:
- Root cause is unknown
- Debugging becomes guesswork
- Production issues take hours to resolve

**Interview statement**
> “In microservices, logs are the primary source of truth for debugging failures.”

---

## 2. Why Logging Comes BEFORE Resilience

Resilience patterns:
- Retry
- Circuit Breaker
- Timeout
- Fallback

These can **hide failures**.

If logging is missing:
- You don’t know *why* retries happened
- You don’t know *how often* failures occur
- You don’t know *which service failed first*

**Conclusion**
> Logging must be implemented **before** resilience mechanisms.

---

## 3. Logging Architecture in Spring Boot

```
Application Code
↓
SLF4J (Logging API)
↓
Logback (Implementation)
↓
Console + Rolling Log Files
```

Key points:
- SLF4J = abstraction
- Logback = actual logger
- Logs are written to **files**, not only console

---

## 4. Why File-Based Logging (Not Only Console)

### Problems with console-only logging
- Logs disappear after restart
- Not usable in containers / production
- Cannot be collected centrally

### Benefits of file-based logging
- Persistent logs
- Can be shipped to ELK / Datadog / Loki
- Required for real-world systems

**Interview line**
> “Applications write logs to files; centralized logging systems consume those files.”

---

## 5. `logback-spring.xml` Placement (VERY IMPORTANT)

### ❌ Wrong
- Putting `logback-spring.xml` in **Config Server repo**

### ✅ Correct
```
src/main/resources/logback-spring.xml
```

### Why?

Logging initializes **before Spring Cloud Config**.

### Startup order
```
JVM starts
→ Logging system initializes
→ Config Server configuration loads
→ Spring ApplicationContext starts
```

**Interview gold line**
> “Logging configuration must be on the application classpath because logging initializes before Config Server.”

---

## 6. Purpose of `logback-spring.xml`

`logback-spring.xml` controls:
- Log format
- Console + file logging
- Rolling policy
- Retention period

Example log:
```
2026-02-06 10:12:45 INFO [http-nio-8081-exec-1]
RequestLoggingFilter [cid=API-GATEWAY-xxx] - [REQ] GET /employees
```

---

## 7. API Request Logging (How We Implemented)

### Tool Used
`OncePerRequestFilter`

### Why `OncePerRequestFilter`?
- Executes **once per request**
- Prevents duplicate logs
- Safe for async dispatch
- Recommended by Spring

### What we log
- HTTP method
- Request URI
- Response status
- Execution time

### What we DO NOT log
- Request body
- Headers
- PII / sensitive data

**Interview line**
> “We log request metadata, not payloads, to avoid performance and security issues.”

---

## 8. Request Logging Flow
```
Client Request
↓
RequestLoggingFilter
→ log request
↓
Controller → Service → Database
↓
RequestLoggingFilter
→ log response + duration
↓
Client Response
```

---

## 9. Core Request Logging Code (Simplified)

```java
@Component
public class RequestLoggingFilter extends OncePerRequestFilter {

    private static final Logger log =
            LoggerFactory.getLogger(RequestLoggingFilter.class);

    @Override
    protected void doFilterInternal(
            HttpServletRequest request,
            HttpServletResponse response,
            FilterChain filterChain) throws IOException, ServletException {

        long startTime = System.currentTimeMillis();

        log.info("[REQ] method={} uri={}",
                request.getMethod(),
                request.getRequestURI());

        filterChain.doFilter(request, response);

        long duration = System.currentTimeMillis() - startTime;

        log.info("[RES] status={} time={}ms",
                response.getStatus(),
                duration);
    }
}
```
```
10. What Is Correlation ID?

Simple language

Correlation ID = unique ID for one request
Generated when request enters the system
Same ID travels across services
Printed in every log line
Interview GOLD line
“Correlation ID helps trace a single request across multiple microservices using logs.”
```
```
11. Correlation ID Flow
Client Request
↓
API Gateway
→ generate Correlation ID (if missing)
→ add header: X-Correlation-Id
↓
Employee Service
→ read header
→ put into MDC
↓
Department Service
→ same Correlation ID
↓
All logs contain same ID
```
```
12. MDC (Mapped Diagnostic Context)
MDC is provided by SLF4J.
Think of MDC as:
A thread-local key-value map automatically attached to logs
Example:
MDC.put("cid", correlationId);
Logback pattern:
[cid=%X{cid}]
```
```
13. Reactive vs Servlet Logging
Servlet-based services

One request = one thread

MDC works correctly

Reactive API Gateway

Request hops across threads

MDC unreliable

Correlation ID logged explicitly

Interview answer

“MDC works well in servlet services but not reliably in reactive gateways.”

14. Centralized Logging (Concept Only)

Not implemented yet (by design).

Service Logs
→ Filebeat / Fluentd
→ Elasticsearch / OpenSearch
→ Kibana


Interview line

“Each service writes its own logs; centralized systems aggregate them using correlation IDs.”

15. Logs vs Metrics vs Traces
Pillar	Answers
Logs	Why did it fail
Metrics	How often / how slow
Traces	Where time was spent
16. Interview Q&A

Q: Why not log request body?
A: Performance + security risk.

Q: Why correlation ID instead of one log file?
A: Microservices are distributed.

Q: Why logging before resilience?
A: Resilience hides failures.

Q: Does Feign propagate headers automatically?
A: No, a RequestInterceptor is required.

17. Final Interview Summary
“I implemented end-to-end request logging with correlation IDs across API Gateway and downstream services, making the system observable and debuggable.”
```

