---
title: Core Library Helpers Overview
---
# Introduction to Core Library Helpers

Helpers in the core library are specialized utility functions crafted to support and streamline the primary operations of the HTTP client. They encapsulate common, reusable tasks that are essential for the client's functionality but are abstracted away from the main logic to promote clarity and maintainability.

These helpers cover a range of focused operations including URL construction, parameter encoding, configuration resolution, stream management, and data parsing. By handling these tasks, helpers enable the HTTP client to maintain a clean and modular codebase.

For example, helpers manage the combination of URLs and encoding of parameters to ensure requests are correctly formatted. They also handle asynchronous data streams and blob reading, which are critical for processing response data efficiently.

In addition, helpers are responsible for managing request configurations such as resolving headers, handling authentication credentials, and managing cross-site request forgery (CSRF) tokens. This ensures that requests are consistently and securely configured.

# Benefits of Helpers

By isolating specific logic into small, focused functions, helpers enhance the modularity and maintainability of the codebase. This approach reduces code duplication and guarantees consistent behavior across different parts of the library, making the code easier to test and evolve.

# Usage of Helpers in the Codebase

Helpers are utilized internally by the core components of the HTTP client to perform common operations efficiently. This internal usage allows the main code to focus on higher-level logic without being cluttered by repetitive utility code.

<SwmSnippet path="/lib/helpers/spread.js" line="8">

---

For instance, the <SwmToken path="lib/helpers/spread.js" pos="14:6:6" line-data=" * With `spread` this example can be re-written.">`spread`</SwmToken> helper function simplifies invoking a callback with an array of arguments by expanding the array into individual parameters. This provides syntactic sugar over the traditional <SwmToken path="lib/helpers/spread.js" pos="6:18:22" line-data=" * Common use case would be to use `Function.prototype.apply`.">`Function.prototype.apply`</SwmToken> method, improving code readability and usability.

````javascript
 *  ```js
 *  function f(x, y, z) {}
 *  var args = [1, 2, 3];
 *  f.apply(null, args);
 *  ```
 *
 * With `spread` this example can be re-written.
 *
 *  ```js
 *  spread(function(x, y, z) {})([1, 2, 3]);
 *  ```
````

---

</SwmSnippet>

# Helper Functions for Signals, Headers, and Streams

Helpers also provide utility functions to manage complex aspects such as abort signals, HTTP headers, and data streams within axios.

<SwmSnippet path="/lib/helpers/composeSignals.js" line="5">

---

The <SwmToken path="lib/helpers/composeSignals.js" pos="5:2:2" line-data="const composeSignals = (signals, timeout) =&gt; {">`composeSignals`</SwmToken> helper function merges multiple abort signals and an optional timeout into a single abort signal. This unified signal allows axios to handle cancellation requests from various sources seamlessly. Internally, it creates an <SwmToken path="lib/helpers/composeSignals.js" pos="9:9:9" line-data="    let controller = new AbortController();">`AbortController`</SwmToken> and listens for abort events from all provided signals, triggering the composed signal's abort event with an appropriate error if any source aborts or the timeout expires.

```javascript
const composeSignals = (signals, timeout) => {
  const {length} = (signals = signals ? signals.filter(Boolean) : []);

  if (timeout || length) {
    let controller = new AbortController();

    let aborted;

    const onabort = function (reason) {
      if (!aborted) {
        aborted = true;
        unsubscribe();
        const err = reason instanceof Error ? reason : this.reason;
        controller.abort(err instanceof AxiosError ? err : new CanceledError(err instanceof Error ? err.message : err));
      }
    }

    let timer = timeout && setTimeout(() => {
      timer = null;
      onabort(new AxiosError(`timeout ${timeout} of ms exceeded`, AxiosError.ETIMEDOUT))
    }, timeout)

    const unsubscribe = () => {
      if (signals) {
        timer && clearTimeout(timer);
        timer = null;
        signals.forEach(signal => {
          signal.unsubscribe ? signal.unsubscribe(onabort) : signal.removeEventListener('abort', onabort);
        });
        signals = null;
      }
    }

    signals.forEach((signal) => signal.addEventListener('abort', onabort));

    const {signal} = controller;

    signal.unsubscribe = () => utils.asap(unsubscribe);

    return signal;
  }
```

---

</SwmSnippet>

<SwmSnippet path="/lib/helpers/parseHeaders.js" line="28">

---

Another important helper, `parseHeaders`, converts raw HTTP header strings into a structured JavaScript object. It correctly handles multiple headers with the same name, such as <SwmToken path="lib/helpers/parseHeaders.js" pos="43:9:11" line-data="    if (key === &#39;set-cookie&#39;) {">`set-cookie`</SwmToken>, and follows Node.js conventions to ignore duplicates for certain headers. This parsing is essential for axios to work with response headers in a consistent and accessible format.

```javascript
export default rawHeaders => {
  const parsed = {};
  let key;
  let val;
  let i;

  rawHeaders && rawHeaders.split('\n').forEach(function parser(line) {
    i = line.indexOf(':');
    key = line.substring(0, i).trim().toLowerCase();
    val = line.substring(i + 1).trim();

    if (!key || (parsed[key] && ignoreDuplicateOf[key])) {
      return;
    }

    if (key === 'set-cookie') {
      if (parsed[key]) {
        parsed[key].push(val);
      } else {
        parsed[key] = [val];
      }
    } else {
      parsed[key] = parsed[key] ? parsed[key] + ', ' + val : val;
    }
  });

  return parsed;
};
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBYXhpb3MlM0ElM0FTd2ltbS1EZW1v" repo-name="axios"><sup>Powered by [Swimm](/)</sup></SwmMeta>
