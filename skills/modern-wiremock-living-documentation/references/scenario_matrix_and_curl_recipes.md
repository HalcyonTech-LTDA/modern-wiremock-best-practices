# Scenario Matrix & `curl` Verification Recipes Guide

The Scenario Matrix and `curl` verification snippets are the operational heart of any WireMock `README.md`. They bridge the gap between static JSON definitions and actionable developer workflows.

---

## 1. Crafting the Scenario Matrix

The Scenario Matrix acts as a live API catalog for all simulated interactions.

### Standard Schema:

| Column | Purpose | Example |
| :--- | :--- | :--- |
| **Scenario Name** | Business identifier for the test case | `Fraud Score Exceeded` |
| **Method & Path** | HTTP verb and exact/regex path | `POST /api/v1/payments/verify` |
| **Match Preconditions** | Required headers, query params, or body matching rules | Body contains `"riskScore": 95` |
| **HTTP Status** | Returned HTTP response code | `422 Unprocessable Entity` |
| **Key Behavior / Attributes** | Summary of simulated behavior, Handlebars logic, or delays | Returns RFC 7807 problem details with `FRAUD_DETECTED` code |
| **Mapping & Payload Files** | Clickable links to repository source files | `mappings/pay-fraud.json`<br>`__files/pay-fraud-response.json` |

### Matrix Best Practices:
- **Order by Precedence**: List explicit high-priority overrides before catch-all dynamic fallback rows.
- **Explicit Negative Paths**: Ensure 4xx and 5xx edge cases are explicitly cataloged, not just happy paths.
- **Link Directly to Files**: Use relative file links (e.g. `[mappings/pay-fraud.json](./mappings/pay-fraud.json)`) so reviewers and developers can jump directly to the code from the markdown viewer.

---

## 2. Authoring Reproducible `curl` Recipes

A good `curl` recipe is executable with zero manual edits, providing instant feedback for developers verifying mock behavior.

### Standard Rules for WireMock `curl` Snippets:
1. **Always Include `-i`**: Emits HTTP response headers (`HTTP/1.1 200 OK`, `Content-Type`), verifying status codes and content negotiations.
2. **Comment Expected Responses Directly Below**: Include the expected HTTP status line and a formatted JSON snippet.
3. **Use `-H "Content-Type: application/json"` on Mutations**: Never omit Content-Type headers when sending JSON bodies in POST, PUT, or PATCH.
4. **Use `-s` or `-S` in Automated Scripts**: Keep CLI noise minimal when piped into CI assertions or `jq`.

### 4-Point Smoke Test Suite Pattern:

```bash
# ==============================================================================
# 1. DYNAMIC TEMPLATED FALLBACK (Echoes ID and generates dynamic timestamp)
# ==============================================================================
curl -i http://localhost:8080/api/v1/accounts/acc-live-99
# HTTP/1.1 200 OK
# Content-Type: application/json; charset=UTF-8
# {"accountId": "acc-live-99", "status": "ACTIVE", "tier": "STANDARD"}

# ==============================================================================
# 2. SPECIFIC HAPPY-PATH OVERRIDE (Explicit VIP status, Priority 1)
# ==============================================================================
curl -i http://localhost:8080/api/v1/accounts/acc-vip-01
# HTTP/1.1 200 OK
# Content-Type: application/json; charset=UTF-8
# {"accountId": "acc-vip-01", "status": "ACTIVE", "tier": "VIP", "discount": 20}

# ==============================================================================
# 3. CLIENT ERROR PATH (Explicit 404 Not Found)
# ==============================================================================
curl -i http://localhost:8080/api/v1/accounts/acc-not-found
# HTTP/1.1 404 Not Found
# Content-Type: application/problem+json; charset=UTF-8
# {"title": "Account Not Found", "status": 404, "detail": "Account does not exist"}

# ==============================================================================
# 4. WIREMOCK HEALTH CHECK (Verifies container and admin listener readiness)
# ==============================================================================
curl -i http://localhost:8080/__admin/health
# HTTP/1.1 200 OK
# Content-Type: application/json
# {"status": "healthy"}
```

---

## 3. Pull Request Synchronization Checklist

When creating or reviewing pull requests involving WireMock, enforce this checklist:

- [ ] **Mapping File Created/Modified**: Added or altered files in `mappings/`.
- [ ] **Payload File Created/Modified**: Response body saved in `__files/` (not inlined in mapping).
- [ ] **Priority Assigned**: Explicit numerical `priority` assigned if path pattern overlaps with existing stubs.
- [ ] **Scenario Matrix Updated**: New row added to `README.md` documenting scenario name, path, status, and file links.
- [ ] **`curl` Recipe Added**: Copy-pasteable `curl` command with expected response added to Section 4 of `README.md`.
- [ ] **Local Verification Executed**: `curl` commands tested against a running instance before committing.
- [ ] **Zero Secrets/PII**: Verified no internal API keys, passwords, or real production PII exist in fixtures.
