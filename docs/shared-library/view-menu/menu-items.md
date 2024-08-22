---
title: Menu items
description: Comprehensive guide to menu items
---

The main building blocks of the menu system are the menu items. They are used to define the structure of the menu and can be nested to create complex menus.

At the top of the hierarchy stands the `MenuItem` class. It is an abstract class that is not meant to be used directly. Instead, there are several concrete classes that inherit from it and provide actual functionality.

A menu item can be enabled or disabled using the `enabled` property. By default, all menu items are enabled.

The `MenuItem` class is a `StyleableComponent` and as such, using the `custom_css` property, it is possible to apply custom CSS classes to the menu item. More information about styling can be found on the [Styling](../styleable-components/index.md) page.

The following sections describe the concrete menu item classes.

## LinkMenuItem

Base class for all menu items that represent a simple link.

```Python title="Basic usage"
# A simple link
LinkMenuItem('Link label', 'app:view')
# Actual URL instead of a view name
LinkMenuItem('Link label', '/some/url', reversible=False)
```

A set of **get** and **query** parameters can be passed to the link using the `get_params` and `query_params` arguments. Note that both arguments can be used at the same time.

```Python title="Get and query parameters"
# Given the URL definition 'view/<int:pk>/', the url will be 'view/1/'
LinkMenuItem('Link label', 'app:view', get_params={'pk': '1'})
# Results in /app/view/?param1=value1
LinkMenuItem('Link label', 'app:view', query_params={'param1': 'value1'})
```

There are several classes that inherit from the `LinkMenuItem` class and provide additional functionality. The constructor signature is identical to the `LinkMenuItem` class.

### SuccessLink

Adds a success class to the link.

```Python
LinkMenuItem('Link label', 'app:view', custom_css=StyleModifier(('text-success',)))
# vs
SuccessLink('Link label', 'app:view')
```

### DangerLink

Adds a danger class to the link.

```Python
LinkMenuItem('Link label', 'app:view', custom_css=StyleModifier(('text-danger',)))
# vs
DangerLink('Link label', 'app:view')
```

### ListingLink

Sets the label to 'Všechny záznamy' by default.

```Python
LinkMenuItem('Všechny záznamy', 'app:view',)
# vs
ListingLink('app:view')
```

### CreateLink

Sets the label to 'Přidat nový záznam' by default.

```Python
LinkMenuItem('Přidat nový záznam', 'app:view')
# vs
CreateLink('app:view')
```

### ObjectRelatedLink

Represents a link related to a single object, i.e. update the object, delete, view details, etc.

It expects two arguments, `view_name` and `pk`. The `view_name` is the name of the view that the link points to and `pk` is the primary key of the object. The default label is 'Object link'.

The raw use of the `ObjectRelatedLink` class on its own is not very helpful, so other subclasses are implemented.

#### UpdateLink

The label defaults to 'Upravit'.

#### DeleteLink

The label defaults to 'Vymazat' and a danger class is applied to the link.

## SpacerMenuItem

A simple menu item that renders a specific number of `<br>` tags (**1** by default).

```Python title="Basic usage"
# A single spacer
SpacerMenuItem()
# Multiple spacers
SpacerMenuItem(3)
```

## SectionTitle

Because we might want to split menus into different sections, the `SectionTitle` class is used to create a section title. It renders as a `<b>` tag.

```Python
# Renders as <b>Section title</b>
SectionTitle('Section title')
```

## Filters

Filters usually come in handy in listing views. A filter is a list of links that simply add a query parameter to the URL and the server uses that to filter the results.

The `Filter` class expects 3 arguments:

`request`

:   The request object. 

    Mainly used to check if the filter has been applied, if so, the filter is not rendered.

`queryset`

:   The list of objects that can be used for filtering.

`query_parameter`

:   The name of the query parameter that will be added to the URL.

```Python title="Basic usage"
Filter(request, queryset, 'filter')
```

Each object is rendered as a link. The actual link label is by default the string representation of the object. If the object is a model instance, the `__str__` method is used. It is possible to pass in a callable `display_name` argument.

```Python title="Custom display name"
Filter(request, queryset, 'filter', display_name=lambda obj: obj.name)
```

The filter is rendered as a underscored heading and a list of links. The heading is by default 'Filtry zobrazení' but can be changed with the `title` property.

### Extending the base class

```Python title="DumFilter"
class DumFilter(Filters):
    def __init__(self, request: HttpRequest, queryset: QuerySet = None, 
                 title: str = None, hide: bool = True, enabled: bool = True,
                 custom_css: StyleModifier = None) -> None:
        super().__init__(request, queryset or Domy.objects.account_can_see(request.user), 'referraldumid',
                         lambda dum: dum.display_altername_or_street_name, title, hide, enabled,
                         custom_css)
```

