# Proxyma

Smart HTTP/HTTPS proxy rotator с Prometheus метриками и REST API для управления.

## Зачем?

Существующие прокси-ротаторы (mubeng, go-proxy-rotator) не предоставляют:
- Per-proxy метрики (success rate, latency, errors)
- Health checks и circuit breaker
- API для управления прокси
- Информацию о том, какой прокси использовался

Proxyma решает все эти проблемы.

## Возможности

- **HTTP/HTTPS proxy** с поддержкой CONNECT tunneling
- **Стратегии ротации**: round-robin, random, weighted, least-latency
- **Health management**: passive/active checks, circuit breaker, auto-recovery
- **Prometheus метрики**: per-proxy статистика, latency histograms
- **REST API**: управление прокси в runtime
- **Hot reload** конфигурации

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

## Конфигурация

```yaml
server:
  address: "0.0.0.0:8089"
  auth:
    username: "user"
    password: "pass"

rotation:
  strategy: "weighted"  # round-robin, random, weighted, least-latency

health_check:
  enabled: true
  interval: 30s
  timeout: 10s
  test_url: "http://httpbin.org/ip"

circuit_breaker:
  failure_threshold: 5
  cooldown: 60s

proxies:
  - url: "http://user:pass@proxy1:8080"
    weight: 10
  - url: "http://user:pass@proxy2:8080"
    weight: 5
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
proxyma_success_rate
```

## REST API

```
GET    /api/v1/proxies              # Список прокси со статистикой
POST   /api/v1/proxies              # Добавить прокси
DELETE /api/v1/proxies/{id}         # Удалить прокси
PUT    /api/v1/proxies/{id}/enable  # Включить
PUT    /api/v1/proxies/{id}/disable # Отключить
POST   /api/v1/proxies/{id}/reset   # Сбросить статистику

GET    /health                      # Health check
GET    /metrics                     # Prometheus metrics
```

## Использование

### Docker

```bash
docker run -p 8089:8089 -v ./config.yaml:/config.yaml ghcr.io/dapi/proxyma
```

### Kubernetes (Helm)

```bash
helm install proxyma ./charts/proxyma -n production
```

### CLI

```bash
proxyma -config config.yaml
```

## Тестирование

```bash
# Через proxyma
curl -x http://user:pass@localhost:8089 http://httpbin.org/ip

# Проверка метрик
curl http://localhost:8089/metrics

# API статистика
curl http://localhost:8089/api/v1/proxies
```

## Разработка

```bash
# Запуск
go run ./cmd/proxyma -config config.yaml

# Тесты
go test ./...

# Линтер
golangci-lint run

# Билд
go build -o proxyma ./cmd/proxyma
```

## License

MIT
