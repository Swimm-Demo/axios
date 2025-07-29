---
title: The InterceptorManager class
---
This document covers the <SwmToken path="lib/core/Axios.js" pos="25:6:6" line-data="      request: new InterceptorManager(),">`InterceptorManager`</SwmToken> class in detail, focusing on:

1. What <SwmToken path="lib/core/Axios.js" pos="25:6:6" line-data="      request: new InterceptorManager(),">`InterceptorManager`</SwmToken> is and its purpose
2. The variables and functions defined in <SwmToken path="lib/core/Axios.js" pos="25:6:6" line-data="      request: new InterceptorManager(),">`InterceptorManager`</SwmToken>, with explanations and code citations for each

# What is <SwmToken path="lib/core/Axios.js" pos="25:6:6" line-data="      request: new InterceptorManager(),">`InterceptorManager`</SwmToken>

<SwmToken path="lib/core/Axios.js" pos="25:6:6" line-data="      request: new InterceptorManager(),">`InterceptorManager`</SwmToken> is a class responsible for managing a stack of interceptors in Axios. It allows the registration, removal, and iteration of interceptor functions that can modify requests or responses before they are handled by the main logic. This mechanism is essential for features like logging, authentication, or request/response transformation, and is used internally by Axios to provide flexible middleware-like capabilities.

<SwmSnippet path="/lib/core/InterceptorManager.js" line="18">

---

The function <SwmToken path="lib/core/InterceptorManager.js" pos="18:1:1" line-data="  use(fulfilled, rejected, options) {">`use`</SwmToken> adds a new interceptor to the stack. It accepts fulfilled and rejected handler functions, as well as an optional options object. The function stores these handlers in the internal <SwmToken path="lib/core/InterceptorManager.js" pos="19:3:3" line-data="    this.handlers.push({">`handlers`</SwmToken> array and returns an ID that can be used to remove the interceptor later.

```javascript
  use(fulfilled, rejected, options) {
    this.handlers.push({
      fulfilled,
      rejected,
      synchronous: options ? options.synchronous : false,
      runWhen: options ? options.runWhen : null
    });
    return this.handlers.length - 1;
  }
```

---

</SwmSnippet>

<SwmSnippet path="/lib/core/InterceptorManager.js" line="35">

---

The function <SwmToken path="lib/core/InterceptorManager.js" pos="35:1:1" line-data="  eject(id) {">`eject`</SwmToken> removes an interceptor from the stack by setting the corresponding entry in the <SwmToken path="lib/core/InterceptorManager.js" pos="36:6:6" line-data="    if (this.handlers[id]) {">`handlers`</SwmToken> array to null. This allows the stack to skip over removed interceptors without changing the indices of other handlers.

```javascript
  eject(id) {
    if (this.handlers[id]) {
      this.handlers[id] = null;
    }
  }
```

---

</SwmSnippet>

<SwmSnippet path="/lib/core/InterceptorManager.js" line="46">

---

The function <SwmToken path="lib/core/InterceptorManager.js" pos="46:1:1" line-data="  clear() {">`clear`</SwmToken> removes all interceptors from the stack by resetting the <SwmToken path="lib/core/InterceptorManager.js" pos="47:6:6" line-data="    if (this.handlers) {">`handlers`</SwmToken> array to an empty array. This is useful for scenarios where all registered interceptors need to be discarded at once.

```javascript
  clear() {
    if (this.handlers) {
      this.handlers = [];
    }
  }
```

---

</SwmSnippet>

<SwmSnippet path="/lib/core/InterceptorManager.js" line="62">

---

The function <SwmToken path="lib/core/InterceptorManager.js" pos="62:1:1" line-data="  forEach(fn) {">`forEach`</SwmToken> iterates over all registered interceptors in the <SwmToken path="lib/core/InterceptorManager.js" pos="63:7:7" line-data="    utils.forEach(this.handlers, function forEachHandler(h) {">`handlers`</SwmToken> array and invokes a provided function for each non-null handler. This is particularly useful for applying logic to all active interceptors, such as executing them in sequence.

```javascript
  forEach(fn) {
    utils.forEach(this.handlers, function forEachHandler(h) {
      if (h !== null) {
        fn(h);
      }
    });
  }
```

---

</SwmSnippet>

# Usage

## <SwmToken path="lib/core/Axios.js" pos="25:6:6" line-data="      request: new InterceptorManager(),">`InterceptorManager`</SwmToken>

The <SwmToken path="lib/core/Axios.js" pos="25:6:6" line-data="      request: new InterceptorManager(),">`InterceptorManager`</SwmToken> class is used to create and manage collections of interceptors for both requests and responses within an Axios instance. It allows adding, removing, and executing interceptors that can modify or handle HTTP requests and responses.

<SwmSnippet path="/lib/core/Axios.js" line="22">

---

Within the Axios class constructor, two instances of <SwmToken path="lib/core/Axios.js" pos="25:6:6" line-data="      request: new InterceptorManager(),">`InterceptorManager`</SwmToken> are created: one for request interceptors and one for response interceptors. This setup enables Axios to maintain separate chains of interceptors that process outgoing requests and incoming responses respectively.

```javascript
  constructor(instanceConfig) {
    this.defaults = instanceConfig || {};
    this.interceptors = {
      request: new InterceptorManager(),
      response: new InterceptorManager()
    };
  }
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBYXhpb3MlM0ElM0FTd2ltbS1EZW1v" repo-name="axios"><sup>Powered by [Swimm](/)</sup></SwmMeta>
