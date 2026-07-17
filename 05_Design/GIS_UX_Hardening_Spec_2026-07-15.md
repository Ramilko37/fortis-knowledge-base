# Спецификация GIS UX Hardening для `/prototype` — 2026-07-15

**Статус:** Implemented and verified 2026-07-17  
**Владелец:** Product / UX / Frontend  
**Целевой контур:** desktop `/prototype`, `1280×720` и `1440×960`  
**Источник:** UX/UI-аудит `/prototype` от 2026-07-14 и [[05_Design/Prototype_GIS_UI_Audit_2026-07-10|аудит от 2026-07-10]]

**Ограничение среды:** backend поднимается только локально. Интеграционные сценарии используют `http://localhost:8090`; недоступность, offline, timeout и `409 conflict` воспроизводятся локальным backend либо контролируемыми mock-ответами frontend-тестов. Внешний staging не является условием приёмки.

## 1. Решение

Не создавать второй параллельный backlog по GIS-интерфейсу. Существующие Linear-задачи актуализируются и закрывают релевантные findings; новые задачи создаются только для пробелов, которых нет в текущем backlog.

Новая нарезка должна устранить семь разрывов:

1. безопасное удаление размещённого объекта;
2. сохранение активного эшелона при изменении видимости;
3. спокойная картографическая графика и компактная легенда;
4. минимальный профессиональный набор GIS-инструментов;
5. точный поиск и compatible-first выдача библиотеки;
6. доступность desktop GIS workspace;
7. понятные ошибки сохранения и действия восстановления.

## 2. Пользовательский результат

После реализации пользователь может настроить карту без потери рабочего контекста и данных:

- случайный клик не удаляет объект необратимо;
- скрытие эшелона не переключает пользователя на другой эшелон;
- карта остаётся главным рабочим холстом и визуально не подавляется кольцами покрытия;
- базовые навигация, reset extent и измерение находятся в одном предсказуемом наборе инструментов;
- поиск `МОГ` показывает МОГ выше нерелевантных совпадений вроде `Дымогенерация`;
- состояние `saving / saved / offline / conflict / error` однозначно и сопровождается следующим действием;
- основные сценарии доступны с клавиатуры и не зависят только от цвета или иконки.

## 3. Принципы поведения

1. **Selection и visibility независимы.** Выбранный/активный слой может быть скрыт, но приложение не меняет selection без явного действия пользователя.
2. **Destructive action всегда обратим или подтверждён.** Для удаления объекта используются confirmation и короткое окно undo.
3. **Карта имеет приоритет.** Empty-state панели не открываются автоматически; редакторы не должны без необходимости перекрывать основную область карты.
4. **Состояние выражено текстом и семантикой.** Цвет и иконка усиливают, но не заменяют подпись, `aria-*` и доступный feedback.
5. **UI не сообщает об успехе раньше persistence.** `Сохранено` показывается только после подтверждённой записи в активный источник данных.
6. **Детали реализации не выходят в пользовательский текст.** Форматы источников, dev/license flags, HTTP/transport errors и внутренние идентификаторы не показываются как основной copy.

## 4. Функциональная спецификация

### 4.1. Безопасное удаление размещённого объекта — P0

#### Текущее нарушение

Действие `Удалить` сразу исключает объект из проекта. Нет confirmation, undo и предварительного объяснения последствий.

#### Требуемое поведение

1. При нажатии `Удалить` открывается confirmation dialog.
2. Заголовок содержит название объекта: `Удалить «{objectName}»?`.
3. Описание сообщает, что объект исчезнет с карты, из эшелона и расчёта текущей конфигурации.
4. Основное destructive-действие называется `Удалить объект`; безопасное — `Отмена`.
5. После успешного удаления появляется snackbar `Объект удалён` с действием `Отменить` на 10 секунд.
6. Undo восстанавливает объект с прежними `id`, `layerId`, координатами, настройками МОГ, coverage и visibility.
7. Ошибка удаления не закрывает контекст объекта и показывает локализованный recovery action `Повторить`.
8. Пока операция выполняется, повторное удаление заблокировано.

#### Acceptance criteria

