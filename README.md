# Prompt Store

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

Коллекция промптов и правил для AI-ассистентов в PHP/Symfony-разработке, распространяемая как Composer-пакет.

---

## Навигация

- [Структура](#структура)
- [Установка](#установка)
- [Автоматическая синхронизация](#автоматическая-синхронизация)
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

## Установка

**1. Подключить пакет:**

```bash
composer require united-software-platform/prompt-store
```

**2. Синхронизировать промпты в `.claude/`:**

```bash
vendor/bin/sync-prompts
```

Скрипт предложит выбрать типы (`skills`, `commands`, `rules`, `agents`) и категории внутри каждого из них. Для установки всего без вопросов:

```bash
vendor/bin/sync-prompts --all
```

---

## Автоматическая синхронизация

Чтобы промпты обновлялись автоматически при каждом `composer install` и `composer update`, добавьте в `composer.json` проекта:

```json
"scripts": {
    "post-install-cmd": ["vendor/bin/sync-prompts --all"],
    "post-update-cmd":  ["vendor/bin/sync-prompts --all"]
}
```

---

## Лицензия

MIT
