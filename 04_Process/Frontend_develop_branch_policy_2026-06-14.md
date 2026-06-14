# Frontend develop branch policy

Дата: 2026-06-14

## Правило

Все frontend-задачи Fortis должны попадать в `frontend/develop`.

Недостаточно запушить код только в feature/codex-ветку и обновить parent submodule pointer. После реализации frontend-задачи порядок такой:

1. Довести изменения в рабочей ветке frontend.
2. Проверить релевантные тесты, lint и TypeScript baseline.
3. Влить изменения в `frontend/develop`.
4. Запушить `frontend/develop`.
5. Обновить parent submodule pointer на новый commit `frontend/develop`.
6. Запушить parent `develop`.

## Причина

Parent-репозиторий хранит только указатель submodule. Если frontend-код остаётся только в feature-ветке, команда легко теряет задачу при работе от `frontend/develop`, даже если parent временно указывает на нужный commit.

## Проверка перед закрытием frontend-задачи

- `git -C frontend branch --show-current` должен быть `develop` перед финальным push frontend-кода, либо изменения должны быть явно merged в `develop`.
- `git -C frontend log --oneline origin/develop..HEAD` не должен показывать незапушенные frontend-коммиты.
- `git -C frontend branch -r --no-merged origin/develop` нужно просмотреть, если есть риск потерянных frontend-задач.

## Связанные заметки

- [[Продуктовый_план_Fortis]]
- [[04_Process/Linear_FE_closing_workflow_2026-06-14]]
