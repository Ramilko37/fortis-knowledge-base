# ADR-0014: Map-first рабочее пространство GIS

- **Date:** 2026-07-24
- **Status:** superseded by [[03_Architecture/ADR-0015-rollback-map-first-prototype-redesign|ADR-0015]]

## Context

После миграции `/prototype` на [[03_Architecture/ADR-0013-gis-workspace-fortis-ui-kit-migration|Fortis UI Kit]] рабочее пространство одновременно показывало дерево, библиотеку, плавающую карточку объектов, постоянный нижний обзор эшелонов и инспектор. Несколько поверхностей дублировали выбор и закрывали полезную площадь карты. Это противоречило роли карты как основного рабочего полотна профессиональной GIS.

## Decision

- Карта становится приоритетной областью desktop workspace; левая панель имеет два взаимоисключающих режима: «Структура» и «Библиотека».
- Состояние выбора задаётся единым контрактом `WorkspaceState`: активный эшелон остаётся в `DefenseProject.activeLayerId`, а `selectedEntity` описывает выбранный эшелон или размещённый объект.
- Инспектор существует только при выборе сущности или операции. На `1024–1439 px` он накладывается поверх карты, а с `1440 px` становится отдельной колонкой.
- Постоянный нижний обзор эшелонов и плавающая карточка «Объекты эшелона» удаляются из runtime и исходников. Все пространственные сущности остаются на карте, а их иерархия — в «Структуре».
- Создание и редактирование карточки средства выполняются в Fortis Drawer шириной 544 px: в нём сохраняются inline validation, focus trap, возврат фокуса к opener, fixed footer и подтверждение отмены dirty-формы.
- deck.gl `TextLayer` получает `characterSet: "auto"` верхнего уровня, чтобы русскоязычные подписи карты попадали в atlas шрифта.

## Consequences

- В `/prototype` нет нескольких конкурирующих источников selection state и постоянных непустых панелей без пользовательского контекста.
- Взаимодействия с эшелонами и объектами проходят через дерево, карту и контекстный инспектор; `DefenseProject` и backend boundary не меняются.
- Browser smoke обязан проверять отсутствие legacy drawer, one-inspector behavior, ширину карты, отсутствие горизонтального overflow и отсутствие deck.gl warnings о кириллических glyphs.

## Verification

- unit/source-contract tests `gis-workspace-state`, `gis-workspace-layout`, `gis-workspace-panels`, `gis-p1p2-unification`, `asset-library-layout`;
- Playwright scenarios для Structure/Library, inspector, toolbar и отсутствия lower drawer;
- TypeScript и production `pnpm build`.
