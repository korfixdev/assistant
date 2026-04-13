# Changelog

Все значимые изменения плагина будут здесь.

Формат — [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), версионирование — [SemVer](https://semver.org/).

## [0.2.0] — 2026-04-14

### Changed

- **Canonical plugin layout** — `plugin.json` перенесён в `.claude-plugin/`, skills в подпапках с `SKILL.md` (требование Claude Code plugin spec).
- **Install instructions multi-variant** — UI-флоу `/plugin`, CLI-команды для старых версий, ручная установка как fallback.
- **Cross-client note** — для Codex, Cursor, Claude Desktop указано как подключить MCP напрямую без плагина.
- **`repository` field в plugin.json** — теперь строка (было объект), соответствует schema.
- **Bilingual README** (EN + RU).

### Fixed

- Marketplace source format в listing'е (`url` type с full https URL) — избегает SSH clone fallback.

## [0.1.0] — 2026-04-13

### Added

- Initial release.
- 1 агент: `korfix-ops` — бизнес-запросы (сводки, поиск, создание записей через MCP).
- 4 skills: `korfix-data-read`, `korfix-find-records`, `korfix-data-modify`, `korfix-catalog-reference`.
- Bundled docs: `docs/catalogs.md` — полный список каталогов Korfix ERP.
- MCP connection config (обязательно — плагин работает только через MCP).

---

## Рекомендации по обновлению

Плагины **не обновляются автоматически**. Чтобы получить свежую версию:

```
/plugin marketplace update korfixdev
/plugin update korfix-assistant
/reload-plugins
```
