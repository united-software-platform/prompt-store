# Excel MCP — Справочник методов API

Полный список 25 методов MCP-сервера `excel`. Все методы вызываются с префиксом `mcp__excel__`.

---

## Метаданные и чтение

### `get_workbook_metadata`

Возвращает структуру книги: список листов, именованные диапазоны, свойства файла.

**Параметры:**

| Параметр | Тип | Обязательный | Описание |
|---|---|---|---|
| `filepath` | string | да | Полный путь к файлу .xlsx |

**Пример вызова:**
```
mcp__excel__get_workbook_metadata(
  filepath="/home/user/docs/report-v1.3.xlsx"
)
```

**Возвращает:** список имён листов, именованные диапазоны, количество строк/столбцов на каждом листе.

---

### `read_data_from_excel`

Читает данные из указанного диапазона ячеек.

**Параметры:**

| Параметр | Тип | Обязательный | Описание |
|---|---|---|---|
| `filepath` | string | да | Полный путь к файлу .xlsx |
| `sheet_name` | string | да | Имя листа |
| `range` | string | да | Диапазон в формате `A1:D10` или одна ячейка `B5` |

**Пример вызова:**
```
mcp__excel__read_data_from_excel(
  filepath="/home/user/docs/report-v1.3.xlsx",
  sheet_name="Данные",
  range="A1:E20"
)
```

**Рекомендация:** всегда указывать минимально необходимый диапазон (принцип minimal-data).

---

## Создание файлов и листов

### `create_workbook`

Создаёт новый пустой файл .xlsx.

**Параметры:**

| Параметр | Тип | Обязательный | Описание |
|---|---|---|---|
| `filepath` | string | да | Полный путь для нового файла |

**Пример вызова:**
```
mcp__excel__create_workbook(
  filepath="/home/user/docs/report-v1.4.xlsx"
)
```

**Примечание:** создаёт файл с одним пустым листом "Sheet1". Переименовать или удалить его перед использованием.

---

### `create_worksheet`

Добавляет новый лист в существующую книгу.

**Параметры:**

| Параметр | Тип | Обязательный | Описание |
|---|---|---|---|
| `filepath` | string | да | Полный путь к файлу .xlsx |
| `sheet_name` | string | да | Имя нового листа |

**Пример вызова:**
```
mcp__excel__create_worksheet(
  filepath="/home/user/docs/report-v1.4.xlsx",
  sheet_name="Аналитика"
)
```

---

### `delete_worksheet`

Удаляет лист из книги.

**Параметры:**

| Параметр | Тип | Обязательный | Описание |
|---|---|---|---|
| `filepath` | string | да | Полный путь к файлу .xlsx |
| `sheet_name` | string | да | Имя листа для удаления |

**Пример вызова:**
```
mcp__excel__delete_worksheet(
  filepath="/home/user/docs/report-v2.0.xlsx",
  sheet_name="Черновик"
)
```

**Предупреждение:** операция необратима. Использовать только в новой версии файла.

---

### `rename_worksheet`

Переименовывает лист.

**Параметры:**

| Параметр | Тип | Обязательный | Описание |
|---|---|---|---|
| `filepath` | string | да | Полный путь к файлу .xlsx |
| `old_name` | string | да | Текущее имя листа |
| `new_name` | string | да | Новое имя листа |

**Пример вызова:**
```
mcp__excel__rename_worksheet(
  filepath="/home/user/docs/report-v2.0.xlsx",
  old_name="Sheet1",
  new_name="Сводка"
)
```

---

### `copy_worksheet`

Копирует лист из одной книги в другую (или в ту же книгу).

**Параметры:**

| Параметр | Тип | Обязательный | Описание |
|---|---|---|---|
| `source_filepath` | string | да | Путь к исходному файлу |
| `source_sheet` | string | да | Имя исходного листа |
| `target_filepath` | string | да | Путь к целевому файлу |
| `target_sheet` | string | да | Имя листа в целевом файле |

**Пример вызова:**
```
mcp__excel__copy_worksheet(
  source_filepath="/home/user/docs/report-v1.3.xlsx",
  source_sheet="Данные",
  target_filepath="/home/user/docs/report-v1.4.xlsx",
  target_sheet="Данные"
)
```

**Применение:** основной метод переноса листов при создании новой версии файла.

---

## Запись данных

### `write_data_to_excel`

