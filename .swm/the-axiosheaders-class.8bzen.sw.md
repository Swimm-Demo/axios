---
title: The AxiosHeaders class
---
This document covers the <SwmToken path="lib/core/AxiosHeaders.js" pos="74:2:2" line-data="class AxiosHeaders {">`AxiosHeaders`</SwmToken> class and its API. We'll address:

1. What <SwmToken path="lib/core/AxiosHeaders.js" pos="74:2:2" line-data="class AxiosHeaders {">`AxiosHeaders`</SwmToken> is and its purpose
2. All variables and functions defined in <SwmToken path="lib/core/AxiosHeaders.js" pos="74:2:2" line-data="class AxiosHeaders {">`AxiosHeaders`</SwmToken>, with code citations for each

# What is <SwmToken path="lib/core/AxiosHeaders.js" pos="74:2:2" line-data="class AxiosHeaders {">`AxiosHeaders`</SwmToken>

<SwmToken path="lib/core/AxiosHeaders.js" pos="74:2:2" line-data="class AxiosHeaders {">`AxiosHeaders`</SwmToken> is a class that provides a structured and normalized way to manage HTTP headers within Axios. It is designed to handle header names and values in a case-insensitive manner, support multiple formats for setting and retrieving headers, and offer utility methods for common header operations such as normalization, merging, and serialization. <SwmToken path="lib/core/AxiosHeaders.js" pos="74:2:2" line-data="class AxiosHeaders {">`AxiosHeaders`</SwmToken> is used throughout Axios to ensure consistent header handling in both browser and Node.js environments.

<SwmSnippet path="/lib/core/AxiosHeaders.js" line="79">

---

The function <SwmToken path="lib/core/AxiosHeaders.js" pos="79:1:1" line-data="  set(header, valueOrRewrite, rewrite) {">`set`</SwmToken> allows you to set one or more headers on the <SwmToken path="lib/core/AxiosHeaders.js" pos="74:2:2" line-data="class AxiosHeaders {">`AxiosHeaders`</SwmToken> instance. It supports various input formats, including plain objects, iterables, and header strings. The method normalizes header names and values, and can optionally overwrite existing values. It returns the <SwmToken path="lib/core/AxiosHeaders.js" pos="74:2:2" line-data="class AxiosHeaders {">`AxiosHeaders`</SwmToken> instance for chaining.

```javascript
  set(header, valueOrRewrite, rewrite) {
    const self = this;

    function setHeader(_value, _header, _rewrite) {
      const lHeader = normalizeHeader(_header);

      if (!lHeader) {
        throw new Error('header name must be a non-empty string');
      }

      const key = utils.findKey(self, lHeader);

      if(!key || self[key] === undefined || _rewrite === true || (_rewrite === undefined && self[key] !== false)) {
        self[key || _header] = normalizeValue(_value);
      }
    }

    const setHeaders = (headers, _rewrite) =>
      utils.forEach(headers, (_value, _header) => setHeader(_value, _header, _rewrite));

    if (utils.isPlainObject(header) || header instanceof this.constructor) {
      setHeaders(header, valueOrRewrite)
    } else if(utils.isString(header) && (header = header.trim()) && !isValidHeaderName(header)) {
      setHeaders(parseHeaders(header), valueOrRewrite);
    } else if (utils.isObject(header) && utils.isIterable(header)) {
      let obj = {}, dest, key;
      for (const entry of header) {
        if (!utils.isArray(entry)) {
          throw TypeError('Object iterator must return a key-value pair');
        }

        obj[key = entry[0]] = (dest = obj[key]) ?
          (utils.isArray(dest) ? [...dest, entry[1]] : [dest, entry[1]]) : entry[1];
      }

      setHeaders(obj, valueOrRewrite)
    } else {
      header != null && setHeader(valueOrRewrite, header, rewrite);
    }

    return this;
  }
```

---

</SwmSnippet>

<SwmSnippet path="/lib/core/AxiosHeaders.js" line="122">

