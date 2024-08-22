---
title: Menu items
description: Comprehensive guide to menu items
---

The main building blocks of the menu system are the menu items. They are used to define the structure of the menu and can be nested to create complex menus.

At the top of the hierarchy stands the `MenuItem` class. It is an abstract class that is not meant to be used directly. Instead, there are several concrete classes that inherit from it and provide actual functionality.

A menu item can be enabled or disabled using the `enabled` property. By default, all menu items are enabled.

The `MenuItem` class is a `StyleableComponent` and as such, using the `custom_css` property, it is possible to apply custom CSS classes to the menu item. More information about styling can be found on the [Styling](/shared-library/styleable-components/) page.

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

## CollapsibleSectionToggle

## CollapsibleSection

## Conditional

:  Described in it's own dedicated [section](#filters).

## Filters

## Conditionals

## Collapsibles

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