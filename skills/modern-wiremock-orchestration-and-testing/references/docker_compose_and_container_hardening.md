# Docker, Compose & Container Hardening Guide

Running WireMock in containers ensures consistent behavior across developer workstations, ephemeral CI/CD agents, and preview environments.

---

## 1. Standalone Immutable `Dockerfile.wiremock`

For CI/CD and immutable staging deployments, package stubs, payloads, and optional extensions directly into a container image.

### Production Dockerfile Blueprint:

```dockerfile
# syntax=docker/dockerfile:1
FROM wiremock/wiremock:3.13.2-alpine

# Set working directory to default WireMock container root
WORKDIR /home/wiremock

# Copy declarative mappings and external payloads
COPY --chown=wiremock:wiremock mappings/ /home/wiremock/mappings/
COPY --chown=wiremock:wiremock __files/ /home/wiremock/__files/

# Copy standalone Java extensions if present (optional)
COPY --chown=wiremock:wiremock extensions/ /var/wiremock/extensions/

# Expose internal default WireMock HTTP port
EXPOSE 8080

# Run with global response templating enabled
CMD [ \
  "--global-response-templating", \
  "--disable-gzip", \
  "--verbose" \
]
```

### Build & Run Commands:
```bash
# Build the standalone container:
docker build -f Dockerfile.wiremock -t mock-service:1.0.0 .

# Run the hardened container:
docker run --rm \
  -p 8081:8080 \
  --security-opt no-new-privileges:true \
  --cap-drop ALL \
  mock-service:1.0.0
```

---

## 2. Docker Compose Orchestration (`compose.yaml`)

In local development, developers benefit from hot-reloading mappings and payloads directly from the host filesystem without rebuilding images.

### Compose Spec Blueprint:

```yaml
# Note: Modern Compose Specification is versionless (no top-level `version:` attribute)
name: mock-environment

services:
  wiremock:
    image: wiremock/wiremock:3.13.2-alpine
    container_name: mock-external-api
    restart: unless-stopped
    command:
      - --global-response-templating
      - --disable-gzip
      - --verbose
    ports:
      # Map host port via environment variable with fallback to 8081
      - "${WIREMOCK_PORT:-8081}:8080"
    volumes:
      # Live read-only bind mounts for instant hot-reload during local development
      - ./wiremock/mappings:/home/wiremock/mappings:ro
      - ./wiremock/__files:/home/wiremock/__files:ro
      # Optional: mount extensions only if custom JARs are present
      - ./wiremock/extensions:/var/wiremock/extensions:ro
    security_opt:
      - no-new-privileges:true
    cap_drop:
      - ALL
    healthcheck:
      test: ["CMD-SHELL", "wget -qO- http://localhost:8080/__admin/health || exit 1"]
      interval: 5s
      timeout: 3s
      retries: 5
      start_period: 5s
    networks:
      - internal-net

  # Consuming backend microservice waiting for WireMock readiness
  backend-service:
    build:
      context: .
      dockerfile: Dockerfile
    environment:
      EXTERNAL_API_BASE_URL: http://wiremock:8080
    depends_on:
      wiremock:
        condition: service_healthy
    networks:
      - internal-net

networks:
  internal-net:
    driver: bridge
```

---

## 3. Container Hardening Checklist

- [x] **Pinned Image Tag**: Use pinned versions such as `wiremock/wiremock:3.9.1-alpine` (never `:latest`).
- [x] **Unprivileged User**: The official WireMock image runs under the `wiremock` (UID 1000) non-root user. Preserve permissions via `--chown=wiremock:wiremock`.
- [x] **Read-Only Bind Mounts**: Append `:ro` to volume mounts to prevent container processes from altering host JSON files.
- [x] **Security Opts**: Set `no-new-privileges:true` and drop Linux capabilities (`cap_drop: [ALL]`).
- [x] **Choreographed Healthcheck**: Always configure healthchecks against `/__admin/health` and guard downstream services with `condition: service_healthy`.
