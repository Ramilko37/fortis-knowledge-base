# Linear FE closing workflow

Дата: 2026-06-14

## Контекст

В Linear не используется отдельный статус `Blocked`: для команды Fortis доступны `Backlog`, `Todo`, `In Progress`, `In Review`, `Done`, `Duplicate`, `Canceled`.

Из-за этого часть frontend-задач выглядела заблокированной не через статус, а через устаревшее положение в `Backlog` после закрытия backend parent-задач.

## Правило

Когда backend-задача, от которой зависит frontend-интеграция, переходит в `Done`:

- дочерняя или связанная FE-задача переводится из `Backlog` в `Todo`;
- в FE-задачу добавляется комментарий `Workflow update` с next action, границами работы и проверками закрытия;
- если остаётся реальный backend-блокер, он оформляется как Linear relation `blocked by`, а не как неявная договорённость в описании;
- если FE-задача только тематически связана с другой FE-задачей, используется `related`, а не блокировка;
- задача переводится в `In Review` только после комментария с verification notes;
- задача закрывается в `Done` только после фиксации результата и пройденных или явно пропущенных проверок.

## Текущая уборка Linear

- FRT-47 переведена в `Todo`; реальный blocker: FRT-56.
- FRT-50 переведена в `Todo`; backend parent FRT-35 уже `Done`.
- FRT-51 переведена в `Todo`; backend parent FRT-36 уже `Done`.
- FRT-56 назначена на Sergey U. как P0 backend blocker для полного update `DefenseProject` через `projectJson + version`.
- FRT-19 связана как blocked by FRT-37.
- FRT-28 связана как blocked by FRT-38.
- FRT-20 связана с FRT-51 как с канонической FE-интеграцией backend cost/budget API.

## Практический порядок закрытия

1. Закрыть backend blocker, если он оформлен через `blocked by`.
2. В связанной FE-задаче проверить, не устарели ли API-контракты и acceptance criteria.
3. Перевести FE-задачу в `In Progress` только с понятным владельцем.
4. После реализации добавить closing comment: что сделано, какие проверки выполнены, какие риски остались.
5. Перевести в `In Review`.
6. После review и подтверждения проверок перевести в `Done`.

## Связанные заметки

- [[Продуктовый_план_Fortis]]
- [[03_Architecture/ADR-0008-asset-library-api-frontend-boundary]]
- [[03_Architecture/Linear_Telegram_notifications_2026-06-13]]
