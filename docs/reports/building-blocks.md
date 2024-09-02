---
title: Building blocks
description: A comprehensive guide to the building blocks of the report system
---

All tables are constructed from several basic building blocks that are comprehensively described in the following sections.

## ReportTableCell

The cell is the smallest building block within a table. It has two main properties:

```Python
def __init__(self, value=None, default_value='-', custom_css=None) -> None:
    ...
```

`value`

:   Value to be rendered inside of the cell

    Defaults to `None`.

`default_value`

:   Gets rendered inside of the cell if the `value` is Falsy (None or an empty string)

    Defaults to '*-*' (a dash).

Accepts a `StyleModifier`.

The `ReportTableCell` class can be extended to allow for creation of custom cells, which increases efficiency.

### EmptyCell

Simply a cell with no value.

A `ReportTableCell` with both `value` and `default_value` set to an empty string.

```Python
ReportTableCell(default_value='')
# vs
EmptyCell()
```

### BooleanCell

Expects a boolean value.

Based on the `true` and `false` arguments, the boolean value is translated.

```Python
def __init__(self, value, true='ANO', false='NE', default_value='-', custom_css=None) -> None:
    ... 
```

The default translation:

| Value | Translation |
| ----- | ----------- |
| True  | Ano         |
| False | Ne          |

```Python
ReportTableCell('Ano' if True else 'Ne')
# vs
BooleanCell(True) # -> Ano
BooleanCell(True, true='Yes') # -> Yes
```

### DateCell

Expects a value of type `datetime.date`.

Formats the date to a specified format.

```Python
def __init__(self, value, _format='%d.%m.%Y', default_value='-', custom_css=None) -> None:
    ...
```

The date format defaults to '*%d.%m.%Y*'.

```Python
ReportTableCell(datetime.date(1, 1, 2024).strftime('%d.%m.%Y'))
# vs
DateCell(datetime.date(1, 1, 2024)) # -> 1.1.2024
DateCell(datetime.date(1, 1, 2024), _format='%d/%m/%Y') # -> 1/1/2024
```

### ChoiceCell

Best demonstrated with an example.

```Python
typy_jednotek = (
    ('A', 'Bytová jednotka'),
    ('B', 'Nájemní prostor'),
)

ReportTableCell(dict(typy_jednotek).get('A'))
# vs
ChoiceCell('A', typy_jednotek) # -> Bytová jednotka
```

### CurrencyCell

Expects a `float` or a list of `float`s.

Text within these cells is aligned to the **right**.

```Python
ReportTableCell('69 420,00 Kč')
# vs
CurrencyCell(69420) # -> 69 420,00 Kč
```

```Python title="Prefix and suffix"
CurrencyCell(1000, prefix='(-)') # -> (-) 1 000,00 Kč
CurrencyCell(1000, suffix=' (+)') # -> 1 000,00 Kč (+)
CurrencyCell(1000, prefix='(-)', suffix=' (+)') # -> (-) 1 000,00 Kč (+)
```

Under the hood, Babel is used for formatting. The function call to `format_currency()` used by `CurrencyCell` can be seen below.

```Python title="Format call to Babel"
babel.numbers.format_currency(value, "CZK", format=u"#,##0.00 ¤", locale="cs_CZ")
```

### NumberCell

Expects a number.

A custom format can be passed in using `custom_format`.

The Babel library is used for formatting. More on formatting string [here](https://babel.pocoo.org/en/latest/numbers.html).

```Python
ReportTableCell('42.000')
# vs
NumberCell(42, custom_format='#,##0.000') # -> 42.000
```

## ReportTableRow

A row is just a collection of cells.

Similarly as `ReportTableCell`, it accepts a `StyleModifier`.

```Python
def __init__(self, cells, custom_css=None) -> None:
    ...
```

```Python
ReportTableRow((
    ReportTableCell('1'),
    ReportTableCell('2'),
    ReportTableCell('3')
))
```

## ReportTable

At last, we arrive at the top level building block - the `ReportTable`. Every table in a report corresponds to an instance of this class.

```Python
def __init__(self, columns: Iterable[str | ReportTableCell] = None,
                heading: str = None, info: str = None,
                extra_data: dict = None,
                custom_css: StyleModifier = None) -> None:
    self.custom_css = custom_css or StyleModifier()
    if columns:
        self.columns = tuple(
            map(lambda column: column if isinstance(column, ReportTableCell) else ReportTableCell(column),
                columns))
    self.heading = heading
    self.extra_data = {} if extra_data is None else extra_data
    self.info = info
    self.rows = []
    self.translations = {}
```

The class properties are the following:

`heading`

:   An optional heading text.

    By default rendered right above the table.

`info`

:   An optional info text.

    By default rendered below the heading.

`columns`

:   A collection column cells.

    Can be a mix of strings and instances of `ReportTableCell` (e.g. if custom CSS is necessary).

    If left empty, the table becomes headless.

`rows`

:   A collection of actual table rows (excluding the first row - the *&lt;thead>*).

    Rows are added using dedicated methods, cannot be passed into to the constructor.

`extra_data`

:   An optional dictionary can be bound to a table and later accessed in the template for more complex rendering.

`translations`

:   Optional mapping dictionary for translating values in cells.

```Python
ReportTable(('column 1', 'column 2'), 'heading', 'info text', {'key':'val'})
```

There are two ways of adding rows to tables - **explicit** and **implicit**.

```Python title="Explicit function"
table = ReportTable()
# Takes in a row object
table.add_row_expl(ReportTableRow.of(
    ReportTableCell(),
    ...
))
```

```Python title="Implicit function"
table = ReportTable()
# Takes in an arbitrary amount of cell objects
table.add_row_impl(
    ReportTableCell(),
    ...
)
```

### HeadlessReportTable

A headless table is simply put a table without a header row (without the *&lt;th>* tag).

```Python
# We have to specify an empty tuple for the columns
ReportTable((), heading="Headless table")
# vs
# Implicit empty columns tuple
HeadlessReportTable(heading="Headless table")
```