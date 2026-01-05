# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Proxyma - Smart HTTP/HTTPS proxy rotator with Prometheus metrics and REST API. Written in Go.

**Перед началом работы обязательно прочитай `DESIGN.md`** — там мотивация, архитектурные решения и план реализации (subissues).

## Development Commands

```bash
# Setup (installs Go via mise)
./init.sh

# Build
make build

# Run locally
./proxyma -config config.yaml

# Run tests
make test

# Lint
make lint

# Format
make fmt

# Install to /usr/local/bin
make install

# Release snapshot (for testing goreleaser)
make release-snapshot
```

## Versioning

Version is embedded at build time via `-ldflags "-X main.version=$(VERSION)"`. Git tags are used for releases.

## Architecture

```
Client → Proxyma → Upstream Proxies → Destination
```

Core components:
- **HTTP/HTTPS Proxy Handler** - Handles incoming requests with CONNECT tunneling support
- **Rotation Engine** - Selects upstream proxy (strategies: round-robin, random, weighted, least-latency)
- **Health Checker** - Background passive/active checks with auto-recovery
- **Circuit Breaker** - Per-proxy failure isolation with cooldown
- **Prometheus /metrics** - Per-proxy stats, latency histograms
- **REST API /api/v1/** - Runtime proxy management

## Expected Directory Structure

```
cmd/proxyma/       # Main entry point
internal/
  proxy/           # HTTP/HTTPS proxy handler
  rotation/        # Rotation strategies
  health/          # Health checker
  circuit/         # Circuit breaker
  metrics/         # Prometheus collectors
  api/             # REST API handlers
```

## Configuration

Единый YAML формат для config и state. При старте можно подать минимальный конфиг (только URLs) или полный дамп с состоянием. См. `DESIGN.md` для деталей.

## Language

Документация и комментарии на русском.
