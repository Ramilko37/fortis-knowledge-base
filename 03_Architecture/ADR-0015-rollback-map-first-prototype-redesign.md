# ADR-0015: Откат `/prototype` к прошлому дизайну

- **Date:** 2026-07-26
- **Status:** accepted

## Context

После [[03_Architecture/ADR-0013-gis-workspace-fortis-ui-kit-migration|миграции `/prototype` на Fortis UI Kit]] и последующего [[03_Architecture/ADR-0014-map-first-gis-workspace|map-first редизайна]] рабочий интерфейс ушёл от прежней визуальной и операционной модели. Последний откат только ADR-0014 оказался недостаточным: нужен возврат к прошлому дизайну `/prototype`, существовавшему до Fortis Studio / UI Kit runtime redesign.

При этом [[03_Architecture/ADR-0012-fortis-design-system-foundations|foundations дизайн-системы]], Storybook и `frontend/src/shared/ui/fortis/` остаются в кодовой базе как общий UI Kit для дальнейшего развития. Откат касается runtime-композиции страницы `/prototype`, а не удаления дизайн-системы.

## Decision

- Откатить runtime UI `/prototype` к состоянию `frontend` commit `63cb6fb` (`revert: restore frontend before Fortis Studio redesign`) для `src/modules/drone-defense/ui/` и связанных prototype workflow/map files.
- Не удалять `frontend/src/shared/ui/fortis/`, tokens, Storybook и foundations дизайн-системы.
- Считать [[03_Architecture/ADR-0013-gis-workspace-fortis-ui-kit-migration|ADR-0013]] и [[03_Architecture/ADR-0014-map-first-gis-workspace|ADR-0014]] superseded для runtime `/prototype`.
- Сохранить текущие shared/domain изменения вне prototype UI, если они не требуются для совместимости прошлого дизайна.

## Consequences

- `/prototype` снова использует прошлую визуальную модель с Ant Design/Ant Icons в runtime-слое страницы.
- Fortis UI Kit остаётся доступным для Storybook, будущих экранов и последующей более осторожной интеграции.
- Документы ADR-0013 и ADR-0014 сохраняются в истории как отклонённые для текущего `/prototype` runtime.
- Новая попытка миграции `/prototype` на UI Kit должна начинаться отдельным решением и отдельной UX-проверкой.

## Verification

- TypeScript или production build frontend;
- smoke `/prototype`;
- при необходимости визуальная проверка desktop GIS workflow на `1280×720` и `1440×960`.