- объект нельзя удалить одним кликом;
- `Escape` и `Отмена` закрывают confirmation без изменений;
- destructive-кнопка получает initial focus только по принятому в приложении dialog-паттерну; безопасный keyboard flow проверен;
- отмена после удаления полностью восстанавливает объект и расчёт;
- сообщение об успехе появляется только после изменения source of truth;
- удаление и undo покрыты unit/store test и browser regression.

### 4.2. Скрытый активный эшелон — P0

#### Текущее нарушение

Скрытие активного L5 автоматически переключает selection на L1. Пользователь теряет рабочий контекст и может разместить следующий объект не в том эшелоне.

#### Требуемое поведение

1. Toggle видимости изменяет только visibility; `activeLayerId` остаётся прежним.
2. Скрытый активный эшелон остаётся выбранным в layer strip.
3. Карточка показывает совместное состояние: `Активный · Скрыт` и доступную иконку crossed-eye.
4. В collapsed-состоянии сохраняется тот же контекст.
5. Размещение нового объекта в скрытый эшелон заблокировано, пока пользователь явно не выберет `Показать эшелон`.
6. Пользователь может явно выбрать другой эшелон; только это действие меняет `activeLayerId`.
7. Калькулятор, отчёт и стоимость не зависят от visibility.

#### Acceptance criteria

- скрытие L5 не выбирает L1 и не меняет `activeLayerId`;
- UI явно сообщает одновременно active и hidden state;
- placement CTA объясняет блокировку и позволяет показать активный эшелон;
- состояние корректно восстанавливается после reload;
- regression проверяет visible active, hidden active и explicit layer switch.

### 4.3. Картографическая графика и легенда — P1

#### Требуемое поведение

1. Неактивные эшелоны используют спокойную заливку и тонкий контур; стартовый ориентир токенов — fill opacity `0.06–0.10` и border `1px`, подложка и объекты остаются читаемыми.
2. Активный эшелон отличается не только цветом: стартовый ориентир — fill opacity `0.12–0.18`, border `2px`/halo плюс состояние в layer manager.
3. Hidden-слои не рисуют fill, border, markers и coverage, но остаются в layer manager.
4. Компактная сворачиваемая легенда объясняет:
   - цвета/коды эшелонов;
   - маркеры типов защиты;
   - coverage;
   - constraints/warnings, когда режим доступен.
5. Легенда отражает только доступные на текущей карте категории и обновляется при фильтрации.
6. Подписи защищаемого объекта и слоёв поддерживают кириллицу; runtime не сообщает о missing glyphs для `Завод Альфа`.
7. Basemap selector показывает пользовательские названия и атрибуцию, но скрывает `vector-style-url`, `raster-xyz`, `demo/dev` и `license check`.

#### Acceptance criteria

- на `1280×720` подложка, объект защиты, маркеры и активный эшелон различимы одновременно;
- active/hidden state читается без зависимости только от цвета;
- легенда сворачивается, не перекрывает ключевые объекты и доступна с клавиатуры;
- Cyrillic labels рендерятся без missing-glyph warning;
- visual regression зафиксирован для светлой подложки минимум на двух viewport.

### 4.4. Минимальный набор GIS-инструментов — P1

#### Home / reset extent

1. Кнопка `Показать весь объект` возвращает камеру к extent защищаемого объекта и всех видимых эшелонов.
2. Если геометрии эшелонов нет, fallback — extent объекта защиты с безопасным padding.
3. Действие не изменяет selection, visibility и данные проекта.

#### Измерение расстояния

1. Кнопка `Измерить расстояние` включает отдельный mode и имеет `aria-pressed`.
2. Первый клик задаёт старт, следующие добавляют сегменты; double-click/Enter завершает линию.
3. Текущая длина обновляется после каждого сегмента; единицы автоматически переключаются `м / км`.
4. `Escape` отменяет незавершённое измерение; `Очистить` удаляет результат.
5. Выход из measure mode возвращает обычные pan/zoom/select без изменения проекта.

#### Общие требования

- zoom, compass, home/reset, measure и basemap образуют единый визуальный control cluster;
- интерактивная область каждого control не меньше `44×44px`;
- tooltip и доступное имя объясняют действие до клика;
- координатная форма показывает реальные initial values либо пустые поля с примерами, которые не выглядят введёнными данными.

#### Acceptance criteria

