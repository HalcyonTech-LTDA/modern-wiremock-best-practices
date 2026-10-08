# Stateful Scenarios & Fault Injection Guide

Modern distributed systems must handle transient failures, retries, polling lifecycles, and network anomalies. WireMock enables deterministic simulation of stateful workflows and transport faults.

---

## 1. Stateful Scenarios (Finite State Machines)

WireMock Scenarios allow stubs to change their behavior based on previous interactions, functioning as a Finite State Machine (FSM).

### Core Concepts:
- `scenarioName`: A string identifier grouping related stateful stubs together.
- `requiredScenarioState`: The prerequisite state for this stub to match. Every scenario begins automatically in the default state: `"Started"`.
- `newScenarioState`: The state the scenario transitions into *after* this stub is executed.

---

## 2. Stateful Recipes

### Recipe A: Asynchronous Polling Lifecycle
Simulate a background job that is initially `PENDING`, transitions to `PROCESSING`, and finally finishes as `COMPLETED`:

#### Step 1: Initial Polling Call (`Started` -> `PROCESSING`)
Mapping (`mappings/reports/report-polling-step1.json`):
```json
{
  "scenarioName": "ReportGeneration",
  "requiredScenarioState": "Started",
  "newScenarioState": "PROCESSING",
  "request": {
    "method": "GET",
    "urlPath": "/api/v1/reports/rep-100"
  },
  "response": {
    "status": 200,
    "headers": { "Content-Type": "application/json; charset=UTF-8" },
    "bodyFileName": "reports/report-rep-100-pending.json"
  }
}
```
Payload (`__files/reports/report-rep-100-pending.json`):
```json
{
  "reportId": "rep-100",
  "status": "PENDING"
}
```

#### Step 2: Intermediate Polling Call (`PROCESSING` -> `COMPLETED`)
Mapping (`mappings/reports/report-polling-step2.json`):
```json
{
  "scenarioName": "ReportGeneration",
  "requiredScenarioState": "PROCESSING",
  "newScenarioState": "COMPLETED",
  "request": {
    "method": "GET",
    "urlPath": "/api/v1/reports/rep-100"
  },
  "response": {
    "status": 200,
    "headers": { "Content-Type": "application/json; charset=UTF-8" },
    "bodyFileName": "reports/report-rep-100-processing.json"
  }
}
```
Payload (`__files/reports/report-rep-100-processing.json`):
```json
{
  "reportId": "rep-100",
  "status": "PROCESSING"
}
```

#### Step 3: Terminal Polling Call (`COMPLETED` -> Terminal)
Mapping (`mappings/reports/report-polling-step3.json`):
```json
{
  "scenarioName": "ReportGeneration",
  "requiredScenarioState": "COMPLETED",
  "request": {
    "method": "GET",
    "urlPath": "/api/v1/reports/rep-100"
  },
  "response": {
    "status": 200,
    "headers": { "Content-Type": "application/json; charset=UTF-8" },
    "bodyFileName": "reports/report-rep-100-completed.json"
  }
}
```
Payload (`__files/reports/report-rep-100-completed.json`):
```json
{
  "reportId": "rep-100",
  "status": "COMPLETED",
  "downloadUrl": "https://s3.example.com/rep-100.pdf"
}
```

---

### Recipe B: Transient Failure with Automatic Retry Recovery
Simulate a flakey payment service: fails on the first attempt with `503 Service Unavailable`, but recovers with `200 OK` on retry.

#### Call 1 (Attempt 1 Fails - No Body):
Mapping (`mappings/charges/charge-transient-fail.json`):
```json
{
  "scenarioName": "PaymentRetryFlow",
  "requiredScenarioState": "Started",
  "newScenarioState": "FAILED_ONCE",
  "request": {
    "method": "POST",
    "urlPath": "/api/v1/charges"
  },
  "response": {
    "status": 503,
    "headers": { "Retry-After": "1" }
  }
}
```

#### Call 2 (Retry Succeeds and Resets State):
Mapping (`mappings/charges/charge-retry-success.json`):
```json
{
  "scenarioName": "PaymentRetryFlow",
  "requiredScenarioState": "FAILED_ONCE",
  "newScenarioState": "Started",
  "request": {
    "method": "POST",
    "urlPath": "/api/v1/charges"
  },
  "response": {
    "status": 200,
    "headers": { "Content-Type": "application/json; charset=UTF-8" },
    "bodyFileName": "charges/charge-success.json"
  }
}
```
Payload (`__files/charges/charge-success.json`):
```json
{
  "chargeId": "ch_999",
  "status": "SUCCESS"
}
```

*Note: Looping `newScenarioState` back to `"Started"` ensures the test is idempotent and reusable across test runs.*

---

## 3. Scenario Hygiene & Isolation

To prevent state contamination across independent test suites:
- **Always reset scenarios** between test runs in automated CI pipelines:
  ```bash
  curl -X POST http://localhost:8080/__admin/scenarios/reset
  ```
- In JUnit/Java test fixtures:
  ```java
  @AfterEach
  void resetWireMockScenarios() {
      wireMockClient.resetScenarios();
  }
  ```

---

## 4. Latency & Delay Simulation

Simulate slow responses, timeouts, and network degradation:

### 1. Fixed Latency:
Test client-side timeout configurations (e.g., Feign, WebClient, HttpClient):
```json
{
  "response": {
    "status": 200,
    "fixedDelayMilliseconds": 3500,
    "bodyFileName": "data.json"
  }
}
```

### 2. Random Distribution (Network Jitter):
Model real-world internet latency variance using lognormal or uniform distributions:
```json
{
  "response": {
    "status": 200,
    "randomDelay": {
      "type": "lognormal",
      "median": 120,
      "sigma": 0.4
    },
    "bodyFileName": "data.json"
  }
}
```

### 3. Chunked Dribble Delay:
Simulate slow, throttled bandwidth connections:
```json
{
  "response": {
    "status": 200,
    "chunkedDribbleDelay": {
      "numberOfChunks": 5,
      "totalDuration": 2000
    },
    "bodyFileName": "large-report.json"
  }
}
```

---

## 5. Network Fault Injection

Test how resilient your application's connection pool, circuit breaker, and retry policies are when low-level socket connections fail abruptly.

| Fault Mode | Socket / Protocol Behavior |
| :--- | :--- |
| `EMPTY_RESPONSE` | Returns a completely empty response, abruptly closing the TCP connection without HTTP headers. |
| `MALFORMED_RESPONSE_CHUNK` | Sends standard 200 OK headers, streams part of the body, then injects malformed bytes mid-stream. |
| `RANDOM_DATA_THEN_CLOSE` | Sends random junk bytes across the socket and immediately terminates TCP. |
| `CONNECTION_RESET_BY_PEER` | Instantly sends a TCP RST packet to simulate abrupt process termination or network partition. |

### Fault Stub Example:
```json
{
  "id": "e4a7a8d5-1c39-4475-b903-82c81525e982",
  "name": "Simulate Socket Drop on Authorization",
  "request": {
    "method": "POST",
    "urlPath": "/api/v1/auth/token"
  },
  "response": {
    "fault": "CONNECTION_RESET_BY_PEER"
  }
}
```
