---
title: Cancellation Mechanism in Core Library
---
# Introduction to Cancellation in Core Library

Cancellation in the Core Library provides a mechanism to abort HTTP requests that are currently in progress. This feature helps prevent unnecessary network activity and allows applications to handle request cancellations gracefully.

# <SwmToken path="lib/cancel/CancelToken.js" pos="125:9:9" line-data="    const token = new CancelToken(function executor(c) {">`CancelToken`</SwmToken>: The Core of Cancellation

The cancellation mechanism is primarily implemented through the <SwmToken path="lib/cancel/CancelToken.js" pos="125:9:9" line-data="    const token = new CancelToken(function executor(c) {">`CancelToken`</SwmToken> class. A <SwmToken path="lib/cancel/CancelToken.js" pos="125:9:9" line-data="    const token = new CancelToken(function executor(c) {">`CancelToken`</SwmToken> represents a token that can be used to request cancellation of an operation. It is created with an executor function that receives a cancel function. Invoking this cancel function triggers the cancellation process.

When cancellation is requested, a <SwmToken path="lib/cancel/CancelToken.js" pos="3:2:2" line-data="import CanceledError from &#39;./CanceledError.js&#39;;">`CanceledError`</SwmToken> instance is created and stored as the reason for cancellation within the <SwmToken path="lib/cancel/CancelToken.js" pos="125:9:9" line-data="    const token = new CancelToken(function executor(c) {">`CancelToken`</SwmToken>. The token maintains a promise that resolves upon cancellation, notifying all subscribed listeners about the event.

# Subscribing to Cancellation Events

Listeners can subscribe to the <SwmToken path="lib/cancel/CancelToken.js" pos="125:9:9" line-data="    const token = new CancelToken(function executor(c) {">`CancelToken`</SwmToken> to be notified when cancellation occurs. They can also unsubscribe if they no longer need to receive cancellation notifications. This subscription model allows different parts of the codebase to react appropriately to cancellation events.

# Integration with <SwmToken path="lib/cancel/CancelToken.js" pos="106:9:9" line-data="    const controller = new AbortController();">`AbortController`</SwmToken>

The <SwmToken path="lib/cancel/CancelToken.js" pos="125:9:9" line-data="    const token = new CancelToken(function executor(c) {">`CancelToken`</SwmToken> class provides a method to convert itself into an `AbortSignal`, enabling integration with the standard <SwmToken path="lib/cancel/CancelToken.js" pos="106:9:9" line-data="    const controller = new AbortController();">`AbortController`</SwmToken> API. This compatibility allows cancellation to be handled consistently alongside other web APIs that use `AbortSignal`.

# Detecting and Handling Cancellation

The utility function `isCancel` checks whether a given value represents a cancellation by looking for a specific cancellation property. Additionally, the <SwmToken path="lib/cancel/CancelToken.js" pos="68:1:1" line-data="  throwIfRequested() {">`throwIfRequested`</SwmToken> method on <SwmToken path="lib/cancel/CancelToken.js" pos="125:9:9" line-data="    const token = new CancelToken(function executor(c) {">`CancelToken`</SwmToken> throws the stored <SwmToken path="lib/cancel/CancelToken.js" pos="3:2:2" line-data="import CanceledError from &#39;./CanceledError.js&#39;;">`CanceledError`</SwmToken> if cancellation has been requested. This method enables clean interruption of the request flow when cancellation occurs.

# Cancellation in the Request Lifecycle

During the dispatch of an HTTP request, if a <SwmToken path="lib/cancel/CancelToken.js" pos="125:9:9" line-data="    const token = new CancelToken(function executor(c) {">`CancelToken`</SwmToken> is present and cancellation has already been requested, the request is immediately aborted without making the HTTP call. Otherwise, the request subscribes to cancellation events to handle any cancellation that might occur while the request is in progress.

<SwmSnippet path="/lib/cancel/CancelToken.js" line="123">

---

The static method `CancelToken.source` offers a convenient way to create a new <SwmToken path="lib/cancel/CancelToken.js" pos="125:9:9" line-data="    const token = new CancelToken(function executor(c) {">`CancelToken`</SwmToken> instance along with its associated cancel function. Calling this cancel function triggers the cancellation of the token. This approach simplifies the creation and usage of cancellation tokens by bundling the token and its cancel function together.

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

<SwmSnippet path="/lib/cancel/CancelToken.js" line="68">

---

The instance method <SwmToken path="lib/cancel/CancelToken.js" pos="68:1:1" line-data="  throwIfRequested() {">`throwIfRequested`</SwmToken> allows code to check if cancellation has been requested on the <SwmToken path="lib/cancel/CancelToken.js" pos="125:9:9" line-data="    const token = new CancelToken(function executor(c) {">`CancelToken`</SwmToken>. If so, it throws the stored <SwmToken path="lib/cancel/CancelToken.js" pos="3:2:2" line-data="import CanceledError from &#39;./CanceledError.js&#39;;">`CanceledError`</SwmToken>. This method is useful for interrupting the flow of an HTTP request or any operation that supports cancellation, ensuring that further processing stops immediately when cancellation is detected.

```javascript
  throwIfRequested() {
    if (this.reason) {
      throw this.reason;
    }
  }
```

---

</SwmSnippet>

# Example Usage of Cancellation

To use cancellation, create a <SwmToken path="lib/cancel/CancelToken.js" pos="125:9:9" line-data="    const token = new CancelToken(function executor(c) {">`CancelToken`</SwmToken> and its cancel function using the <SwmToken path="lib/cancel/CancelToken.js" pos="125:9:9" line-data="    const token = new CancelToken(function executor(c) {">`CancelToken`</SwmToken> constructor or the <SwmToken path="lib/cancel/CancelToken.js" pos="123:3:3" line-data="  static source() {">`source`</SwmToken> method. Pass the token in the request configuration. When you want to abort the request, call the cancel function. The request will throw a <SwmToken path="lib/cancel/CancelToken.js" pos="3:2:2" line-data="import CanceledError from &#39;./CanceledError.js&#39;;">`CanceledError`</SwmToken>, which can be detected using `isCancel` to handle cancellation separately from other errors.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBYXhpb3MlM0ElM0FTd2ltbS1EZW1v" repo-name="axios"><sup>Powered by [Swimm](/)</sup></SwmMeta>
