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

## Следствия

- `/workspace` становится авторизованной точкой входа GIS-flow: предприятие -> конфигурация -> `/prototype` -> `/calculator`.
- Local fallback остаётся допустимым только для unsaved drafts и offline/empty catalog states.
- Любые будущие frontend save/load/report/compare изменения должны уважать backend optimistic locking и current project context.
