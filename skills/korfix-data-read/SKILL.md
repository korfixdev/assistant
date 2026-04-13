---
name: korfix-data-read
description: Use when the assistant needs to read records from Korfix catalogs via MCP — single records, lists with filters, paginated data, or FK-joined values. Covers db_read tool patterns.
---

# korfix-data-read

Чтение данных из каталогов Korfix через MCP.

## Основной инструмент — `db_read`

```
db_read(catalog, {
  filter: { status: 'open', assignee_id: 12345 },
  order_by: 'created_at',
  order: 'DESC',
  limit: 20,
  offset: 0,
  load_values: 1,        // читаемые FK вместо ID
  select: ['name', 'status', 'assignee_id']  // только нужные поля
})
```

Возвращает: `{ data: [...], total: N }` или подобную структуру.

## Типовые сценарии

### Одна запись по ID/alias

```
db_read('tt_tasks', { filter: { alias: 'abc123' }, limit: 1 })
```

### Список с фильтром

```
db_read('tt_tasks', {
  filter: { status: 'in_progress' },
  load_values: 1,
  limit: 50
})
```

### За период

```
db_read('ag_cashflows', {
  filter: { date_accrual: '>=2026-03-01' },  // синтаксис зависит от API — уточни
  order_by: 'date_accrual',
  order: 'DESC',
  limit: 100
})
```

Для сложных периодов может потребоваться два отдельных фильтра (>= и <=), или использование диапазонов.

### Агрегаты (count, sum)

Если MCP даёт только raw-записи — агрегируй на клиенте:
```python
records = db_read('ag_sales', { filter: { month: '2026-03' }, limit: 999 })
total = sum(r['summa'] for r in records['data'])
by_manager = groupby(records['data'], 'manager_id')
```

Либо через `/api/db/` endpoint с параметрами агрегации (если поддерживается — уточняй в документации).

## Пагинация

Если `total > limit`:
```
page = 0
while True:
    batch = db_read(catalog, { limit: 100, offset: page * 100 })
    if not batch['data']: break
    process(batch['data'])
    page += 1
```

Разумный лимит — 100-500 записей за запрос. Для массовых пересчётов использовать большие страницы.

## Схема каталога

Перед первым чтением незнакомого каталога:
```
catalog_schema(catalog) → { field: { type, arr (options), catalog (FK target), ... } }
```

Это даёт:
- Какие поля есть
- Какие обязательные
- Какие это FK и на какой каталог
- Какие значения допустимы в select-полях (например, список статусов)

## Read vs load_values

| Без load_values | С load_values=1 |
|---|---|
| `assignee_id: 12345` | `assignee_id: "Алексей Григорьев"` |

Для отображения пользователю — всегда `load_values=1`. Для дальнейших запросов по ID — без.

## Документация

- `${CLAUDE_PLUGIN_ROOT}/docs/catalogs.md` — список каталогов с описаниями