---

The function <SwmToken path="lib/core/AxiosHeaders.js" pos="122:1:1" line-data="  get(header, parser) {">`get`</SwmToken> retrieves the value of a header by name. It supports an optional parser argument, which can be a boolean, function, or regular expression to process the header value. If the parser is true, it parses <SwmToken path="lib/core/AxiosHeaders.js" pos="107:16:18" line-data="          throw TypeError(&#39;Object iterator must return a key-value pair&#39;);">`key-value`</SwmToken> pairs from the header string. If a function or RegExp is provided, it applies them to the value.

```javascript
  get(header, parser) {
    header = normalizeHeader(header);

    if (header) {
      const key = utils.findKey(this, header);

      if (key) {
        const value = this[key];

        if (!parser) {
          return value;
        }

        if (parser === true) {
          return parseTokens(value);
        }

        if (utils.isFunction(parser)) {
          return parser.call(this, value, key);
        }

        if (utils.isRegExp(parser)) {
          return parser.exec(value);
        }

        throw new TypeError('parser must be boolean|regexp|function');
      }
    }
  }
```

---

</SwmSnippet>

<SwmSnippet path="/lib/core/AxiosHeaders.js" line="152">

---

The function <SwmToken path="lib/core/AxiosHeaders.js" pos="152:1:1" line-data="  has(header, matcher) {">`has`</SwmToken> checks if a header exists in the <SwmToken path="lib/core/AxiosHeaders.js" pos="74:2:2" line-data="class AxiosHeaders {">`AxiosHeaders`</SwmToken> instance. It can also accept a matcher to further filter the check based on the header value.

```javascript
  has(header, matcher) {
    header = normalizeHeader(header);

    if (header) {
      const key = utils.findKey(this, header);

      return !!(key && this[key] !== undefined && (!matcher || matchHeaderValue(this, this[key], key, matcher)));
    }

    return false;
  }
```

---

</SwmSnippet>

<SwmSnippet path="/lib/core/AxiosHeaders.js" line="164">

---

The function <SwmToken path="lib/core/AxiosHeaders.js" pos="164:1:1" line-data="  delete(header, matcher) {">`delete`</SwmToken> removes one or more headers from the <SwmToken path="lib/core/AxiosHeaders.js" pos="74:2:2" line-data="class AxiosHeaders {">`AxiosHeaders`</SwmToken> instance. It supports deleting by header name or an array of names, and can use a matcher to conditionally delete headers based on their values.

```javascript
  delete(header, matcher) {
    const self = this;
    let deleted = false;

    function deleteHeader(_header) {
      _header = normalizeHeader(_header);

      if (_header) {
        const key = utils.findKey(self, _header);

        if (key && (!matcher || matchHeaderValue(self, self[key], key, matcher))) {
          delete self[key];

          deleted = true;
        }
      }
    }

    if (utils.isArray(header)) {
      header.forEach(deleteHeader);
    } else {
      deleteHeader(header);
    }

    return deleted;
  }
```

---

</SwmSnippet>

<SwmSnippet path="/lib/core/AxiosHeaders.js" line="191">

---

The function <SwmToken path="lib/core/AxiosHeaders.js" pos="191:1:1" line-data="  clear(matcher) {">`clear`</SwmToken> deletes all headers from the <SwmToken path="lib/core/AxiosHeaders.js" pos="74:2:2" line-data="class AxiosHeaders {">`AxiosHeaders`</SwmToken> instance. It can also accept a matcher to selectively clear headers that match certain criteria.

```javascript
  clear(matcher) {
    const keys = Object.keys(this);
    let i = keys.length;
    let deleted = false;

    while (i--) {
      const key = keys[i];
      if(!matcher || matchHeaderValue(this, this[key], key, matcher, true)) {
        delete this[key];
        deleted = true;
      }
    }

    return deleted;
  }
```

---

</SwmSnippet>

<SwmSnippet path="/lib/core/AxiosHeaders.js" line="207">

