# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the OpenTelemetry Astronomy Shop Demo, a microservice-based distributed system designed to illustrate OpenTelemetry instrumentation and observability in a near real-world environment.

## Architecture

### Microservices Architecture
The demo consists of multiple microservices implemented in different languages:

- **Frontend** (Next.js/TypeScript): Main web UI at `/src/frontend/`
- **Frontend Proxy** (Nginx): Reverse proxy for frontend at `/src/frontend-proxy/`
- **Backend Services**: Located in individual directories under `/src/`:
  - `accounting` (.NET): Processes order accounting events
  - `ad` (Java): Serves advertisements  
  - `cart` (DotNet): Shopping cart service
  - `checkout` (Go): Handles order checkout process
  - `currency` (Node.js): Currency conversion service
  - `email` (Ruby): Email notification service
  - `fraud-detection` (Kotlin): Fraud detection service
  - `payment` (JavaScript): Payment processing
  - `product-catalog` (Go): Product catalog and inventory
  - `quote` (PHP): Shipping quote calculation
  - `recommendation` (Python): Product recommendation engine
  - `shipping` (Rust): Shipping management

### Infrastructure Components
- **OpenTelemetry Collector**: Central telemetry collection (`/src/otel-collector/`)
- **Flagd**: Feature flag service (`/src/flagd/`)
- **Flagd UI**: Feature flag management interface (`/src/flagd-ui/`)
- **Load Generator**: Traffic simulation (`/src/load-generator/`)
- **Grafana**: Observability dashboard configuration (`/src/grafana/`)
- **Prometheus**: Metrics collection (`/src/prometheus/`)
- **Postgres**: Database service (`/src/postgres/`)
- **Kafka**: Message queue service (`/src/kafka/`)
- **Image Provider**: Static image serving (`/src/image-provider/`)

## Development Commands

### Docker-based Development (Primary Method)
```bash
# Start the full demo
make start

# Start minimal version (fewer services)
make start-minimal

# Stop all services
make stop

# Build all Docker images
make build

# Run integration tests
make run-tests

# Restart a specific service
make restart service=frontend

# Rebuild and restart a service
make redeploy service=frontend
```

### Individual Service Development
Some services have their own development scripts:

**Frontend (Next.js)**:
```bash
cd src/frontend
npm run dev          # Development server
npm run build        # Production build
npm run lint         # ESLint
```

**Other JavaScript services**:
```bash
cd src/[service-name]
npm run dev          # Development server (if available)
npm run build        # Build (if available)
```

### Code Quality and Testing
```bash
# Lint all markdown files
make markdownlint

# Check spelling in documentation
make misspell

# Fix spelling issues
make misspell-correction

# Run all code quality checks
make check

# Check license headers
make checklicense

# Add license headers
make addlicense

# Check links in documentation
make checklinks
```

### Protocol Buffers
```bash
# Generate protobuf files (requires Docker)
make docker-generate-protobuf

# Generate protobuf files (local tools)
make generate-protobuf
```

## Environment Configuration

### Environment Files
- `.env`: Main configuration with image versions and service ports
- `.env.override`: Local overrides (create this for custom settings)
- `.env.arm64`: ARM64-specific settings (automatically used on Apple Silicon)

### Key Environment Variables
- `DEMO_VERSION`: Demo version tag (default: "latest")
- `IMAGE_NAME`: Base Docker image name
- `OTEL_COLLECTOR_HOST`: OpenTelemetry Collector host
- `FLAGD_HOST/FLAGD_PORT`: Feature flag service configuration

## Testing

### Integration Testing
The project uses Docker Compose for integration testing:
```bash
make run-tests                    # Run all tests
make run-tracetesting            # Run trace-based tests
SERVICES_TO_TEST=frontend make run-tracetesting  # Test specific service
```

### Frontend Testing
```bash
cd src/frontend
npm run cy:open      # Open Cypress interactive testing
npm run cy:run       # Run Cypress tests headlessly
```

## Linting and Type Checking

### Frontend
```bash
cd src/frontend
npm run lint         # ESLint for TypeScript/JavaScript
npx tsc --noEmit     # TypeScript type checking
```

### Documentation
```bash
make markdownlint    # Lint all markdown files
make yamllint        # Lint YAML files
```

## Key Deployment Targets

- **Local Demo**: Full Docker Compose setup for local development
- **Kubernetes**: Helm charts and manifests in `/kubernetes/` directory
- **Various Observability Backends**: The demo supports multiple vendors (see README for full list)

## Port Configuration

Services expose ports based on environment variables defined in `.env`. Key ports:
- Frontend: `8080` (main demo UI)
- Jaeger UI: `8080/jaeger/ui`
- Grafana: `8080/grafana/`
- Load Generator UI: `8080/loadgen/`
- Feature Flags UI: `8080/feature/`

## Common Workflows

1. **Adding a new service**: Create directory under `/src/`, add Dockerfile, update `docker-compose.yml`
2. **Modifying observability**: Update configurations in `/src/otel-collector/`, `/src/grafana/`, or `/src/prometheus/`
3. **Changing feature flags**: Modify `/src/flagd/` configuration
4. **Testing changes**: Use `make run-tests` for integration testing
5. **Documentation updates**: Run `make check` to verify formatting and links

### Code Optimization Principles
**CRITICAL**: Always prioritize code optimization and efficiency when writing or modifying code:

**Line Minimization:**
- Use list comprehensions instead of explicit for loops: `[f(x) for x in items]`
- Leverage dictionary comprehensions: `{k: v for k, v in items}`
- Combine operations using method chaining when possible
- Use functional programming patterns with `map`, `filter`, `reduce`
- Prefer vectorized operations over iterative approaches

**Conditional Minimization:**
- Replace if/else chains with dictionary-driven lookups: `actions[key]()`
- Use ternary operators for simple conditions: `value if condition else default`
- Leverage boolean algebra to eliminate nested conditionals
- Use `all()` and `any()` for multiple condition checks
- Replace switch-like patterns with configuration dictionaries

**Exception Handling Optimization:**
- Use EAFP (Easier to Ask for Forgiveness than Permission) pattern
- Combine multiple exception types: `except (ValueError, TypeError) as e:`
- Use context managers (`with` statements) for resource management
- Prefer early returns to reduce nesting depth
- Use exception chaining with `raise ... from e` for debugging
- Implement centralized error handling where possible

**Examples:**
```python
# BAD - Explicit loops and conditionals
results = []
for item in data:
    if item.status == 'active':
        if item.value > threshold:
            results.append(process_item(item))

# GOOD - Optimized functional approach  
results = [process_item(item) for item in data 
          if item.status == 'active' and item.value > threshold]

# BAD - Multiple if/else statements
if action == 'buy':
    execute_buy()
elif action == 'sell':
    execute_sell()
elif action == 'hold':
    execute_hold()

# GOOD - Dictionary-driven approach
actions = {'buy': execute_buy, 'sell': execute_sell, 'hold': execute_hold}
actions.get(action, lambda: None)()
```