- reset extent помещает объект защиты и все видимые эшелоны в viewport с padding и не меняет проект;
- измерение поддерживает start, промежуточные точки, завершение, cancel и clear;
- результат измерения не попадает в `DefenseProject` и исчезает после clear/reload;
- после выхода из measure mode обычные pan/zoom/select работают без дополнительного клика;
- все map controls имеют доступные имена, state semantics и target не меньше `44×44px`;
- сценарии проверены unit-тестами геометрии/форматирования и browser regression.

### 4.5. Поиск и compatible-first библиотека — P1

#### Поиск

1. Поисковая строка имеет видимый label или программно связанное имя, а не только placeholder.
2. Запрос нормализуется по регистру, пробелам и punctuation.
3. Ранжирование: exact name/abbreviation → отдельный token → prefix token → description/secondary fields → substring.
4. Запрос `МОГ` ранжирует карточку МОГ выше `Дымогенерация`; совпадение внутри слова не считается равным token match.
5. Empty state показывает запрос и действия `Сбросить поиск` / `Сбросить фильтры`.

#### Compatible-first

1. По умолчанию сначала показываются средства, совместимые с активным эшелоном.
2. Несовместимые позиции не исчезают: они находятся ниже и имеют причину `Не рекомендуется для L5` или эквивалентную.
3. Пользователь может явно выбрать режим `Все`.
4. Фасеты категории, типа защиты и совместимости комбинируются с текстовым запросом.
5. Смена visibility активного эшелона не сбрасывает запрос и фильтры; явная смена active layer обновляет совместимость предсказуемо.

#### Acceptance criteria

- `МОГ` не выдаёт `Дымогенерация` на равной или более высокой позиции;
- exact/token/prefix/substring ranking покрыт unit tests;
- фильтры и query отражены в одном понятном result count/empty state;
- keyboard focus не теряется при обновлении результатов.

### 4.6. Desktop accessibility hardening — P2

#### Требуемое поведение

1. Основная навигация объявляет активный пункт через `aria-current`.
2. Visibility, measure, basemap/legend toggles используют `aria-pressed`, `aria-expanded` или нативную семантику по типу контрола.
3. Активный слой объявляется программно; изменение active/hidden state отправляется в ненавязчивый live region.
4. Все icon-only действия имеют доступное имя и tooltip.
5. Критические targets имеют минимум `44×44px`; визуальная и интерактивная области могут различаться.
6. Focus indicator имеет достаточный contrast и не обрезается floating panels.
7. Dialog/drawer управляют focus: вход, trap при modal-поведении, возврат на trigger.
8. При `200%` zoom на `1280×720` нет обязательного горизонтального скролла страницы; внутренний scroll допускается в каталогах и inspector.
9. Состояния error/success/hidden/active не выражаются только цветом.

#### Acceptance criteria

- пройден keyboard-only сценарий: выбрать L5 → найти МОГ → разместить по координатам → открыть настройки → отменить → скрыть/показать объект;
- automated a11y smoke не обнаруживает critical/serious violations в основном состоянии;
- размеры search, coordinate action и layer visibility не меньше `44×44px`;
- screen-reader smoke подтверждает понятные названия карты, панели, активного слоя и save state.

### 4.7. Ошибки сохранения и recovery — P1

#### Модель состояния

Интерфейс использует одно из явных состояний:

| Состояние | Пользовательский текст | Действия |
|---|---|---|
| `saving` | `Сохраняем…` | повторный save заблокирован |
| `saved` | `Сохранено {time}` | нет обязательного действия |
| `offline-draft` | `Офлайн · изменения сохранены на устройстве` | `Повторить синхронизацию` |
| `conflict` | `Версия проекта изменилась` | `Обновить данные`, `Сохранить копию` |
| `error` | `Не удалось сохранить изменения` | `Повторить`, `Подробнее` |

#### Требуемое поведение

1. Transport/backend text вроде `Failed to reach backend` не показывается как основной copy.
2. Диалог вариантов сохраняет введённое название и не закрывается при ошибке.
3. Ошибка показывается inline рядом с действием, а также в общем save status.
4. `Повторить` повторяет тот же intent и защищено от double-submit.
5. Offline/local draft не выдаётся за server-saved version.
6. Conflict не перезаписывает удалённую версию молча.
7. После успешного retry ошибка очищается, состояние переходит `saving → saved`.
8. Технические детали доступны по `Подробнее` для диагностики, но отделены от пользовательского сообщения.