Записывает данные в указанный диапазон ячеек.

**Параметры:**

| Параметр | Тип | Обязательный | Описание |
|---|---|---|---|
| `filepath` | string | да | Полный путь к файлу .xlsx |
| `sheet_name` | string | да | Имя листа |
| `start_cell` | string | да | Начальная ячейка диапазона, например `A1` |
| `data` | array | да | Двумерный массив значений `[[строка1], [строка2]]` |

**Пример вызова:**
```
mcp__excel__write_data_to_excel(
  filepath="/home/user/docs/report-v1.4.xlsx",
  sheet_name="Данные",
  start_cell="A1",
  data=[
    ["Имя", "Отдел", "Оклад"],
    ["Иванов И.И.", "Финансы", 85000],
    ["Петрова А.С.", "HR", 72000]
  ]
)
```

---

## Управление строками и столбцами

### `insert_rows`

Вставляет пустые строки в указанную позицию.

**Параметры:**

| Параметр | Тип | Обязательный | Описание |
|---|---|---|---|
| `filepath` | string | да | Полный путь к файлу .xlsx |
| `sheet_name` | string | да | Имя листа |
| `row_index` | integer | да | Номер строки перед которой вставить (1-based) |
| `count` | integer | да | Количество вставляемых строк |

**Пример вызова:**
```
mcp__excel__insert_rows(
  filepath="/home/user/docs/report-v1.4.xlsx",
  sheet_name="Данные",
  row_index=5,
  count=2
)
```

---

### `insert_columns`

Вставляет пустые столбцы в указанную позицию.

**Параметры:**

| Параметр | Тип | Обязательный | Описание |
|---|---|---|---|
| `filepath` | string | да | Полный путь к файлу .xlsx |
| `sheet_name` | string | да | Имя листа |
| `column_index` | integer | да | Номер столбца перед которым вставить (1-based, A=1) |
| `count` | integer | да | Количество вставляемых столбцов |

**Пример вызова:**
```
mcp__excel__insert_columns(
  filepath="/home/user/docs/report-v1.4.xlsx",
  sheet_name="Данные",
  column_index=3,
  count=1
)
```

---

### `delete_sheet_rows`

Удаляет строки из листа.

**Параметры:**

| Параметр | Тип | Обязательный | Описание |
|---|---|---|---|
| `filepath` | string | да | Полный путь к файлу .xlsx |
| `sheet_name` | string | да | Имя листа |
| `row_index` | integer | да | Номер первой удаляемой строки (1-based) |
| `count` | integer | да | Количество удаляемых строк |

**Пример вызова:**
```
mcp__excel__delete_sheet_rows(
  filepath="/home/user/docs/report-v1.4.xlsx",
  sheet_name="Данные",
  row_index=10,
  count=3
)
```

---

### `delete_sheet_columns`

Удаляет столбцы из листа.

**Параметры:**

| Параметр | Тип | Обязательный | Описание |
|---|---|---|---|
| `filepath` | string | да | Полный путь к файлу .xlsx |
| `sheet_name` | string | да | Имя листа |
| `column_index` | integer | да | Номер первого удаляемого столбца (1-based) |
| `count` | integer | да | Количество удаляемых столбцов |

**Пример вызова:**
```
mcp__excel__delete_sheet_columns(
  filepath="/home/user/docs/report-v1.4.xlsx",
  sheet_name="Данные",
  column_index=5,
  count=2
)
```

---

## Управление диапазонами

### `delete_range`

Очищает содержимое диапазона (данные и/или форматирование).

**Параметры:**

| Параметр | Тип | Обязательный | Описание |
|---|---|---|---|
| `filepath` | string | да | Полный путь к файлу .xlsx |
| `sheet_name` | string | да | Имя листа |
| `range` | string | да | Диапазон в формате `A1:D10` |

**Пример вызова:**
```
mcp__excel__delete_range(
  filepath="/home/user/docs/report-v1.4.xlsx",
  sheet_name="Данные",
  range="C5:F15"
)
```

---

### `copy_range`

Копирует диапазон ячеек в другое место.

**Параметры:**

| Параметр | Тип | Обязательный | Описание |
|---|---|---|---|
| `filepath` | string | да | Полный путь к файлу .xlsx |
| `sheet_name` | string | да | Имя исходного листа |
| `source_range` | string | да | Исходный диапазон |
| `target_sheet` | string | да | Имя целевого листа |
| `target_cell` | string | да | Начальная ячейка целевого диапазона |