This filter by default, if no queryset was provided, uses the `Domy` model and gets all the objects the current user has access to. 

The `query_parameter` is set to `referraldumid`.

And lastly, the `display_name` is set to a lambda function that returns the `display_altername_or_street_name` property of the `Dum` model instance.

```Python title="Usage"
# The simplest way to use this filter
DumFilter(request)
# Using a custom queryset
DumFilter(request, Domy.objects.all())
```

## Conditionals

As we discussed, every menu item has to possibility of being disabled. This behavior is controlled by the `enabled` property. If we have a bunch of menu items that need to be hidden given a common condition, adding that condition to each menu item is not very efficient. Instead, we can use the `Conditional` class.

The `Conditional` class expects is just a wrapper for multiple menu items that is given a condition and based on its result, it enables or disables all the menu items it wraps around.

```Python title="Basic usage" hl_lines="15-22"
ViewMenu(
    VsechnyJednotky(),
    SpacerMenuItem(enabled=self.referral_dum is not None),
    CreateLink(
        'byty:byty-create',
        query_params={'referraldumid': referraldumid},
        enabled=self.referral_dum is not None
    ),
    SpacerMenuItem(),
    DumFilters(self.request, Domy.objects.account_can_see(self.request.user))
)
# vs
ViewMenu(
    VsechnyJednotky(),
    Conditional(
        self.referral_dum is not None, # (1)!
        SpacerMenuItem(),
        CreateLink(
            'byty:byty-create',
            query_params={'referraldumid': referraldumid}
        )
    ),
    SpacerMenuItem(),
    DumFilters(self.request, Domy.objects.account_can_see(self.request.user))
)
```

1. The condition that needs to be met for the menu items to be enabled.

The `Conditional` class also has a `otherwise()` function that takes in a list of menu items that will be enabled if the condition is not met.

```Python title="Using otherwise"
Conditional(condition, menu_item1, menu_item2).otherwise(menu_item3, menu_item4)
```

!!! warning "No conditionals in `otherwise()`"

    The `otherwise()` function does not take any conditionals. It is assumed that if the condition is not met, the menu items passed to the `otherwise()` function will be enabled.

## Collapsibles

The menu system has support for collapsible sections. It is a way to group menu items together and hide them behind a toggle button.

### Toggle button/link

```Python
toggle = CollapsibleSectionTitle('Sestavy', 'sestavyCollapse') # (1)!
# or
toggle = CollapsibleSectionToggle('Sestavy', 'sestavyCollapse') # (2)!
```

1. The title of the collapsible section.
2. Just a simple link that toggles the collapsible section.

### CollapsibleSection

```Python
section = CollapsibleSection(
    'sestavyCollapse', # (1)!
    LinkMenuItem('Sestava jednotky', 'byty:sestava-byty', query_params=query),
    LinkMenuItem('Sestava služby', 'sluzby:sestava-sluzby', query_params=query),
    LinkMenuItem('Sestava kategorie zálohy', 'sluzby:sestava-kategorie-zaloh',
                 query_params=query),
    LinkMenuItem('Sestava definice záloh', 'sluzby:sestava-zalohy-definice',
                 query_params=query),
    LinkMenuItem('Sestava platby záloh', 'sluzby:sestava-zalohy-platby',
                 query_params=query),
    LinkMenuItem('Sestava odečty', 'sluzby:sestava-odecty', query_params=query),
    LinkMenuItem('Sestava faktury', 'sluzby:sestava-faktury', query_params=query),
    LinkMenuItem('Sestava vlastníci', 'byty:sestava-byty-vlastnici', query_params=query),
    LinkMenuItem('Sestava nájemníci', 'byty:sestava-byty-najemnici', query_params=query),
)
```

1. The `collapse_id`. Must match with the `collapse_id` of the toggle button.

## Creating a custom menu item

The `MenuItem` class has an empty function called `html` that needs to be implemented in the concrete classes. The function is responsible for rendering the menu item.

As an example, let's take the spacer menu item. It is a simple menu item that renders a number of `<br>` tags.

```Python title="The spacer menu item implementation"
class SpacerMenuItem(MenuItem):
    def __init__(self, units: int = 1, enabled: bool = True) -> None:
        super().__init__(enabled)
        self.units = units

    def html(self):
        return '<br>' * self.units
```

Because the `extra_css` argument in the `MenuItem` class default to `None`, we only need to pass in the `enabled` parameter if we wish to be able to control that from the `SpacerMenuItem` class. If our custom menu item for whatever reason was never going to be disabled, we could omit the `enabled` parameter altogether and just call the parent constructor with `super().__init__()`.