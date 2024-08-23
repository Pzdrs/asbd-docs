---
title: Styleable components
description: Styleable components
---

!!! note "Depracation notice"
    Currently, styleable components use inheritance. In the future, this will be replaced with composition.

    ```Python
    class Component(StyleableComponent):
        def __init__(self, custom_css: StyleModifier):
            super().__init__(custom_css)

    # to be replaced with composition

    class Commponent:
        def __init__(self, custom_css: StyleModifier):
            self.custom_css = custom_css
    ```

When it comes to generating HTML in code, it is often necessary to apply CSS to the generated elements. For this reason, the shared library provides the `StyleModifier` class.

The `StyleModifier` class encapsulates an arbitrary number of CSS classes and also individual CSS properties. The `get_attributes()` function can then be used to generate the HTML attributes for the element.

!!! example "Generated attributes"
    ```Python
    StyleModifier(('class1', 'class2'), {'color':'blue'}).get_attributes()
    ```
    
    If this style modifier were to be applied to a `div` element, the resulting HTML would look like this:
    
    ```HTML
    <div class="class1 class2" style="color: blue;"></div>
    ```

The `get_attributes()` function pretty much just concatenates the classes and properties into a string (as shown in the code block below). So, if for some reason you need to get the classes and properties separately, you can use the `get_class_attribute()` and `get_style_attribute()` functions. 

```Python title="Implementation of get_attributes()"
def get_attributes(self):
    return f'{self.get_class_attribute()}{self.get_style_attribute()}'
```

The `StyleModifier` also has an extension functionality, which allows for combination of different style modifier objects.

```Python title="Combining style modifiers"
StyleModifier(styles={'text-align': 'right'}).extend(StyleModifier(('class1',)))
```

## An example

```Python title="View"
def get_context_data(self):
    return {
        'style': StyleModifier(('class1', 'class2'), {'color':'blue'}),
        **super().get_context_data()
    }
```


```HTML title="Template"
<div {{ style.get_class_attribute() }} {{ style.get_style_attribute() }}>Hello, world!</div>
<!-- or use the convenient concatenation -->
<div {{ style.get_attributes() }}>Hello, world!</div>
```

```HTML title="Result"
<div class="class1 class2" style="color: blue;">Hello, world!</div>
```