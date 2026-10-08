# WireMock Standard `README.md` Template Blueprint

Use this template as the standard blueprint whenever establishing or refactoring a dedicated `README.md` inside any WireMock mock server directory.

---

```markdown
# WireMock - <Service Name> Mock Server

This directory contains the declarative stubs, response payloads, extensions, and container infrastructure used to simulate the upstream **<Service Name>** API for local development and integration testing.

---

## 1. Directory Structure

\`\`\`text
<mock-root>/
├── Dockerfile.wiremock       # Dockerfile to build an immutable, standalone WireMock image
├── README.md                 # Living documentation and operational contract guide
├── __files/                  # Static and Handlebars-templated response payloads
│   └── <domain>/
│       ├── <resource>-default.json
│       ├── <resource>-success.json
│       └── <resource>-error.json
├── extensions/               # Standalone WireMock Java extension JARs (optional)
│   └── <custom-extension>.jar
└── mappings/                 # Declarative stub definitions and routing rules
    └── <domain>/
        ├── <resource>-default.json
        ├── <resource>-success.json
        └── <resource>-error.json
\`\`\`

---

## 2. Simulated API Contract

The mock server simulates the following upstream contract:

- **Target Service**: `<Upstream Service Name>`
- **Consuming Client**: [`<ClientClassName>`](<path/to/client/interface>)
- **Base URL (Local)**: `http://localhost:<HOST_PORT>` (mapped to port `8080` internally via `<PORT_ENV_VAR>`)
- **Protocol**: HTTP/1.1 (JSON REST)

### Mapped Scenarios Matrix

| Scenario | Method & Path | Match Preconditions | HTTP Status | Key Behavior / Attributes | Mapping & Payload Files |
| :--- | :--- | :--- | :---: | :--- | :--- |
| **Dynamic Default** | `GET /api/v1/<resources>/{id}` | Any valid identifier pattern | `200 OK` | Dynamically echoes `{id}` via Handlebars `{{request.pathSegments.[3]}}` | `mappings/<domain>/<resource>-default.json`<br>`__files/<domain>/<resource>-default.json` |
| **Explicit Success** | `GET /api/v1/<resources>/<explicit-id>` | Exact ID match | `200 OK` | Returns full activated profile with static attributes | `mappings/<domain>/<resource>-success.json`<br>`__files/<domain>/<resource>-success.json` |
| **Validation Error** | `POST /api/v1/<resources>` | Missing required body attribute | `400 Bad Request` | RFC 7807 problem details error payload | `mappings/<domain>/<resource>-invalid.json`<br>`__files/<domain>/<resource>-invalid.json` |
| **Not Found** | `GET /api/v1/<resources>/<not-found-id>` | Explicit missing identifier | `404 Not Found` | Error payload indicating entity does not exist | `mappings/<domain>/<resource>-not-found.json`<br>`__files/<domain>/<resource>-not-found.json` |
| **Transient Fault** | `GET /api/v1/<resources>/<timeout-id>` | Explicit timeout identifier | `200 OK` (delayed) | Injects 5000ms delay to test client timeout thresholds | `mappings/<domain>/<resource>-delay.json`<br>`__files/<domain>/<resource>-default.json` |

---

## 3. How to Run

### Option A: Local Development via Docker Compose (Live Hot-Reload)
The mock service is pre-configured in `compose.yaml` with bind-mounted `mappings/` and `__files/`:

\`\`\`bash
# Start the mock server alongside dependencies:
docker compose up -d <wiremock-service-name>

# Check service logs:
docker compose logs -f <wiremock-service-name>
\`\`\`

### Option B: Immutable Standalone Container (CI/CD Pipelines)
Build an immutable Docker image embedding all stubs, payloads, and extensions:

\`\`\`bash
# Build the container:
docker build -f <mock-root>/Dockerfile.wiremock -t <image-name>:latest <mock-root>

# Run standalone on host port <HOST_PORT>:
docker run --rm -p <HOST_PORT>:8080 <image-name>:latest
\`\`\`

---

## 4. Verification with `curl`

Once the mock server is running on port `<HOST_PORT>`:

\`\`\`bash
# 1. Dynamic query (echoes the ID from path)
curl -i http://localhost:<HOST_PORT>/api/v1/<resources>/sample-123
# HTTP/1.1 200 OK
# {"id": "sample-123", "status": "ACTIVE"}

# 2. Explicit success scenario
curl -i http://localhost:<HOST_PORT>/api/v1/<resources>/<explicit-id>
# HTTP/1.1 200 OK
# {"id": "<explicit-id>", "status": "APPROVED", "tier": "VIP"}

# 3. Not Found scenario (404)
curl -i http://localhost:<HOST_PORT>/api/v1/<resources>/<not-found-id>
# HTTP/1.1 404 Not Found
# {"type": "https://example.com/errors/not-found", "title": "Resource Not Found"}

# 4. WireMock Admin Health Check
curl -i http://localhost:<HOST_PORT>/__admin/health
# HTTP/1.1 200 OK
# {"status": "healthy"}
\`\`\`

---

## 5. Extensions & Custom Features (If Applicable)

Document any custom transformers, state machines, or standalone JAR extensions located in `extensions/` (e.g. JWT token validation, custom response encoders).
```
