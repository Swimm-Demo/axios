---
title: Axios Utility Functions
---
# Introduction

This document explains the main ideas behind the implementation of utility functions in <SwmPath>[lib/utils.js](lib/utils.js)</SwmPath> for Axios. The file provides type checks, object manipulation helpers, and environment-agnostic utilities. The design choices focus on:

1. Why type detection is handled with custom helpers instead of relying on <SwmToken path="lib/utils.js" pos="263:12:12" line-data="  if (obj === null || typeof obj === &#39;undefined&#39;) {">`typeof`</SwmToken>.
2. How object merging and extension are implemented and why.
3. How iteration over objects and arrays is unified.
4. Why certain helpers (like <SwmToken path="lib/utils.js" pos="48:2:2" line-data="function isBuffer(val) {">`isBuffer`</SwmToken>, <SwmToken path="lib/utils.js" pos="130:2:2" line-data="const isPlainObject = (val) =&gt; {">`isPlainObject`</SwmToken>, <SwmToken path="lib/utils.js" pos="146:2:2" line-data="const isEmptyObject = (val) =&gt; {">`isEmptyObject`</SwmToken>) are more defensive than typical implementations.
5. How the file handles cross-environment compatibility for globals and async scheduling.

# Type detection: Why not just use <SwmToken path="lib/utils.js" pos="263:12:12" line-data="  if (obj === null || typeof obj === &#39;undefined&#39;) {">`typeof`</SwmToken>?

<SwmSnippet path="/lib/utils.js" line="11">

---

JavaScript's <SwmToken path="lib/utils.js" pos="263:12:12" line-data="  if (obj === null || typeof obj === &#39;undefined&#39;) {">`typeof`</SwmToken> is unreliable for many types (e.g., arrays, null, buffers). The file uses a cached approach to get the internal `[[Class]]` string and normalizes it for consistent type checks. This avoids false positives and negatives.

```javascript
const kindOf = (cache => thing => {
    const str = toString.call(thing);
    return cache[str] || (cache[str] = str.slice(8, -1).toLowerCase());
})(Object.create(null));

const kindOfTest = (type) => {
  type = type.toLowerCase();
  return (thing) => kindOf(thing) === type
}
```

---

</SwmSnippet>

This approach is used to build specific type checkers, like for arrays, buffers, and more. For example, <SwmToken path="lib/utils.js" pos="724:1:1" line-data="  isArrayBuffer,">`isArrayBuffer`</SwmToken>, <SwmToken path="lib/utils.js" pos="739:1:1" line-data="  isDate,">`isDate`</SwmToken>, and others are all built on top of this.

# Defensive type checks for edge cases

<SwmSnippet path="/lib/utils.js" line="41">

---

Some types, like Buffers, need more careful detection, especially in Node.js. The implementation checks for the presence of the constructor and its <SwmToken path="lib/utils.js" pos="48:2:2" line-data="function isBuffer(val) {">`isBuffer`</SwmToken> method, and ensures the value is not undefined or null.

```javascript
/**
 * Determine if a value is a Buffer
 *
 * @param {*} val The value to test
 *
 * @returns {boolean} True if value is a Buffer, otherwise false
 */
function isBuffer(val) {
  return val !== null && !isUndefined(val) && val.constructor !== null && !isUndefined(val.constructor)
    && isFunction(val.constructor.isBuffer) && val.constructor.isBuffer(val);
}
```

---

</SwmSnippet>

This avoids errors when dealing with objects that might look like Buffers but aren't, or when running in environments where Buffer isn't defined.

# Plain object detection

<SwmSnippet path="/lib/utils.js" line="123">

---

Not all objects are plain objects (created by `{}` or <SwmToken path="lib/utils.js" pos="474:5:5" line-data=" * Returns new array from array like object or null if failed">`new`</SwmToken>` Object()`). The utility checks the prototype chain and ensures the object doesn't have special symbols, which is important for Axios to distinguish between plain data and special objects (like <SwmToken path="lib/utils.js" pos="206:15:15" line-data=" * Determine if a value is a FormData">`FormData`</SwmToken>, Streams, etc).

```javascript
/**
 * Determine if a value is a plain Object
 *
 * @param {*} val The value to test
 *
 * @returns {boolean} True if value is a plain Object, otherwise false
 */
const isPlainObject = (val) => {
  if (kindOf(val) !== 'object') {
    return false;
  }

  const prototype = getPrototypeOf(val);
  return (prototype === null || prototype === Object.prototype || Object.getPrototypeOf(prototype) === null) && !(toStringTag in val) && !(iterator in val);
}
```

---

</SwmSnippet>

# Empty object detection with buffer safety

