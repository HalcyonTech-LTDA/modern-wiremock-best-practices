---
name: modern-wiremock-declarative-stubs
description: >-
  Use this skill when authoring, reviewing, or refactoring WireMock declarative stubs (mappings/) and external response payloads (__files/):
  request matching criteria, bodyFileName decoupling, Handlebars response templating, priority choreography, stateful scenarios, and fault injection.
---

# Modern WireMock Declarative Stubs & Payloads

This skill provides comprehensive architectural guidelines, schema specifications, and best practices for authoring declarative, production-grade WireMock stubs using `mappings/` and external payload files in `__files/`.

---

## Core Principles

1. **Strict Separation of Concerns (`mappings/` vs `__files/`)**:
   - `mappings/`: Dedicated exclusively to declarative HTTP routing rules, URL/path pattern matching, header/cookie validations, request body assertions, HTTP response status codes, content-type headers, priority definitions, and transformer declarations.
   - `__files/`: Dedicated exclusively to response body payloads (JSON, XML, HTML, plain text, or binary assets) and Handlebars response templates.
   - **Anti-Pattern**: Inlining JSON structures or stringified bodies (`"body": "{ ... }"` or `"jsonBody": { ... }`) directly inside `mappings/` files. Inlining pollutes git diffs, breaks JSON syntax highlighting in editors, prevents schema validation, and creates maintenance headaches.
   - **Best Practice**: Always use `"bodyFileName": "relative/path/to/payload.json"`, pointing to a file in the `__files/` directory. For empty responses (e.g., 204 No Content), omit `bodyFileName`.

2. **Hierarchical Request Matching**:
   - Match by path, not full URL, unless querying external host domains: prefer `urlPath` (literal match) or `urlPathPattern` (regex match) over `url` and `urlPattern`, decoupling query parameters into the dedicated `queryParameters` block.
   - Match JSON request bodies with `equalToJson` (leveraging `ignoreArrayOrder: true` and `ignoreExtraElements: true` for resilience) or `matchesJsonPath` for targeted attribute verification.
   - Never write fragile exact-string body matches (`equalTo`) for serialized JSON/XML payloads where key order or whitespace can vary.

3. **Handlebars Templating over Fixture Duplication**:
   - Do not generate separate static stubs for 20 different IDs if the payload structure is identical.
   - Author a single dynamic stub with `"transformers": ["response-template"]` and extract dynamic parameters via Handlebars:
     - Path segments: `{{request.pathSegments.[n]}}`
     - Query parameters: `{{request.query.<param>}}`
     - Request body fields: `{{{jsonPath request.body '$.fieldName'}}}`
     - Identifiers & timestamps: `{{randomValue type='UUID'}}`, `{{now format='yyyy-MM-ddTHH:mm:ssZ'}}`
   - Use triple braces `{{{...}}}` when rendering unescaped JSON values or strings containing quotes/slashes.

4. **Deterministic Priority Choreography**:
   - Assign explicit numerical priorities (`priority: 1` through `priority: 10`) when URL patterns overlap.
   - Specific stubs (e.g. testing an exact customer ID `cust-rejected`, specific error codes, or explicit HTTP header requirements) MUST have high priority (lower number: `1` to `5`).
   - Catch-all fallback stubs (e.g. matching any UUID or regex path `([a-zA-Z0-9_-]+)`) MUST have lower priority (`10`).
   - Never rely on default priority (`5`) or non-deterministic filesystem traversal order.

5. **Stateful Scenarios & State Machine Integrity**:
   - Model stateful interactions (e.g. polling an async job from `PENDING` to `COMPLETED`, testing circuit breakers, or simulating rate limit retries) with WireMock Scenarios (`scenarioName`, `requiredScenarioState`, `newScenarioState`).
   - The initial state must always be `Started`.
   - Ensure a clean transition path back to `Started` or provide explicit reset mechanisms (`POST /__admin/scenarios/reset`).

6. **Resilience & Chaos Simulation**:
   - Simulate realistic latency using `fixedDelayMilliseconds` or statistical distribution models (`randomDelay`).
   - Simulate network interruptions and transport drops using fault injection (`EMPTY_RESPONSE`, `MALFORMED_RESPONSE_CHUNK`, `RANDOM_DATA_THEN_CLOSE`, `CONNECTION_RESET_BY_PEER`).

---

## Detailed Topic Guides

1. **Request Matching & Mapping Anatomy**:
   - Structure of declarative mapping definitions.
   - Path, query, header, and cookie matchers.
   - Advanced JSON/XML body matching operators.
   - Priority and fallback choreography.
   - [Read Guide](./references/mappings_and_request_matching.md)

2. **External Payloads & Handlebars Response Templating**:
   - Linking `bodyFileName` to `__files/`.
   - Handlebars request model and extraction helpers.
   - Dynamic data generation (UUIDs, dates, math, strings).
   - Escaping, formatting, and JSON safety.
   - [Read Guide](./references/files_and_handlebars_templating.md)

3. **Stateful Scenarios & Fault Simulation**:
   - Finite State Machine (FSM) scenario modeling.
   - Asynchronous polling and retry lifecycle recipes.
   - Latency modeling and network fault injection.
   - [Read Guide](./references/scenarios_and_fault_injection.md)

---

## Blueprint Example: Mapping vs Payload Separation

### Mapping Definition (`mappings/order-dynamic-lookup.json`):
```json
{
  "id": "e6f49726-89c2-48ec-b44c-9828d54c4101",
  "name": "Dynamic Order Lookup",
  "priority": 10,
  "request": {
    "method": "GET",
    "urlPathPattern": "/api/v1/orders/([a-zA-Z0-9_-]+)",
    "headers": {
      "Accept": {
        "contains": "application/json"
      }
    }
  },
  "response": {
    "status": 200,
    "headers": {
      "Content-Type": "application/json; charset=UTF-8"
    },
    "bodyFileName": "orders/order-dynamic-response.json",
    "transformers": ["response-template"]
  }
}
```

### External Payload Template (`__files/orders/order-dynamic-response.json`):
```json
{
  "orderId": "{{request.pathSegments.[3]}}",
  "status": "PROCESSING",
  "currency": "USD",
  "createdAt": "{{now format='yyyy-MM-ddTHH:mm:ssZ'}}",
  "trackingCode": "{{randomValue type='ALPHANUMERIC' length=12 uppercase=true}}",
  "totalAmount": 149.99
}
```

### Specific Override Mapping (`mappings/order-cancelled.json`):
```json
{
  "id": "8a72c1e4-3d12-4c6e-bf22-a9b091f93f10",
  "name": "Explicit Cancelled Order",
  "priority": 1,
  "request": {
    "method": "GET",
    "urlPath": "/api/v1/orders/ord-cancelled-99"
  },
  "response": {
    "status": 200,
    "headers": {
      "Content-Type": "application/json; charset=UTF-8"
    },
    "bodyFileName": "orders/order-cancelled-response.json"
  }
}
```
