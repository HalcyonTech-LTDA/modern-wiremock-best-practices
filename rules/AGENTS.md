---
description: >-
  Automatically enforces modern WireMock engineering standards: declarative stubbing (mappings/ vs __files/),
  response body separation, Handlebars dynamic templating, deterministic priority scoring, stateful scenarios,
  official WireMock container orchestration, and mandatory dedicated README.md living documentation.
trigger: model_decision
---

# Modern WireMock Engineering Guardrails & Skill Router

When authoring, reviewing, or refactoring WireMock stubs, response payloads, directory structures, Docker configurations, or mock documentation, strictly adhere to Modern WireMock Engineering Standards and the corresponding specialized skills:

| When working on... | Consult & Activate Skill | Key Focus Areas |
| :--- | :--- | :--- |
| **Declarative Stubs & Payloads** | `modern-wiremock-declarative-stubs` | `mappings/` vs `__files/` decoupling, request matching operators, Handlebars dynamic templating, stateful scenarios, priority scoring, and fault injection. |
| **Living Documentation & Contracts** | `modern-wiremock-living-documentation` | Mandatory dedicated `README.md`, simulated API contract specification, scenario matrix tables, curl verification recipes, and PR sync checklists. |
| **Container Orchestration & Testing** | `modern-wiremock-orchestration-and-testing` | Official `wiremock/wiremock:3.x` Docker images, Compose Spec orchestration, healthcheck choreography (`/__admin/health`), Testcontainers integration, and zero-PII security. |

---

## Non-Negotiable Guardrails (Absolute Invariants)

1. **Strict Separation of Routing (`mappings/`) and Payloads (`__files/`)**:
   - Absolute ban on inlining response payloads directly into `mappings/*.json` files using `"body"` or `"jsonBody"`. Never write stringified escaped JSON strings (`"body": "{\"...\": ...}"`).
   - Decouple all non-empty response payloads into `__files/` and reference them exclusively via `"bodyFileName": "relative/path/to/payload.json"`. For empty responses (such as `204 No Content`), simply omit `bodyFileName`.
   - Reserve `mappings/` exclusively for declarative routing: HTTP methods, URL/path matching, request headers, query parameters, request body predicates, response status codes, response headers, priority, delays, faults, and transformer declarations.

2. **Mandatory Dedicated `README.md` (Living Documentation Mandate)**:
   - Every directory or project module housing WireMock stubs MUST maintain a dedicated, up-to-date `README.md` in that mock root directory.
   - Any PR or commit that introduces, alters, or removes stubs or payloads MUST synchronize this `README.md` in the exact same commit.
   - The `README.md` must clearly document:
     - The simulated upstream API contract (target service, paths, verbs, ports, protocol).
     - A structured **Scenario Matrix** table mapping scenario names, matching criteria, status codes, key response fields, and corresponding mapping/`__files` paths.
     - Operational instructions for running the mock server (Docker Compose, standalone container, Testcontainers).
     - Executable, copy-pasteable `curl` verification snippets with expected responses for smoke testing.
     - Any required extensions or custom transformers.

3. **Deterministic Priority Choreography**:
   - Always assign explicit numerical priorities (`priority: 1` through `priority: 10`) when multiple stubs can match overlapping URL patterns.
   - Concrete, specific stubs (e.g., matching a distinct customer ID or explicit error scenario) MUST have higher priority (lower integer, e.g., `priority: 1` to `5`).
   - Generic, wildcard, or catch-all templated fallback stubs MUST have lower priority (higher integer, e.g., `priority: 10`).
   - Never rely on filesystem loading order or implicit default priority (`5`).

4. **Handlebars Templating over Fixture Duplication**:
   - Use dynamic response templating (`"transformers": ["response-template"]`) to echo request paths (`{{request.pathSegments.[n]}}`), query parameters (`{{request.query.<name>}}`), or body fields (`{{jsonPath request.body '$.field'}}`).
   - Do not generate dozens of redundant static stub files for identical payload structures that only differ by identifier.
   - Always use Handlebars triple braces `{{{jsonPath ...}}}` when rendering raw JSON or strings that must not be HTML-escaped.

5. **Zero Production PII or Secrets in Fixtures**:
   - Never store real credentials, production API keys, real social security numbers, banking details, or customer PII inside `__files/` or `mappings/`.
   - Always synthesize realistic, anonymized fixture data or leverage Handlebars random helpers (`{{randomValue type='UUID'}}`, `{{randomValue type='ALPHANUMERIC'}}`).

6. **Operational Healthchecks & Orchestration Choreography**:
   - WireMock services in `compose.yaml` and production/CI containers MUST declare a native healthcheck targeting `http://localhost:8080/__admin/health`.
   - Downstream dependent microservices MUST declare `depends_on: { wiremock: { condition: service_healthy } }`.
   - Standalone images must use pinned, modern versions of `wiremock/wiremock:3.x` (never `:latest`) and run as an unprivileged user.

7. **Symmetrical & Modular Subdirectory Architecture**:
   - In multi-service or multi-domain mocking setups, organize `mappings/` and `__files/` symmetrically by service or domain (e.g., `mappings/payment/charge.json` pairs with `__files/payment/charge-success.json`).
   - Keep filenames kebab-case, descriptive, and aligned between mapping rules and payload files.

8. **Stateful Scenario Isolation & Resets**:
   - When modeling stateful transitions (e.g., polling, retries, multi-step workflows), explicitly declare `scenarioName`, `requiredScenarioState`, and `newScenarioState`.
   - The initial state must always be `Started`.
   - Provide a deterministic loopback to `Started` or document the reset endpoint (`POST /__admin/scenarios/reset`) to prevent inter-test state bleed.

Consult the corresponding skills for complete technical deep dives, configuration recipes, and reference implementations.
