# DESIGN.ru.md — Proxyma

## Мотивация

Существующие прокси-ротаторы (mubeng, go-proxy-rotator) не предоставляют:
- Per-proxy метрики (success rate, latency, errors)
- Health checks и circuit breaker
- API для управления прокси
- Информацию о том, какой прокси использовался

Proxyma решает эти проблемы.

## Архитектура

```
┌─────────┐     ┌──────────────────────────────────────────┐     ┌──────────────┐
│ Client  │────▶│              Proxyma                     │────▶│  Upstream    │
│         │     │                                          │     │  Proxies     │
└─────────┘     │  ┌────────────┐    ┌──────────────────┐  │     └──────────────┘
                │  │ HTTP/HTTPS │    │ Rotation Engine  │  │            │
                │  │ Proxy      │───▶│ - round-robin    │  │            ▼
                │  │ Handler    │    │ - weighted       │  │     ┌──────────────┐
                │  └────────────┘    │ - least-errors   │  │     │ Destination  │
                │                    └──────────────────┘  │     │ Server       │
                │  ┌────────────┐    ┌──────────────────┐  │     └──────────────┘
                │  │ Prometheus │    │ Health Checker   │  │
                │  │ /metrics   │    │ (background)     │  │
                │  └────────────┘    └──────────────────┘  │
                │                                          │
                │  ┌────────────┐    ┌──────────────────┐  │
                │  │ REST API   │    │ Circuit Breaker  │  │
                │  │ /api/v1/*  │    │ (per-proxy)      │  │
                │  └────────────┘    └──────────────────┘  │
                └──────────────────────────────────────────┘
```

## Ключевые решения

### Storage: Config = State

Единый YAML формат для входа и выхода. При старте можно подать:
- Минимальный конфиг (только URLs)
- Полный дамп с сохранённым состоянием

```yaml
# Минимальный вход
proxies:
  - url: "http://user:pass@proxy1:8080"
  - url: "http://user:pass@proxy2:8080"
```

```yaml
# Полный дамп (GET /api/v1/config или автосохранение)
server:
  address: "0.0.0.0:8089"

rotation:
  strategy: "weighted"

proxies:
  - url: "http://user:pass@proxy1:8080"
    weight: 10                    # default: 1
    healthy: true
    enabled: true                 # можно отключить вручную
    circuit: "closed"             # closed | open | half-open
    stats:
      requests: 1523
      errors: 12
      success_rate: 0.992
      latency_p50_ms: 145
      latency_p99_ms: 890
    last_used_at: "2024-01-15T10:29:55Z"
    last_check_at: "2024-01-15T10:29:00Z"

  - url: "http://user:pass@proxy2:8080"
    weight: 5
    healthy: false
    enabled: true
    circuit: "open"
    circuit_until: "2024-01-15T10:35:00Z"
    stats:
      requests: 892
      errors: 47
      success_rate: 0.947
      latency_p50_ms: 230
      latency_p99_ms: 1200
    last_error: "connection refused"
    last_used_at: "2024-01-15T10:25:12Z"

exported_at: "2024-01-15T10:30:00Z"  # информационное, игнорируется при импорте
```

### Логика парсинга при старте

```
Читаем YAML:
├── Есть только url? → weight=1, healthy=true, stats=zero
├── Есть weight? → используем
├── Есть stats/healthy/circuit? → восстанавливаем состояние
└── exported_at? → игнорируем
```

### Persistence

In-memory state с опциональным автосохранением:

```bash
# Без persistence (state теряется при рестарте)
proxyma -config proxies.yaml

# С автосохранением каждые 60 сек
proxyma -config config.yaml -state-file /pvc/state.yaml -state-interval 60s

# Восстановление из дампа
proxyma -config /pvc/state-dump.yaml
```

## REST API

```
GET    /api/v1/proxies              # Список прокси со статистикой
POST   /api/v1/proxies              # Добавить прокси
DELETE /api/v1/proxies/{id}         # Удалить прокси
PUT    /api/v1/proxies/{id}/enable  # Включить
PUT    /api/v1/proxies/{id}/disable # Отключить
POST   /api/v1/proxies/{id}/reset   # Сбросить статистику

GET    /api/v1/config               # Полный дамп (YAML/JSON)
POST   /api/v1/reload               # Перечитать конфиг

GET    /health                      # Health check
GET    /metrics                     # Prometheus metrics
```

## Prometheus Metrics

```prometheus
# Per-proxy request counts
proxyma_requests_total{upstream="proxy1", status="success"}
proxyma_requests_total{upstream="proxy1", status="error"}

# Per-proxy error types
proxyma_errors_total{upstream="proxy1", error="timeout"}
proxyma_errors_total{upstream="proxy1", error="connection_refused"}

# Per-proxy latency histogram
proxyma_request_duration_seconds_bucket{upstream="proxy1", le="0.5"}

# Health status
proxyma_proxy_healthy{upstream="proxy1"} 1

# Global stats
proxyma_active_proxies
proxyma_healthy_proxies
```

## Subissues (план реализации)

1. **Config Parser** — YAML с graceful defaults
2. **Proxy Pool + State** — in-memory с sync.RWMutex
3. **Core Proxy** — HTTP/HTTPS handler с CONNECT tunneling
4. **Rotation Engine** — round-robin, random, weighted, least-latency
5. **Health Checker** — passive + active checks
6. **Circuit Breaker** — per-proxy изоляция с cooldown
7. **Metrics** — Prometheus collectors
8. **REST API** — /config, /reload, /proxies/*
9. **State Persistence** — автодамп в файл
