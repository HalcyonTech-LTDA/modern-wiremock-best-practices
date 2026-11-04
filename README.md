<div align="center">

# Modern Wiremock Best Practices Plugin
### Deterministic Service Virtualization, API Mocking & Fault Injection

[![Antigravity Plugin](https://img.shields.io/badge/Antigravity-Plugin%20v1.0.0-4F46E5.svg)](https://antigravity.google)
[![License: MIT](https://img.shields.io/badge/License-MIT-10B981.svg)](./LICENSE)
[![Organization](https://img.shields.io/badge/Organization-HalcyonTech--LTDA-black.svg)](https://github.com/HalcyonTech-LTDA)

<p align="center">
  <b>Professional modern WireMock best practices covering declarative stubs architecture (mappings/ and __files/), response payload separation, Handlebars dynamic templating, stateful scenarios, fault injection, container orchestration, and mandatory living README.md documentation discipline.</b>
</p>

</div>

---

## Installation & Verification

### Antigravity CLI (`agy`)

Install directly into your Antigravity environment via the CLI:

```bash
agy plugin install https://github.com/HalcyonTech-LTDA/modern-wiremock-best-practices
```

Verify that the plugin is active and adheres to the strict official schema:

```bash
agy plugin list
agy plugin validate --strict ./modern-wiremock-best-practices
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

---

## What's Included (Component Catalog)

### Contextual Rules (`rules/`)
Automatically loaded to govern and protect project contexts:
- **[`modern-wiremock-engineering.md`](rules/modern-wiremock-engineering.md)**: >-

### Procedural Skills (`skills/`)
On-demand actionable workflows and runbooks:
- **[`/modern-wiremock-declarative-stubs`](skills/modern-wiremock-declarative-stubs/SKILL.md)**: Use this skill when authoring, reviewing, or refactoring WireMock declarative stubs (mappings/) and external response payloads (__files/): request matching criteria, bodyFileName decoupling, Handlebars response templating, priority choreography, stateful scenarios, and fault injection.
- **[`/modern-wiremock-living-documentation`](skills/modern-wiremock-living-documentation/SKILL.md)**: Use this skill when authoring, updating, or reviewing documentation for WireMock mock servers: establishing the mandatory dedicated README.md living documentation, building contract scenario matrices, providing copy-pasteable curl verification recipes, and enforcing documentation synchronization in pull requests.
- **[`/modern-wiremock-orchestration-and-testing`](skills/modern-wiremock-orchestration-and-testing/SKILL.md)**: Use this skill when deploying, orchestrating, or testing WireMock in Docker, Docker Compose, CI/CD pipelines, and automated test suites: standalone container hardening, healthcheck choreography (/__admin/health), Testcontainers integration, high-load performance tuning, and zero-PII security hygiene.

---

## Directory Hierarchy

```text
modern-wiremock-best-practices/
├── plugin.json              # Official Antigravity manifest (schema v1)
├── LICENSE                  # MIT License (Copyright 2026 HalcyonTech LTDA)
├── .gitignore               # System ignores & installed_version.json exclusions
├── .gitattributes           # Deterministic line endings (* text=auto eol=lf)
├── README.md                # Golden Standard documentation & runbook
├── assets/
│   └── logo.svg             # Official Halcyon Tech corporate vector symbol
├── rules/                   # Contextual rules discovered automatically
│   └── modern-wiremock-engineering.md
├── skills/                  # Actionable procedural skills (Progressive Disclosure)
│   └── modern-wiremock-declarative-stubs/
│       └── SKILL.md
│   └── modern-wiremock-living-documentation/
│       └── SKILL.md
│   └── modern-wiremock-orchestration-and-testing/
│       └── SKILL.md
```

---

## 💡 Prompt Recipes & Use Cases

```markdown
- "Create deterministic WireMock stubs and dynamic response transformers"
- "Configure WireMock standalone or Testcontainers integration for consumer tests"
- "Simulate network faults, rate limits, and slow responses with WireMock"
```

---

## 📄 License & Corporate Governance

Distributed under the **MIT License**. Copyright (c) 2026 **HalcyonTech LTDA**.  
Engineered with precision and excellence by [Halcyon Tech](https://github.com/HalcyonTech-LTDA).
