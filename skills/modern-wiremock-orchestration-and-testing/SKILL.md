---
name: modern-wiremock-orchestration-and-testing
description: >-
  Use this skill when deploying, orchestrating, or testing WireMock in Docker, Docker Compose, CI/CD pipelines, and automated test suites:
  standalone container hardening, healthcheck choreography (/__admin/health), Testcontainers integration, high-load performance tuning, and zero-PII security hygiene.
---

# Modern WireMock Orchestration & Automated Testing

This skill provides architectural guidelines for packaging, containerizing, and orchestrating WireMock instances across local development, CI/CD pipelines, and automated integration test suites.

---

## Core Engineering Principles

1. **Immutable Container Builds for CI/CD**:
   - In production CI/CD pipelines, never rely on ephemeral host mounts that can desynchronize or fail across runner agents.
   - Author a standalone Dockerfile (e.g., `Dockerfile.wiremock`) that copies `mappings/`, `__files/`, and `extensions/` directly into an immutable container image based on `wiremock/wiremock:3.x`.
   - Tag images deterministically (using Git commit SHA or semantic version, never floating `:latest`).

2. **Compose Spec Choreography & Healthchecks**:
   - Always declare an explicit healthcheck probing the WireMock health endpoint: `http://localhost:8080/__admin/health`.
   - In `compose.yaml`, dependent services (backend APIs, gateway services) must declare `depends_on: { wiremock: { condition: service_healthy } }`.
   - For rapid local iteration, bind-mount `mappings/` and `__files/` with read-only (`:ro`) access to enable hot-reloading without container rebuilds.

3. **In-Test Orchestration with Testcontainers**:
   - For integration test suites (JUnit 5, pytest, mocha), spin up WireMock using Testcontainers (`wiremock/wiremock:3.x`) rather than embedding heavyweight in-process mock libraries.
   - Configure dynamic URLs via framework hooks (e.g. `@DynamicPropertySource` in Spring Boot, test fixtures in Python/Node).
   - Ensure containers run on random ephemeral host ports to enable parallel test execution without port conflicts.

4. **Zero-PII & Secret Sanitization**:
   - Never commit production secrets, private certificates, or customer Personally Identifiable Information (PII) inside `__files/` or `mappings/`.
   - Sanitize mock payloads to use realistic synthetic data or Handlebars random generators (`{{randomValue type='UUID'}}`).

5. **High-Throughput Performance Tuning**:
   - By default, WireMock records all received HTTP requests into an in-memory request journal for verification assertions (`wireMock.verify(...)`).
   - In performance, load, or soak testing environments, this in-memory journal will exhaust container RAM and trigger JVM `OutOfMemoryError`.
   - Pass `--no-request-journal` when running WireMock under high concurrency or prolonged load test suites.

---

## Detailed Topic Guides

1. **Docker, Compose & Container Hardening**:
   - Standalone `Dockerfile.wiremock` recipe.
   - Versionless `compose.yaml` with healthcheck choreography.
   - Non-root user execution, read-only volumes, and resource limits.
   - [Read Guide](./references/docker_compose_and_container_hardening.md)

2. **Testcontainers & CI Performance Tuning**:
   - Java Spring Boot / JUnit 5 Testcontainers integration.
   - Python / Node.js Testcontainers patterns.
   - Disabling request journaling (`--no-request-journal`) for performance testing.
   - Scenario cleanup and parallel test isolation.
   - [Read Guide](./references/testcontainers_and_ci_performance.md)