<SwmSnippet path="/lib/utils.js" line="139">

---

Checking if an object is empty can throw errors for certain types (like Buffers). The implementation short-circuits for <SwmToken path="lib/utils.js" pos="147:9:11" line-data="  // Early return for non-objects or Buffers to prevent RangeError">`non-objects`</SwmToken> and Buffers, and uses a try-catch to avoid RangeErrors.

```javascript
/**
 * Determine if a value is an empty object (safely handles Buffers)
 *
 * @param {*} val The value to test
 *
 * @returns {boolean} True if value is an empty object, otherwise false
 */
const isEmptyObject = (val) => {
  // Early return for non-objects or Buffers to prevent RangeError
  if (!isObject(val) || isBuffer(val)) {
    return false;
  }
  
  try {
    return Object.keys(val).length === 0 && Object.getPrototypeOf(val) === Object.prototype;
  } catch (e) {
    // Fallback for any other objects that might cause RangeError with Object.keys()
    return false;
  }
}
```

---

</SwmSnippet>

# Unified iteration for arrays and objects

<SwmSnippet path="/lib/utils.js" line="246">

---

Instead of writing separate loops for arrays and objects, the file provides a single <SwmToken path="lib/utils.js" pos="261:2:2" line-data="function forEach(obj, fn, {allOwnKeys = false} = {}) {">`forEach`</SwmToken> that handles both. It also skips Buffers and can optionally include non-enumerable properties.

```javascript
/**
 * Iterate over an Array or an Object invoking a function for each item.
 *
 * If `obj` is an Array callback will be called passing
 * the value, index, and complete array for each item.
 *
 * If 'obj' is an Object callback will be called passing
 * the value, key, and complete object for each property.
 *
 * @param {Object|Array} obj The object to iterate
 * @param {Function} fn The callback to invoke for each item
 *
 * @param {Boolean} [allOwnKeys = false]
 * @returns {any}
 */
function forEach(obj, fn, {allOwnKeys = false} = {}) {
  // Don't bother if no value provided
  if (obj === null || typeof obj === 'undefined') {
    return;
  }

  let i;
  let l;

  // Force an array if not already something iterable
  if (typeof obj !== 'object') {
    /*eslint no-param-reassign:0*/
    obj = [obj];
  }

  if (isArray(obj)) {
    // Iterate over array values
    for (i = 0, l = obj.length; i < l; i++) {
      fn.call(null, obj[i], i, obj);
    }
  } else {
    // Buffer check
    if (isBuffer(obj)) {
      return;
    }

    // Iterate over object keys
    const keys = allOwnKeys ? Object.getOwnPropertyNames(obj) : Object.keys(obj);
    const len = keys.length;
    let key;

    for (i = 0; i < len; i++) {
      key = keys[i];
      fn.call(null, obj[key], key, obj);
    }
  }
}
```

---

</SwmSnippet>

This makes it easier to write generic code that works with both arrays and objects, and avoids subtle bugs.

# Deep merging with context awareness

<SwmSnippet path="/lib/utils.js" line="325">

---

Merging objects is common in Axios (e.g., merging configs). The <SwmToken path="lib/utils.js" pos="335:9:9" line-data=" * var result = merge({foo: 123}, {foo: 456});">`merge`</SwmToken> function handles deep merging, array copying, and respects case-insensitive keys if a context is provided. It uses the unified <SwmToken path="lib/utils.js" pos="360:8:8" line-data="    arguments[i] &amp;&amp; forEach(arguments[i], assignValue);">`forEach`</SwmToken> and checks for plain objects to avoid merging prototypes or special objects.

````javascript
/**
 * Accepts varargs expecting each argument to be an object, then
 * immutably merges the properties of each object and returns result.
 *
 * When multiple objects contain the same key the later object in
 * the arguments list will take precedence.
 *
 * Example:
 *
 * ```js
 * var result = merge({foo: 123}, {foo: 456});
 * console.log(result.foo); // outputs 456
 * ```
 *
 * @param {Object} obj1 Object to merge
 *
 * @returns {Object} Result of all merge properties
 */
function merge(/* obj1, obj2, obj3, ... */) {
  const {caseless} = isContextDefined(this) && this || {};
  const result = {};
  const assignValue = (val, key) => {
    const targetKey = caseless && findKey(result, key) || key;
    if (isPlainObject(result[targetKey]) && isPlainObject(val)) {
      result[targetKey] = merge(result[targetKey], val);
    } else if (isPlainObject(val)) {
      result[targetKey] = merge({}, val);
    } else if (isArray(val)) {
      result[targetKey] = val.slice();
    } else {
      result[targetKey] = val;
    }
  }

  for (let i = 0, l = arguments.length; i < l; i++) {
    arguments[i] && forEach(arguments[i], assignValue);
  }
  return result;
}
````

