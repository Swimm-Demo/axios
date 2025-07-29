---
title: The CancelToken class
---
This document covers the <SwmToken path="lib/cancel/CancelToken.js" pos="125:9:9" line-data="    const token = new CancelToken(function executor(c) {">`CancelToken`</SwmToken> class and its API. We'll address:

1. What <SwmToken path="lib/cancel/CancelToken.js" pos="125:9:9" line-data="    const token = new CancelToken(function executor(c) {">`CancelToken`</SwmToken> is and its purpose
2. The main functions and variables in <SwmToken path="lib/cancel/CancelToken.js" pos="125:9:9" line-data="    const token = new CancelToken(function executor(c) {">`CancelToken`</SwmToken>, including:
   - <SwmToken path="lib/cancel/CancelToken.js" pos="68:1:1" line-data="  throwIfRequested() {">`throwIfRequested`</SwmToken>
   - subscribe
   - unsubscribe
   - <SwmToken path="lib/cancel/CancelToken.js" pos="105:1:1" line-data="  toAbortSignal() {">`toAbortSignal`</SwmToken>
   - source

# What is <SwmToken path="lib/cancel/CancelToken.js" pos="125:9:9" line-data="    const token = new CancelToken(function executor(c) {">`CancelToken`</SwmToken>

<SwmToken path="lib/cancel/CancelToken.js" pos="125:9:9" line-data="    const token = new CancelToken(function executor(c) {">`CancelToken`</SwmToken> is a class that provides a mechanism to signal and handle cancellation of asynchronous operations, such as HTTP requests. It is used to allow consumers to cancel requests before they complete, helping to manage resources and avoid unnecessary network activity. <SwmToken path="lib/cancel/CancelToken.js" pos="125:9:9" line-data="    const token = new CancelToken(function executor(c) {">`CancelToken`</SwmToken> is typically used in scenarios where a user may want to abort a request, for example, when navigating away from a page or when a newer request supersedes an older one.

<SwmSnippet path="/lib/cancel/CancelToken.js" line="68">

---

The function <SwmToken path="lib/cancel/CancelToken.js" pos="68:1:1" line-data="  throwIfRequested() {">`throwIfRequested`</SwmToken> checks if a cancellation has already been requested for the token. If so, it throws the stored cancellation reason, which is typically a <SwmToken path="lib/axios.js" pos="53:2:2" line-data="axios.CanceledError = CanceledError;">`CanceledError`</SwmToken>. This allows consumers to proactively abort processing if cancellation has occurred.

```javascript
  throwIfRequested() {
    if (this.reason) {
      throw this.reason;
    }
  }
```

---

</SwmSnippet>

<SwmSnippet path="/lib/cancel/CancelToken.js" line="78">

---

The function subscribe registers a listener function that will be called when cancellation is requested. If cancellation has already occurred, the listener is invoked immediately with the cancellation reason. Otherwise, the listener is added to an internal list to be notified upon cancellation.

```javascript
  subscribe(listener) {
    if (this.reason) {
      listener(this.reason);
      return;
    }

    if (this._listeners) {
      this._listeners.push(listener);
    } else {
      this._listeners = [listener];
    }
  }
```

---

</SwmSnippet>

<SwmSnippet path="/lib/cancel/CancelToken.js" line="95">

---

The function unsubscribe removes a previously registered listener from the internal list. This prevents the listener from being called if cancellation occurs after it has been unsubscribed.

```javascript
  unsubscribe(listener) {
    if (!this._listeners) {
      return;
    }
    const index = this._listeners.indexOf(listener);
    if (index !== -1) {
      this._listeners.splice(index, 1);
    }
  }
```

---

</SwmSnippet>

<SwmSnippet path="/lib/cancel/CancelToken.js" line="105">

---

The function <SwmToken path="lib/cancel/CancelToken.js" pos="105:1:1" line-data="  toAbortSignal() {">`toAbortSignal`</SwmToken> creates and returns an AbortSignal instance that is linked to the <SwmToken path="lib/cancel/CancelToken.js" pos="125:9:9" line-data="    const token = new CancelToken(function executor(c) {">`CancelToken`</SwmToken>. When the token is cancelled, the AbortSignal is also aborted, allowing integration with APIs that support the <SwmToken path="lib/cancel/CancelToken.js" pos="106:9:9" line-data="    const controller = new AbortController();">`AbortController`</SwmToken> pattern.

```javascript
  toAbortSignal() {
    const controller = new AbortController();

    const abort = (err) => {
      controller.abort(err);
    };

    this.subscribe(abort);

    controller.signal.unsubscribe = () => this.unsubscribe(abort);

    return controller.signal;
  }
```

---

</SwmSnippet>

<SwmSnippet path="/lib/cancel/CancelToken.js" line="123">

---

The static function source is a factory method that returns an object containing a new <SwmToken path="lib/cancel/CancelToken.js" pos="125:9:9" line-data="    const token = new CancelToken(function executor(c) {">`CancelToken`</SwmToken> instance and a cancel function. Calling the cancel function triggers cancellation for the associated token. This pattern simplifies token creation and cancellation management.

```javascript
  static source() {
    let cancel;
    const token = new CancelToken(function executor(c) {
      cancel = c;
    });
    return {
      token,
      cancel
    };
  }
```

---

</SwmSnippet>

# Usage

<SwmSnippet path="/lib/axios.js" line="52">

---

<SwmToken path="lib/axios.js" pos="52:8:8" line-data="// Expose Cancel &amp; CancelToken">`CancelToken`</SwmToken> is exposed as a property of the main axios export, allowing users to create cancellation tokens that can be used to cancel HTTP requests. This is done alongside other related exports such as <SwmToken path="lib/axios.js" pos="53:2:2" line-data="axios.CanceledError = CanceledError;">`CanceledError`</SwmToken> and <SwmToken path="lib/axios.js" pos="55:2:2" line-data="axios.isCancel = isCancel;">`isCancel`</SwmToken>, which help manage and identify cancellation events.

```javascript
// Expose Cancel & CancelToken
axios.CanceledError = CanceledError;
axios.CancelToken = CancelToken;
axios.isCancel = isCancel;
axios.VERSION = VERSION;
```

---

</SwmSnippet>

## Example Usage of <SwmToken path="lib/cancel/CancelToken.js" pos="125:9:9" line-data="    const token = new CancelToken(function executor(c) {">`CancelToken`</SwmToken>

To use <SwmToken path="lib/cancel/CancelToken.js" pos="125:9:9" line-data="    const token = new CancelToken(function executor(c) {">`CancelToken`</SwmToken>, a developer creates a new token and passes it as part of the request configuration. This token can then be triggered to cancel the request if needed, for example, to abort a request that is taking too long or is no longer needed. This mechanism improves control over HTTP requests and resource management.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBYXhpb3MlM0ElM0FTd2ltbS1EZW1v" repo-name="axios"><sup>Powered by [Swimm](/)</sup></SwmMeta>
