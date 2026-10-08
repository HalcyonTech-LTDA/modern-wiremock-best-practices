---
name: modern-wiremock-living-documentation
description: >-
  Use this skill when authoring, updating, or reviewing documentation for WireMock mock servers:
  establishing the mandatory dedicated README.md living documentation, building contract scenario matrices,
  providing copy-pasteable curl verification recipes, and enforcing documentation synchronization in pull requests.
---

# Modern WireMock Living Documentation Discipline

This skill defines the non-negotiable living documentation standard for WireMock mock servers. In modern engineering, mock servers are first-class service contracts, not throwaway testing artifacts.

---

## The Living Documentation Philosophy

### Why Undocumented Mocks Cause System Degradation:
1. **The "Black-Box" Antipattern**: When a mock directory contains dozens of JSON mappings with cryptic filenames, engineers cannot determine what endpoints are simulated without manually reverse-engineering regexes and JSON payloads.
2. **Silent Contract Drift**: As upstream external APIs evolve, mocks without documentation drift out of alignment with production reality, leading to false-positive test suites.
3. **Developer Friction & Flaky Onboarding**: New team members struggle to spin up local mocks or verify whether an edge case (e.g. rate limit, 404, authorization failure) is already supported.
4. **Disjointed PRs**: When developers update stubs in `mappings/` without updating documentation, integration knowledge remains trapped in private heads.

### The Invariant Rule: Dedicated `README.md` Mandate
Whenever WireMock is integrated into a repository (e.g. `wiremock/`, `mock-server/`, or `src/test/resources/wiremock/`), it **MUST** be accompanied by a dedicated, meticulously maintained `README.md` located directly in that directory.

Any pull request that introduces, alters, or deletes a stub in `mappings/` or a fixture in `__files/` **MUST** update this `README.md` within the exact same atomic commit.

---

## Anatomy of a Production-Grade WireMock `README.md`

A professional WireMock `README.md` consists of six mandatory sections:

### 1. Overview & Simulated API Contract
- Clarifies which third-party, partner, or upstream microservice is being mocked.
- References the consuming application clients (e.g., Spring Cloud OpenFeign client, WebClient, Axios instance).
- Identifies protocols, base URLs, default ports, and environment variable bindings.

### 2. Directory Architecture
- Displays a clean visual ASCII tree of the mock setup:
  - Configuration/Dockerfile (`Dockerfile.wiremock`, `compose.yaml`)
  - Documentation (`README.md`)
  - Payload directory (`__files/`)
  - Declarative stubs directory (`mappings/`)
  - Custom extensions (`extensions/`, if applicable)

### 3. Contract Scenario Matrix (The Core Catalog)
A comprehensive markdown table cataloging every mapped interaction:
- **Scenario Name**: Business description (e.g., Dynamic Default, Explicit Suspended, Card Declined, 404 Not Found).
- **HTTP Verb & Path / Parameters**: Exact path or regex pattern.
- **Match Criteria**: Key headers, query parameters, or payload body patterns required to trigger the stub.
- **HTTP Status**: Response status code (e.g., `200 OK`, `400 Bad Request`, `404 Not Found`, `503 Service Unavailable`).
- **Key Attributes / Behaviors**: Summary of payload contents, templated attributes, or injected latency.
- **Mapping & Payload Path**: Clickable file links to `mappings/<file>.json` and `__files/<file>.json`.

### 4. How to Run & Orchestrate
- Copy-pasteable shell commands for:
  - Starting via Docker Compose (local development hot reload).
  - Building and running an immutable standalone container (CI/CD environments).
  - Running via CLI or Testcontainers if applicable.

### 5. Verification with `curl` (Smoke Testing Suite)
- A sequence of runnable `curl -i` commands covering:
  - Dynamic fallback endpoint (verifying Handlebars templating).
  - Explicit happy-path overrides.
  - Error and edge-case paths (400, 403, 404, 500).
  - Health check endpoint (`curl -i http://localhost:<port>/__admin/health`).
- Every command must include commented HTTP status codes and JSON response snippets showing expected output.

### 6. Extensions & Advanced Capabilities
- Documentation of custom transformers, standalone JARs (e.g., JWT signing extensions, OAuth2 helpers), and runtime flags.

---

## Detailed Topic Guides

1. **Standard `README.md` Template Blueprint**:
   - Production-ready, domain-agnostic Markdown template.
   - [Read Guide](./references/mock_readme_standard_template.md)

2. **Scenario Matrix & Verification Recipe Standards**:
   - Tabular schema for scenario catalogs.
   - Writing bulletproof `curl` verification commands with response assertions.
   - Code review checklist for PR synchronization.
   - [Read Guide](./references/scenario_matrix_and_curl_recipes.md)