#### Acceptance criteria

- обработаны success, timeout/network, HTTP error, offline и `409 conflict`;
- введённые данные и текущая карта не теряются при каждом error path;
- save status одинаково трактуется в app bar, диалоге вариантов и inspector;
- сценарии покрыты contract/unit tests и browser tests с mocked responses.

## 5. Актуализация существующих Linear-задач

| Задача | Решение | Findings, которые она закрывает |
|---|---|---|
| FRT-66 — layer manager | Сохранить In Progress; уточнить границы и связать с новыми задачами active-hidden и cartography | читаемость layer strip, состояния и связь карточки с картой |
| FRT-93 — перегруз первого экрана | Расширить AC: пустая objects panel закрыта по умолчанию, карта сохраняет рабочую площадь на `1280×720` | плотный стартовый workspace |
| FRT-101 — модель черновика/вариантов | Расширить моделью `saving/saved/offline/conflict/error`; реализацию transport recovery вынести в новую задачу | неоднозначный save status |
| FRT-49 — polish настроек МОГ | Добавить dock/resizable либо не перекрывающий карту inspector, сохранение map context и проверку `1280×720` | редактор конкурирует с картой |
| FRT-114 — tooltips эшелона | Расширить доступными именами и описанием последствий hide/edit/delete | icon-only действия и предупреждение до клика |
| FRT-115 — фильтры библиотеки | Оставить задачей фасетов; связать с новой задачей search relevance/compatible-first | масштабирование каталога |
| FRT-12 — visibility типов | Оставить основным функциональным тикетом видимости по типам | быстрый показ категорий защиты |
| FRT-118 — слои по типам | Пометить duplicate of FRT-12, не вести параллельную реализацию | дублирующий scope |

Завершённые FRT-11 и FRT-13 не переоткрываются: новые regressions и missing UX оформляются отдельными задачами и связываются с ними как follow-up.

## 6. Новые Linear-задачи только для пробелов

| Linear | Название | Priority | Связи |
|---|---|---:|---|
| FRT-133 | `[FE][P0] Защитить удаление размещённого объекта: confirm + undo` | Urgent | related FRT-10, FRT-13 |
| FRT-136 | `[FE][P0] Не менять активный эшелон при его скрытии` | Urgent | follow-up FRT-11; related FRT-66 |
| FRT-134 | `[FE][P1] Ослабить графику эшелонов и добавить GIS-легенду` | High | related FRT-66, FRT-118/FRT-12 |
| FRT-135 | `[FE][P1] Добавить home/reset extent и измерение расстояния` | High | related FRT-66 |
| FRT-137 | `[FE][P1] Исправить поиск библиотеки и compatible-first ranking` | High | related/parallel FRT-115 |
| FRT-139 | `[FE][P1] Обработать ошибки сохранения, offline и recovery` | High | related FRT-101, FRT-47 |
| FRT-138 | `[FE][P2] Провести desktop accessibility hardening GIS workspace` | Medium | depends on stable controls from FRT-136/FRT-135 |

Каждая новая задача получает ссылку на эту спецификацию, собственные acceptance criteria из раздела 4, явный out-of-scope и verification checklist.

## 7. Зависимости и порядок

1. UXG-01 и UXG-02 — первый safety sprint, выполняются независимо.
2. FRT-93, FRT-49 и FRT-66 можно продолжать параллельно при сохранении границ этой спецификации.
3. UXG-03 и UXG-04 используют существующую карту и не требуют изменения backend-контракта.
4. FRT-115 отвечает за facets; UXG-05 — за ranking, compatible-first и search states.
5. FRT-101 задаёт общую save terminology; UXG-06 реализует transport/offline/conflict states.
6. UXG-07 выполняется после стабилизации control set, но a11y semantics добавляются в каждую предшествующую задачу, а не откладываются целиком.

## 8. Out of scope

- отдельная mobile-компоновка и mobile bottom sheet;
- изменение 3D-прототипа;
- geocoder/address search;
- измерение площади и сложные measurement profiles;
- новая backend-модель слоёв или видимости;
- полноценная история версий и collaborative editing;
- изменение формул бюджета, coverage или эффективности;
- unrelated visual redesign всего Fortis.

## 9. Общий Definition of Done

