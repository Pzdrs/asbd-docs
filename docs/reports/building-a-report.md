---
title: Building a Report
description: Building a Report
---

A report is just like any other view in Django. We understandably have to use CBV as the whole report system is built with the approach in mind.

First things first, let's create our view and extend the `ReportView` class.

```Python title="Create view"
class MyReportView(ReportView):
    pass
```

When we open our view we get an `ImproperlyConfigured` exception saying *ReportMixin requires either a definition of 'report_headline' or an implementation of 'get_report_headline()'*. So we have to define our report headline.

```Python title="Define headline"
class MyReportView(ReportView):
    headline = 'My fancy report'
    tables = [] # So we can see the complete barebones report
```

<img src="/reports/empty_report.png" style="border: 1px solid lightgray" alt="An empty report">

At this stage, our report isn't very useful. Before we get into building tables, we need to familiarize ourselves with the basic building blocks.

## Building blocks

### ReportTableCell

The cell is the smallest building block within a table. It has two main properties:

`value`

:   Value to be rendered inside of the cell

    Defaults to `None`.

`default_value`

:   Gets rendered inside of the cell if the `value` is Falsy (None or an empty string)

    Defaults to '*-*' (a dash).

Accepts a `StyleModifier`.

The `ReportTableCell` class can be extended to allow for creation of custom cells, which increases efficiency.

#### EmptyCell

Simply a cell with no value.

A `ReportTableCell` with both `value` and `default_value` set to an empty string.

```Python
ReportTableCell(default_value='')
# vs
EmptyCell()
```

#### BooleanCell

Expects a boolean value.

Based on the `true` and `false` arguments, the boolean value is translated.

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

#### DateCell

Expects a value of type `datetime.date`.

Formats the date to a specified format.

Defaults to '*%d.%m.%Y*'.

```Python
ReportTableCell(datetime.date(1, 1, 2024).strftime('%d.%m.%Y'))
# vs
DateCell(datetime.date(1, 1, 2024)) # -> 1.1.2024
DateCell(datetime.date(1, 1, 2024), _format='%d/%m/%Y') # -> 1/1/2024
```

#### ChoiceCell

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

#### CurrencyCell

Expects a `float` or a list of `float`s.

Text within these cells is aligned to the **right**.

```Python
ReportTableCell('69 420,00 Kč')
# vs
CurrencyCell(69420) # -> 69 420,00 Kč
```

```Python title="Prefix and suffix"
CurrencyCell(1000, prefix='(-)') # -> (-) 1 000,00 Kč
CurrencyCell(1000, suffix=' //') # -> 1 000,00 Kč //
CurrencyCell(1000, prefix='(-)', suffix=' //') # -> (-) 1 000,00 Kč //
```

Under the hood, Babel is used for formatting. The function call to `format_currency()` used by `CurrencyCell` can be seen below.

```Python title="Format call to Babel"
babel.numbers.format_currency(value, "CZK", format=u"#,##0.00 ¤", locale="cs_CZ")
```

#### NumberCell

Expects a number.

A custom format can be passed in using `custom_format`.

```Python
ReportTableCell('42.000')
# vs
NumberCell(42, custom_format='#,##0.000') # -> 42.000
```

### ReportTableRow

A row is just a collection of cells.

Similarly as `ReportTableCell`, it accepts a `StyleModifier`.

```Python
ReportTableRow((
    ReportTableCell('1'),
    ReportTableCell('2'),
    ReportTableCell('3')
))
```

### ReportTable

At last, we arrive at the top level building block - the `ReportTable`. Every table in a report corresponds to an instance of this class.

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

## Creating our first table

Let's say we want to create a two variable truth table. A table with two columns for the two variables would look like this:

```Python title="Table instantiation"
ReportTable(('A', 'B'), heading="Two variable truth table")
```

At this point, we could add this table instance into the table list in our report, but we would only get the table header (the first row), because we haven't actually added any data to it yet.

We have two options when it comes to adding rows, an implicit function (`add_row_impl()`) and an explicit function (`add_row_expl()`).

`add_row_impl()`

:   This function takes in an arbitrary number of cells.

`add_row_expl()`

:   This function expects an instance of `ReportTableRow`.

```Python
class MyReportView(ReportView):
    headline = 'My fancy report'

    def get_tables(self):
        truth_table = ReportTable(('A', 'B'), heading="Two variable truth table")

        truth_table.add_row_impl(ReportTableCell('0'), ReportTableCell('0'))
        truth_table.add_row_impl(ReportTableCell('1'), ReportTableCell('0'))
        truth_table.add_row_impl(ReportTableCell('0'), ReportTableCell('1'))
        truth_table.add_row_impl(ReportTableCell('1'), ReportTableCell('1'))

        return [truth_table]
```
<img src="/reports/truth_table.png" style="border: 1px solid lightgray" alt="Truth table example">
