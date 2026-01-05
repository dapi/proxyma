# Proxyma

Smart HTTP/HTTPS proxy rotator with Prometheus metrics and REST API.

📖 **[DESIGN.md](DESIGN.md)** — motivation, architecture, and implementation plan.

🇷🇺 **[README.ru.md](README.ru.md)** — Russian version.

## Features

- HTTP/HTTPS proxy with CONNECT tunneling
- Rotation strategies: round-robin, random, weighted, least-latency
- Health checks (passive + active) and circuit breaker
- Per-proxy Prometheus metrics
- REST API for runtime management
- Unified config/state format with auto-save

## Quick Start

```bash
# Minimal config
cat > config.yaml << EOF
proxies:
  - url: "http://user:pass@proxy1:8080"
  - url: "http://user:pass@proxy2:8080"
EOF

# Run
proxyma -config config.yaml

# Test
curl -x http://localhost:8089 http://httpbin.org/ip
```

## Usage

```bash
# CLI
proxyma -config config.yaml

# With state auto-save
proxyma -config config.yaml -state-file /data/state.yaml -state-interval 60s

# Docker
docker run -p 8089:8089 -v ./config.yaml:/config.yaml ghcr.io/dapi/proxyma
```

## API

```
GET    /api/v1/proxies              # List proxies
GET    /api/v1/config               # Full dump (config + state)
GET    /metrics                     # Prometheus
GET    /health                      # Health check
```

## Development

```bash
make build    # Build
make test     # Tests
make lint     # Linter
```

## License

MIT
