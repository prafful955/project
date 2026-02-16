# 🛡 Resilience Architecture – Employee Service

## 📌 Overview

This project implements production-grade resilience using **Resilience4j** in a Spring Boot microservices architecture.

Architecture Flow:

Client  
↓  
API Gateway (CircuitBreaker + Timeout + Fallback)  
↓  
Employee Service (Retry + TimeLimiter + CircuitBreaker + Bulkhead + Fallback)  
↓  
Department Service  

The goal is to prevent cascading failures, isolate dependencies, and ensure graceful degradation.

---

# 🔥 Resilience Mechanisms Implemented

## 1️⃣ Circuit Breaker

### 🎯 Purpose
Stops calling a failing downstream service after a failure threshold is reached.

### ⚙ Configuration (application.yml)

```yaml
resilience4j:
  circuitbreaker:
    instances:
      departmentService:
        sliding-window-size: 5
        minimum-number-of-calls: 5
        failure-rate-threshold: 50
        wait-duration-in-open-state: 10s
        permitted-number-of-calls-in-half-open-state: 2
        slow-call-rate-threshold: 50
        slow-call-duration-threshold: 2s
```

### 📖 Explanation

- **sliding-window-size: 5**  
  Last 5 calls are considered for calculating failure rate.

- **minimum-number-of-calls: 5**  
  Circuit will evaluate failure rate only after 5 calls.

- **failure-rate-threshold: 50**  
  If more than 50% calls fail → circuit opens.

- **wait-duration-in-open-state: 10s**  
  Circuit stays OPEN for 10 seconds before moving to HALF_OPEN.

- **slow-call-rate-threshold: 50**  
  If more than 50% calls are slow → circuit may open.

- **slow-call-duration-threshold: 2s**  
  Any call taking more than 2 seconds is considered slow.

---

## Circuit States

- **CLOSED** → Normal operation
- **OPEN** → Downstream calls blocked, fallback triggered
- **HALF_OPEN** → Limited test calls allowed
- **CLOSED** → Recovery if test calls succeed

---

## 2️⃣ Retry

### 🎯 Purpose
Handles temporary failures (network glitch, short downtime).

### ⚙ Configuration

```yaml
resilience4j:
  retry:
    instances:
      departmentService:
        max-attempts: 3
        wait-duration: 1s
        retry-exceptions:
          - feign.RetryableException
          - java.io.IOException
```

### 📖 Explanation

- **max-attempts: 3** → 1 original + 2 retries  
- **wait-duration: 1s** → 1 second delay between retries  
- Retries only on network-related exceptions  

---

## 3️⃣ TimeLimiter

### 🎯 Purpose
Prevents long-running calls from blocking threads.

### ⚙ Configuration

```yaml
resilience4j:
  timelimiter:
    instances:
      departmentService:
        timeout-duration: 2s
```

### 📖 Explanation

If Department call exceeds 2 seconds:
- Call is cancelled
- Fallback is triggered

This ensures fail-fast behavior.

---

## 4️⃣ ThreadPool Bulkhead

### 🎯 Purpose
Prevents thread pool exhaustion by isolating dependency calls.

### ⚙ Configuration

```yaml
resilience4j:
  thread-pool-bulkhead:
    instances:
      departmentService:
        coreThreadPoolSize: 5
        maxThreadPoolSize: 10
        queueCapacity: 20
```

### 📖 Explanation

- **coreThreadPoolSize: 5** → Minimum dedicated threads
- **maxThreadPoolSize: 10** → Maximum concurrent threads
- **queueCapacity: 20** → 20 additional calls can wait

If limit exceeded:
- Calls are rejected
- Fallback is triggered
- Main service remains stable

---

# 🔁 Service-Level Resilience Annotation

```java
@CircuitBreaker(name = "departmentService", fallbackMethod = "departmentFallback")
@Retry(name = "departmentService")
@TimeLimiter(name = "departmentService")
@Bulkhead(name = "departmentService", type = Bulkhead.Type.THREADPOOL)
public CompletableFuture<Map<String, Object>> getEmployeeWithDepartment(Long id) {
    return CompletableFuture.supplyAsync(() -> {
        Employee employee = repository.findById(id)
                .orElseThrow(() -> new RuntimeException("Employee not found"));

        DepartmentDto department = departmentClient.getDepartment(1L);

        Map<String, Object> response = new HashMap<>();
        response.put("employee", employee);
        response.put("department", department);

        return response;
    });
}
```

---

# 🛑 Fallback Strategy

```java
public CompletableFuture<Map<String, Object>> departmentFallback(Long id, Throwable ex) {

    log.warn("Fallback triggered for employeeId={}, reason={}",
            id, ex.getClass().getSimpleName());

    Employee employee = repository.findById(id)
            .orElseThrow(() -> new RuntimeException("Employee not found"));

    DepartmentDto fallbackDepartment =
            new DepartmentDto(1L, "Department Service Unavailable");

    Map<String, Object> fallbackResponse = new HashMap<>();
    fallbackResponse.put("employee", employee);
    fallbackResponse.put("department", fallbackDepartment);
    fallbackResponse.put("message", "Fallback applied");

    return CompletableFuture.completedFuture(fallbackResponse);
}
```

---

# 📊 Monitoring & Metrics

Metrics exposed via:

```
/actuator/prometheus
```

Important Metrics:

- `resilience4j_circuitbreaker_state`
- `resilience4j_circuitbreaker_calls`
- `resilience4j_retry_calls_total`
- `resilience4j_bulkhead_available_concurrent_calls`

These metrics are visualized in Grafana.

---

# 🧪 Failure Testing Performed

## Scenario 1 – Department Service Down

- Retry attempts executed
- CircuitBreaker recorded failures
- Circuit transitioned to OPEN
- Fallback triggered
- No cascading failure occurred

## Scenario 2 – Slow Department Response

- TimeLimiter triggered
- Slow calls recorded
- CircuitBreaker reacted accordingly

## Scenario 3 – High Concurrent Load

- Bulkhead isolated dependency calls
- Thread pool remained stable
- System did not freeze

---

# 🎯 Interview Questions & Answers (Resilience)

## Q1: Why use both Retry and CircuitBreaker?

Retry handles transient failures.  
CircuitBreaker handles persistent failures and prevents overload.

---

## Q2: What happens when CircuitBreaker is OPEN?

Downstream service is not invoked.  
Fallback is executed immediately.

---

## Q3: Why use Bulkhead?

To prevent thread pool exhaustion and isolate dependency failures.

---

## Q4: How do you monitor resilience in production?

Using Prometheus metrics:
- Circuit state
- Failure rate
- Retry count
- Bulkhead availability

---

## Q5: What is slow call detection?

Even if calls don’t fail, frequent slow responses can degrade performance.  
Slow-call threshold ensures circuit reacts to latency spikes.

---

## Q6: How do you prevent cascading failure?

Using:
- Retry
- TimeLimiter
- CircuitBreaker
- Bulkhead
- Fallback

Each protects a different failure scenario.

---

# 🏁 Conclusion

The resilience layer ensures:

- Fail-fast behavior
- Dependency isolation
- Graceful degradation
- Production-level observability
- Prevention of cascading failures

This implementation is production-ready and aligned with SDE-2 backend expectations.