---

</SwmSnippet>

# Cross-environment global detection

<SwmSnippet path="/lib/utils.js" line="317">

---

Axios needs to run in browsers, Node.js, and web workers. The file provides a <SwmToken path="lib/utils.js" pos="317:2:2" line-data="const _global = (() =&gt; {">`_global`</SwmToken> variable that always points to the right global object, regardless of environment.

```javascript
const _global = (() => {
  /*eslint no-undef:0*/
  if (typeof globalThis !== "undefined") return globalThis;
  return typeof self !== "undefined" ? self : (typeof window !== 'undefined' ? window : global)
})();

const isContextDefined = (context) => !isUndefined(context) && context !== _global;
```

---

</SwmSnippet>

This is used throughout the codebase to avoid environment-specific bugs.

# Async scheduling: <SwmToken path="lib/utils.js" pos="693:3:3" line-data="    return setImmediate;">`setImmediate`</SwmToken>, <SwmToken path="lib/utils.js" pos="705:3:3" line-data="      _global.postMessage(token, &quot;*&quot;);">`postMessage`</SwmToken>, <SwmToken path="lib/utils.js" pos="714:26:26" line-data="  queueMicrotask.bind(_global) : ( typeof process !== &#39;undefined&#39; &amp;&amp; process.nextTick || _setImmediate);">`nextTick`</SwmToken>, <SwmToken path="lib/utils.js" pos="713:8:8" line-data="const asap = typeof queueMicrotask !== &#39;undefined&#39; ?">`queueMicrotask`</SwmToken>

<SwmSnippet path="/lib/utils.js" line="691">

---

Different environments have different ways to schedule microtasks. The file provides a unified <SwmToken path="lib/utils.js" pos="713:2:2" line-data="const asap = typeof queueMicrotask !== &#39;undefined&#39; ?">`asap`</SwmToken> function that uses the best available method, falling back as needed.

```javascript
const _setImmediate = ((setImmediateSupported, postMessageSupported) => {
  if (setImmediateSupported) {
    return setImmediate;
  }

  return postMessageSupported ? ((token, callbacks) => {
    _global.addEventListener("message", ({source, data}) => {
      if (source === _global && data === token) {
        callbacks.length && callbacks.shift()();
      }
    }, false);

    return (cb) => {
      callbacks.push(cb);
      _global.postMessage(token, "*");
    }
  })(`axios@${Math.random()}`, []) : (cb) => setTimeout(cb);
})(
  typeof setImmediate === 'function',
  isFunction(_global.postMessage)
);

const asap = typeof queueMicrotask !== 'undefined' ?
  queueMicrotask.bind(_global) : ( typeof process !== 'undefined' && process.nextTick || _setImmediate);

// *********************


const isIterable = (thing) => thing != null && isFunction(thing[iterator]);
```

---

</SwmSnippet>

This ensures Axios can schedule tasks efficiently everywhere.

# Export structure

<SwmSnippet path="/lib/utils.js" line="722">

---

All helpers are exported as a single object, making them easy to import and use elsewhere in Axios.

```javascript
export default {
  isArray,
  isArrayBuffer,
  isBuffer,
  isFormData,
  isArrayBufferView,
  isString,
  isNumber,
  isBoolean,
  isObject,
  isPlainObject,
  isEmptyObject,
  isReadableStream,
  isRequest,
  isResponse,
  isHeaders,
  isUndefined,
  isDate,
  isFile,
  isBlob,
  isRegExp,
  isFunction,
  isStream,
  isURLSearchParams,
  isTypedArray,
  isFileList,
  forEach,
  merge,
  extend,
  trim,
  stripBOM,
  inherits,
  toFlatObject,
  kindOf,
  kindOfTest,
  endsWith,
  toArray,
  forEachEntry,
  matchAll,
  isHTMLForm,
  hasOwnProperty,
  hasOwnProp: hasOwnProperty, // an alias to avoid ESLint no-prototype-builtins detection
  reduceDescriptors,
  freezeMethods,
  toObjectSet,
  toCamelCase,
  noop,
  toFiniteNumber,
  findKey,
  global: _global,
  isContextDefined,
  isSpecCompliantForm,
  toJSONObject,
  isAsyncFn,
  isThenable,
  setImmediate: _setImmediate,
  asap,
  isIterable
};
```

---

</SwmSnippet>

This design keeps the codebase DRY and consistent, and makes it easy to add or update utilities in one place.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBYXhpb3MlM0ElM0FTd2ltbS1EZW1v" repo-name="axios"><sup>Powered by [Swimm](/)</sup></SwmMeta>
