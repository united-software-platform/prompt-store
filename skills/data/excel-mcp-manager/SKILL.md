---
name: Excel MCP Manager
description: >
  This skill manages the full lifecycle of Excel documents via MCP API (mcp__excel__* tools only).
  Use when working with .xlsx files: reading and writing data, managing sheets/rows/columns/ranges,
  applying formatting, creating tables, pivot tables, and charts. Handles document versioning
  ({name}-v{X}.{Y}.xlsx) and maintains a Changelog sheet on every modification.
  Requires excel MCP server to be active. Never use Bash, Python, shell, or file tools.
allowed-tools:
  - mcp__excel__get_workbook_metadata
  - mcp__excel__read_data_from_excel
  - mcp__excel__write_data_to_excel
  - mcp__excel__create_workbook
  - mcp__excel__create_worksheet
  - mcp__excel__delete_worksheet
  - mcp__excel__rename_worksheet
  - mcp__excel__copy_worksheet
  - mcp__excel__insert_rows
  - mcp__excel__insert_columns
  - mcp__excel__delete_sheet_rows
  - mcp__excel__delete_sheet_columns
  - mcp__excel__delete_range
  - mcp__excel__copy_range
  - mcp__excel__merge_cells
  - mcp__excel__unmerge_cells
  - mcp__excel__get_merged_cells
  - mcp__excel__format_range
  - mcp__excel__apply_formula
  - mcp__excel__validate_formula_syntax
  - mcp__excel__validate_excel_range
  - mcp__excel__create_table
  - mcp__excel__create_pivot_table
  - mcp__excel__create_chart
  - mcp__excel__get_data_validation_info
  - Bash
---

# Excel MCP Manager

Управление полным жизненным циклом Excel-документов исключительно через MCP API. Skill охватывает
чтение и запись данных, управление листами, строками, столбцами и диапазонами, применение
форматирования, создание таблиц, сводных таблиц и диаграмм. Каждое изменение фиксируется
в системе версионирования и сопровождается записью в лист Changelog.

---

## Принципы работы

### 1. mcp-only
Для всех операций с данными использовать **только** инструменты `mcp__excel__*`.

**Единственное исключение — Bash:** разрешён только для побайтового копирования файла при создании новой версии:
```bash
cp "source-v1.0.xlsx" "source-v1.1.xlsx"
```
Bash запрещён для любых других целей: чтения данных, записи, анализа, редактирования содержимого.

Инструменты Read, Write, Edit, Glob, Grep — запрещены полностью.

Если Excel MCP-сервер недоступен — сообщить об этом пользователю и остановиться.

### 2. minimal-data
Читать только те диапазоны и ячейки, которые необходимы для конкретной задачи.
- Не читать лист целиком, если нужна одна ячейка
- Использовать `validate_excel_range` для проверки границ перед чтением
- При неизвестной структуре — начинать с `get_workbook_metadata`

### 3. versioning-on-every-write
Любое изменение документа требует:
1. Определить тип изменения (revision или release)
2. Вычислить новое имя файла
3. Скопировать исходный файл побайтно (binary-safe copy) в новый файл с вычисленным именем, сохранив структуру (листы, стили, формулы, метаданные), без каких-либо модификаций содержимого
4. Выполнить все операции в новом файле
5. Записать в Changelog

Оригинальный файл остаётся нетронутым.

### 4. verify-before-write
Перед записью всегда проверять:
- Существование листа через `get_workbook_metadata`
- Корректность диапазона через `validate_excel_range`
- Синтаксис формул через `validate_formula_syntax`

---

## Схема версионирования

Формат имени файла: `{name}-v{release}.{revision}.xlsx`

Пример: `report-v1.3.xlsx`, `budget-v2.0.xlsx`

### Когда увеличивать revision (v1.3 → v1.4)

| Тип изменения | Примеры |
|---|---|
| Обновление данных | Изменение значений ячеек, исправление данных |
| Форматирование | Цвета, шрифты, границы, выравнивание |
| Добавление строк/столбцов | Вставка новых строк в существующий лист |
| Обновление формул | Изменение формул в существующих ячейках |
| Обновление данных таблицы | Добавление строк в существующую таблицу |

### Когда увеличивать release (v1.9 → v2.0)

| Тип изменения | Примеры |
|---|---|
| Новый лист | Добавление листа в книгу |
| Удаление листа | Удаление существующего листа |
| Переименование листа | Изменение имени листа |
| Новая таблица/диаграмма | Создание структурных объектов |
| Новая сводная таблица | Добавление pivot table |
| Одновременно структурные и данные | Любое сочетание структурных изменений с другими |

### Процедура создания новой версии

1. Определить тип изменения → выбрать revision или release
2. Разобрать текущее имя файла: извлечь `name`, `release`, `revision`
3. Вычислить новое имя:
   - revision: `revision + 1`, release без изменений
   - release: `release + 1`, revision сбрасывается в `0`
