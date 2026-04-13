# Contributing to korfix-assistant

## Release workflow — версии и CHANGELOG

**Каждый push, который меняет поведение агента/skills — сопровождается bump версии и записью в CHANGELOG.**

### 1. Bump version в `.claude-plugin/plugin.json`

Формат — [SemVer](https://semver.org/). Правило выбора уровня:

| Уровень | Когда применять | Примеры |
|---|---|---|
| **PATCH** `0.2.0 → 0.2.1` | Багфикс, опечатка, уточнение, небольшая правка примера | Опечатка в skill, уточнение фильтра в примере |
| **MINOR** `0.2.0 → 0.3.0` | Новая фича/skill, новый паттерн, расширение skill'а | Добавили skill для работы со складом, новый helper поиска |
| **MAJOR** `0.9.0 → 1.0.0` | Breaking change, удаление/переименование | Удалили skill, изменили формат env-var |

Если сомнения — **бери старший**.

**Не bump'ать:** правки только в CHANGELOG.md / README.md / CONTRIBUTING.md; форматирование; комментарии.

### 2. Добавь запись в CHANGELOG.md

Новая версия **сверху**. Формат [Keep a Changelog](https://keepachangelog.com/en/1.1.0/):

```markdown
## [0.3.0] — YYYY-MM-DD

### Added
- Что-то новое — зачем

### Changed
- Что изменилось — почему
```

Разделы: `Added`, `Changed`, `Deprecated`, `Removed`, `Fixed`, `Security`.

Пиши **с точки зрения пользователя**: «новый skill для работы со складом», не «коммит xyz».

### 3. Commit message

- `release: vX.Y.Z` — bump версии
- `docs:`, `skill(<name>):`, `agent(<name>):`, `fix:`

### 4. Push

```bash
git add -A && git commit -m "release: v0.3.0" && git push
```

Пользователи увидят «Update available» в `/plugin` UI.

## Breaking changes (MAJOR)

1. GitHub Release с тегом `vX.0.0`
2. Миграционная инструкция в release notes
3. `BREAKING CHANGE:` в commit message

## Для AI-агентов

Агент, меняющий конфиг плагина (skill/agent/структуру), **обязан**:

1. Определить уровень bump
2. Обновить `version` в `.claude-plugin/plugin.json`
3. Добавить запись в CHANGELOG.md сверху с сегодняшней датой
4. Использовать префикс в commit message (`release:`, `skill(X):`, ...)
5. Один атомарный коммит (версия + CHANGELOG + сами изменения)

**Нарушение:** push без bump'а = пользователи пропустят уведомление и важные правила.

## Контакт

info@korfix.ru
