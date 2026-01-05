# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Proxyma - Smart HTTP/HTTPS proxy rotator with Prometheus metrics and REST API. Written in Go.

**Read `DESIGN.md` first** — motivation, architecture decisions, and implementation plan (subissues).

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

Unified YAML format for config and state. On startup, can accept minimal config (URLs only) or full dump with state. See `DESIGN.md` for details.

## Language Policy

- **Documentation**: English first (`*.md`), Russian second (`*.ru.md`)
- **Code comments**: English only
- **When updating docs**: Always update both English and Russian versions
- **Cross-links**: English docs must include link to Russian version (e.g., `🇷🇺 **[FILE.ru.md](FILE.ru.md)**`)
