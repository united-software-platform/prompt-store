# Excel MCP Manager — Справочник по форматированию

Полный справочник параметров `mcp__excel__format_range` с примерами и кодами числовых форматов.

---

## Структура объекта `format`

```json
{
  "font": {
    "name": "string",
    "size": "number",
    "bold": "boolean",
    "italic": "boolean",
    "underline": "boolean",
    "color": "HEX без #"
  },
  "fill": {
    "color": "HEX без #"
  },
  "border": {
    "style": "none|thin|medium|thick|double|dashed|dotted",
    "color": "HEX без #",
    "top": { "style": "...", "color": "..." },
    "bottom": { "style": "...", "color": "..." },
    "left": { "style": "...", "color": "..." },
    "right": { "style": "...", "color": "..." }
  },
  "alignment": {
    "horizontal": "left|center|right|fill|justify",
    "vertical": "top|middle|bottom",
    "wrap_text": "boolean",
    "indent": "number"
  },
  "number_format": "string"
}
```

Все секции необязательны — указывать только те параметры, которые нужно изменить.

---

## Параметры шрифта (font)

| Параметр | Тип | Описание | Пример |
|---|---|---|---|
| `name` | string | Название шрифта | `"Calibri"`, `"Arial"`, `"Times New Roman"` |
| `size` | number | Размер в пунктах | `11`, `14`, `16` |
| `bold` | boolean | Жирный | `true`, `false` |
| `italic` | boolean | Курсив | `true`, `false` |
| `underline` | boolean | Подчёркивание | `true`, `false` |
| `color` | string | Цвет текста в HEX | `"FF0000"` (красный), `"FFFFFF"` (белый) |

---

## Параметры заливки (fill)

| Параметр | Тип | Описание | Пример |
|---|---|---|---|
| `color` | string | Цвет фона в HEX | `"FFFF00"` (жёлтый), `"2F5496"` (синий) |

---

## Стили границ (border)

| Стиль | Описание |
|---|---|
| `none` | Без границы |
| `thin` | Тонкая линия (наиболее распространённая) |
| `medium` | Средняя линия |
| `thick` | Толстая линия |
| `double` | Двойная линия |
| `dashed` | Штриховая |
| `dotted` | Пунктирная |
| `hair` | Волосяная (тончайшая) |

Границы можно задавать для всего диапазона (`border`) или для каждой стороны отдельно (`top`, `bottom`, `left`, `right`).

---

## Коды числовых форматов (number_format)

### Общие форматы

| Код | Описание | Пример результата |
|---|---|---|
| `General` | Автоматический | как есть |
| `@` | Текст | `12345` → `"12345"` |
| `0` | Целое число | `1234.5` → `1235` |
| `0.00` | Два знака после запятой | `1234.5` → `1234.50` |
| `#,##0` | Целое с разделителем тысяч | `1234567` → `1 234 567` |
| `#,##0.00` | С разделителем и 2 знаками | `1234.5` → `1 234.50` |

### Форматы дат

| Код | Описание | Пример результата |
|---|---|---|
| `YYYY-MM-DD` | ISO 8601 | `2026-04-04` |
| `DD.MM.YYYY` | Русский формат | `04.04.2026` |
| `DD/MM/YYYY` | Европейский формат | `04/04/2026` |
| `DD MMMM YYYY` | С полным названием месяца | `04 Апрель 2026` |
| `YYYY-MM-DD HH:MM` | Дата и время | `2026-04-04 14:30` |
| `HH:MM:SS` | Время | `14:30:00` |

### Форматы процентов

| Код | Описание | Пример результата |
|---|---|---|
| `0%` | Целый процент | `0.125` → `13%` |
| `0.0%` | Один знак | `0.125` → `12.5%` |
| `0.00%` | Два знака | `0.125` → `12.50%` |

### Денежные форматы

| Код | Описание | Пример результата |
|---|---|---|
| `#,##0 ₽` | Рубли без копеек | `1500` → `1 500 ₽` |
| `#,##0.00 ₽` | Рубли с копейками | `1500.5` → `1 500.50 ₽` |
| `$#,##0.00` | Доллары | `1500` → `$1,500.00` |
| `€#,##0.00` | Евро | `1500` → `€1,500.00` |

---

## Готовые примеры форматирования

### Заголовок таблицы (тёмный фон, белый жирный текст)
```
mcp__excel__format_range(
  filepath="...",
  sheet_name="Данные",
  range="A1:F1",
  format={
    "font": {"bold": true, "color": "FFFFFF", "size": 12},
    "fill": {"color": "2F5496"},
    "alignment": {"horizontal": "center", "vertical": "middle"},
    "border": {"style": "thin", "color": "1F3864"}
  }
)
```

