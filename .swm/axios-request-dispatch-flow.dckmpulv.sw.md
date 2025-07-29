---
title: Axios Request Dispatch Flow
---
# introduction

This document explains the request dispatch flow in axios, focusing on how a request is prepared, sent, and how the response or errors are handled. We will cover:

1. How cancellation is checked before and after the request.
2. How request data and headers are transformed before sending.
3. How the adapter is selected and used to send the request.
4. How response data and headers are transformed after receiving.
5. How errors are processed, especially in relation to cancellation and response transformation.

# cancellation checks before dispatch

<SwmSnippet path="/lib/core/dispatchRequest.js" line="10">

---

Before sending the request, axios verifies if cancellation has been requested to avoid unnecessary network activity. This is done by the function that throws a <SwmToken path="lib/core/dispatchRequest.js" pos="11:8:8" line-data=" * Throws a `CanceledError` if cancellation has been requested.">`CanceledError`</SwmToken> if the cancellation token or abort signal indicates the request should be canceled.

```javascript
/**
 * Throws a `CanceledError` if cancellation has been requested.
 *
 * @param {Object} config The config that is to be used for the request
 *
 * @returns {void}
 */
function throwIfCancellationRequested(config) {
  if (config.cancelToken) {
    config.cancelToken.throwIfRequested();
  }

  if (config.signal && config.signal.aborted) {
    throw new CanceledError(null, config);
  }
}
```

---

</SwmSnippet>

This early cancellation check prevents wasted resources and aligns with axios's support for cancellation tokens and abort signals.

# preparing request data and headers

<SwmSnippet path="/lib/core/dispatchRequest.js" line="37">

---

Once cancellation is cleared, the request headers are normalized into an <SwmToken path="lib/core/dispatchRequest.js" pos="37:7:7" line-data="  config.headers = AxiosHeaders.from(config.headers);">`AxiosHeaders`</SwmToken> instance for consistent manipulation. Then, the request data is transformed using the configured <SwmToken path="lib/core/dispatchRequest.js" pos="42:3:3" line-data="    config.transformRequest">`transformRequest`</SwmToken> functions. This allows users to preprocess data (e.g., serialize JSON) before sending.

```javascript
  config.headers = AxiosHeaders.from(config.headers);

  // Transform request data
  config.data = transformData.call(
    config,
    config.transformRequest
  );
```

---

</SwmSnippet>

<SwmSnippet path="/lib/core/dispatchRequest.js" line="45">

---

Additionally, for HTTP methods that usually send a body (<SwmToken path="lib/core/dispatchRequest.js" pos="45:6:6" line-data="  if ([&#39;post&#39;, &#39;put&#39;, &#39;patch&#39;].indexOf(config.method) !== -1) {">`post`</SwmToken>, <SwmToken path="lib/core/dispatchRequest.js" pos="45:11:11" line-data="  if ([&#39;post&#39;, &#39;put&#39;, &#39;patch&#39;].indexOf(config.method) !== -1) {">`put`</SwmToken>, <SwmToken path="lib/core/dispatchRequest.js" pos="45:16:16" line-data="  if ([&#39;post&#39;, &#39;put&#39;, &#39;patch&#39;].indexOf(config.method) !== -1) {">`patch`</SwmToken>), the content-type header is set to <SwmToken path="lib/core/dispatchRequest.js" pos="46:8:16" line-data="    config.headers.setContentType(&#39;application/x-www-form-urlencoded&#39;, false);">`application/x-www-form-urlencoded`</SwmToken> if not already specified. This ensures the server interprets the payload correctly.

```javascript
  if (['post', 'put', 'patch'].indexOf(config.method) !== -1) {
    config.headers.setContentType('application/x-www-form-urlencoded', false);
  }
```

---

</SwmSnippet>

# selecting and invoking the adapter

<SwmSnippet path="/lib/core/dispatchRequest.js" line="49">

---

Axios supports multiple environments by abstracting the actual HTTP request behind adapters. The adapter is selected based on the config or defaults, then invoked with the prepared config. The adapter returns a Promise representing the HTTP transaction.

```javascript
  const adapter = adapters.getAdapter(config.adapter || defaults.adapter);

  return adapter(config).then(function onAdapterResolution(response) {
    throwIfCancellationRequested(config);
```

---

</SwmSnippet>

# handling the response

When the adapter resolves, axios again checks for cancellation to avoid processing a response for a canceled request.

The response data is then transformed using the configured <SwmToken path="lib/core/dispatchRequest.js" pos="57:3:3" line-data="      config.transformResponse,">`transformResponse`</SwmToken> functions, allowing users to parse or modify the response payload (e.g., JSON parsing).

Response headers are normalized into an <SwmToken path="lib/core/dispatchRequest.js" pos="37:7:7" line-data="  config.headers = AxiosHeaders.from(config.headers);">`AxiosHeaders`</SwmToken> instance for consistent access.

<SwmSnippet path="/lib/core/dispatchRequest.js" line="54">

---

Finally, the processed response is returned.

```javascript
    // Transform response data
    response.data = transformData.call(
      config,
      config.transformResponse,
      response
    );

    response.headers = AxiosHeaders.from(response.headers);

    return response;
```

---

</SwmSnippet>

# handling errors

If the adapter rejects (network error, timeout, etc.), axios first checks if the error is a cancellation. If not, it performs another cancellation check to ensure the error is not due to a late cancellation.

If the error contains a response (e.g., HTTP error status), axios applies the same response data and headers transformations as in the success case. This ensures error responses are processed consistently.

<SwmSnippet path="/lib/core/dispatchRequest.js" line="64">

---

The error is then propagated as a rejected Promise.

```javascript
  }, function onAdapterRejection(reason) {
    if (!isCancel(reason)) {
      throwIfCancellationRequested(config);

      // Transform response data
      if (reason && reason.response) {
        reason.response.data = transformData.call(
          config,
          config.transformResponse,
          reason.response
        );
        reason.response.headers = AxiosHeaders.from(reason.response.headers);
      }
    }

    return Promise.reject(reason);
  });
}
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBYXhpb3MlM0ElM0FTd2ltbS1EZW1v" repo-name="axios"><sup>Powered by [Swimm](/)</sup></SwmMeta>
