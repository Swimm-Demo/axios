---
title: Core Library Overview
---
# Core Library Overview

The Core Library forms the foundation of the HTTP client, implementing essential functionalities required to perform HTTP requests and handle responses efficiently. It centralizes the logic that enables communication between the client and servers.

# Key Functionalities

This library includes modules responsible for dispatching requests, merging configuration options, transforming request and response data, and managing errors. A significant feature is the management of interceptors, which allow developers to modify requests or responses before they are processed by the application.

# Main Interface: Axios Class

At the heart of the Core Library is the Axios class, which serves as the primary interface for developers to create and send HTTP requests. This class encapsulates the core features, ensuring that requests are properly constructed, sent, and that responses are processed and settled according to the specified configuration. It exposes methods for common HTTP operations such as GET, POST, PUT, and DELETE, all handled through promises for asynchronous control.

# Request Lifecycle in the Core Library

When a request is initiated, the Core Library uses the `dispatchRequest` module to send the HTTP request. Before dispatching, it applies interceptors to allow modifications to the request or response. Configuration options are merged using the `mergeConfig` module to ensure all settings are correctly applied. Data transformations are handled by the `transformData` module, which processes request payloads and response data as needed. Finally, the response is settled using the `settle` module, which resolves or rejects the promise based on the HTTP status and other criteria.

<SwmSnippet path="/lib/cancel/CancelToken.js" line="5">

---

The Core Library also includes the <SwmToken path="lib/cancel/CancelToken.js" pos="6:6:6" line-data=" * A `CancelToken` is an object that can be used to request cancellation of an operation.">`CancelToken`</SwmToken> class, which provides a mechanism to cancel ongoing HTTP requests. Developers can create a cancellation token and pass it with a request. This token can later be used to signal cancellation, which is particularly useful for aborting requests that are no longer necessary, thereby improving resource management and user experience. The <SwmToken path="lib/cancel/CancelToken.js" pos="6:6:6" line-data=" * A `CancelToken` is an object that can be used to request cancellation of an operation.">`CancelToken`</SwmToken> class supports subscribing to cancellation events, throwing errors if cancellation is requested, and converting to an `AbortSignal` for compatibility with other APIs.

```javascript
/**
 * A `CancelToken` is an object that can be used to request cancellation of an operation.
 *
 * @param {Function} executor The executor function.
 *
 * @returns {CancelToken}
 */
class CancelToken {
  constructor(executor) {
    if (typeof executor !== 'function') {
      throw new TypeError('executor must be a function.');
    }

    let resolvePromise;

    this.promise = new Promise(function promiseExecutor(resolve) {
      resolvePromise = resolve;
    });

    const token = this;

    // eslint-disable-next-line func-names
    this.promise.then(cancel => {
      if (!token._listeners) return;

      let i = token._listeners.length;

      while (i-- > 0) {
        token._listeners[i](cancel);
      }
      token._listeners = null;
    });

    // eslint-disable-next-line func-names
    this.promise.then = onfulfilled => {
      let _resolve;
      // eslint-disable-next-line func-names
      const promise = new Promise(resolve => {
        token.subscribe(resolve);
        _resolve = resolve;
      }).then(onfulfilled);

      promise.cancel = function reject() {
        token.unsubscribe(_resolve);
      };

      return promise;
    };

    executor(function cancel(message, config, request) {
      if (token.reason) {
        // Cancellation has already been requested
        return;
      }

      token.reason = new CanceledError(message, config, request);
      resolvePromise(token.reason);
    });
  }

  /**
   * Throws a `CanceledError` if cancellation has been requested.
   */
  throwIfRequested() {
    if (this.reason) {
      throw this.reason;
    }
  }

  /**
   * Subscribe to the cancel signal
   */

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

  /**
   * Unsubscribe from the cancel signal
   */

  unsubscribe(listener) {
    if (!this._listeners) {
      return;
    }
    const index = this._listeners.indexOf(listener);
    if (index !== -1) {
      this._listeners.splice(index, 1);
    }
  }

  toAbortSignal() {
    const controller = new AbortController();

    const abort = (err) => {
      controller.abort(err);
    };

    this.subscribe(abort);

    controller.signal.unsubscribe = () => this.unsubscribe(abort);

    return controller.signal;
  }

  /**
   * Returns an object that contains a new `CancelToken` and a function that, when called,
   * cancels the `CancelToken`.
   */
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
}

export default CancelToken;

```

---

</SwmSnippet>

# Benefits of the Core Library

By encapsulating these core features, the Core Library ensures consistent and extensible HTTP communication across different platforms supported by the client. This modular and centralized design simplifies development, enhances maintainability, and allows for easy extension or customization of HTTP request handling.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBYXhpb3MlM0ElM0FTd2ltbS1EZW1v" repo-name="axios"><sup>Powered by [Swimm](/)</sup></SwmMeta>
