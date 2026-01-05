# Proxyma

Smart HTTP/HTTPS proxy rotator с Prometheus метриками и REST API.

📖 **[DESIGN.ru.md](DESIGN.ru.md)** — мотивация, архитектура и план реализации.

## Возможности

- HTTP/HTTPS proxy с CONNECT tunneling
- Стратегии ротации: round-robin, random, weighted, least-latency
- Health checks (passive + active) и circuit breaker
- Prometheus метрики per-proxy
- REST API для управления в runtime
- Единый формат config/state с автосохранением

## Быстрый старт

```bash
# Минимальный конфиг
cat > config.yaml << EOF
proxies:
  - url: "http://user:pass@proxy1:8080"
  - url: "http://user:pass@proxy2:8080"
EOF

# Запуск
proxyma -config config.yaml

# Проверка
curl -x http://localhost:8089 http://httpbin.org/ip
```

## Использование

```bash
# CLI
proxyma -config config.yaml

# С автосохранением состояния
proxyma -config config.yaml -state-file /data/state.yaml -state-interval 60s

# Docker
docker run -p 8089:8089 -v ./config.yaml:/config.yaml ghcr.io/dapi/proxyma
```

## API

```
GET    /api/v1/proxies              # Список прокси
GET    /api/v1/config               # Полный дамп (config + state)
GET    /metrics                     # Prometheus
GET    /health                      # Health check
```

## Разработка

```bash
make build    # Сборка
make test     # Тесты
make lint     # Линтер
```

## License

MIT