**Пример вызова:**
```
mcp__excel__copy_range(
  filepath="/home/user/docs/report-v1.4.xlsx",
  sheet_name="Данные",
  source_range="A1:D20",
  target_sheet="Архив",
  target_cell="A1"
)
```

---

## Объединение ячеек

### `merge_cells`

Объединяет ячейки в указанном диапазоне.

**Параметры:**

| Параметр | Тип | Обязательный | Описание |
|---|---|---|---|
| `filepath` | string | да | Полный путь к файлу .xlsx |
| `sheet_name` | string | да | Имя листа |
| `range` | string | да | Диапазон для объединения |

**Пример вызова:**
```
mcp__excel__merge_cells(
  filepath="/home/user/docs/report-v1.4.xlsx",
  sheet_name="Отчёт",
  range="A1:E1"
)
```

---

### `unmerge_cells`

Разъединяет объединённые ячейки.

**Параметры:**

| Параметр | Тип | Обязательный | Описание |
|---|---|---|---|
| `filepath` | string | да | Полный путь к файлу .xlsx |
| `sheet_name` | string | да | Имя листа |
| `range` | string | да | Диапазон для разъединения |

**Пример вызова:**
```
mcp__excel__unmerge_cells(
  filepath="/home/user/docs/report-v1.4.xlsx",
  sheet_name="Отчёт",
  range="A1:E1"
)
```

---

### `get_merged_cells`

Возвращает список всех объединённых диапазонов на листе.

**Параметры:**

| Параметр | Тип | Обязательный | Описание |
|---|---|---|---|
| `filepath` | string | да | Полный путь к файлу .xlsx |
| `sheet_name` | string | да | Имя листа |

**Пример вызова:**
```
mcp__excel__get_merged_cells(
  filepath="/home/user/docs/report-v1.4.xlsx",
  sheet_name="Отчёт"
)
```

---

## Форматирование

### `format_range`

Применяет форматирование к диапазону ячеек.

**Параметры:**

| Параметр | Тип | Обязательный | Описание |
|---|---|---|---|
| `filepath` | string | да | Полный путь к файлу .xlsx |
| `sheet_name` | string | да | Имя листа |
| `range` | string | да | Диапазон форматирования |
| `format` | object | да | Объект с параметрами форматирования |

**Структура объекта `format`:**
```json
{
  "font": {
    "name": "Calibri",
    "size": 11,
    "bold": true,
    "italic": false,
    "color": "FF0000"
  },
  "fill": {
    "color": "FFFF00"
  },
  "border": {
    "style": "thin",
    "color": "000000"
  },
  "alignment": {
    "horizontal": "center",
    "vertical": "middle",
    "wrap_text": true
  },
  "number_format": "YYYY-MM-DD"
}
```

**Пример вызова:**
```
mcp__excel__format_range(
  filepath="/home/user/docs/report-v1.4.xlsx",
  sheet_name="Данные",
  range="A1:F1",
  format={
    "font": {"bold": true, "color": "FFFFFF"},
    "fill": {"color": "2F5496"},
    "alignment": {"horizontal": "center"}
  }
)
```

Подробные примеры — в [formatting-guide.md](formatting-guide.md).

---

## Формулы

### `apply_formula`

Записывает формулу в ячейку или диапазон.

**Параметры:**

| Параметр | Тип | Обязательный | Описание |
|---|---|---|---|
| `filepath` | string | да | Полный путь к файлу .xlsx |
| `sheet_name` | string | да | Имя листа |
| `cell` | string | да | Ячейка для формулы |
| `formula` | string | да | Формула, начинающаяся с `=` |

**Пример вызова:**
```
mcp__excel__apply_formula(
  filepath="/home/user/docs/report-v1.4.xlsx",
  sheet_name="Данные",
  cell="G2",
  formula="=SUM(D2:F2)"
)
```

---

### `validate_formula_syntax`

Проверяет синтаксическую корректность формулы без записи в файл.

**Параметры:**

| Параметр | Тип | Обязательный | Описание |
|---|---|---|---|
| `formula` | string | да | Формула для проверки |

**Пример вызова:**
```
mcp__excel__validate_formula_syntax(
  formula="=VLOOKUP(A2,Sheet2!A:C,3,FALSE)"
)
```

