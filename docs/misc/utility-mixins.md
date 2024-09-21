---
title: Utility Mixins
description: Guide to the plethora 
---

### Hotkey Mixin

The `HotkeyMixin` allows us to use keyboard shortcuts in the browser using JavaScript. On the backend, we define what shortcuts do what actions and that information is handed of to the JavaScript on the frontend which takes care of detecting occurances and performing the predefined actions.

Currently, a keyboard shortcut can either perform a **redirection** or **run a piece of JavaScript code**.

The `HotkeyMixin` defines a singular property:

`hotkeys_enabled`
:   Allows us to enable/disable the hotkey funcionality.

    Defaults to `True`.

#### Usage

First, we need to extend our view with the `HotkeyMixin` class and provide our hotkeys by overriding the `get_hotkeys()` function. The original implementaion returns an empty list so not overriding it will not raise any exceptions.

The `get_hotkeys()` function needs to return a list of `Hotkey` instances. The `Hotkey` class itself is abstract so it's children must be used.

The key combinations are represented as tuples of individual keys.

```Python title="Example key combination constant"
KATEGORIEZALOHY_LISTING_ADD_NEW_HOT_KEY = ('CTRL', 'ALT', 'N')
```

```Python
class MyView(HotkeyMixin):
    def get_hotkeys(self):
        return [
            RedirectHotkey(('ALT', 'P'), reverse('app:view_name'))
        ]
```

#### Creating custom hotkey classes

The base `Hotkey` class is abstract so it cannot be used on its own.

The constructor only accepts the key combination as a tuple.

Avery class extending the base `Hotkey` class must implement the `serialize()` function which will be unique to every type of hotkey.

```Python title="The base class implementation"
class Hotkey(ABC):
    def __init__(self, combination: tuple) -> None:
        self.combination = combination

    @property
    def serialized_combination(self) -> str:
        return '+'.join(self.combination)

    @abstractmethod
    def serialize(self) -> tuple:
        pass
```

The most commonly used hotkey is the `RedirectHotkey` which upon triggering simply redirects the user to a predefined page.

We can see that the constructor now also accepts a link alongside the combination.

The implementation of the `serialize()` function returns a tuple containing the serialized key combination and a dictionary that is specifically structured so the JavaScript on the other side knows what to do with it.

```Python title="RedirectHotkey class implementation"
class RedirectHotkey(Hotkey):
    def __init__(self, combination: tuple, link: str) -> None:
        super().__init__(combination)
        self.link = link

    def serialize(self) -> tuple:
        return self.serialized_combination, {'link': self.link}
```
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

If we need the user to be redirected somewhere after loading a page, we can use the `DelayedRedirectMixin`.

For example, after a user logged in, we display a welcome page and after 5 seconds we redirect them somewhere (anywhere, doesn't matter).

The `AutoRefreshMixin` defines three properties:

`redirect_enabled`
:   Can be used to control if the page will perform the redirect after the configured period - we can dynamically set when to use the mixin and when to not.   

    Defaults to `True`.

`redirect_link`
:   This property must be set or `ImproperlyConfigured` exception is raised.

    The link to redirect the user to after a given period after loding the page.

`redirect_delay`
:   This property must be set or `ImproperlyConfigured` exception is raised.

    How long to wait before performing the redirection.

```Python title="Usage"
class MyView(DelayedRedirectMixin):
    redirect_delay = 5000
    redirect_link = reverse_lazy('app:view_name')

# or

class MyView(DelayedRedirectMixin):
    def get_redirect_to(self):
        return reverse('app:view_name')
    
    def get_delay(self):
        return 5000
```