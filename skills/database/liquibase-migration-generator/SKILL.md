---
name: Liquibase Migration Generator
description: "Generates Liquibase .sql migration files for PostgreSQL. Use when user asks to create schema, create table, add column, or update column. Triggers: создай миграцию, создай схему, создай таблицу, добавь колонку, обнови колонку, migration."
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Bash
---

# Liquibase Migration Generator

Генератор Liquibase `.sql` файлов миграций для проекта task-trecker (PostgreSQL 16).

Навык только создаёт файлы. Запрещено запускать Liquibase, выполнять команды базы данных или применять миграции.

## Поддерживаемые действия

| action          | SQL-операция                                                       |
|-----------------|-------------------------------------------------------------------|
| `create schema` | `CREATE SCHEMA`                                                   |
| `create table`  | `CREATE TABLE`                                                    |
| `add column`    | `ALTER TABLE ... ADD COLUMN`                                      |
| `update column` | `ALTER TABLE ... ALTER COLUMN / RENAME / SET DEFAULT / ADD CONSTRAINT` |

## Алгоритм работы

1. Определить `action` из запроса пользователя.
2. Извлечь из запроса: схему (`schema`), таблицу (`table`), поля и их типы.
3. Проверить полноту данных:
   - Если данных недостаточно — задать **один** уточняющий вопрос и ждать ответа.
   - Если данных достаточно — перейти к шагу 4.
4. Glob: проверить существующие файлы в `migrations/sql/` на конфликт имён. Grep: проверить существующие `.sql` файлы на дублирование `changeset`-идентификатора.
5. Сформировать имя файла по шаблону.
6. Сгенерировать содержимое `.sql` файла.
7. Write файл в `migrations/sql/`.
8. Актуализировать `docs/database/schema.sql` согласно правилам раздела **Актуализация docs/database/schema.sql**.
9. Сообщить пользователю: имя файла миграции и что схема обновлена.

## Правила уточнения

**Запрещено** придумывать поля и их типы самостоятельно.

Когда задавать уточняющий вопрос:
- `create table` — поля не указаны: уточнить названия полей и их типы.
- `add column` — тип колонки не указан: уточнить тип.
- `update column` — не указано, что именно менять: уточнить характер изменения.

Всегда задавать **один** вопрос за раз.

## Автодобавляемые поля (только create table)

При `create table` добавлять автоматически, если явно не указаны в запросе:

- `id BIGSERIAL PRIMARY KEY` — **первым** в списке колонок.
- `created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()` — **в конце**.
- `updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()` — **в конце**.

## Шаблон имени файла

```
Version{timestamp}_{action}_{schema}_{table}.sql
```

- `{action}` — одно из: `create`, `add`, `update`.
- `{timestamp}` — текущая дата и время в формате `YYYYMMDDHHmmss`. Брать **только из системного контекста `currentDate`**, не выводить из имён существующих файлов миграций.
- Для `create schema` (без таблицы): `create-{schema}-v{timestamp}.sql`.

Примеры:
- `Version20260408143000_create_billing.sql`
- `Version20260408143000_create_billing_invoices.sql`
- `Version20260408143000_add_core_tasks.sql`
- `Version20260408143000_update_core_orders.sql`

## Путь сохранения

```
/home/llm-station/Project/Process/req-control/migrations/sql/
```

## Формат генерируемого .sql файла

```sql
--liquibase formatted sql

--changeset auto:{action}-{schema}-{table} labels:{action} comment:{описание на русском}
--preconditions onFail:MARK_RAN
--precondition-sql-check expectedResult:0 {precondition_query}

{SQL DDL}
```

### Precondition queries

| action          | precondition_query |
|-----------------|--------------------|
| `create schema` | `SELECT COUNT(*) FROM information_schema.schemata WHERE schema_name='{schema}'` |
| `create table`  | `SELECT COUNT(*) FROM information_schema.tables WHERE table_schema='{schema}' AND table_name='{table}'` |
| `add column`    | `SELECT COUNT(*) FROM information_schema.columns WHERE table_schema='{schema}' AND table_name='{table}' AND column_name='{column}'` — повторить для каждой добавляемой колонки |
| `update column` | precondition **не добавляется** |

### Выравнивание SQL

Имена колонок и типы выравниваются по столбцам, как в `migrations/sql/001_init_schema.sql`:

```sql
CREATE TABLE billing.invoices (
    id          BIGSERIAL       PRIMARY KEY,
    amount      NUMERIC(12,2)   NOT NULL,
    created_at  TIMESTAMPTZ     NOT NULL DEFAULT NOW(),
    updated_at  TIMESTAMPTZ     NOT NULL DEFAULT NOW()
);
```

