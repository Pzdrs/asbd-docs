---
title: Report View
description: This document describes the structure of the report view.
---

At the heart of every report lies the `ReportView` class. It holds the following properties:

`title`

:   The title of the report page.

    Defaults to '*Sestava*'.

`headline`

:   The headline of the report page.

<img src="/reports/headline.png" style="border: 1px solid lightgray">

`tables`

:   A list of `ReportTable` instances.

Each report view also needs a **template** and a **renderer** (also just a template).

`template_name`

:   Because reports are a highly specialized feature (mainly CSS-wise), we can't use the standard base template. Instead, a dedicated report base template is used.

    Takes care of the headline, extra content, dividers, etc. and passes the list of tables to a renderer.

    Multiple blocks (injection points) are available for customization:

    `header`

    :   Renders the headline and extra content.

        ```HTML title="Header block"
        {% block headline %}
            <div class="headline">
                <p>{{ report_headline }}</p>
            </div>
        {% endblock %}
        {% resolve_extra_content extra_header %}
        ```

    `content-header_divider`

    :   Divider between `header` and `main_content_data`.

        Defaults to '*&lt;hr>*'.

    `main_content_data`

    :   Includes the renderer template.

        ```HTML
        <br>
        {% include report_renderer with tables=tables %}
        ```

    `content-footer_divider`

    :   Divider between `main_content_data` and `footer`.

        Defaults to '*&lt;hr>*'.

    `footer`

    :   Renders the extra content and a signature.

        ```HTML
        {% resolve_extra_content extra_footer %}
        {% block footer_signature %}
            {% include 'snippets/report_footer_signature.html' %}
        {% endblock %}
        ```

    Defaults to '*asbd_report_base_template.html*'.

`report_renderer`

:   As previously mentioned, it's another template. It is responsible for rendering each table.

    !!! note "Future changes"

        The for loop itself will be moved from the report renderer to the report template so it can be renamed to `table_renderer`.

    Can also be extended to accomodate special tables.

    Multiple blocks (injection points) are available for customization:

    `loop`

    :   The top-level block encapsulating everything.

        `for_each_table`

        :   Encapsulates the following blocks:

            `table_before_first` - Content to insert before the first table

            `table_before_last` - Content to insert before the last table

            `table_before_other` - Content to insert before any other table

            `table_before`

            :   Content to insert before every table.

                Renders the optional heading and info by default.

                ```HTML title="Default table_before block"
                {% if table.heading %}
                    <h4 class="table-heading">{{ table.heading }}</h4>
                {% endif %}
                <p class="table-info">{{ table.info|if_present }}</p>
                ```

            `render` - Holds the tag call (`{% render_report_table table %}`) to render the table.

            `table_after` - Content to insert after every table

            `table_after_first` - Content to insert after the first table

            `table_after_last` - Content to insert after the last table

            `table_after_other` - Content to insert after any other table

## Extending the report template

A good example of a modified report template is the `zaverecne_vyuctovani_rozklad_report_template.html`. We only needed to render the tables and nothing else.

```HTML title="zaverecne_vyuctovani_rozklad_report_template.html"
{% extends 'asbd_report_base_template.html' %}
{% block header %}{% endblock %}
{% block content-header_divider %}{% endblock %}
{% block content-footer_divider %}{% endblock %}
{% block footer %}{% endblock %}
```

## Extending the report renderer

The purpose of the `zaverecne_vyuctovani_sluzeb_report_renderer.html` is to insert a print only page break after every table (except the last one, because the printer would just spit out an empty paper).

In this case, we couldn't have used the `table_after_other` block because the page break wouldn't be inserted after the first table. For that reason, we had to use the broader `table_after` block and manually check if the table is not the last one.

```HTML title="zaverecne_vyuctovani_sluzeb_report_renderer.html"
{% extends 'snippets/report_renderer.html' %}
{% block table_after %}
    {{ block.super }}
    {% if not forloop.last %}
        <div class="print-page-break"></div>
    {% endif %}
{% endblock %}
```
## Extending the base view

The base `ReportView` may be extended to abstract out common functionality. The most common example is the `DumReportView` class, which is specifically tailored for reports that are in some way tied to a single `Dum` or `Byt`.

As far as the report system goes, the `DumReportView` just automates the process of setting a headline. 

```Python title="The headline logic in the DumReportView class"
def get_report_headline(self):
    if self.referral_dum:
        return f'Dům: {self.referral_dum.display_altername_or_street_name}'
    elif self.referral_byt:
        return f'Jednotka: {self.referral_byt.name_variabilni_symbol}'
    else:
        return 'Název není k dispozici'
```

## Extra content

!!! warning "Work in progress"

    The extra content system will be going through a major overhaul in the near future. The current implementation is not very scalable.

Every report can have extra content in the header and footer. This is what the `extra_header` and `extra_footer` properties are for.

### TemplateContent

The `TemplateContent` is used to include one or more templates in the report.

```Python title="Example 1: Assigning the property directly"
class SestavaPredpisyPlatbyMesicni(DumReportView):
    title = 'Měsíční rekapitulace předpisů a plateb'
    extra_header = TemplateContent('snippets/report_kontrolni_sestava_period_extra_header.html',
                                   sestava='měsíční rekapitulace předpisů a plateb (kumulovaně)')
```

```Python title="Example 2: Using a function - dynamic context"
def get_extra_header_content(self):
    return TemplateContent(
        'snippets/report_kontrolni_sestava_period_extra_header.html',
        sestava=f'seznam odečtů měřidel pro službu {self.sluzba}'
    )
```

### StringContent

```Python
class MyReport(ReportView):
    extra_header = StringContent(
        'This is a string content', 
        'This is a second line'
    )
```