4. Создать новый файл через `mcp__excel__create_workbook` с новым именем
5. Скопировать необходимые листы через `mcp__excel__copy_worksheet`
6. Выполнить целевые операции
7. Записать в Changelog нового файла

Новый файл всегда начинается с версии `v1.0`.

---

## Протокол Changelog

В каждом документе должен существовать лист **"Changelog"**.

### Структура таблицы

| Колонка | Тип | Описание |
|---|---|---|
| Version | Текст | Версия файла, например `v1.4` |
| Date | Дата | Дата изменения в формате `YYYY-MM-DD` |
| Author | Текст | Имя инициатора изменений — уточнить у пользователя перед первой записью |
| Change Type | Текст | `DATA`, `STRUCTURE`, `FORMAT` или `FORMULA` |
| Sheet | Текст | Имя затронутого листа или `—` если несколько |
| Description | Текст | Краткое описание на русском языке |

### Правила ведения

- **Append-only**: никогда не редактировать существующие записи
- Новые записи добавляются в конец таблицы
- Перед первой записью в Changelog **обязательно уточнить у пользователя** имя автора (инициатора изменений); использовать это имя во всех записях текущей сессии
- Если лист "Changelog" отсутствует — создать его автоматически перед первой записью (см. [changelog-protocol.md](references/changelog-protocol.md))

### Значения Change Type

- `DATA` — изменение данных в ячейках
- `STRUCTURE` — добавление/удаление/переименование листов, таблиц, диаграмм
- `FORMAT` — форматирование ячеек, диапазонов
- `FORMULA` — добавление или изменение формул

---

## Стандартный workflow (6 шагов)

```
READ → PLAN → VERSION → WRITE → CHANGELOG → REPORT
```

### Шаг 1: READ — Прочитать состояние документа
```
mcp__excel__get_workbook_metadata(filepath)
```
Получить список листов, именованных диапазонов, текущую версию из имени файла.

### Шаг 2: PLAN — Спланировать операции
Определить:
- Тип изменения (revision / release)
- Какие листы затронуты
- Какие MCP-вызовы потребуются и в каком порядке

### Шаг 3: VERSION — Создать новую версию
Вычислить новое имя → скопировать файл побайтово через Bash → обновить символическую ссылку `{name}-last.xlsx` → все дальнейшие операции только в новом файле:
```bash
cp "/path/to/source-v1.0.xlsx" "/path/to/source-v1.1.xlsx"
ln -sf "source-v1.1.xlsx" "/path/to/source-last.xlsx"
```
Ссылка `{name}-last.xlsx` всегда указывает на актуальную версию файла; обновлять её при каждом создании новой версии.

`mcp__excel__create_workbook` и `mcp__excel__copy_worksheet` для создания версии **не использовать** — они не переносят стили, формулы и метаданные. Bash `cp` — единственный корректный способ клонирования файла.

### Шаг 4: WRITE — Выполнить операции
Применить все изменения в новом файле. Проверять каждый шаг.

### Шаг 5: CHANGELOG — Записать изменения
Найти или создать лист "Changelog" → добавить запись.

### Шаг 6: REPORT — Отчитаться
Сообщить пользователю:
- Новое имя файла
- Что именно было изменено
- Версию (было → стало)

---

## Routing по типу запроса

| Запрос пользователя | Тип | Действие |
|---|---|---|
| "Прочитай данные из..." | Read-only | Только READ, без версионирования |
| "Покажи структуру файла" | Read-only | Только `get_workbook_metadata` |
| "Обнови значение ячейки" | Revision | READ → VERSION (revision++) → WRITE → CHANGELOG |
| "Добавь строку" | Revision | READ → VERSION (revision++) → WRITE → CHANGELOG |
| "Измени форматирование" | Revision | READ → VERSION (revision++) → WRITE → CHANGELOG |
| "Добавь новый лист" | Release | READ → VERSION (release++) → WRITE → CHANGELOG |
| "Удали лист" | Release | READ → VERSION (release++) → WRITE → CHANGELOG |
| "Создай таблицу/диаграмму" | Release | READ → VERSION (release++) → WRITE → CHANGELOG |
| "Переименуй лист" | Release | READ → VERSION (release++) → WRITE → CHANGELOG |
| "Вставь/удали колонки" | Уточнить | Уточнить у пользователя (см. references/versioning-rules.md) |

---

## Примеры

### Пример 1: Точечное обновление ячейки

Задача: обновить значение ячейки B5 на листе "Данные" в файле `report-v1.3.xlsx`.

```
1. READ
   mcp__excel__get_workbook_metadata("report-v1.3.xlsx")
   → убедиться, что лист "Данные" существует

2. PLAN
   Тип: DATA → revision++
   Новое имя: report-v1.4.xlsx

3. VERSION
   Bash: cp "report-v1.3.xlsx" "report-v1.4.xlsx"
   Bash: ln -sf "report-v1.4.xlsx" "report-last.xlsx"

4. WRITE
   mcp__excel__validate_excel_range("report-v1.4.xlsx", "Данные", "B5")
   mcp__excel__write_data_to_excel("report-v1.4.xlsx", "Данные", "B5", [[новое_значение]])

5. CHANGELOG
   mcp__excel__write_data_to_excel("report-v1.4.xlsx", "Changelog", "A{next_row}",
     [["v1.4", "2026-04-04", "{автор}", "DATA", "Данные", "Обновлено значение ячейки B5"]])

6. REPORT
   "Создан файл report-v1.4.xlsx. Обновлена ячейка B5 на листе 'Данные'. Версия: v1.3 → v1.4"
```

