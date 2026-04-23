---
name: model-dsl
description: "Генерация доменных моделей в DSL-формате. Использовать когда пользователь просит создать модель, описать сущность, добавить поля к модели. Триггерные слова: создай модель, опиши модель, model DSL, новая сущность, добавь поле."
allowed-tools: []
---

# model-dsl

Навык интерактивно собирает описание модели и выводит готовый DSL-блок.

## Синтаксис DSL

```
model <snake_case_name>
  field <snake_case_name> <type> [modifiers]
end
```

## Разрешённые типы

| Тип | Описание |
|-----|----------|
| `uuid` | UUID-идентификатор |
| `string` | Строка |
| `int` | Целое число |
| `float` | Число с плавающей точкой |
| `bool` | Булево значение |
| `timestamp` | Дата и время |
| `date` | Дата |
| `json` | JSON-объект |

## Разрешённые модификаторы

| Модификатор | Описание |
|-------------|----------|
| `pk` | Первичный ключ |
| `unique` | Уникальное значение |
| `nullable` | Допускает NULL |
| `default <val>` | Значение по умолчанию |
| `fk <model>.<field>` | Внешний ключ |

## Обязательные правила

1. **Первое поле всегда** `field id uuid pk` — без исключений.
2. **Отступ** — ровно два пробела перед каждым `field`.
3. **Имена** — только `snake_case`, символы `[a-z0-9_]`.
4. **NOT NULL по умолчанию** — `nullable` пишется явно только при необходимости.
5. **Запрещено** комбинировать `pk` и `nullable` в одном поле.
6. **Пустых строк** внутри блока нет.
7. **Комментарии** запрещены.

## Протокол выполнения

### Шаг 1. Уточнить имя модели

Если пользователь не указал имя, спросить:

> Как назвать модель? (snake_case, например: `user_profile`)

### Шаг 2. Уточнить поля

Для каждого поля спросить (по одному или списком):

> Перечислите поля в формате `<name> <type> [modifiers]`, по одному на строку.  
> Поле `id uuid pk` добавляется автоматически.  
> Пример: `email string unique`, `deleted_at timestamp nullable`, `project_id uuid fk project.id`

### Шаг 3. Валидация

Перед выводом проверить каждое поле:

- Имя соответствует `[a-z][a-z0-9_]*`.
- Тип входит в список разрешённых.
- Модификаторы совместимы (`pk` + `nullable` → ошибка).
- `fk` ссылается в формате `<model>.<field>`.
- `default` имеет значение.

При ошибке — сообщить и попросить исправить конкретное поле.

### Шаг 4. Вывести DSL-блок

Вывести готовый блок в код-блоке без лишних пояснений:

```
model user_profile
  field id uuid pk
  field email string unique
  field full_name string
  field bio string nullable
  field project_id uuid fk project.id
  field created_at timestamp default now()
end
```

## Шаблон

```
model ___
  field id uuid pk
  field ___ ___
end
```

## Примеры

### Простая модель

```
model tag
  field id uuid pk
  field name string unique
  field color string default '#000000'
end
```

### Модель со связями и nullable

```
model comment
  field id uuid pk
  field body string
  field author_id uuid fk user.id
  field parent_id uuid nullable fk comment.id
  field created_at timestamp default now()
  field deleted_at timestamp nullable
end
```

### Модель с JSON

```
model audit_log
  field id uuid pk
  field entity_type string
  field entity_id uuid
  field payload json
  field occurred_at timestamp
end
```