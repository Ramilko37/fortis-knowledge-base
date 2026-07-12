# ADR-0012: явное серверное сохранение и recovery проекта

**Дата:** 2026-07-12  
**Статус:** Accepted  
**Теги:** Frontend, Backend, GIS, Sync, Investor Demo

## Контекст

Для investor-flow проект Fortis должен восстанавливаться из backend без неявного auto-save и без подмены серверных финансовых данных локальными вычислениями. Backend уже поддерживает полную запись `projectJson` и optimistic locking через `version`, а также cost, budget, report и compare API.

## Решение

- `DefenseProject` остаётся немедленной локальной моделью редактирования.
- Сохранение на сервер выполняется явно: frontend отправляет полный `projectJson` и текущую `version` в update API.
- UI sync state ограничен пятью состояниями: `clean`, `dirty`, `saving`, `conflict`, `error` (`Сохранено`, `Есть изменения`, `Сохранение`, `Конфликт версии`, `Ошибка`).
- Успешный save получает новую `version`, записывает canonical snapshot и очищает recovery draft.
- `localStorage` не хранит body сохранённого backend-проекта. Для backend-проекта разрешён только recovery envelope локальных несохранённых изменений и metadata активного project ID; после reload body заново загружается с backend.
- При сетевой ошибке recovery draft сохраняется и действие retry повторяет server save.
- При `409 Conflict` текущий draft не перезаписывает сервер. Пользователь выбирает `Загрузить серверную версию` или `Сохранить как новый вариант`.
- Для сохранённого backend-проекта `/calculator` показывает только server cost, budget и report. Ошибка API не заменяется локальными финансовыми цифрами.
- A/B сравнение принимает два разных сохранённых project ID и показывает snapshots, верхнеуровневые дельты и per-echelon differences.
- `/report?id=<projectId>` является HTML-представлением server report; `Печать / сохранить PDF` вызывает browser print, отдельный PDF service не используется.

## Последствия

- Browser storage помогает восстановить несохранённую работу, но не является источником сохранённой конфигурации.
- Server API contracts становятся обязательными для calculator, comparison и report UI.
- Staging, deployment и demo seed не входят в это ADR и требуют отдельного решения.
