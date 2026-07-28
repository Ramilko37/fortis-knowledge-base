# ADR-0008: frontend boundary для API библиотеки средств

- **Дата:** 2026-06-13
- **Статус:** Accepted
- **Теги:** GIS, Frontend, Backend, API, FRT-48

## Контекст

Backend в FRT-33 предоставляет CRUD API библиотеки средств защиты:

- `GET /api/v1/assets`
- `GET /api/v1/assets/get`
- `POST /api/v1/assets`
- `PUT /api/v1/assets/update`
- `DELETE /api/v1/assets/delete`

Frontend FRT-48 должен использовать эти эндпоинты из `/prototype`, но Next.js с `trailingSlash: true` перенаправлял same-origin запрос `/api/v1/assets` на `/api/v1/assets/`, после чего запрос попадал в Next fallback и возвращал `404`.

## Решение

Frontend оставляет browser-facing URL same-origin (`/api/v1/assets...`) и проксирует asset API через Next rewrites на backend.

Переменная backend base URL:

- `FORTIS_API_BASE_URL`
- default для локального frontend dev: `http://85.208.87.187` (dev VM frontend proxy, публичный `/api/v1/*`)
- `NEXT_PUBLIC_FORTIS_API_BASE_URL` допускается как compatibility fallback, но секретов в ней быть не должно.

Docker/deploy окружения должны задавать `FORTIS_API_BASE_URL` явно. На dev VM frontend container использует внутренний backend `http://backend:8090`, а не публичный IP.

Для asset routes включён `skipTrailingSlashRedirect: true`, чтобы Next не превращал `/api/v1/assets` в `/api/v1/assets/`.

## Контракт единиц измерения

Backend DTO хранит и возвращает дальности в километрах:

- `minEffectiveDistance`
- `maxEffectiveDistance`
- `coverageRadius`

Frontend `DefenseProject.assetLibrary` работает в метрах.

Правило преобразования:

- backend -> frontend: километры умножаются на `1000`;
- frontend -> backend mutations: метры делятся на `1000`;
- `coverageAngle` не конвертируется.

## Поведение при недоступном backend

Если backend недоступен, `/prototype` должен оставаться работоспособным и использовать мягкий fallback на локальную библиотеку средств. Ошибка proxy не должна ломать рендер карты.

## Проверка

FRT-48 проверяется focused tests через `pnpm exec tsx`, `pnpm exec eslint`, `pnpm build`, а также browser smoke `/prototype` с backend:

- `GET /api/v1/assets?isPublic=true&limit=5` через Next proxy возвращает `200`;
- `POST /api/v1/assets` через Next proxy создаёт карточку и возвращает `201`;
- созданная backend-карточка появляется в `/prototype`;
- при остановленном backend fallback остаётся доступным.
