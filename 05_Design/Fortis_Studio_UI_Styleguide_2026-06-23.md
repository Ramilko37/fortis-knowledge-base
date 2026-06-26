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

- Brand/display: Syne.
- UI: Manrope.
- Числа/коды: IBM Plex Mono.
- Primary blue: `#2563eb`.
- Primary hover/accent: `#1d4ed8`, `#38bdf8`.
- App bar: `#0f172a`.
- App bar surface: `#1e293b`.
- Surfaces: `#ffffff`, `#f8fafc`, `#f1f5f9`.
- Selected surface/border: `#eff6ff`, `#bfdbfe`.
- Semantic: green `#10b981`, amber `#f59e0b`, red `#ef4444`.
- Radius: `8px` для панелей/карточек/контролов, `6px` для внутренних fields.
- Desktop layout constants from source: app bar `54px`, left panel `312px`, inspector `328px`.

Frontend UI kit source of truth: `frontend/docs/product/fortis-studio-styleguide.md`.

## UI kit from `Fortis Studio.html`

На 2026-06-25 bundled HTML размотан в полноценный UI kit-контракт во frontend-документации.

Канонические части:

- тёмный app bar с `FORTIS Studio`, route links `Карта защиты`, `Калькулятор`, `Сценарии BETA`, low-emphasis `Анализ`, disabled undo/redo, save/export;
- desktop three-pane workspace: left `Эшелоны / Библиотека`, center live GIS map shell, right object inspector;
- compact echelon tree с кодом слоя, цветом, диапазоном, счётчиком и nested placed objects;
- library groups: `Обнаружение`, `РЭБ / Подавление`, `Огневое поражение`, `Пассивная защита`;
- map toolbar: `Покрытие`, `Подписи`, `Ограничения`, `Линейка`, basemap `Карта`, zoom `+/-`;
- warning stack: budget/info, blind sector warning, conflict danger;
- inspector fields: координаты, азимут, сектор, дальность, количество, статус, заметки;
- calculator sibling view with estimate list and sticky summary/budget aside.

Runtime-ограничение: массивы `ECHELONS`, `SEED`, `LIBRARY` из HTML могут быть только demo seed/reference. В рабочем `/prototype` source of truth остаётся `DefenseProject.layers`, `DefenseProject.placedObjects`, `DefenseProject.assetLibrary`.

## Карта

Карта остаётся production stack: MapLibre + Deck.gl. Концептный SVG/mock не используется вместо карты.

Новые UI toggles:

- `showCoverage`
- `showPlacementLabels`
- `showConstraintWarnings`

Оверлеи: compact toolbar сверху, warning stack снизу слева над status footer, dark status footer снизу.

На 2026-06-24 compact toolbar `/prototype` объединяет режимы карты и технические controls в одну поверхность: `Покрытие`, `Подписи`, `Ограничения`, `Линейка`, basemap dropdown `Карта`, zoom `+/-`. Отдельный крупный basemap/zoom cluster не является каноном.

Первый экран `/prototype` должен открываться как живой редактор, а не empty state: при наличии demo/local проекта выбран первый размещённый объект, в дереве и inspector подсвечен `МОГ — пост №1`, на карте видны размещённые маркеры средств защиты и summary warnings. Demo-conflict для `МОГ — пост №2` сохраняется при последующем редактировании объектов, пока полноценный conflict engine не заменит placeholder flags.

## Документация

Frontend styleguide: `frontend/docs/product/fortis-studio-styleguide.md`.

Старый `frontend/prototype-ui-audit.md` заменён migration-complete заметкой и больше не является источником будущих UI-решений.
