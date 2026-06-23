# Fortis Studio UI Styleguide 2026-06-23

## Контекст

Defense Studio обновлён по концепту `/Users/rr/Downloads/Fortis Studio.html` для экранов `/prototype` и `/calculator`.

Цель: тихий B2B GIS/finance инструмент без hero/marketing композиции и без военной визуальной стилизации. Главная иерархия: карта, эшелоны, выбранный объект, стоимость, покрытие, статус, ограничения.

## Решение

- Studio использует верхний тёмный app bar `#0f172a` с брендом `FORTIS Studio`, вкладками `Карта защиты`, `Калькулятор`, `Сценарии BETA`, низкоприоритетным `Анализ`, disabled undo/redo, save/export.
- `/prototype` теперь строится как desktop three-pane workspace: слева `Эшелоны / Библиотека`, в центре реальная MapLibre/Deck.gl карта, справа `Инспектор объекта`.
- Нижняя полоса эшелонов больше не является каноном; канон — левое дерево эшелонов с nested placed objects.
- Выбранный `PlacedDefenseObject` редактируется через инспектор: координаты, азимут, сектор, дальность, количество, статус, заметки, стоимость, фокус карты, видимость, удаление.
- МОГ сохраняет текущий composition editor как детальный flow, открываемый из инспектора.
- `/calculator` стал sibling view Studio: compact summary, tabs, sticky estimate/budget surfaces, сохранён print/PDF report.

## Токены

- UI: Manrope.
- Числа/коды: IBM Plex Mono.
- Primary blue: `#2563eb`.
- App bar: `#0f172a`.
- Surfaces: `#ffffff`, `#f8fafc`, `#f1f5f9`.
- Semantic: green `#10b981`, amber `#f59e0b`, red `#ef4444`.
- Radius: `8px` для панелей/карточек/контролов, `6px` для внутренних fields.

## Карта

Карта остаётся production stack: MapLibre + Deck.gl. Концептный SVG/mock не используется вместо карты.

Новые UI toggles:

- `showCoverage`
- `showPlacementLabels`
- `showConstraintWarnings`

Оверлеи: compact toolbar сверху, warning stack справа, dark status footer снизу, basemap/zoom controls справа.

## Документация

Frontend styleguide: `frontend/docs/product/fortis-studio-styleguide.md`.

Старый `frontend/prototype-ui-audit.md` заменён migration-complete заметкой и больше не является источником будущих UI-решений.