---

The function <SwmToken path="lib/core/AxiosHeaders.js" pos="207:1:1" line-data="  normalize(format) {">`normalize`</SwmToken> standardizes the header names in the <SwmToken path="lib/core/AxiosHeaders.js" pos="74:2:2" line-data="class AxiosHeaders {">`AxiosHeaders`</SwmToken> instance. It can format header names to a canonical form (e.g., <SwmToken path="lib/core/AxiosHeaders.js" pos="299:6:8" line-data="AxiosHeaders.accessor([&#39;Content-Type&#39;, &#39;Content-Length&#39;, &#39;Accept&#39;, &#39;Accept-Encoding&#39;, &#39;User-Agent&#39;, &#39;Authorization&#39;]);">`Content-Type`</SwmToken>) or simply trim whitespace. This helps prevent duplicate headers with different casing.

```javascript
  normalize(format) {
    const self = this;
    const headers = {};

    utils.forEach(this, (value, header) => {
      const key = utils.findKey(headers, header);

      if (key) {
        self[key] = normalizeValue(value);
        delete self[header];
        return;
      }

      const normalized = format ? formatHeader(header) : String(header).trim();

      if (normalized !== header) {
        delete self[header];
      }

      self[normalized] = normalizeValue(value);

      headers[normalized] = true;
    });

    return this;
  }
```

---

</SwmSnippet>

<SwmSnippet path="/lib/core/AxiosHeaders.js" line="234">

---

The function <SwmToken path="lib/core/AxiosHeaders.js" pos="234:1:1" line-data="  concat(...targets) {">`concat`</SwmToken> merges the current <SwmToken path="lib/core/AxiosHeaders.js" pos="74:2:2" line-data="class AxiosHeaders {">`AxiosHeaders`</SwmToken> instance with one or more other header sources. It returns a new <SwmToken path="lib/core/AxiosHeaders.js" pos="74:2:2" line-data="class AxiosHeaders {">`AxiosHeaders`</SwmToken> instance containing the combined headers.

```javascript
  concat(...targets) {
    return this.constructor.concat(this, ...targets);
  }
```

---

</SwmSnippet>

<SwmSnippet path="/lib/core/AxiosHeaders.js" line="238">

---

The function <SwmToken path="lib/core/AxiosHeaders.js" pos="238:1:1" line-data="  toJSON(asStrings) {">`toJSON`</SwmToken> serializes the headers into a plain object. Optionally, it can join array values into a comma-separated string if the <SwmToken path="lib/core/AxiosHeaders.js" pos="238:3:3" line-data="  toJSON(asStrings) {">`asStrings`</SwmToken> argument is true.

```javascript
  toJSON(asStrings) {
    const obj = Object.create(null);

    utils.forEach(this, (value, header) => {
      value != null && value !== false && (obj[header] = asStrings && utils.isArray(value) ? value.join(', ') : value);
    });

    return obj;
  }
```

---

</SwmSnippet>

<SwmSnippet path="/lib/core/AxiosHeaders.js" line="252">

---

The function <SwmToken path="lib/core/AxiosHeaders.js" pos="252:1:1" line-data="  toString() {">`toString`</SwmToken> returns a string representation of the headers, with each header on a new line in the format 'Header-Name: value'.

```javascript
  toString() {
    return Object.entries(this.toJSON()).map(([header, value]) => header + ': ' + value).join('\n');
  }
```

---

</SwmSnippet>

<SwmSnippet path="/lib/core/AxiosHeaders.js" line="256">

---

The function <SwmToken path="lib/core/AxiosHeaders.js" pos="256:1:1" line-data="  getSetCookie() {">`getSetCookie`</SwmToken> retrieves the value of the <SwmToken path="lib/core/AxiosHeaders.js" pos="257:8:10" line-data="    return this.get(&quot;set-cookie&quot;) || [];">`set-cookie`</SwmToken> header as an array. If the header is not present, it returns an empty array.