### Чередование строк (зебра)

Чётные строки (светло-синий фон):
```
mcp__excel__format_range(
  filepath="...",
  sheet_name="Данные",
  range="A2:F2",
  format={"fill": {"color": "DCE6F1"}}
)
```

Нечётные строки (белый фон):
```
mcp__excel__format_range(
  filepath="...",
  sheet_name="Данные",
  range="A3:F3",
  format={"fill": {"color": "FFFFFF"}}
)
```

Повторить для всех строк данных.

### Форматирование дат (ISO формат)
```
mcp__excel__format_range(
  filepath="...",
  sheet_name="Данные",
  range="B2:B100",
  format={"number_format": "YYYY-MM-DD"}
)
```

### Форматирование денежных сумм
```
mcp__excel__format_range(
  filepath="...",
  sheet_name="Данные",
  range="D2:D100",
  format={"number_format": "#,##0.00 ₽"}
)
```

### Форматирование процентов
```
mcp__excel__format_range(
  filepath="...",
  sheet_name="Данные",
  range="E2:E100",
  format={"number_format": "0.0%"}
)
```

### Подсветка статусов

**Done — зелёный фон:**
```
mcp__excel__format_range(
  filepath="...",
  sheet_name="Задачи",
  range="A5:F5",
  format={
    "fill": {"color": "C6EFCE"},
    "font": {"color": "375623"}
  }
)
```

**In Progress — жёлтый фон:**
```
mcp__excel__format_range(
  filepath="...",
  sheet_name="Задачи",
  range="A6:F6",
  format={
    "fill": {"color": "FFEB9C"},
    "font": {"color": "9C5700"}
  }
)
```

**Blocked — красный фон:**
```
mcp__excel__format_range(
  filepath="...",
  sheet_name="Задачи",
  range="A7:F7",
  format={
    "fill": {"color": "FFC7CE"},
    "font": {"color": "9C0006"}
  }
)
```

**On Hold — серый фон:**
```
mcp__excel__format_range(
  filepath="...",
  sheet_name="Задачи",
  range="A8:F8",
  format={
    "fill": {"color": "EDEDED"},
    "font": {"color": "595959"}
  }
)
```

### Выделение итоговой строки
```
mcp__excel__format_range(
  filepath="...",
  sheet_name="Данные",
  range="A101:F101",
  format={
    "font": {"bold": true, "color": "000000"},
    "fill": {"color": "BDD7EE"},
    "border": {
      "top": {"style": "medium", "color": "000000"},
      "bottom": {"style": "double", "color": "000000"}
    }
  }
)
```

### Текст с переносом (длинные описания)
```
mcp__excel__format_range(
  filepath="...",
  sheet_name="Данные",
  range="F2:F100",
  format={
    "alignment": {"wrap_text": true, "vertical": "top"},
    "font": {"size": 10}
  }
)
```

---

## Справочник цветов

### Стандартные цвета Excel

| Цвет | HEX | Применение |
|---|---|---|
| Белый | `FFFFFF` | Фон, текст на тёмном |
| Чёрный | `000000` | Основной текст, границы |
| Серый светлый | `EDEDED` | Фон нечётных строк |
| Серый средний | `A6A6A6` | Вспомогательный текст |
| Синий Excel | `2F5496` | Заголовки таблиц |
| Голубой светлый | `DCE6F1` | Фон чётных строк |
| Голубой средний | `BDD7EE` | Итоговые строки |
| Зелёный успех | `C6EFCE` | Статус Done |
| Зелёный текст | `375623` | Текст статуса Done |
| Жёлтый предупреждение | `FFEB9C` | Статус In Progress |
| Жёлтый текст | `9C5700` | Текст статуса In Progress |
| Красный ошибка | `FFC7CE` | Статус Blocked/Error |
| Красный текст | `9C0006` | Текст статуса Blocked |
| Тёмно-синий | `1F3864` | Акцентные заголовки |
| Оранжевый | `F4B942` | Предупреждения |

### Корпоративная палитра (нейтральная)

| Название | HEX |
|---|---|
| Основной синий | `2E74B5` |
| Тёмный синий | `1F4E79` |
| Акцентный оранжевый | `ED7D31` |
| Тёмный оранжевый | `C55A11` |
| Серо-синий | `4472C4` |
| Тёмный серо-синий | `2F5496` |