**Рекомендация:** всегда вызывать перед `apply_formula` для сложных формул.

---

## Валидация

### `validate_excel_range`

Проверяет, что диапазон корректен для указанного листа (не выходит за границы, правильный синтаксис).

**Параметры:**

| Параметр | Тип | Обязательный | Описание |
|---|---|---|---|
| `filepath` | string | да | Полный путь к файлу .xlsx |
| `sheet_name` | string | да | Имя листа |
| `range` | string | да | Диапазон для проверки |

**Пример вызова:**
```
mcp__excel__validate_excel_range(
  filepath="/home/user/docs/report-v1.4.xlsx",
  sheet_name="Данные",
  range="A1:Z1000"
)
```

---

### `get_data_validation_info`

Возвращает настройки валидации данных (выпадающие списки, ограничения) для ячеек.

**Параметры:**

| Параметр | Тип | Обязательный | Описание |
|---|---|---|---|
| `filepath` | string | да | Полный путь к файлу .xlsx |
| `sheet_name` | string | да | Имя листа |
| `range` | string | нет | Диапазон (если не указан — весь лист) |

**Пример вызова:**
```
mcp__excel__get_data_validation_info(
  filepath="/home/user/docs/report-v1.4.xlsx",
  sheet_name="Данные",
  range="D2:D100"
)
```

---

## Структурные объекты

### `create_table`

Создаёт именованную таблицу Excel из диапазона данных.

**Параметры:**

| Параметр | Тип | Обязательный | Описание |
|---|---|---|---|
| `filepath` | string | да | Полный путь к файлу .xlsx |
| `sheet_name` | string | да | Имя листа |
| `range` | string | да | Диапазон данных включая заголовки |
| `table_name` | string | да | Уникальное имя таблицы |
| `style` | string | нет | Стиль таблицы, например `TableStyleMedium2` |

**Пример вызова:**
```
mcp__excel__create_table(
  filepath="/home/user/docs/report-v1.4.xlsx",
  sheet_name="Данные",
  range="A1:D50",
  table_name="ТаблицаДанных",
  style="TableStyleMedium9"
)
```

---

### `create_pivot_table`

Создаёт сводную таблицу.

**Параметры:**

| Параметр | Тип | Обязательный | Описание |
|---|---|---|---|
| `source_filepath` | string | да | Путь к файлу с исходными данными |
| `source_sheet` | string | да | Лист с исходными данными |
| `source_range` | string | да | Диапазон исходных данных |
| `target_filepath` | string | да | Путь к целевому файлу |
| `target_sheet` | string | да | Лист для размещения сводной таблицы |
| `target_cell` | string | да | Начальная ячейка |
| `rows` | array | да | Поля для строк |
| `columns` | array | нет | Поля для столбцов |
| `values` | array | да | Поля для значений с агрегацией |
| `filters` | array | нет | Поля для фильтров |

**Пример вызова:**
```
mcp__excel__create_pivot_table(
  source_filepath="/home/user/docs/sales-v2.0.xlsx",
  source_sheet="Продажи",
  source_range="A1:E500",
  target_filepath="/home/user/docs/sales-v2.0.xlsx",
  target_sheet="Сводная",
  target_cell="A3",
  rows=["Регион", "Менеджер"],
  values=[{"field": "Сумма", "aggregation": "SUM"}]
)
```

---

### `create_chart`

Создаёт диаграмму на основе данных.

**Параметры:**

| Параметр | Тип | Обязательный | Описание |
|---|---|---|---|
| `filepath` | string | да | Полный путь к файлу .xlsx |
| `sheet_name` | string | да | Лист для размещения диаграммы |
| `chart_type` | string | да | Тип: `bar`, `line`, `pie`, `scatter`, `area` |
| `data_range` | string | да | Диапазон данных для диаграммы |
| `target_cell` | string | да | Ячейка для верхнего левого угла диаграммы |
| `title` | string | нет | Заголовок диаграммы |
| `width` | integer | нет | Ширина в пикселях |
| `height` | integer | нет | Высота в пикселях |

**Пример вызова:**
```
mcp__excel__create_chart(
  filepath="/home/user/docs/sales-v2.0.xlsx",
  sheet_name="Сводная",
  chart_type="bar",
  data_range="A3:C15",
  target_cell="H3",
  title="Продажи по регионам",
  width=600,
  height=400
)
```
