---
title: Colorize Helper Function for CLI Output
---
# introduction

This document explains the rationale behind the colorize helper function in <SwmPath>[bin/helpers/colorize.js](bin/helpers/colorize.js)</SwmPath>. It answers these questions:

1. Why does the function accept a variable number of colors and provide defaults?
2. How does the function apply colors to template literal values?
3. Why use the chalk library and the chosen approach for styling?

# handling colors and defaults

The function accepts any number of color names as arguments. If none are provided, it defaults to a set of visually distinct colors: green, cyan, magenta, blue, yellow, and red. This design allows flexibility for different CLI output needs while ensuring there is always a palette to cycle through.

<SwmSnippet path="/bin/helpers/colorize.js" line="1">

---

The colors are stored and counted to enable cycling through them when applying styles. This avoids index errors and supports any number of values to be colorized.

```javascript
import chalk from 'chalk';

export const colorize = (...colors)=> {
  if(!colors.length) {
    colors = ['green', 'cyan', 'magenta', 'blue', 'yellow', 'red'];
  }

  const colorsCount = colors.length;
```

---

</SwmSnippet>

# applying colors to template literals

The function returns a tagged template literal handler. It receives the static string parts and the dynamic values separately. For each value, it applies a color from the list in a round-robin fashion, using modulo arithmetic to cycle through colors if there are more values than colors.

<SwmSnippet path="/bin/helpers/colorize.js" line="10">

---

Each value is styled with bold formatting using chalk before being concatenated back with the surrounding strings. This approach cleanly separates styling logic from string construction and leverages template literals for readable syntax.

```javascript
  return (strings, ...values) => {
    const {length} = values;
    return strings.map((str, i) => i < length ? str + chalk[colors[i%colorsCount]].bold(values[i]) : str).join('');
  }
}
```

---

</SwmSnippet>

# using chalk for styling

Chalk is used because it provides a simple, chainable API for terminal string styling. Accessing colors dynamically via <SwmToken path="bin/helpers/colorize.js" pos="1:2:2" line-data="import chalk from &#39;chalk&#39;;">`chalk`</SwmToken>`[`<SwmToken path="bin/helpers/colorize.js" pos="3:10:10" line-data="export const colorize = (...colors)=&gt; {">`colors`</SwmToken>`[i]]` allows the function to support any valid chalk color without hardcoding.

Bold styling is applied consistently to make the colored values stand out in CLI output. This choice improves readability and highlights important dynamic parts of messages.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBYXhpb3MlM0ElM0FTd2ltbS1EZW1v" repo-name="axios"><sup>Powered by [Swimm](/)</sup></SwmMeta>
