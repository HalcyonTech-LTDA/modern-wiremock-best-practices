# WireMock Mappings & Request Matching Guide

In WireMock, declarative stub mappings define the contract criteria for intercepting incoming HTTP requests and dispatching mock responses.

---

## 1. Top-Level Mapping Anatomy

Every mapping file located in `mappings/` is a JSON object adhering to the WireMock StubMapping schema:

```json
{
  "id": "b1d0a514-4a41-4890-a7d1-0f7f32997e3a",
  "name": "Lookup Customer By ID",
  "priority": 5,
  "request": {
    "method": "GET",
    "urlPathPattern": "/api/v1/customers/([a-zA-Z0-9_-]+)"
  },
  "response": {
    "status": 200,
    "headers": {
      "Content-Type": "application/json; charset=UTF-8"
    },
    "bodyFileName": "customers/customer-lookup-response.json",
    "transformers": ["response-template"]
  },
  "persistent": true
}
```

### Key Attributes:
- `id`: Unique UUID identifier. Mandatory for predictable API management via the Admin API (`/__admin/mappings/{id}`).
- `name`: Descriptive human-readable title. Essential for log readability and scenario debugging.
- `priority`: Integer from `1` (highest priority) to `10` (lowest priority). Defaults to `5` if omitted.
- `request`: Defines the HTTP request matching predicates (method, path, headers, query, body).
- `response`: Defines the HTTP response attributes (status code, headers, payload, transformers, delays, faults).
- `persistent`: Boolean indicating whether the stub should be saved to disk across restarts (default `true` for file-based stubs).

---

## 2. URL and Path Matching Strategy

WireMock provides four primary URL matching operators:

| Operator | Scope | Evaluates Query Parameters? | Best Use Case |
| :--- | :--- | :---: | :--- |
| `urlPath` | Exact Path | ❌ No | Static endpoints without regex (e.g. `/api/v1/health`) |
| `urlPathPattern` | Regex Path | ❌ No | Dynamic path parameters (e.g. `/api/v1/users/[0-9]+`) |
| `url` | Exact URL | ✅ Yes | Full static URL when query parameter order is rigid |
| `urlPattern` | Regex URL | ✅ Yes | Complex matching including host/query via regex |

### ⚠️ The Path Decoupling Principle
Never embed query parameters inside `url` or `urlPattern` strings (e.g., `url: "/api/v1/items?status=active&sort=desc"` ❌).
Query parameter order can fluctuate across HTTP clients and JVM implementations, causing intermittent matching failures.

**Always pair `urlPath` or `urlPathPattern` with the dedicated `queryParameters` block**:

```json
{
  "request": {
    "method": "GET",
    "urlPath": "/api/v1/search",
    "queryParameters": {
      "query": {
        "matches": "[a-zA-Z0-9 ]+"
      },
      "limit": {
        "equalTo": "20"
      },
      "includeInactive": {
        "absent": true
      }
    }
  }
}
```

---

## 3. Advanced Request Matching Operators

WireMock supports rich matchers across headers, query parameters, cookies, and bodies:

### Common Predicates:
- `equalTo`: Exact literal string match (case-sensitive by default).
- `equalToIgnoreCase`: Case-insensitive string match.
- `contains`: Substring match.
- `matches`: Regular expression match.
- `doesNotMatch`: Negative regular expression match.
- `absent`: Asserts that the header, query parameter, or field is omitted in the request.

### Header Validation Example:
```json
{
  "headers": {
    "Authorization": {
      "matches": "Bearer eyJ[a-zA-Z0-9_-]+\\.[a-zA-Z0-9_-]+\\.[a-zA-Z0-9_-]+"
    },
    "Accept": {
      "contains": "application/json"
    },
    "X-Deprecated-Flag": {
      "absent": true
    }
  }
}
```

---

## 4. Request Body Matching

When mocking POST, PUT, or PATCH endpoints, asserting request payloads is critical.

### Option A: `equalToJson` (Recommended for Full Payloads)
Evaluates JSON semantically, ignoring whitespace, indentation, and key order:

```json
{
  "request": {
    "method": "POST",
    "urlPath": "/api/v1/payments",
    "bodyPatterns": [
      {
        "equalToJson": {
          "currency": "USD",
          "amount": 250.00,
          "sourceAccount": "${json-unit.any-string}"
        },
        "ignoreArrayOrder": true,
        "ignoreExtraElements": true
      }
    ]
  }
}
```
- `ignoreArrayOrder: true`: Array elements match regardless of position.
- `ignoreExtraElements: true`: Ignores additional attributes sent by the client, preventing fragile contract breaks when non-critical fields are added.

### Option B: `matchesJsonPath` (Targeted Field Assertions)
Evaluates specific attributes using JSONPath expressions without matching the entire object:

```json
{
  "request": {
    "method": "POST",
    "urlPath": "/api/v1/webhooks/orders",
    "bodyPatterns": [
      {
        "matchesJsonPath": "$.event[?(@ == 'ORDER_PAID')]"
      },
      {
        "matchesJsonPath": {
          "expression": "$.customer.email",
          "matches": "^[A-Za-z0-9+_.-]+@(.+)$"
        }
      }
    ]
  }
}
```

### Option C: XML Body Matching
For SOAP or legacy XML endpoints:
```json
{
  "request": {
    "method": "POST",
    "urlPath": "/soap/customerService",
    "bodyPatterns": [
      {
        "equalToXml": "<customerRequest><id>12345</id></customerRequest>",
        "enablePlaceholders": true
      }
    ]
  }
}
```

---

## 5. Priority Choreography & The Fallback Ladder

When multiple stubs match an incoming request, WireMock executes the stub with the **lowest numerical priority value** (`1` beats `5`, `5` beats `10`).

### Priority Rules of Thumb:
1. **Priority 1–3 (Explicit Edge Cases & Overrides)**:
   - Specific IDs testing exceptional behavior (e.g. `/api/v1/users/user-blocked`, `/api/v1/payments/card-declined`).
   - Explicit error simulations (400 Bad Request, 403 Forbidden, 404 Not Found, 500 Internal Error).
2. **Priority 4–7 (Standard Business Scenarios)**:
   - Happy-path scenarios with explicit query parameters or headers.
3. **Priority 8–10 (Catch-All Dynamic Fallbacks)**:
   - Templated regex stubs that extract parameters and return simulated generic data.

```text
Incoming Request: GET /api/v1/users/user-banned
 │
 ├── Priority 1: Stub `/api/v1/users/user-banned` -> 403 Forbidden  [MATCHED! Dispatches 403]
 └── Priority 10: Stub `/api/v1/users/([a-zA-Z0-9_-]+)` -> 200 OK  [IGNORED]
```
