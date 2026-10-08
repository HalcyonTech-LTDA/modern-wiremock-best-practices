# Testcontainers & CI Performance Tuning Guide

Integrating WireMock into automated testing suites with Testcontainers enables isolated, deterministic integration tests that run identically on developer machines and ephemeral CI/CD pipelines.

---

## 1. Testcontainers Integration (Java / Spring Boot)

Using the official WireMock Testcontainers extension or a standard generic container isolates tests without port collisions.

### Dependency Declaration (`gradle/libs.versions.toml`):
```toml
[versions]
wiremock-testcontainers = "1.0.0"

[libraries]
wiremock-testcontainers = { module = "org.wiremock.integrations.testcontainers:wiremock-testcontainers-extension", version.ref = "wiremock-testcontainers" }
```

### Modern Spring Boot 3.x+ / JUnit 5 Test Example:
```java
package com.example.customer.infrastructure.webclient;

import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.DynamicPropertyRegistry;
import org.springframework.test.context.DynamicPropertySource;
import org.testcontainers.junit.jupiter.Container;
import org.testcontainers.junit.jupiter.Testcontainers;
import org.wiremock.integrations.testcontainers.WireMockContainer;

import static org.assertj.core.api.Assertions.assertThat;

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@Testcontainers
class CustomerApiClientIntegrationTest {

    @Container
    static final WireMockContainer wiremock = new WireMockContainer("wiremock/wiremock:3.13.2-alpine")
            .withMappingFromResource("customer-lookup", "wiremock/mappings/customer-lookup.json")
            .withFileFromResource("customer-lookup-response.json", "wiremock/__files/customer-lookup-response.json");

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        // Dynamically bind the random ephemeral host port to Spring properties
        registry.add("clients.customer-service.base-url", wiremock::getBaseUrl);
    }

    private final CustomerApiClient client;

    @Autowired
    CustomerApiClientIntegrationTest(CustomerApiClient client) {
        this.client = client;
    }

    @AfterEach
    void resetMocks() {
        // Reset scenarios and request journal between tests to prevent pollution
        wiremock.resetAll();
    }

    @Test
    void shouldFetchCustomerDetailsSuccessfully() {
        var response = client.getCustomerById("cust-100");

        assertThat(response).isNotNull();
        assertThat(response.id()).isEqualTo("cust-100");
    }
}
```

---

## 2. Multi-Language Testcontainers Recipes

### Python (`testcontainers`):
```python
from testcontainers.core.container import DockerContainer
import pytest
import requests

@pytest.fixture(scope="session")
def wiremock_server():
    container = (
        DockerContainer("wiremock/wiremock:3.9.1-alpine")
        .with_bind_ports(8080, None)
        .with_volume_mapping("./wiremock/mappings", "/home/wiremock/mappings", "ro")
        .with_volume_mapping("./wiremock/__files", "/home/wiremock/__files", "ro")
        .with_command("--global-response-templating")
    )
    with container as started:
        host_port = started.get_exposed_port(8080)
        yield f"http://localhost:{host_port}"
```

### TypeScript / Node.js (`testcontainers`):
```typescript
import { GenericContainer, StartedTestContainer } from "testcontainers";

let container: StartedTestContainer;
let wiremockUrl: string;

beforeAll(async () => {
  container = await new GenericContainer("wiremock/wiremock:3.9.1-alpine")
    .withExposedPorts(8080)
    .withBindMounts([
      { source: "./wiremock/mappings", target: "/home/wiremock/mappings", mode: "ro" },
      { source: "./wiremock/__files", target: "/home/wiremock/__files", mode: "ro" },
    ])
    .withCommand(["--global-response-templating"])
    .start();

  wiremockUrl = `http://localhost:${container.getMappedPort(8080)}`;
});

afterAll(async () => {
  await container.stop();
});
```

---

## 3. High-Throughput Performance Tuning

Under standard configurations, WireMock records every incoming request in an internal in-memory data structure to support assertions like `verify(...)`.

### The Memory Leak in Load Testing:
During load tests, soak tests, or benchmark suites processing hundreds of thousands of HTTP requests, the in-memory journal will continuously expand until it throws `java.lang.OutOfMemoryError: Java heap space`.

### Optimization Flags for Performance Suites:
When executing load testing against or through WireMock, pass these critical CLI flags:

| Flag | Purpose | Recommended Setting |
| :--- | :--- | :--- |
| `--no-request-journal` | Disables in-memory recording of received requests. Prevents memory leaks under high request volumes. | Mandatory for load / soak tests. |
| `--container-threads` | Number of worker threads allocated to handling incoming HTTP requests. | Set to `100`–`200` for high concurrency. |
| `--jetty-acceptor-threads` | Number of threads accepting socket connections. | Set to `4` for multi-core runners. |
| `--async-response-enabled` | Enables asynchronous servlet response processing for non-blocking I/O. | `true` |

### Load Testing Command Example:
```bash
docker run --rm -p 8081:8080 wiremock/wiremock:3.9.1-alpine \
  --global-response-templating \
  --no-request-journal \
  --container-threads=150 \
  --async-response-enabled=true
```
