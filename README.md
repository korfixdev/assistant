# korfix-assistant

**AI assistant plugin for Claude Code — query [Korfix](https://korfix.ru) ERP data via MCP.**

For business users: build summaries, search records, create tasks and entries through natural-language requests. MCP-based, no coding required.

**Platform:** [korfix.ru](https://korfix.ru) · **Docs:** [docs.korfix.info](https://docs.korfix.info) · **Related plugin:** [korfixdev/devkit](https://github.com/korfixdev/devkit)

Unlike [korfix-devkit](https://github.com/korfixdev/devkit) (miniapp development), this plugin is for **platform usage** — managers, analysts, automations.

## Install

The plugin is distributed via the [Korfix Marketplace](https://github.com/korfixdev/marketplace).

**New Claude Code (interactive `/plugin` UI):**

1. Type `/plugin` in Claude Code
2. **Add marketplace** → paste `korfixdev/marketplace`
3. Find `korfix-assistant` in list → **Install**

**Older Claude Code (commands):**

```
/plugin marketplace add korfixdev/marketplace
/plugin install korfix-assistant@korfixdev
```

**Manual install** (fallback):

```bash
git clone https://github.com/korfixdev/assistant ~/.claude/plugins/korfix-assistant
```

### After install — activate in current session

The plugin won't be active in your current Claude session immediately. Run:

```
/reload-plugins
```

Or restart Claude Code — plugins load on next start.

> For other AI clients (Codex, Cursor, Claude Desktop) — just connect the MCP server directly: `https://mcp.korfix.ru/${KORFIX_TOKEN}/sse` via your client's MCP config. The plugin format here is Claude Code-specific, but the MCP behind it works everywhere.

## Setup

The plugin works **only through MCP** — requires token and MCP URL.

```bash
export KORFIX_TOKEN="your-token-from-db-api"
export KORFIX_MCP_URL="https://mcp.korfix.ru/${KORFIX_TOKEN}/sse"
```

Get a token in your Korfix panel → `/db/api` → Add, with read/write access to the catalogs you need.

## What's inside

| Component | Role |
|-----------|------|
| Agent `korfix-ops` | Business queries, summaries, record search, create/update |
| Skill `korfix-data-read` | Reading catalog records with filters and pagination |
| Skill `korfix-find-records` | Fuzzy/partial search patterns |
| Skill `korfix-data-modify` | Creating and updating records with proper ownership |
| Skill `korfix-catalog-reference` | Mapping business terms to catalog aliases |
| Docs | `docs/catalogs.md` — full Korfix ERP catalog list |

## Usage examples

```
Show all tasks with status "In progress" from the last week
```

```
How many sales in March, grouped by manager?
```

```
Create a task "Call ACME client" assigned to Alex, due tomorrow
```

```
Find a client with a name similar to "Petrov" — show contacts and recent deals
```

The agent does not write miniapp code. For that — [korfix-devkit](https://github.com/korfixdev/devkit).

## When to use which plugin

| Task | Plugin |
|---|---|
| Write a new miniapp for marketplace | `devkit` |
| Deploy miniapp update | `devkit` |
| Get summary "how many tasks on the team" | `assistant` |
| Build financial report from data | `assistant` |
| Create a client/task via text command | `assistant` |
| Automation via n8n or external script | MCP directly, no plugin needed |

Both plugins can be installed simultaneously — no conflicts.

## Security

- The token = your assistant's access level. Grant **minimum required** — only needed catalogs.
- For automations/scripts — separate tokens (not personal), with restricted scope.
- The assistant can create/modify records when authorized by the token. Be mindful of write permissions.

## License

MIT — see [LICENSE](LICENSE).

## Contact

info@korfix.ru

---

# korfix-assistant — на русском

**AI-ассистент для работы с данными Korfix ERP через Claude Code.**

Отвечает на бизнес-вопросы, строит сводки, ищет записи, создаёт задачи — через MCP без написания кода.

В отличие от [korfix-devkit](https://github.com/korfixdev/devkit), который для **разработки миниапов**, этот плагин для **использования платформы** — руководителей, менеджеров, автоматизаций.

## Установка

Плагин распространяется через [маркетплейс Korfix](https://github.com/korfixdev/marketplace).

**Новый Claude Code (интерактивный `/plugin` UI):**

1. Набери `/plugin` в Claude Code
2. **Add marketplace** → вставь `korfixdev/marketplace`
3. Найди `korfix-assistant` в списке → **Install**

**Старый Claude Code (командой):**

```
/plugin marketplace add korfixdev/marketplace
/plugin install korfix-assistant@korfixdev
```

**Ручная установка** (fallback):

```bash
git clone https://github.com/korfixdev/assistant ~/.claude/plugins/korfix-assistant
```

### После установки — активация в текущей сессии

Плагин не станет активен в текущей сессии сразу. Выполни:

```
/reload-plugins
```

Либо перезапусти Claude Code — плагины подтянутся при следующем старте.

> Для других AI-клиентов (Codex, Cursor, Claude Desktop) — подключай MCP-сервер напрямую: `https://mcp.korfix.ru/${KORFIX_TOKEN}/sse` через MCP-конфиг своего клиента. Plugin-формат тут специфичен для Claude Code, а MCP за ним работает везде.

## Настройка

Плагин работает **только через MCP** — требует токен и MCP-URL.

```bash
export KORFIX_TOKEN="your-token-from-db-api"
export KORFIX_MCP_URL="https://mcp.korfix.ru/${KORFIX_TOKEN}/sse"
```

Получить токен — в панели Korfix → `/db/api` → Добавить, с доступом на чтение/запись нужных каталогов.

## Что внутри

| Компонент | Роль |
|-----------|------|
| Агент `korfix-ops` | Бизнес-запросы, сводки, поиск записей, создание/обновление |
| Skill `korfix-data-read` | Чтение записей из каталогов с фильтрами и пагинацией |
| Skill `korfix-find-records` | Fuzzy/частичный поиск |
| Skill `korfix-data-modify` | Создание и обновление записей с правильными полями владения |
| Skill `korfix-catalog-reference` | Маппинг бизнес-терминов на alias каталогов |
| Документация | `docs/catalogs.md` — полный список каталогов Korfix ERP |

## Примеры

```
Покажи все задачи со статусом "В работе" за последнюю неделю
```

```
Сколько сделок в марте, сгруппируй по менеджерам
```

```
Создай задачу "Позвонить клиенту ACME" на Алексея, срок — завтра
```

```
Найди клиента с именем похожим на "Петров" — покажи контакты и последние сделки
```

Агент не пишет код миниапов. Для этого — [korfix-devkit](https://github.com/korfixdev/devkit).

## Когда какой плагин ставить

| Задача | Плагин |
|---|---|
| Написать новый миниап для маркетплейса | `devkit` |
| Задеплоить обновление миниапа | `devkit` |
| Получить сводку «сколько задач у команды» | `assistant` |
| Построить финансовый отчёт из данных | `assistant` |
| Создать клиента/задачу через команду | `assistant` |
| Автоматизация через n8n / внешний скрипт | только MCP, плагин не нужен |

Можно поставить **оба плагина одновременно** — они не конфликтуют.

## Безопасность

- Токен = уровень доступа ассистента. Выдавай **минимум прав** — только каталоги, которые можно читать/править.
- Для автоматизаций и скриптов — отдельный токен (не личный), с ограниченным scope.
- Ассистент может создавать/менять записи — если токен это разрешает.

## Лицензия

MIT — см. [LICENSE](LICENSE).

## Контакт

info@korfix.ru
