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
| **Declarative Stubs & Payloads** | `modern-wiremock-declarative-stubs` | `mappings/` vs `__files/` decoupling, request matching, Handlebars dynamic templating, stateful scenarios, and priority scoring. |
| **Living Documentation & Contracts** | `modern-wiremock-living-documentation` | Mandatory dedicated `README.md`, simulated API contract specification, scenario matrix tables, and curl verification recipes. |
| **Container Orchestration & Testing** | `modern-wiremock-orchestration-and-testing` | Official `wiremock/wiremock:3.x` Docker images, Compose Spec orchestration, healthchecks (`/__admin/health`), and Testcontainers. |

---

## Non-Negotiable Guardrails (Absolute Invariants)

1. **Strict Decoupling of Routing (`mappings/`) and Payloads (`__files/`)**: Absolute ban on inlining non-empty response bodies into `mappings/*.json` files. Decouple response bodies into `__files/` referenced exclusively via `bodyFileName`, organizing both directories with symmetrical domain subdirectories.
2. **Deterministic Priority Scoring**: Explicit numerical priorities (`priority: 1` through `10`) are mandatory whenever stubs match overlapping URL patterns. Specific stubs must possess higher priority (lower integer) than generic or catch-all fallback stubs.
3. **Handlebars Templating over Fixture Duplication**: Use dynamic response templating (`response-template`) to parameterize dynamic request attributes. Prohibit creating duplicate static stub files for identical responses that differ only by entity identifier.
4. **Zero Production PII & Credential Hygiene**: Never commit real passwords, API keys, tokens, or customer PII inside mock mappings or payload files; synthesize realistic, anonymized fixture data.
5. **Stateful Scenario Isolation & Deterministic Resets**: Stubs modeling stateful workflows must define explicit scenario state transitions starting at `Started`. Document and support reset endpoints to eliminate state leakage across automated tests.
6. **Container Healthchecks & Dedicated Living Documentation**: Mock servers in containerized environments must declare native healthchecks targeting `/__admin/health`. Maintain and synchronize a dedicated living `README.md` containing a complete scenario matrix in every atomic stub commit.

Consult the corresponding skills for complete technical deep dives, configuration recipes, and reference implementations.
