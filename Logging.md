# Logging in Spring Boot Microservices (Real-World Notes)

🚧 **Status: Implemented (Phase – Logging Foundation)**

This document explains **why logging is critical in microservices**, how it is implemented in this project, and how it scales to production systems.  
These notes are written with **backend interview expectations (SDE-1 / SDE-2)** in mind.

---
SLF4J = Simple Logging Facade for Java
## 1. Why Logging is Important in Microservices

**Simple definition**  
> Logging explains *WHY* something happened, metrics explain *WHAT* happened.

In a microservices system:
- One request flows through multiple services
- Failures may be hidden by retries or circuit breakers
- Without logs, debugging becomes impossible

**Interview statement**
> “In microservices, logs are the primary source of truth for debugging failures.”

---

## 2. Why Logging Comes Before Resilience

Resilience patterns like:
- Retry
- Circuit Breaker
- Fallback

can **hide failures**.

Without logging:
- Root cause is unknown
- Failure frequency is invisible
- Debugging becomes guesswork

**Conclusion**
> Logging must be implemented before resilience mechanisms.

---

## 3. Logging Architecture in Spring Boot
```
Application Code
↓
SLF4J (Logging API)
↓
Logback (Implementation)
↓
Console + Log Files
```

Key points:
- SLF4J is only an abstraction
- Logback is the default Spring Boot logging implementation
- No extra dependency required

---

## 4. Why File-Based Logging (Not Only Console)

### Problems with console logging
- Logs disappear after restart
- Not usable in production environments

### Benefits of file-based logging
- Logs are persistent
- Can be shipped to ELK / Datadog / Loki
- Required for real-world systems

**Interview line**
> “Applications write logs to files; centralized systems consume those files.”

---

## 5. logback-spring.xml Placement (Very Important)

### ❌ Wrong
- Putting `logback-spring.xml` in Config Server repo

### ✅ Correct
src/main/resources/logback-spring.xml


### Reason
Logging initializes **before** Spring Cloud Config.

### Startup order

JVM starts
→ Logging system initializes
→ Config Server configuration loads
→ Spring ApplicationContext starts


**Interview gold line**
> “Logging configuration must be on the application classpath because logging initializes before Config Server.”

---

## 6. Purpose of logback-spring.xml

The `logback-spring.xml` file:
- Defines log format
- Writes logs to console and rolling log files
- Controls log rotation and retention

Example log output:
```
2026-02-06 10:12:45 INFO [http-nio-8081-exec-1] RequestLoggingFilter - [REQ] GET /employees
2026-02-06 10:12:45 INFO [http-nio-8081-exec-1] RequestLoggingFilter - [RES] 200 in 32ms
```

---

## 7. API Request Logging Implementation

### Tool Used
`OncePerRequestFilter`

### Why OncePerRequestFilter
- Executes once per HTTP request
- Prevents duplicate logging
- Safe for async dispatch

### What is logged
- HTTP method
- Request URI
- Response status
- Execution time

### What is NOT logged
- Request body
- Headers
- Sensitive data

**Interview line**
> “We log request metadata, not payloads, to avoid performance and security issues.”

---
```
## 8. Request Logging Flow

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

## 9. Key Code Concepts (High-Level)

### Start time capture
```java
package com.example.employeeservice.logging;

import jakarta.servlet.FilterChain;
import jakarta.servlet.ServletException;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;
import org.springframework.web.filter.OncePerRequestFilter;

import java.io.IOException;

@Component
public class RequestLoggingFilter extends OncePerRequestFilter {

    private static final Logger log =
            LoggerFactory.getLogger(RequestLoggingFilter.class);

    @Override
    protected void doFilterInternal(
            HttpServletRequest request,
            HttpServletResponse response,
            FilterChain filterChain
    ) throws ServletException, IOException {

        long startTime = System.currentTimeMillis();

        log.info("[REQ] method={} uri={}",
                request.getMethod(),
                request.getRequestURI()
        );

        filterChain.doFilter(request, response);

        long duration = System.currentTimeMillis() - startTime;

        log.info("[RES] status={} time={}ms",
                response.getStatus(),
                duration
        );
    }
}
```

Response logging

HTTP status

Total execution time

This helps detect:

Slow APIs

Error responses

Performance regressions
