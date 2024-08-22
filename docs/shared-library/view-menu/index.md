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
