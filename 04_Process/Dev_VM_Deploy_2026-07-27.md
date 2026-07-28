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
./deploy/production/write-deploy-info.sh
rsync -az --delete --exclude 'production/.env' deploy/ fortis-dev-vm:~/fortis/deploy/

cd ~/fortis/deploy/production
./deploy-with-notify.sh
```

Скрипт отправляет события `started`, `succeeded` и `failed`. Успешная сборка
фиксируется только после `docker compose up -d --build` и healthcheck backend и
frontend внутри Compose-сети.

`write-deploy-info.sh` должен запускаться перед копированием `deploy/` на VM.
Он создаёт `deploy/production/.deploy-info` с commit metadata parent, frontend и
backend (`ref`, branch, subject). Уведомления читают этот файл, потому что на VM
код лежит без `.git`.

## Ограничения

На 2026-07-27 backend, frontend и PostgreSQL подняты на VM через Docker Compose. Контейнеры healthy, backend probes отвечают, frontend доступен внутри VM по `127.0.0.1` и `10.0.0.5`.

Публичный доступ по `http://85.208.87.187/` таймаутится несмотря на то, что Docker публикует `0.0.0.0:80` и `ufw` неактивен. Вероятная причина — внешний firewall/security group провайдера: требуется открыть inbound TCP `80` для public IP. В дальнейшем для HTTPS потребуется также inbound TCP `443`.

После открытия TCP `80` frontend публично доступен по `http://85.208.87.187/`. Публичный API-контур остаётся same-origin: `http://85.208.87.187/api/v1/*` обрабатывается Next.js rewrite и проксируется во внутренний backend `http://backend:8090/api/v1/*`. Backend-порт `8090` наружу не публикуется.

## Операционный доступ

SSH:

```bash
ssh -i /path/to/id_rsa user1@85.208.87.187
```

Project dir на VM: `~/fortis`.

Docker Compose:

```bash
cd ~/fortis/deploy/production
sudo docker compose --env-file .env ps
sudo docker compose --env-file .env logs -f backend
sudo docker compose --env-file .env logs -f frontend
sudo docker compose --env-file .env up -d --build
```

Публичные URL:

- Frontend: `http://85.208.87.187/`
- Prototype: `http://85.208.87.187/prototype`
- API через frontend proxy: `http://85.208.87.187/api/v1/*`

Health внутри VM/container:

- Liveness: `http://127.0.0.1:8090/_/liveness`
- Readiness: `http://127.0.0.1:8090/_/readiness`

Локальный frontend dev по умолчанию использует dev VM public proxy как backend base URL. Docker/deploy окружение должно явно задавать `FORTIS_API_BASE_URL=http://backend:8090`, чтобы контейнер frontend ходил во внутренний backend service.
