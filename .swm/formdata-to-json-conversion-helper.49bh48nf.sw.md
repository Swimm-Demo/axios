---
title: FormData to JSON Conversion Helper
---
# introduction

This document explains the rationale and main points behind the implementation of a helper that converts <SwmToken path="lib/helpers/formDataToJSON.js" pos="43:9:9" line-data=" * It takes a FormData object and returns a JavaScript object">`FormData`</SwmToken> into a JSON object. We will cover:

1. How the property paths from <SwmToken path="lib/helpers/formDataToJSON.js" pos="43:9:9" line-data=" * It takes a FormData object and returns a JavaScript object">`FormData`</SwmToken> keys are parsed.
2. How arrays and objects are constructed from these paths.
3. How the recursive building of the final JSON structure works.
4. How edge cases like repeated keys and prototype pollution are handled.

# parsing form data keys into property paths

<SwmToken path="lib/helpers/formDataToJSON.js" pos="43:9:9" line-data=" * It takes a FormData object and returns a JavaScript object">`FormData`</SwmToken> keys can represent nested structures using syntax like <SwmToken path="lib/helpers/formDataToJSON.js" pos="6:14:23" line-data=" * It takes a string like `foo[x][y][z]` and returns an array like `[&#39;foo&#39;, &#39;x&#39;, &#39;y&#39;, &#39;z&#39;]">`foo[x][y][z]`</SwmToken>. To convert these keys into a usable form, the code parses them into arrays of property names or indices. This is done by matching word characters or bracketed segments and normalizing them into a flat array of strings or empty strings for array indices.

<SwmSnippet path="/lib/helpers/formDataToJSON.js" line="1">

---

This parsing step is crucial because it transforms complex keys into a sequence that can be used to build nested objects or arrays programmatically.

```javascript
'use strict';

import utils from '../utils.js';

/**
 * It takes a string like `foo[x][y][z]` and returns an array like `['foo', 'x', 'y', 'z']
 *
 * @param {string} name - The name of the property to get.
 *
 * @returns An array of strings.
 */
function parsePropPath(name) {
  // foo[x][y][z]
  // foo.x.y.z
  // foo-x-y-z
  // foo x y z
  return utils.matchAll(/\w+|\[(\w*)]/g, name).map(match => {
    return match[0] === '[]' ? '' : match[1] || match[0];
  });
}
```

---

</SwmSnippet>

# converting arrays to objects for sparse arrays

<SwmSnippet path="/lib/helpers/formDataToJSON.js" line="22">

---

When building the JSON structure, some arrays may become sparse or have non-numeric keys. To handle this, the code includes a utility that converts an array into an object by copying all keys and values. This ensures that the final structure can represent both arrays and objects correctly, especially when keys are not continuous numeric indices.

```javascript
/**
 * Convert an array to an object.
 *
 * @param {Array<any>} arr - The array to convert to an object.
 *
 * @returns An object with the same keys and values as the array.
 */
function arrayToObject(arr) {
  const obj = {};
  const keys = Object.keys(arr);
  let i;
  const len = keys.length;
  let key;
  for (i = 0; i < len; i++) {
    key = keys[i];
    obj[key] = arr[key];
  }
  return obj;
}
```

---

</SwmSnippet>

# recursive building of the JSON structure

The core of the conversion is a recursive function that walks through the parsed property path and assigns values accordingly. It takes the current path segment, the value to assign, the target object or array, and the current index in the path.

Key points in this recursion:

- It prevents prototype pollution by ignoring any property named <SwmToken path="lib/helpers/formDataToJSON.js" pos="53:9:9" line-data="    if (name === &#39;__proto__&#39;) return true;">`__proto__`</SwmToken>.
- It distinguishes between numeric keys (array indices) and string keys (object properties).
- When it reaches the last segment of the path, it assigns the value. If the property already exists, it converts it into an array to hold multiple values.
- For intermediate segments, it ensures the target is an array or object as needed, initializing empty arrays when necessary.
- After recursion, if the target is an array but has non-numeric keys, it converts it into an object to preserve all keys.

<SwmSnippet path="/lib/helpers/formDataToJSON.js" line="42">

---

This approach allows the function to handle deeply nested structures, arrays, and repeated keys gracefully.

```javascript
/**
 * It takes a FormData object and returns a JavaScript object
 *
 * @param {string} formData The FormData object to convert to JSON.
 *
 * @returns {Object<string, any> | null} The converted object.
 */
function formDataToJSON(formData) {
  function buildPath(path, value, target, index) {
    let name = path[index++];

    if (name === '__proto__') return true;

    const isNumericKey = Number.isFinite(+name);
    const isLast = index >= path.length;
    name = !name && utils.isArray(target) ? target.length : name;

    if (isLast) {
      if (utils.hasOwnProp(target, name)) {
        target[name] = [target[name], value];
      } else {
        target[name] = value;
      }

      return !isNumericKey;
    }

    if (!target[name] || !utils.isObject(target[name])) {
      target[name] = [];
    }

    const result = buildPath(path, value, target[name], index);

    if (result && utils.isArray(target[name])) {
      target[name] = arrayToObject(target[name]);
    }

    return !isNumericKey;
  }
```

---

</SwmSnippet>

# iterating over <SwmToken path="lib/helpers/formDataToJSON.js" pos="43:9:9" line-data=" * It takes a FormData object and returns a JavaScript object">`FormData`</SwmToken> entries and assembling the final object

The main function first checks if the input is a valid <SwmToken path="lib/helpers/formDataToJSON.js" pos="43:9:9" line-data=" * It takes a FormData object and returns a JavaScript object">`FormData`</SwmToken> object with an <SwmToken path="lib/helpers/formDataToJSON.js" pos="82:19:19" line-data="  if (utils.isFormData(formData) &amp;&amp; utils.isFunction(formData.entries)) {">`entries`</SwmToken> method. It then initializes an empty object and iterates over each entry in the <SwmToken path="lib/helpers/formDataToJSON.js" pos="43:9:9" line-data=" * It takes a FormData object and returns a JavaScript object">`FormData`</SwmToken>.

For each entry, it parses the key into a path and recursively builds the nested structure inside the object. Finally, it returns the fully constructed JSON object.

<SwmSnippet path="/lib/helpers/formDataToJSON.js" line="82">

---

If the input is not a valid <SwmToken path="lib/helpers/formDataToJSON.js" pos="43:9:9" line-data=" * It takes a FormData object and returns a JavaScript object">`FormData`</SwmToken>, it returns null.

```javascript
  if (utils.isFormData(formData) && utils.isFunction(formData.entries)) {
    const obj = {};

    utils.forEachEntry(formData, (name, value) => {
      buildPath(parsePropPath(name), value, obj, 0);
    });

    return obj;
  }

  return null;
}
```

---

</SwmSnippet>

# exporting the helper

<SwmSnippet path="/lib/helpers/formDataToJSON.js" line="95">

---

The helper function is exported as the default export of the module, making it available for use elsewhere in the codebase.

```javascript
export default formDataToJSON;
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBYXhpb3MlM0ElM0FTd2ltbS1EZW1v" repo-name="axios"><sup>Powered by [Swimm](/)</sup></SwmMeta>
