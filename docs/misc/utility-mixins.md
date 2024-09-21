---
title: Utility Mixins
description: Guide to the plethora 
---

### Hotkey Mixin

### Auto Refresh Mixin

If we have a page that needs to be refreshed constantly so the the data is also refreshed this mixin can be uitlized to achieve that goal.

The `AutoRefreshMixin` defines three properties:

`refresh_enabled`
:   Can be used to control if the page will periodically refresh or not - we can dynamically set when to use the mixin and when to not.   

    Defaults to `True`.

`refresh_period`
:   This property must be set or `ImproperlyConfigured` exception is raised.

    ```Python title="Configuring the refresh period"
    class MyView(AutoRefreshMixin):
        refresh_period = 1000

    # or

    class MyView(AutoRefreshMixin):
        def get_period(self):
            return 1000
    ```

`max_refresh_cycles`
:   Defines if there is a limit to how many times we can refresh the page.

    The cycle count is stored in session storage of the browser.

    Defaults to `-1`.

### Delayed Redirect Mixin
