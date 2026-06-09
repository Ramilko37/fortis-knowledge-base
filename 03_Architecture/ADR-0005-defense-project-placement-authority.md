# ADR-0005: единственный источник размещений — DefenseProject

Дата: 2026-06-09

## Статус

Принято.

## Контекст

На прототипе есть несколько представлений размещений:
- проектные `PlacedDefenseObject` внутри `DefenseProject`;
- legacy `Placement[]` для карты и `FacilityDrilldown` (3D/runtime);
- локальные runtime placements.

Нужно, чтобы расчёт, карта и отчёты работали только от фактической пользовательской конфигурации.

## Решение

Принимать `DefenseProject` как единственный источник правды для пользовательских размещений:
- `DefenseProject.layers`
- `DefenseProject.placedObjects`
- `DefenseProject.assetLibrary`

Все пользовательские операции размещения на карте, выборе и удалении на карте/списке должны идти только через `useDefenseProjectStore`:
- создание/перенос/удаление: `placeObject`, `transferObjectToLayer`, `deletePlacedObject`;
- выбор: `selectObject`;
- отображение карты/списка: `placedObjectsToMapPlacements(project)` как адаптер.

`useDefenseStudioStore` и legacy `Placement[]` остаются для сценарных/3D/runtime путей (`FacilityDrilldown`) и не являются расчетным источником пользователя.

## Последствия

- Карта, список эшелона и PDF/калькулятор опираются на `DefenseProject` и единые изменения в `placedObjects`.
- Нужда в синхронизации legacy `Placement` как источника расчёта исчезает.
- При появлении необходимости объединять 3D/runtime с пользовательским проектом потребуется отдельный адаптер миграции из `placedObjects` к runtime placements.
