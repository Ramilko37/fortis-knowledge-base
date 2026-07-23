# ADR-0013: GIS Workspace на Fortis UI Kit

- **Date:** 2026-07-23
- **Status:** accepted

## Context

После [[03_Architecture/ADR-0012-fortis-design-system-foundations]] компоненты Fortis UI Kit были реализованы и опубликованы в Storybook, но основной маршрут `/prototype` продолжал смешивать собственные controls, Ant Design и Ant Icons. Это создавало несколько конкурирующих визуальных контрактов, затрудняло поддержку состояний и давало разное responsive-поведение карты, дерева проекта, библиотеки и инспектора.

## Decision

- Fortis UI Kit из `frontend/src/shared/ui/fortis/` становится каноническим набором интерактивных компонентов для runtime-интерфейса `/prototype`.
- Семантический typed Lucide wrapper расширяется GIS-, asset-, navigation-, action- и status-иконками; прямые импорты `antd` и `@ant-design/icons` из `src/modules/drone-defense/ui/` удаляются.
- Общие элементы — кнопки, поля, select, checkbox, поиск, статусы, карточки, дерево эшелонов, modal, drawer, menu, toast и empty/error states — реализуются через Fortis primitives. Специализированные GIS-элементы (MapLibre/deck.gl canvas, маркеры, зоны покрытия, азимутальные controls) сохраняют собственную domain-реализацию, но используют Fortis tokens и семантические actions.
- Responsive-контракт рабочего пространства:
  - `< 768 px` — карта с компактной выдвижной библиотекой;
  - `768–1023 px` — дерево/библиотека и карта;
  - `1024–1279 px` — дерево/библиотека, карта и инспектор;
  - `>= 1280 px` — полный shell с rail, деревом, картой и инспектором.
- Поведение и модель данных `DefenseProject` не меняются. Миграция является UI-слоем поверх существующего GIS UX Hardening.
- Контракт защищается source-level тестами на отсутствие legacy UI-imports, использование Fortis primitives и responsive breakpoints; Storybook и production build входят в обязательную проверку миграции.

## Consequences

- `/prototype` и Storybook используют один визуальный язык и общие состояния компонентов.
- GIS-модуль больше не зависит от Ant Design на runtime-уровне, хотя зависимость может оставаться в других frontend-модулях до их отдельной миграции.
- Mobile-режим остаётся responsive safeguard, а не отдельным mobile product flow: он обеспечивает доступ к карте и библиотеке, но не расширяет mobile scope GIS MVP.
- Новые общие состояния сначала добавляются в Fortis UI Kit и Storybook, затем используются в GIS. Узкоспециализированная картографическая логика остаётся внутри `src/modules/drone-defense/`.

## Verification

- unit и source-contract tests для UI Kit, GIS composition и responsive-контракта;
- TypeScript и scoped ESLint;
- production Next.js build и Storybook static build;
- browser smoke и проверка отсутствия горизонтального overflow на 390, 768, 1024 и 1440 px.
