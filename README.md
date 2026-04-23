# Prompt Store

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

Коллекция промптов и правил для AI-ассистентов. Работает в любом проекте независимо от технологии.

---

## Навигация

- [Структура](#структура)
- [Установка через Git Submodule](#установка-через-git-submodule)
- [Лицензия](#лицензия)

---

## Структура

```
rules/      — правила и соглашения (стиль кода, архитектура, документация)
skills/     — навыки для AI-ассистентов
commands/   — команды для Claude Code
agents/     — конфигурации AI-агентов
```

---

## Установка через Git Submodule

Подходит для любого проекта — PHP, Python, Node.js, Go и других.

**1. Подключить репозиторий как подмодуль:**

```bash
git submodule add https://github.com/united-software-platform/prompt-store .prompts
git submodule update --init --recursive
```

**2. Синхронизировать промпты в `.claude/`:**

```bash
.prompts/bin/sync-prompts
```

Интерактивный выбор типов и категорий:

```bash
.prompts/bin/sync-prompts --select
```

**3. Обновление промптов:**

```bash
git submodule update --remote
.prompts/bin/sync-prompts
```

---

### Автоматическая синхронизация через Makefile

Добавить в `Makefile` проекта:

```makefile
prompts-install:
	git submodule update --init --recursive
	.prompts/bin/sync-prompts

prompts-update:
	git submodule update --remote
	.prompts/bin/sync-prompts
```

Использование:

```bash
make prompts-install
make prompts-update
```

---

### Первая установка в команде

Новый разработчик клонирует проект и инициализирует подмодуль:

```bash
git clone --recurse-submodules https://github.com/your-org/your-project
.prompts/bin/sync-prompts
```

Или, если проект уже склонирован без подмодулей:

```bash
git submodule update --init --recursive
.prompts/bin/sync-prompts
```

---

## Лицензия

MIT
