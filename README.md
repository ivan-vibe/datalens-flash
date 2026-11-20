# DataLens Flash

Один репозиторий, две команды — и DataLens работает на маке (Apple Silicon):

```bash
docker compose pull
docker compose up -d
```

Открыть **http://localhost:8091**, войти **admin / admin**.

## Что внутри

| сервис | образ | что делает |
|---|---|---|
| `ui` | `ghcr.io/ivan-vibe/datalens-flash-ui` | официальный UI DataLens + наш форк (Theme Studio, стриминг чартов) |
| `monolith` | `ghcr.io/ivan-vibe/datalens-flash-monolith` | весь серверный Rust: US, data-api, charts engine, control-api, meta-manager **и auth** |
| `postgres` | `ghcr.io/datalens-tech/datalens-postgres:16-1.3.0` | официальная база DataLens |

Официальные демо-данные монолит заливает **сам** при первом старте, если в `pg-us-db` нет
воркбуков (в бинарник встроен официальный demo-набор из образа Postgres). Отдельного
seed-контейнера нет.

Ничего не собирается локально: `docker compose pull` скачивает готовые образы. База и демо-данные —
только из официального образа DataLens, в этом репозитории нет ни дампов, ни пользовательских строк.

## Требования

- Docker Desktop для macOS (Apple Silicon);
- ~4 ГБ RAM, свободный порт `8091` (UI) и `8099` (API, только loopback).

## Настройки

Скопируй `.env.example` в `.env`, если нужно поменять порт или тег образов:

```bash
cp .env.example .env
# UI_PORT=8092        # другой порт UI
# FLASH_VERSION=v0.1.0  # тег образов
```

## Первый запуск

Первый `up` занимает ~1–2 минуты: postgres создаёт базы, монолит накатывает миграции,
создаёт схему auth и пользователя `admin/admin`, затем сам заливает демо-воркбук.
Прогресс видно в `docker compose logs -f monolith` (`demo seed: official demo data loaded`).

## Подключения к сервисам на хосте

Монолит работает в контейнере, поэтому `127.0.0.1` внутри него — это сам контейнер, а не
мак. Для внешней базы (например, ClickHouse на хосте) указывайте:

- `host.docker.internal` и **опубликованный на хосте** порт (для ClickHouse это HTTP-порт),
  он уже прописан в `extra_hosts` сервиса `monolith`;
- либо имя контейнера и его внутренний порт, если база в той же docker-сети
  (`finops-clickhouse:8123`).

Проверено: `test_connection_params` для ClickHouse с `host.docker.internal:18123` и с
`finops-clickhouse:8123` отвечает 200.

## Остановить / снести

```bash
docker compose down        # остановить, данные сохранить
docker compose down -v     # остановить и удалить базу (демо зальётся заново)
```

## Обновить

```bash
docker compose pull && docker compose up -d
```

## Откуда образы

- `datalens-flash-monolith` собирается из
  [datalens-rust-monolith-embedded-auth](https://github.com/ivan-vibe/datalens-rust-monolith-embedded-auth)
  (один бинарник, auth внутри);
- `datalens-flash-ui` собирается из официального `datalens-ui:0.3831.0` + собранного dist
  [datalens-ui-crab](https://github.com/ivan-vibe/datalens-ui-crab);
- workflow `.github/workflows/images.yml` пересобирает оба образа и пушит их в GHCR.

## Лицензия и дисклеймер

Apache-2.0 (см. [LICENSE](LICENSE), [NOTICE](NOTICE)). Это неофициальный проект, не связанный
с DataLens, его мейнтейнерами или Яндексом; DataLens и связанные логотипы — товарные знаки
соответствующих владельцев.