Правила выравнивания:
- Имя колонки — дополнить пробелами до ширины самого длинного имени в таблице + минимум 2 пробела.
- Тип данных — дополнить пробелами до ширины самого длинного типа + минимум 2 пробела.
- Ограничения (`NOT NULL`, `DEFAULT`, `REFERENCES`, `CHECK`) следуют за типом.

## Примеры

### Пример 1: create schema

Запрос: "Создай миграцию для новой схемы billing"

Файл: `create-billing-v20260408143000.sql`

```sql
--liquibase formatted sql

--changeset auto:create-billing labels:create comment:Создание схемы billing
--preconditions onFail:MARK_RAN
--precondition-sql-check expectedResult:0 SELECT COUNT(*) FROM information_schema.schemata WHERE schema_name='billing'

CREATE SCHEMA billing;
```

### Пример 2: create table

Запрос: "Создай таблицу invoices в схеме billing, добавь поля amount NUMERIC(12,2) NOT NULL"

Файл: `create-billing-invoices-v20260408143000.sql`

```sql
--liquibase formatted sql

--changeset auto:create-billing-invoices labels:create comment:Создание таблицы invoices в схеме billing
--preconditions onFail:MARK_RAN
--precondition-sql-check expectedResult:0 SELECT COUNT(*) FROM information_schema.tables WHERE table_schema='billing' AND table_name='invoices'

CREATE TABLE billing.invoices (
    id          BIGSERIAL       PRIMARY KEY,
    amount      NUMERIC(12,2)   NOT NULL,
    created_at  TIMESTAMPTZ     NOT NULL DEFAULT NOW(),
    updated_at  TIMESTAMPTZ     NOT NULL DEFAULT NOW()
);
```

### Пример 3: add column

Запрос: "Добавь колонку archived BOOLEAN NOT NULL DEFAULT FALSE в таблицу tasks схемы core"

Файл: `add-core-tasks-v20260408143000.sql`

```sql
--liquibase formatted sql

--changeset auto:add-core-tasks labels:add comment:Добавление колонки archived в таблицу core.tasks
--preconditions onFail:MARK_RAN
--precondition-sql-check expectedResult:0 SELECT COUNT(*) FROM information_schema.columns WHERE table_schema='core' AND table_name='tasks' AND column_name='archived'

ALTER TABLE core.tasks
    ADD COLUMN archived BOOLEAN NOT NULL DEFAULT FALSE;
```

### Пример 4: update column

Запрос: "Обнови колонку status в таблице orders схемы core — изменить DEFAULT на 'PENDING'"

Файл: `update-core-orders-v20260408143000.sql`

```sql
--liquibase formatted sql

--changeset auto:update-core-orders labels:update comment:Изменение DEFAULT для колонки status в таблице core.orders

ALTER TABLE core.orders
    ALTER COLUMN status SET DEFAULT 'PENDING';
```

## Актуализация docs/database/schema.sql

После записи файла миграции **обязательно** обновить `docs/database/schema.sql`.

Файл расположен по пути:
```
/home/llm-station/Project/Process/req-control/docs/database/schema.sql
```

### Порядок действий

1. Bash: выполнить `git config user.name` — получить имя автора для записи в `## Changelog`.
2. Read `docs/database/schema.sql` — прочитать текущее содержимое.
3. Применить изменение согласно таблице ниже.
4. Edit — сохранить только изменённый фрагмент (не перезаписывать файл целиком).

### Правила изменений по action

| action          | Что менять в schema.sql |
|-----------------|-------------------------|
| `create schema` | Добавить строку `CREATE SCHEMA IF NOT EXISTS {schema};` в блок `-- Schema`. |
| `create table`  | Добавить полный блок `CREATE TABLE` с разделителем `-- ---…` и заголовочным комментарием. Добавить FK-ограничения и индексы внутри блока таблицы. Добавить запись в раздел `## Relationships`. |
| `add column`    | Вставить новую строку колонки в нужный `CREATE TABLE` блок. Выровнять по существующим колонкам. Если добавляется FK — добавить `CONSTRAINT` в тот же блок и запись в `## Relationships`. |
| `update column` | Найти и изменить нужную строку колонки: тип, DEFAULT, NOT NULL, имя. Если добавляется/удаляется FK — обновить блок `CONSTRAINT` и раздел `## Relationships`. |
| `create schema` + `create table` вместе | Применить оба правила последовательно. |

### Формат блока таблицы в schema.sql

