---
name: korfix-data-modify
description: Use when the assistant creates, updates, or soft-deletes records in Korfix catalogs via MCP. Enforces alias generation, from_auth/from_group ownership fields, and confirmation for bulk operations.
---

# korfix-data-modify

Создание, обновление и мягкое удаление записей через MCP.

## Создание записи

```
db_insert(catalog, {
  alias: generate_alias(),         // обязательно, уникально
  name: 'Название',
  status: 'new',
  from_auth: user_id,              // обязательно
  from_group: user_id,             // обязательно (совпадает с from_auth для персональных или tenant-id)
  // ...остальные поля
})
```

### Критичные поля

| Поле | Зачем | Как получить |
|---|---|---|
| `alias` | Уникальный ключ записи | `Date.now().toString(36) + Math.random().toString(36).substr(2, 8)` |
| `from_auth` | Владелец записи | `catalog_schema(catalog).data.from_auth.arr` → ID текущего пользователя |
| `from_group` | Тенант / группа | Обычно равен `from_auth`. Для общих записей — ID группы |

Без `from_auth`/`from_group` запись создаётся на суперадмина и не видна обычным пользователям.

### Обязательные поля

Проверь через `catalog_schema(catalog)` перед вставкой — какие поля обязательны. Часто это `name`, иногда `alias`, иногда специфичные для каталога (`contact_id` для сделок, `project_id` для задач в модуле TT).

## Обновление записи

```
db_update(catalog, id_or_alias, {
  status: 'in_progress',
  // только поля которые меняются
})
```

Частичное обновление — передавай только изменяемые поля. Если передать пустое поле — оно обнулится (зависит от API). Будь внимателен.

## Массовые операции

**Обязательно подтверди у пользователя:**
- Создание более 3 записей — «Создать 5 задач, правильно?»
- Обновление более 5 записей — «Обновить статус у 12 записей?»
- Любое удаление — всегда подтверждение

Генерируй `alias` уникально для каждой записи — не используй один и тот же.

## Мягкое удаление

Korfix использует soft-delete через `hidden=1`:

```
db_update(catalog, id_or_alias, { hidden: 1 })
```

**Никогда не удаляй через `db_delete` напрямую** (если такой tool вообще есть) — используй soft-delete. Восстановление полноценного удаления сложно для конечного пользователя.

## Обработка ошибок

Частые ошибки API и как реагировать:

| Ошибка | Значение | Действие |
|---|---|---|
| `field X required` | Не передано обязательное поле | Проверь схему, добавь поле, повтори |
| `alias must be unique` | Коллизия alias | Сгенерируй новый alias, повтори |
| `403 Forbidden` | Токен не имеет доступа к методу/каталогу | Объясни пользователю: «токен не может записывать в X» |
| `from_group mismatch` | Запись пытается создаться в чужой группе | Проверь что `from_group` = твоя группа (или 0 для глобальных) |
| `value X not allowed for select Y` | Неправильное значение enum-поля | Через `catalog_schema().Y.arr` возьми допустимый |

## Примеры

### Создать задачу

```
alias = generate_alias()
db_insert('tt_tasks', {
  alias,
  name: 'Позвонить клиенту ACME',
  status: 'new',
  priority: 'medium',
  assignee_id: 12345,
  project_id: 678,
  due_date: '2026-04-15',
  from_auth: my_user_id,
  from_group: my_user_id
})
// → запись создана, вернуть пользователю: /db/tt_tasks/{alias}
```

### Создать клиента

```
alias = generate_alias()
db_insert('ag_clients', {
  alias,
  name: 'ООО Ромашка',
  phone: '+74951234567',
  email: 'info@romashka.ru',
  manager_id: my_user_id,
  from_auth: my_user_id,
  from_group: my_user_id
})
```

### Обновить статус задачи

```
db_update('tt_tasks', task_alias, { status: 'done' })
```

## Документация

- `${CLAUDE_PLUGIN_ROOT}/docs/catalogs.md` — какие поля у каких каталогов
