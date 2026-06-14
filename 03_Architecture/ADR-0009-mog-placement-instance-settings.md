# ADR-0009: настройки МОГ живут в размещённом инстансе

- **Дата:** 2026-06-14
- **Статус:** accepted
- **Теги:** GIS, Frontend, Domain, МОГ

## Контекст

МОГ создаётся пользователем только после размещения на карте. Библиотека средств должна оставаться источником шаблона, но не полноценным конфигуратором конкретного поста.

ТЗ требует отражать в карточке МОГ:

- количество личного состава;
- тип поста: МОГ, ПВН, ГОР, КПП, другой пост;
- подотчётность: Росгвардия, МО, ЧОП;
- пометку эшелона;
- обмундирование и оснащение с количеством;
- несколько типов оружия с количеством;
- дальность оружия, которая отражается на карте.

## Решение

Библиотечная карточка МОГ остаётся шаблоном `DefenseAsset.compoundProfile`.

При размещении на карте создаётся `PlacedDefenseObject.compoundProfile`, который хранит верхний уровень настроек конкретного поста:

- `postType`;
- `personnelCount`;
- `accountability`;
- `equipment[]`;
- `weapons[]`;
- `visibleCoverageWeaponIds[]`;
- `coverageWeaponId` как legacy/fallback для старых проектов;
- top-level `sectorWidthDeg` и `azimuth` как legacy/fallback;
- `weapons[].coverageSectorWidthDeg`;
- `weapons[].coverageAzimuth`.

Редактор МОГ работает с размещённым объектом, а не с библиотечной карточкой. Пометка эшелона выводится из `PlacedDefenseObject.layerId`.

Покрытие МОГ на карте строится от одного или нескольких типов оружия из `visibleCoverageWeaponIds[]`. Один `PlacedDefenseObject` МОГ может одновременно показывать несколько секторных покрытий с разными цветами. У каждого покрытия теперь свои `weapons[].coverageAzimuth` и `weapons[].coverageSectorWidthDeg`.

Если старый проект ещё не содержит `visibleCoverageWeaponIds[]`, frontend читает `coverageWeaponId` как fallback и интерпретирует его как массив из одного элемента. Если старый проект не содержит per-weapon настроек углов, frontend инициализирует `weapons[].coverageAzimuth` и `weapons[].coverageSectorWidthDeg` из legacy top-level `azimuth` и `sectorWidthDeg`.

При сохранении новой конфигурации основным источником правды становятся `visibleCoverageWeaponIds[]` и per-weapon coverage settings, а `coverageWeaponId`, top-level `azimuth` и top-level `sectorWidthDeg` обновляются как совместимый legacy-слепок последнего выбранного покрытия.

Если у оружия количество становится `0`, frontend автоматически убирает его из `visibleCoverageWeaponIds[]`. Legacy-поля `armament`, `weaponUnits`, `sectorOrRange` остаются fallback-слоем для старых конфигураций и производных UI-summary.

## Последствия

- МОГ можно настраивать после размещения без отдельного backend CRUD.
- Backend-доработка не требуется, пока `DefenseProject.projectJson` сохраняет `placedObjects[].compoundProfile` без потерь.
- Серверный расчёт стоимости или серверная генерация отчёта позднее должны понимать `equipment[]`, `weapons[]`, `visibleCoverageWeaponIds[]`, per-weapon coverage settings и legacy `coverageWeaponId` / top-level `azimuth` / top-level `sectorWidthDeg`.
- ADR-0007 остаётся верным для библиотечного шаблона, но конкретная настройка МОГ теперь явно относится к размещённому инстансу.