### Пример 2: Добавление листа с таблицей

Задача: добавить лист "Сотрудники" с таблицей в файл `hr-v2.1.xlsx`.

```
1. READ
   mcp__excel__get_workbook_metadata("hr-v2.1.xlsx")
   → убедиться, что листа "Сотрудники" нет

2. PLAN
   Тип: STRUCTURE (новый лист + новая таблица) → release++
   Новое имя: hr-v3.0.xlsx

3. VERSION
   Bash: cp "hr-v2.1.xlsx" "hr-v3.0.xlsx"
   Bash: ln -sf "hr-v3.0.xlsx" "hr-last.xlsx"

4. WRITE
   mcp__excel__create_worksheet("hr-v3.0.xlsx", "Сотрудники")
   mcp__excel__write_data_to_excel("hr-v3.0.xlsx", "Сотрудники", "A1",
     [["Имя", "Должность", "Отдел", "Дата найма"]])
   mcp__excel__create_table("hr-v3.0.xlsx", "Сотрудники", "A1:D1", "ТаблицаСотрудников")

5. CHANGELOG
   mcp__excel__write_data_to_excel("hr-v3.0.xlsx", "Changelog", "A{next_row}",
     [["v3.0", "2026-04-04", "{автор}", "STRUCTURE", "Сотрудники",
       "Добавлен лист 'Сотрудники' с таблицей ТаблицаСотрудников"]])

6. REPORT
   "Создан файл hr-v3.0.xlsx. Добавлен лист 'Сотрудники' с таблицей. Версия: v2.1 → v3.0"
```

### Пример 3: Создание сводной таблицы с диаграммой

Задача: создать pivot table и диаграмму на основе листа "Продажи" в файле `sales-v1.0.xlsx`.

```
1. READ
   mcp__excel__get_workbook_metadata("sales-v1.0.xlsx")
   mcp__excel__read_data_from_excel("sales-v1.0.xlsx", "Продажи", "A1:A1")
   → определить структуру данных

2. PLAN
   Тип: STRUCTURE (новые объекты — сводная и диаграмма) → release++
   Новое имя: sales-v2.0.xlsx

3. VERSION
   Bash: cp "sales-v1.0.xlsx" "sales-v2.0.xlsx"
   Bash: ln -sf "sales-v2.0.xlsx" "sales-last.xlsx"

4. WRITE
   mcp__excel__create_worksheet("sales-v2.0.xlsx", "Сводная")
   mcp__excel__create_pivot_table("sales-v2.0.xlsx", "Продажи", "A1:D100",
     "sales-v2.0.xlsx", "Сводная", "A3", ...)
   mcp__excel__create_chart("sales-v2.0.xlsx", "Сводная", "bar", "A3:C20", "H3", ...)

5. CHANGELOG
   mcp__excel__write_data_to_excel("sales-v2.0.xlsx", "Changelog", "A{next_row}",
     [["v2.0", "2026-04-04", "{автор}", "STRUCTURE", "Сводная",
       "Создан лист 'Сводная' со сводной таблицей и столбчатой диаграммой по данным 'Продажи'"]])

6. REPORT
   "Создан файл sales-v2.0.xlsx. Добавлен лист 'Сводная' со сводной таблицей и диаграммой. Версия: v1.0 → v2.0"
```

---

## Обработка ошибок

| Ситуация | Действие |
|---|---|
| Файл не найден | Сообщить пользователю, уточнить полный путь. Не создавать новый файл без явного разрешения. |
| Лист не существует | Проверить `get_workbook_metadata`, уточнить у пользователя имя листа. |
| Ячейка содержит формулу | Перед перезаписью предупредить пользователя, запросить подтверждение. |
| Лист "Changelog" отсутствует | Создать автоматически через `create_worksheet` + записать заголовки. |
| Ошибка MCP-инструмента | Зафиксировать сообщение об ошибке, не создавать новую версию файла, сообщить пользователю. |

---

## Файлы справочника (references/)

Загружать по требованию для деталей:

- **[mcp-api-reference.md](references/mcp-api-reference.md)** — полный справочник всех 23 MCP-методов с параметрами и примерами вызова
- **[versioning-rules.md](references/versioning-rules.md)** — детальные правила версионирования с граничными случаями
- **[changelog-protocol.md](references/changelog-protocol.md)** — полная структура Changelog, шаблоны, примеры записей
- **[formatting-guide.md](references/formatting-guide.md)** — справочник по форматированию через `format_range`, коды числовых форматов