```sql
-- -----------------------------------------------------------------------------
-- TABLE {schema}.{table}
-- {описание на русском}
-- -----------------------------------------------------------------------------

CREATE TABLE {schema}.{table} (
    id          BIGSERIAL       PRIMARY KEY,
    col1        TYPE            NOT NULL,    -- комментарий если неочевидно
    ...
    created_at  TIMESTAMPTZ     NOT NULL DEFAULT NOW(),
    updated_at  TIMESTAMPTZ     NOT NULL DEFAULT NOW(),

    CONSTRAINT fk_{table}_{col}
        FOREIGN KEY ({col})
            REFERENCES {ref_schema}.{ref_table}({ref_col})
            ON DELETE RESTRICT
            ON UPDATE CASCADE
);

CREATE INDEX idx_{schema}_{table}_{col} ON {schema}.{table} ({col});
```

### Формат записи в ## Relationships

```
-- {schema}.{ref_table}.{ref_col}  ←  {schema}.{table}.{col}   (one-to-many)
--   {описание связи}.
--   ON DELETE RESTRICT / CASCADE — {причина выбора}.
--   ON UPDATE CASCADE.
```

### Формат записи в ## Changelog

```
-- {YYYY-MM-DD} | {git user.name} | {краткое описание изменений на русском}
```

Имя файла миграции в запись **не включается**. Только дата, автор и суть изменений.

Пример:
```
-- 2026-04-19 | Alexey Gaybovich | добавлены колонки plan, model в core.tasks
```

### Запрещено

- Удалять или переписывать не изменённые блоки таблиц.
- Менять раздел `## Relationships` для таблиц, которых не касается текущая миграция.
- Добавлять `ON DELETE CASCADE` без явного указания в запросе — по умолчанию использовать `RESTRICT`.
- Использовать `auto` или любой placeholder вместо реального имени из `git config user.name`.

## Best Practices

Имена схем и таблиц писать в snake_case: `task_items`, `billing_accounts`, `user_profiles`.
Аббревиатуры не разворачивать: `id`, `url`, `uuid` — без изменений.

При объявлении колонок всегда явно указывать `NOT NULL` или `DEFAULT`, даже если тип это не требует.
Пример: `status TEXT NOT NULL DEFAULT 'ACTIVE'` — лучше, чем просто `status TEXT`.
Это защищает от неожиданного `NULL` при INSERT без явного значения и делает схему самодокументированной.

## Troubleshooting

### Пользователь не указал тип поля

Ситуация: запрос содержит только имена полей без типов, например "добавь поля name, email, phone".

Действие: задать уточняющий вопрос для каждой колонки за один раз:

> "Укажите типы данных для колонок:
> - `name` — какой тип? (например, `TEXT`, `VARCHAR(255)`)
> - `email` — какой тип?
> - `phone` — какой тип?"

Не генерировать файл до получения ответа. Не угадывать типы самостоятельно.

### Пользователь указал поля без ограничений (только имя и тип)

Ситуация: пользователь написал `amount NUMERIC(12,2)` без `NOT NULL` / `DEFAULT`.

Действие: использовать тип как есть — не добавлять ограничения автоматически, кроме автодобавляемых полей `id`, `created_at`, `updated_at` при `create table`.
Если ситуация неоднозначна — задать уточняющий вопрос: "Поле `amount` должно быть `NOT NULL`?"

### Неверная дата в имени файла

Ситуация: в `migrations/sql/` уже есть файлы с другим датовым префиксом (например, `Version20260415...`), и возникает соблазн продолжить эту же серию.

Действие: **запрещено** копировать или продолжать дату из существующих имён файлов.
Всегда использовать `currentDate` из системного контекста. Например, если сегодня `2026-04-19`, то `{timestamp}` начинается с `20260419`.

### Конфликт имени файла (файл уже существует)

Ситуация: Glob находит файл с идентичным именем в `migrations/sql/`.

Это возможно при повторном запросе с той же секундой в timestamp.

Действие: сгенерировать timestamp заново, добавив 1 секунду к текущему значению.
Не перезаписывать существующий файл. Не использовать суффиксы вида `_2`, `_v2` — они нарушают шаблон именования.

### Для `update column` не указано, что именно менять

Ситуация: запрос "обнови колонку status в таблице orders" без деталей изменения.

Действие: задать вопрос:

> "Что нужно изменить в колонке `status`? Например:
> - переименовать → укажите новое имя
> - изменить тип → укажите новый тип
> - изменить DEFAULT → укажите новое значение
> - добавить ограничение CHECK → укажите условие"