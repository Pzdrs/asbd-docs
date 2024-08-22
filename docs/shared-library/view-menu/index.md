---
title: View menu
description: Navigation menus
---

The view menu system on its own allows for a flexible way to define one or multiple menus for a given view directly from the view class.

By default, the system can handle up to three menus according to their position on the screen - left, right, bottom. For every menu there has to exist a corresponding function that returns the menu object (an instance of `ViewMenu` class).

The default function names are `get_left_menu()`, `get_right_menu()`, and `get_bottom_menu()`. The function names themselves can be changed by overriding the `menu_functions` dictionary (see below) in the `MenuMixin` class.

```Python
menu_functions = {
    'menu_right': 'get_right_menu',
    'menu_left': 'get_left_menu',
    'menu_bottom': 'get_bottom_menu'
}
```

The system looks for the previously mentioned functions in the view class and if they exist, their return values will be injected into the context under the corresponding key.

!!! example "An example"

    A view that has a left and a right menu needs to implement the `get_left_menu()` and `get_right_menu()` functions.
    As a result, the context will contain the `menu_left` and `menu_right` keys with the corresponding menu objects that can be rendered in the template.

!!! warning "Order of execution"

    The `get_context_data()` function in `MenuMixin` gets executed prior to the one in the view itself.

## Prerequisites

1) The view class must inherit from the `MenuMixin` class.

```Python title="views.py"
class MyView(MenuMixin, View):
    pass
```

2) For every menu, the view class must implement a corresponding function that returns the menu object.

The function names are defined in the `menu_functions` dictionary in the `MenuMixin` class.

```Python title="views.py"
class MyView(MenuMixin, View):
    def get_left_menu(self):
        return ViewMenu(
            ...
        )

    def get_right_menu(self):
        return ViewMenu(
            ...
        )
```

3) The menu object needs to be rendered in the template.

A custom template tag is used to render the menus.

```HTML title="template.html"
{% if menu_left %}
    {% render_menu menu_left %}
{% endif %}

{% if menu_right %}
    {% render_menu menu_right %}
{% endif %}
```

In case of ASBD, the `asbd_application_base_template.html` takes care of this and is also prepared to handle three menu positions.

## Examples

### An empty menu

As an example, let's take the `SluzbyDefiniceListingSearchView` view from the `sluzby` application. The class is stripped of anything that is not relevant to the menu system so as not to clutter the example.

```Python title="sluzby/views.py"
class SluzbyDefiniceListingSearchView(LoginRequiredMixin,
                                      DjangoPuzzleMixinWrapper,
                                      GetPostParametersMixin,
                                      UserConfiguredDefaultPaginationMixin,
                                      HotkeyMixin,
                                      ListView):
    def get_right_menu(self):
        return ViewMenu()

    def get_left_menu(self):
        return ViewMenu()

    def get_bottom_menu(self):
        return ViewMenu()
```

The view class has all three menu functions implemented. All of them simply return an empty `ViewMenu` object which is the bare minimum required to render a menu.

!!! info "Return value"

    The function must return a `ViewMenu` object. If the function returns `None`, the menu will not be rendered.

![alt text](<all_views_empty.png>)

As we can see, the view has all three menus rendered but they are empty (well, relatively empty - more on this further down). The `ViewMenu` object has several defaults:

If no menu title is provided, it defaults to 'Dostupné akce...'

Furthermore, a menu is rendered using a template. The default template to be used is the `default_view_menu_template.html`. Even though a specific template can be provided, it is not necessary in most cases.

```HTML title="snippets/default_view_menu_template.html"
{% load static %}

<!-- KONTEXTOVE ACTION MENU -->
<div class='container-fluid mb-2'>
    <!-- ZOBRAZ HLAVICKU -->
    <div class='row mt-2 mb-2'>
        <div class='col-12'>
            <b>{{ title }}</b>
        </div>
    </div>
    <!-- LINK LIST -->
    {% for menu_item in menu_items %}
        {% if menu_item.enabled %}
            <div class='row'>
                <div class='col-12'>
                    {{ menu_item.html|safe }}
                </div>
            </div>
        {% endif %}
    {% endfor %}
</div>
```

Lastly, every `ViewMenu` has a default set of items that are prepended to the menu by default. In case we wanted to omit them, we can set the `default_items` parameter to `False`.

```Python title="Default items"
__default_items = [
    LinkMenuItem('Krok zpět...', 'javascript:window.history.back()', reversible=False),
    SpacerMenuItem()
]
```

### A bit more capable menu

Let's now focus on creating an actual useful menu (we will stick with the right menu only). We have the following requirements:

1) A link to create a new object.

2) A *dům* filter.

- If the `referraldumid` query parameter is present, hide the filter and add a link back to the listing without the filter.

```Python title="sluzby/views.py"
def get_right_menu(self):
    return ViewMenu(
        # A link back to the listing without the filter conditionally displayed based on the value of the query parameter.
        Conditional(
            self.referral_dum,
            VsechnySluzby(), # (1)!
            SpacerMenuItem()
        ), # (2)!
        # A link to create a new object.
        CreateLink('sluzby:sluzbydefinice-create', 'Přidat novou službu', query_params=self.query_params),
        # A spacer to separate the links.
        SpacerMenuItem(),
        # The filter.
        DumFilters(self.request) # (3)!
    )
```

1. The `VsechnySluzby` class is just a simple link that points to the listing view without any filters.
2. The `Conditional` class is used to conditionally display the `VsechnySluzby` link. The `self.referral_dum` property has a value if the query parameter was set - thus the link is only displayed if the property is not `None`.
3. The `DumFilters` class is just a wrapper around the `Filter` class that is specifically tailored for filtering by *dům*.

![alt text](<single_complex_menu.png>)
