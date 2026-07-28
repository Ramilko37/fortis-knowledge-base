# Dev VM deploy 2026-07-27

## Контекст

Создана dev VM `webapp-dev-server` для развёртывания frontend и backend Fortis.

- Public IP: `85.208.87.187`
- Internal IP: `10.0.0.5`
- SSH user: `user1`
- OS: Ubuntu 24.04
- Local SSH alias on the operator machine: `fortis-dev-vm`

## Решение

Первичный dev-deploy поднимается через Docker Compose из parent workspace:

- `postgres` — PostgreSQL 17 с persistent Docker volume;
- `backend` — Go API на внутреннем `:8090`, миграции запускаются при старте приложения;
- `frontend` — Next.js runtime на публичном HTTP-порту, серверные proxy/BFF-запросы идут во внутренний backend.

Deploy-файлы живут в parent repo: `deploy/production/`.

## Переменные

Секреты не коммитятся. Для VM используется локальный `.env` на основе `deploy/production/.env.example`.

Минимально обязательные значения:

- `POSTGRES_PASSWORD`
- `APP_AUTH_JWTSECRET`

Для уведомлений о новых сборках на сервере используется Cloudflare Worker
`fortis-build-telegram`, потому что dev VM находится в РФ и не может стабильно
ходить напрямую в `api.telegram.org`. Worker отправляет сообщения в Telegram
bot `@fortis_ci_cd_bot`.

Секретный Telegram token хранится только в Cloudflare Worker secrets. На VM
хранится только `DEPLOY_NOTIFY_WEBHOOK_URL` с path-secret:

- `~/fortis/deploy/production/.env`
- `DEPLOY_NOTIFY_WEBHOOK_URL=https://fortis-build-telegram.galyamdin.workers.dev/deploy/<secret>`

Публичные параметры Telegram-топика в Worker:

- `TELEGRAM_CHAT_ID=-1004435908726`
- `TELEGRAM_THREAD_ID=370`

Запуск deploy с уведомлениями:

```bash
cd ~/fortis/deploy/production
./deploy-with-notify.sh
```

Скрипт отправляет события `started`, `succeeded` и `failed`. Успешная сборка
фиксируется только после `docker compose up -d --build` и healthcheck backend и
frontend внутри Compose-сети.

## Ограничения

На 2026-07-27 backend, frontend и PostgreSQL подняты на VM через Docker Compose. Контейнеры healthy, backend probes отвечают, frontend доступен внутри VM по `127.0.0.1` и `10.0.0.5`.

Публичный доступ по `http://85.208.87.187/` таймаутится несмотря на то, что Docker публикует `0.0.0.0:80` и `ufw` неактивен. Вероятная причина — внешний firewall/security group провайдера: требуется открыть inbound TCP `80` для public IP. В дальнейшем для HTTPS потребуется также inbound TCP `443`.

После открытия TCP `80` frontend публично доступен по `http://85.208.87.187/`. Публичный API-контур остаётся same-origin: `http://85.208.87.187/api/v1/*` обрабатывается Next.js rewrite и проксируется во внутренний backend `http://backend:8090/api/v1/*`. Backend-порт `8090` наружу не публикуется.