- поведение соответствует разделу 4 и не дублирует существующий Linear scope;
- unit/contract tests покрывают state transitions и ranking;
- browser regression проходит на `1280×720` и `1440×960`;
- выполнен keyboard-only smoke основного сценария;
- нет новых ошибок TypeScript, lint и runtime console в изменённом контуре;
- update не меняет калькулятор/отчёт от визуальной visibility;
- изменения требований и итоговый статус задач отражены в knowledge base.

## 10. Источники и связанные документы

- `audits/prototype-ui-2026-07-14/audit.md` в parent workspace;
- [[05_Design/Prototype_GIS_UI_Audit_2026-07-10]];
- [[01_Requirements/Спецификация_02_Геоданные_покрытие_и_ограничения]];
- [[01_Requirements/Спецификация_03_Сквозной_GIS_бюджет_и_отчёт]];
- [[Продуктовый_план_Fortis]].

## 11. Реализация и evidence — 2026-07-17

Работа выполнена в frontend-ветке `codex/gis-ux-hardening`. Внешний staging не использовался; реальный persistence-path проверен на локальных Go API и PostgreSQL.

| Раздел | Статус | Реализованное поведение | Проверка |
|---|---|---|---|
| 4.1 Безопасное удаление | Verified | confirmation с именем и последствиями, cancel/Escape, exact undo 10 секунд, retry без потери контекста | store contract + Playwright confirm/cancel и keyboard regression |
| 4.2 Скрытый активный эшелон | Verified | selection не меняется, `Активный · Скрыт`, явное восстановление видимости, placement guard; hidden fill/markers/coverage не рисуются | store/source contracts + Playwright hidden-active |
| 4.3 Картография и легенда | Verified | спокойные opacity/border tokens, dynamic keyboard-accessible legend, кириллица, локализованный basemap с атрибуцией без transport/dev copy | config/source contracts + visual captures на `1280×720` и `1440×960` |
| 4.4 GIS-инструменты | Verified | home/reset extent, polyline measurement, Enter/double-click/Escape/clear, compass reset, targets `44×44` | measurement unit test + Playwright controls/keyboard |
| 4.5 Поиск | Verified | нормализация и token ranking, `МОГ` не совпадает с `Дымогенерация` как substring, compatible-first/all, три комбинируемых facet, result count/reset | catalog-search unit test + Playwright keyboard search |
| 4.6 Accessibility | Verified | `aria-current`, live regions, toggle semantics, accessible icon actions, modal/drawer focus, 200% zoom без page overflow | keyboard-only L5→МОГ flow, axe WCAG smoke без serious/critical, two desktop viewports |
| 4.7 Сохранение | Verified | `saving/saved/offline-draft/conflict/error`, локализованный recovery, exact retry intent, input очищается только после success, technical details отделены | state/store contracts + mocked HTTP 500→retry Playwright + real local backend smoke |

### Интеграционное решение по сохранению

Реальный локальный smoke обнаружил, что frontend подставлял локальный `baseObject.id` (`facility-alpha`) в поле `enterpriseId`, тогда как backend хранит его как PostgreSQL UUID. Исправлено на границе контракта:

- локальная карта отправляется без `enterpriseId`;
- UUID сохраняется только для объекта предприятия, полученного с backend;
- save/load не превращают локальный object id в enterprise context;
- opt-in тест `test/playwright/local-backend-smoke.spec.ts` выполняет register-or-login, сохраняет карту через frontend BFF в Go API/PostgreSQL, проверяет UI-статус и удаляет smoke-проект.

### Финальный verification gate

- все `src/**/*test.ts` и source-contract `.mjs` — PASS;
- scoped ESLint всех изменённых TS/TSX/MJS — PASS;
- `pnpm exec tsc --noEmit` — PASS;
- `pnpm build` — PASS, 27 маршрутов;
- `pnpm exec playwright test test/playwright/prototype.spec.ts --reporter=line` — 10/10 PASS;
- `RUN_LOCAL_BACKEND_SMOKE=1 pnpm exec playwright test test/playwright/local-backend-smoke.spec.ts --reporter=line` — 1/1 PASS;
- `git diff --check` — PASS.

Полный repository-wide `pnpm lint` остаётся красным из-за 45 ранее существовавших ошибок React lint в неизменённом `src/app/page.tsx`; изменённый GIS-контур lint-clean.
