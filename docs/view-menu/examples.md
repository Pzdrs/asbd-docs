---
title: Examples
description: Examples of how to use the view menu system.
---

## An empty menu

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

As we can see, the view has all three menus rendered but they are empty (well, relatively empty - more on this in the [Defaults](index.md#defaults) section). 

## A bit more capable menu

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
