# Prompt Store — контекст проекта

## Что это такое

Composer-пакет `united-software-platform/prompt-store` — коллекция промптов и правил для AI-ассистентов в разработке ПО.

**Важно:** пакет содержит только текстовые файлы (Markdown). PHP-кода, тестов и инструментов статического анализа в нём нет и быть не должно.

## Структура

```
rules/
  php/            — стиль PHP-кода, PSR-стандарты
  architecture/   — Clean Architecture, CQRS, DDD
  frameworks/     — Symfony, Laravel и др.
  database/       — SQL-паттерны, схемы, индексы
  documentation/  — стандарты документации, шаблоны
  devops/         — CI/CD, Docker, деплой
  qa/             — стандарты тестирования

skills/
  analysis/       — бизнес-анализ, требования, RAG
  architecture/   — ревью архитектуры, проектирование систем
  database/       — схемы БД, миграции, SQL-оптимизация
  development/    — генерация моделей, нормализация данных
  data/           — работа с Excel, табличными данными

commands/
  review/         — команды ревью кода
  generate/       — генерация кода и артефактов
  analyze/        — анализ архитектуры и БД
  git/            — работа с git (changelog, release notes)
  init/           — инициализация проектов и модулей

agents/
  backend/        — PHP/Symfony разработчик
  database/       — DBA-агент
  devops/         — инфраструктурный агент
  qa/             — QA-инженер
  management/     — project-manager, scrum-master
  analysis/       — системный аналитик, solution-architect
```

## Правила валидации структуры

Полные правила: `rules/documentation/structure-validation.md`

### Перед созданием любого файла проверить:

1. **Тип** — к какому типу относится (`rules`, `skills`, `commands`, `agents`)?
2. **Категория** — категория существует в структуре выше?
3. **Формат имени** — имя директории/файла в `kebab-case`?
4. **Расширение** — файл имеет расширение `.md`?
5. **Вложенность** — не превышает 3 уровней?

### Обязательные точки входа

| Тип      | Обязательный файл | Обязательный фронтматтер          |
|----------|--------------------|----------------------------------|
| skill    | `SKILL.md`         | `name`, `description`, `tools`   |
| agent    | `AGENT.md`         | `name`, `description`, `tools`, `model` |
| command  | `<name>.md`        | заголовок `# /<name>`            |
| rule     | `<name>.md`        | заголовок `# Название`, `## Назначение` |

### Добавление новой категории

Нельзя создавать категории без одновременного обновления:
- `rules/documentation/structure-validation.md` — таблица категорий
- `CLAUDE.md` (этот файл) — таблица структуры

## Активные правила

При работе с этим репозиторием всегда применять:

- `.claude/rules/documentation-rules.md` — стандарты оформления всех `.md` файлов
- `.claude/rules/structure-validation.md` — правила валидации и поддержания структуры репозитория
- `.claude/rules/templates.md` — шаблоны для создания новых документов

## Абсолютные запреты

- PHP-классы, тесты, `vendor/`, `composer.lock`
- `autoload`, `require-dev` в `composer.json`
- Файлы не в формате `.md` (кроме `SKILL.yaml`, `.gitkeep`)
- Вложенность более 3 уровней
- Имена не в `kebab-case`
