# ADR-0010: frontend boundary для backend-backed GIS flow

**Дата:** 2026-06-15  
**Статус:** Accepted

## Контекст

Свежий backend-контур Fortis добавил авторизацию, enterprises, проекты с optimistic locking, budget/cost/report/compare и documents API. Frontend должен работать с этим контуром как с источником правды для сохранённых GIS-конфигураций, не дублируя JWT-хранение и не создавая отдельные локальные project-контексты для `/prototype` и `/calculator`.

## Решение

- Browser-клиент ходит в auth через same-origin routes `/api/auth/login`, `/api/auth/register`, `/api/auth/logout`, `/api/auth/me`.
- JWT хранится только в HttpOnly cookie `access-token`; frontend не кладёт токен в `localStorage`.
- Защита `/prototype`, `/calculator` и `/workspace` выполняется через Next `src/proxy.ts`, с редиректом на `/login?next=...` до рендера protected UI.
- С 2026-07-28 `/login` валидирует `next` только на server page и передаёт в client UI уже нормализованный `redirectTo`; разрешены только внутренние пути `/workspace`, `/prototype`, `/calculator` и их подпути, остальные значения падают в `/workspace`.
- На 2026-07-28 `/prototype` является защищённым auth route по умолчанию: без HttpOnly cookie `access-token` Next `src/proxy.ts` редиректит на `/login?next=/prototype...`. Старый env-флаг `FORTIS_AUTH_ENABLED` больше не управляет доступом к `/prototype`.
- `/calculator` и `/workspace` пока не включены в обязательный auth guard этим шагом, чтобы не расширять изменение за пределы запуска backend-backed `/prototype`.
- После входа `/prototype` bootstrap'ит backend-backed вариант: если в URL есть `projectId`/`project`, загружает его через `/api/defense/projects/:id`; иначе создаёт первый backend-вариант из текущей карты с именем «Тестовый терминал Екатеринбург».
- Для активного backend-варианта `/prototype` выполняет debounce autosave через `PUT /api/defense/projects/:id` и отправляет текущую `version`. Ответ backend обновляет optimistic-lock version, но не затирает более свежие локальные изменения, сделанные во время текущего save.
- Server-side frontend routes, которые проксируют Go backend, обязаны пробрасывать `access-token` как `Authorization: Bearer ...` и сохранять исходный `Cookie`.
- `DefenseProject.version` является optional: локальные черновики могут существовать без версии, backend-проекты должны сохранять версию из ответа и отправлять её при update.
- С 2026-07-28 frontend отправляет `enterpriseId` в save/update только если значение является backend UUID. Локальные demo id вроде `obj-1`/`facility-alpha` остаются внутри `DefenseProject.baseObject.id`, но не попадают в UUID-колонку `defense_projects.enterprise_id`.
- Backend update сохраняет пустой `enterpriseId` как `NULL`, возвращает следующую optimistic-lock `version` после успешного update, а export проекта включает `version`, чтобы загруженный проект можно было сохранить повторно без ложного конфликта.
- Единый current project context для `/prototype` и `/calculator` живёт в `useDefenseProjectStore`: `projectId`, `enterpriseId`, `projectName`, `version?`, полный `DefenseProject`.
- При `409 Conflict` frontend показывает видимое состояние конфликта и действие перезагрузки актуальной версии, без silent overwrite.
- Budget/cost/report/compare/documents подключаются через typed API helpers к backend routes `/api/v1/projects/*` и `/api/v1/assets/documents/*`.
- На 2026-07-27 demo routes `/api/defense/catalog` и `/api/defense/facilities` больше не являются чистыми моками: сначала они пробуют backend `/api/v1/assets` и `/api/v1/enterprises` через server-side proxy, адаптируют ответ в legacy demo-типы и fallback'ятся на mock-data при `401`, `404` или пустом backend-каталоге.
- На 2026-07-28 `/prototype` читает библиотеку средств через auth-aware BFF `/api/defense/assets`, а не через прямой browser rewrite `/api/v1/assets`, потому что rewrite не может превратить HttpOnly `access-token` cookie в backend `Authorization: Bearer ...`.
- `backend-proxy.ts` должен принимать и `BACKEND_URL`, и deploy-переменную `FORTIS_API_BASE_URL`; обе нормализуются до `/api/v1`.
- `/prototype` показывает backend documents metadata для карточек средств защиты через `assets/documents/list`, `assets/documents`, `assets/documents/delete` и download URL.
- `/calculator` для сохранённых backend-проектов загружает backend budget config, даёт сохранить бюджетный лимит на backend и может выполнить явную backend-проверку кандидата через `/api/v1/projects/budget/check`, если frontend asset/layer можно сопоставить с backend id.
- FRT-148 security baseline: frontend держит Next.js на актуальном Active LTS security patch, а `pnpm-workspace.yaml` включает supply-chain hardening (`minimumReleaseAge: 10080`, `trustPolicy: no-downgrade`) с точечными pinned-исключениями только для проверенных security-release/legacy transitive packages.

## Следствия

- `/workspace` становится авторизованной точкой входа GIS-flow: предприятие -> конфигурация -> `/prototype` -> `/calculator`.
- Local fallback остаётся допустимым только для unsaved drafts и offline/empty catalog states.
- Любые будущие frontend save/load/report/compare изменения должны уважать backend optimistic locking и current project context.
- Demo-only routes `/api/defense/layers`, `/api/defense/evaluate` и `/api/defense/recommend` остаются на mock-репозитории до появления соответствующих backend endpoints.
