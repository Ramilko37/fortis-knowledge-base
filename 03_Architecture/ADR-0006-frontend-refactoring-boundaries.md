# ADR-0006: границы frontend-рефакторинга без изменения поведения

- **Дата:** 2026-06-11
- **Статус:** accepted
- **Контекст:** рефакторинг `frontend` после аудита React/Next.js best practices

## Решение

Frontend закрепляет следующие границы:

- `src/app/**` остаётся слоем маршрутизации Next.js App Router: route-файлы должны быть тонкими и рендерить root-компоненты модулей.
- Пользовательские экраны живут в `src/modules/*/ui`, а чистая workflow/domain-логика — в `src/modules/*/domain`.
- Общие UI primitives и helpers должны иметь canonical imports из `src/shared/ui` и `src/shared/lib`.
- Mock-runtime логика, которая зависит от mock data, принадлежит `infra`, а не `domain`.
- Тяжёлые client-only поверхности, например 3D-сцена, можно грузить через `next/dynamic`, если это не меняет route и UX-сценарий.
- `frontend/next.config.ts` явно задаёт `turbopack.root` как cwd запуска frontend, чтобы Next.js не выбирал внешний workspace root из lockfile выше репозитория.
- Build не должен зависеть от сетевого fetch Google Fonts; глобальные font variables задаются локальными fallback families.
- Playwright E2E ограничивается каталогом `test/playwright`, чтобы browser smoke не запускал contract/unit tests из `src`.

## Обоснование

Рефакторинг нужен для поддержки P0-сценария `карта → размещённые объекты → калькулятор` без изменения функциональности. Основной источник правды остаётся прежним: `DefenseProject.layers`, `DefenseProject.placedObjects`, `DefenseProject.assetLibrary`.

## Последствия

- Route `/` теперь должен импортировать landing из модуля, а не содержать полный экран в `app/page.tsx`.
- Route `/prototype` не должен содержать debug side effects в render.
- Проверочный baseline для frontend: `pnpm lint`, `pnpm exec tsc --noEmit --incremental false`, `pnpm build`.
- E2E smoke запускается через `pnpm test:e2e` и проверяет route `/prototype`.
- Долг по fixed-echelon costing остаётся отдельной задачей, потому что его исправление может изменить расчёты.