```javascript
  getSetCookie() {
    return this.get("set-cookie") || [];
  }
```

---

</SwmSnippet>

<SwmSnippet path="/lib/core/AxiosHeaders.js" line="264">

---

The static function `from` creates a new <SwmToken path="lib/core/AxiosHeaders.js" pos="74:2:2" line-data="class AxiosHeaders {">`AxiosHeaders`</SwmToken> instance from a given object or another <SwmToken path="lib/core/AxiosHeaders.js" pos="74:2:2" line-data="class AxiosHeaders {">`AxiosHeaders`</SwmToken> instance. If the input is already an <SwmToken path="lib/core/AxiosHeaders.js" pos="74:2:2" line-data="class AxiosHeaders {">`AxiosHeaders`</SwmToken> instance, it returns it directly.

```javascript
  static from(thing) {
    return thing instanceof this ? thing : new this(thing);
  }
```

---

</SwmSnippet>

<SwmSnippet path="/lib/core/AxiosHeaders.js" line="268">

---

The static function <SwmToken path="lib/core/AxiosHeaders.js" pos="268:3:3" line-data="  static concat(first, ...targets) {">`concat`</SwmToken> merges multiple header sources into a new <SwmToken path="lib/core/AxiosHeaders.js" pos="74:2:2" line-data="class AxiosHeaders {">`AxiosHeaders`</SwmToken> instance. It takes the first source and applies the set method for each additional target.

```javascript
  static concat(first, ...targets) {
    const computed = new this(first);

    targets.forEach((target) => computed.set(target));

    return computed;
  }
```

---

</SwmSnippet>

<SwmSnippet path="/lib/core/AxiosHeaders.js" line="276">

---

The static function <SwmToken path="lib/core/AxiosHeaders.js" pos="276:3:3" line-data="  static accessor(header) {">`accessor`</SwmToken> dynamically creates getter, setter, and checker methods for specified header names on the <SwmToken path="lib/core/AxiosHeaders.js" pos="74:2:2" line-data="class AxiosHeaders {">`AxiosHeaders`</SwmToken> prototype. This allows for convenient property-style access to common headers.

```javascript
  static accessor(header) {
    const internals = this[$internals] = (this[$internals] = {
      accessors: {}
    });

    const accessors = internals.accessors;
    const prototype = this.prototype;

    function defineAccessor(_header) {
      const lHeader = normalizeHeader(_header);

      if (!accessors[lHeader]) {
        buildAccessors(prototype, _header);
        accessors[lHeader] = true;
      }
    }

    utils.isArray(header) ? header.forEach(defineAccessor) : defineAccessor(header);

    return this;
  }
```

---

</SwmSnippet>

<SwmSnippet path="/lib/core/AxiosHeaders.js" line="79">

---

The function <SwmToken path="lib/core/AxiosHeaders.js" pos="79:1:1" line-data="  set(header, valueOrRewrite, rewrite) {">`set`</SwmToken> is also used internally by the constructor and other methods to assign header values, ensuring normalization and proper handling of overwrites.

```javascript
  set(header, valueOrRewrite, rewrite) {
    const self = this;

    function setHeader(_value, _header, _rewrite) {
      const lHeader = normalizeHeader(_header);

      if (!lHeader) {
        throw new Error('header name must be a non-empty string');
      }

      const key = utils.findKey(self, lHeader);

      if(!key || self[key] === undefined || _rewrite === true || (_rewrite === undefined && self[key] !== false)) {
        self[key || _header] = normalizeValue(_value);
      }
    }

    const setHeaders = (headers, _rewrite) =>
      utils.forEach(headers, (_value, _header) => setHeader(_value, _header, _rewrite));

    if (utils.isPlainObject(header) || header instanceof this.constructor) {
      setHeaders(header, valueOrRewrite)
    } else if(utils.isString(header) && (header = header.trim()) && !isValidHeaderName(header)) {
      setHeaders(parseHeaders(header), valueOrRewrite);
    } else if (utils.isObject(header) && utils.isIterable(header)) {
      let obj = {}, dest, key;
      for (const entry of header) {
        if (!utils.isArray(entry)) {
          throw TypeError('Object iterator must return a key-value pair');
        }

        obj[key = entry[0]] = (dest = obj[key]) ?
          (utils.isArray(dest) ? [...dest, entry[1]] : [dest, entry[1]]) : entry[1];
      }

      setHeaders(obj, valueOrRewrite)
    } else {
      header != null && setHeader(valueOrRewrite, header, rewrite);
    }

    return this;
  }
```

