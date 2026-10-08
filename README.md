# Modern WireMock Best Practices Plugin

An Antigravity plugin providing automated engineering guardrails, architectural patterns, declarative stub standards, and procedural skills for modern **WireMock** engineering (WireMock 3.x+): strict payload separation (`mappings/` vs `__files/`), dynamic Handlebars templating, stateful scenarios, fault injection, container orchestration, and the **mandatory living `README.md` documentation discipline**.

---

## Installation

Install directly in Antigravity via the CLI:

```bash
agy plugin install https://github.com/HalcyonTech-LTDA/modern-wiremock-best-practices
```

Verify that the plugin is installed and ready:

```bash
agy plugin list
```

---
## Overview

In modern distributed microservices and cloud architectures, external dependencies (third-party payment gateways, identity providers, credit bureaus, partner APIs) present significant testing hurdles: network flakiness, rate limits, non-deterministic sandbox data, and financial costs.

WireMock is the industry-standard tool for simulating HTTP APIs. However, mock implementations frequently deteriorate into unmaintainable anti-patterns:
- Monolithic, 500-line JSON stub mappings with giant inlined response payloads.
- Proliferation of dozens of nearly identical static stub files differing only by an ID.
- Silent divergence between real upstream contracts and mock server implementations.
- "Black-box" mock directories that developers cannot run, verify, or understand without digging through hundreds of JSON files.

This plugin establishes modern, production-grade engineering standards for WireMock across three fundamental pillars:

1. **Declarative Stub Architecture & Payload Decoupling**: Enforcing strict separation between HTTP matching/routing rules in `mappings/` and externalized response payloads in `__files/`, leveraging Handlebars response templating to eliminate stub bloat.
2. **Mandatory Living Documentation**: Requiring every mock server to maintain an up-to-date, comprehensive `README.md` acting as a living contract catalog with a structured **Scenario Matrix**, local execution recipes, and copy-pasteable `curl` smoke tests.
3. **Container Orchestration & Resilient Testing**: Standardizing official `wiremock/wiremock:3.x` container builds, Docker Compose healthcheck choreography against `/__admin/health`, Testcontainers integration, and zero-PII security hygiene.

---

## What's Included

### 1. Automatic Guardrails (`rules/AGENTS.md`)
Automatically enforced across all projects utilizing WireMock:
- **Strict Separation of Routing and Payloads**: Absolute ban on large inlined JSON/XML strings in `mappings/`; mandatory delegation to `__files/` via `bodyFileName`.
- **Mandatory Dedicated `README.md`**: Any project with WireMock stubs MUST maintain a dedicated, synchronized `README.md` documenting the simulated contract, scenario matrix, run commands, and `curl` tests.
- **Deterministic Priority Choreography**: Explicit priorities (`priority: 1` to `10`) ensuring specific cases take precedence over generic/templated fallbacks.
- **Handlebars Templating over Fixture Duplication**: Using `response-template` helpers (`{{request.pathSegments.[n]}}`, `{{jsonPath ...}}`) to model dynamic APIs without duplicating stubs.
- **Zero Sensitive Data or Production PII**: Enforcing synthetic data and random generators; banning real credentials and customer PII in fixtures.
- **Operational Healthchecks & Choreography**: Requiring native container healthchecks against `/__admin/health` and Compose `condition: service_healthy`.
- **Symmetrical & Modular Subdirectory Layout**: Organizing complex multi-service mocks into clean, symmetrical subdirectories across `mappings/` and `__files/`.
- **Stateful Scenario Isolation & Resets**: Strict modeling of Finite State Machines (`scenarioName`, `requiredScenarioState`, `newScenarioState`) with loopback or reset safety.

---

### 2. Specialized Skills (`skills/`)

#### 🏛️ `modern-wiremock-declarative-stubs`
Guides declarative stub authoring, JSON matching rules, Handlebars templating, and fault simulation:
- `references/mappings_and_request_matching.md`: Schema anatomy of `mappings/*.json`, URL matching (`url`, `urlPattern`, `urlPath`, `urlPathPattern`), header/cookie matching, advanced body matching (`equalToJson`, `matchesJsonPath`, `matchesXPath`), priority hierarchies, and catch-all fallbacks.
- `references/files_and_handlebars_templating.md`: Decoupling payloads into `__files/`, referencing via `bodyFileName`, Handlebars response templating helpers (`request.pathSegments`, `request.query`, `jsonPath`, `randomValue`, `now`, `math`), escaping rules (`{{{...}}}` vs `{{...}}`), and anti-patterns.
- `references/scenarios_and_fault_injection.md`: State machine modeling (FSM) for polling, retries, and CRUD lifecycles (`Started` -> state transitions), chaos engineering with delays (`fixedDelayMilliseconds`, `randomDelay`, `chunkedDribbleDelay`), and network-level faults (`EMPTY_RESPONSE`, `MALFORMED_RESPONSE_CHUNK`, `RANDOM_DATA_THEN_CLOSE`).

#### 📖 `modern-wiremock-living-documentation`
Guides the mandatory living documentation standard for WireMock mock servers:
- `references/mock_readme_standard_template.md`: A complete, professional, domain-agnostic template for a dedicated `wiremock/README.md`.
- `references/scenario_matrix_and_curl_recipes.md`: Standards for building contract catalog matrices (Scenario | Method & Path | Match Criteria | Status | Payload File), crafting copy-pasteable `curl` verification snippets with expected output comments, and PR synchronization checklists.

#### 🐳 `modern-wiremock-orchestration-and-testing`
Guides containerization, CI/CD pipelines, Testcontainers, and performance optimization:
- `references/docker_compose_and_container_hardening.md`: Building standalone immutable WireMock Dockerfiles (`wiremock/wiremock:3.x`), non-root user execution, Docker Compose service choreography, healthcheck integration (`/__admin/health`), and custom extension JAR handling.
- `references/testcontainers_and_ci_performance.md`: WireMock Testcontainers integration in Java, Python, Node, and Go, disabling request journaling (`--no-request-journal`) for performance testing, and CI/CD fixture caching.

---

## Directory Layout

```text
modern-wiremock-best-practices/
├── plugin.json
├── README.md
├── rules/
│   └── AGENTS.md
└── skills/
    ├── modern-wiremock-declarative-stubs/
    │   ├── SKILL.md
    │   └── references/
    │       ├── files_and_handlebars_templating.md
    │       ├── mappings_and_request_matching.md
    │       └── scenarios_and_fault_injection.md
    ├── modern-wiremock-living-documentation/
    │   ├── SKILL.md
    │   └── references/
    │       ├── mock_readme_standard_template.md
    │       └── scenario_matrix_and_curl_recipes.md
    └── modern-wiremock-orchestration-and-testing/
        ├── SKILL.md
        └── references/
            ├── docker_compose_and_container_hardening.md
            └── testcontainers_and_ci_performance.md
```

---

## License

This project is licensed under the [MIT License](LICENSE) - see the LICENSE file for details.
Copyright (c) 2026 Halcyon Tech.
