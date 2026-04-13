---
name: korfix-find-records
description: Use when user asks to find records by partial match, name, phone, email, or complex search criteria across Korfix catalogs. Covers common search patterns and fuzzy matching tricks.
---

# korfix-find-records

Поиск записей по частичному совпадению и сложным условиям.

## По имени / тексту

Korfix API обычно поддерживает `LIKE`-поиск через фильтр по текстовому полю. Точный синтаксис зависит от реализации:

```
db_read('ag_clients', { filter: { name: 'Петров' } })        // точное совпадение
db_read('ag_clients', { filter: { name__like: '%Петров%' } }) // подстрока (если поддерживается)
```

Если API не поддерживает LIKE напрямую — прочитать без фильтра с ограничением, отфильтровать на клиенте:
```
all = db_read('ag_clients', { limit: 999 })
matches = [c for c in all['data'] if 'петров' in c['name'].lower()]
```

## По телефону / email

Поиск по нормализованному полю:
```
db_read('ag_clients', { filter: { phone: '+79...' } })
db_read('ag_clients', { filter: { email: 'ivanov@example.com' } })
```

Для телефонов — учитывай что в базе могут быть разные форматы (`+7 (495) 123-45-67` vs `+74951234567`). Нормализуй перед поиском — убери пробелы, скобки, дефисы.

## По статусу и связям

```
db_read('tt_tasks', {
  filter: {
    status: 'in_progress',
    project_id: 456,
    assignee_id: 12345
  },
  load_values: 1
})
```

Несколько фильтров работают как AND.

## За период

```
# Задачи созданные на этой неделе
from datetime import date, timedelta
week_ago = (date.today() - timedelta(days=7)).isoformat()

db_read('tt_tasks', {
  filter: { created_at__gte: week_ago },  # синтаксис зависит
  order_by: 'created_at',
  order: 'DESC'
})
```

## По пользователю (мои / команды)

```
my_id = current_user_id()  # из getCurrentUserId() или MCP catalog_schema
db_read('tt_tasks', { filter: { assignee_id: my_id } })
db_read('tt_tasks', { filter: { author_id: my_id } })
```

## Fuzzy / похожие записи

Нет прямой поддержки fuzzy-поиска. Подходы:
1. Выбрать кандидатов по первым буквам (`name__startswith: 'Пет'`), отфильтровать Levenshtein'ом на клиенте
2. Разбить запрос на токены, искать каждый по отдельности
3. Если в Korfix есть отдельный search-endpoint — использовать его (уточнить у пользователя)

## Пустой результат

Если ничего не нашлось:
- Не молчи. Скажи «ничего не найдено по запросу X»
- Предложи варианты: «возможно, имел в виду Y?», «расширить поиск до всех каталогов?»
- Если искали в `tt_tasks`, а ожидали и в `tt_subtasks` — предложи второй каталог

## Многокаталожный поиск

«Найди клиента Петров» может значить:
- `ag_clients` (юрлица)
- `ag_persons` (физлица / контакты)
- `auth_pers` (сотрудники)

Проверь несколько каталогов параллельно, верни объединённый результат с указанием источника.

## Документация

- `${CLAUDE_PLUGIN_ROOT}/docs/catalogs.md` — где какие данные лежат (чтобы знать какие каталоги проверять)
