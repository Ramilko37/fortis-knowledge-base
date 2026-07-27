# ADR-0010: frontend boundary для backend-backed GIS flow

**Дата:** 2026-06-15  
**Статус:** Accepted

## Контекст

Свежий backend-контур Fortis добавил авторизацию, enterprises, проекты с optimistic locking, budget/cost/report/compare и documents API. Frontend должен работать с этим контуром как с источником правды для сохранённых GIS-конфигураций, не дублируя JWT-хранение и не создавая отдельные локальные project-контексты для `/prototype` и `/calculator`.

## Решение

- Browser-клиент ходит в auth через same-origin routes `/api/auth/login`, `/api/auth/register`, `/api/auth/logout`, `/api/auth/me`.
- JWT хранится только в HttpOnly cookie `access-token`; frontend не кладёт токен в `localStorage`.
- Защита `/prototype`, `/calculator` и `/workspace` выполняется через Next `src/proxy.ts`, с редиректом на `/login?next=...` до рендера protected UI.
- Временное демо-уточнение от 2026-06-15: пока backend auth не поднят в окружениях, `src/proxy.ts` включает redirect guard только при `FORTIS_AUTH_ENABLED=true`. По умолчанию `/prototype`, `/calculator` и `/workspace` остаются открытыми, чтобы не блокировать frontend demo.
- Server-side frontend routes, которые проксируют Go backend, обязаны пробрасывать `access-token` как `Authorization: Bearer ...` и сохранять исходный `Cookie`.
- `DefenseProject.version` является optional: локальные черновики могут существовать без версии, backend-проекты должны сохранять версию из ответа и отправлять её при update.
- Единый current project context для `/prototype` и `/calculator` живёт в `useDefenseProjectStore`: `projectId`, `enterpriseId`, `projectName`, `version?`, полный `DefenseProject`.
- При `409 Conflict` frontend показывает видимое состояние конфликта и действие перезагрузки актуальной версии, без silent overwrite.
- Budget/cost/report/compare/documents подключаются через typed API helpers к backend routes `/api/v1/projects/*` и `/api/v1/assets/documents/*`.
- На 2026-07-27 demo routes `/api/defense/catalog` и `/api/defense/facilities` больше не являются чистыми моками: сначала они пробуют backend `/api/v1/assets` и `/api/v1/enterprises` через server-side proxy, адаптируют ответ в legacy demo-типы и fallback'ятся на mock-data при `401`, `404` или пустом backend-каталоге.
- `backend-proxy.ts` должен принимать и `BACKEND_URL`, и deploy-переменную `FORTIS_API_BASE_URL`; обе нормализуются до `/api/v1`.
- `/prototype` показывает backend documents metadata для карточек средств защиты через `assets/documents/list`, `assets/documents`, `assets/documents/delete` и download URL.
- `/calculator` для сохранённых backend-проектов загружает backend budget config, даёт сохранить бюджетный лимит на backend и может выполнить явную backend-проверку кандидата через `/api/v1/projects/budget/check`, если frontend asset/layer можно сопоставить с backend id.

## Следствия

- `/workspace` становится авторизованной точкой входа GIS-flow: предприятие -> конфигурация -> `/prototype` -> `/calculator`.
- Local fallback остаётся допустимым только для unsaved drafts и offline/empty catalog states.
- Любые будущие frontend save/load/report/compare изменения должны уважать backend optimistic locking и current project context.
- Demo-only routes `/api/defense/layers`, `/api/defense/evaluate` и `/api/defense/recommend` остаются на mock-репозитории до появления соответствующих backend endpoints.