---

</SwmSnippet>

# Usage

## <SwmToken path="lib/core/AxiosHeaders.js" pos="74:2:2" line-data="class AxiosHeaders {">`AxiosHeaders`</SwmToken> Usage in Configuration Resolution

<SwmToken path="lib/core/AxiosHeaders.js" pos="74:2:2" line-data="class AxiosHeaders {">`AxiosHeaders`</SwmToken> is used to normalize and standardize headers during the configuration resolution phase. For example, in the configuration resolver, headers are converted into an <SwmToken path="lib/core/AxiosHeaders.js" pos="74:2:2" line-data="class AxiosHeaders {">`AxiosHeaders`</SwmToken> instance to ensure consistent header handling before the request is sent.

## <SwmToken path="lib/core/AxiosHeaders.js" pos="74:2:2" line-data="class AxiosHeaders {">`AxiosHeaders`</SwmToken> in HTTP and Fetch Adapters

In both the HTTP and Fetch adapters, <SwmToken path="lib/core/AxiosHeaders.js" pos="74:2:2" line-data="class AxiosHeaders {">`AxiosHeaders`</SwmToken> is used to wrap raw headers from responses or requests. This allows the adapters to work with a consistent header interface, enabling features like normalization and easy header manipulation.

## <SwmToken path="lib/core/AxiosHeaders.js" pos="74:2:2" line-data="class AxiosHeaders {">`AxiosHeaders`</SwmToken> in Request Dispatching

During the dispatch of a request, <SwmToken path="lib/core/AxiosHeaders.js" pos="74:2:2" line-data="class AxiosHeaders {">`AxiosHeaders`</SwmToken> is used to convert the headers in the request configuration into an <SwmToken path="lib/core/AxiosHeaders.js" pos="74:2:2" line-data="class AxiosHeaders {">`AxiosHeaders`</SwmToken> instance. Similarly, when a response is received, its headers are also wrapped in <SwmToken path="lib/core/AxiosHeaders.js" pos="74:2:2" line-data="class AxiosHeaders {">`AxiosHeaders`</SwmToken>. This ensures that headers are always handled in a uniform way throughout the request lifecycle.

## <SwmToken path="lib/core/AxiosHeaders.js" pos="74:2:2" line-data="class AxiosHeaders {">`AxiosHeaders`</SwmToken> in Merging Configurations

When merging configurations, <SwmToken path="lib/core/AxiosHeaders.js" pos="74:2:2" line-data="class AxiosHeaders {">`AxiosHeaders`</SwmToken> instances are converted to plain objects to facilitate merging. This allows the library to combine headers from different sources while preserving the structure and semantics of the headers.

## <SwmToken path="lib/core/AxiosHeaders.js" pos="74:2:2" line-data="class AxiosHeaders {">`AxiosHeaders`</SwmToken> Exposure and Extension

<SwmToken path="lib/core/AxiosHeaders.js" pos="74:2:2" line-data="class AxiosHeaders {">`AxiosHeaders`</SwmToken> is exposed as a property on the main axios instance, allowing users to access and extend it if needed. The class also defines accessors for common HTTP headers, providing convenient getter methods for frequently used headers.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBYXhpb3MlM0ElM0FTd2ltbS1EZW1v" repo-name="axios"><sup>Powered by [Swimm](/)</sup></SwmMeta>
