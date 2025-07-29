---
title: XHR Adapter Implementation for Axios
---
# introduction

This document explains the key design decisions and flow of the XHR adapter implementation in <SwmPath>[lib/adapters/xhr.js](lib/adapters/xhr.js)</SwmPath>. It answers:

1. How does the adapter handle configuration and request setup?
2. How are response and error handling managed?
3. How does the adapter support cancellation and progress events?
4. How does it ensure compatibility and protocol validation?

# configuration and request setup

<SwmSnippet path="/lib/adapters/xhr.js" line="12">

---

The adapter first checks if <SwmToken path="lib/adapters/xhr.js" pos="12:8:8" line-data="const isXHRAdapterSupported = typeof XMLHttpRequest !== &#39;undefined&#39;;">`XMLHttpRequest`</SwmToken> is supported and then returns a function that creates a Promise for the request. It resolves the config to normalize and prepare data and headers, extracting important fields like <SwmToken path="lib/adapters/xhr.js" pos="19:4:4" line-data="    let {responseType, onUploadProgress, onDownloadProgress} = _config;">`responseType`</SwmToken> and progress callbacks. This setup centralizes config handling and ensures consistent request preparation.

```javascript
const isXHRAdapterSupported = typeof XMLHttpRequest !== 'undefined';

export default isXHRAdapterSupported && function (config) {
  return new Promise(function dispatchXhrRequest(resolve, reject) {
    const _config = resolveConfig(config);
    let requestData = _config.data;
    const requestHeaders = AxiosHeaders.from(_config.headers).normalize();
    let {responseType, onUploadProgress, onDownloadProgress} = _config;
    let onCanceled;
    let uploadThrottled, downloadThrottled;
    let flushUpload, flushDownload;
```

---

</SwmSnippet>

<SwmSnippet path="/lib/adapters/xhr.js" line="33">

---

The <SwmToken path="lib/adapters/xhr.js" pos="33:9:9" line-data="    let request = new XMLHttpRequest();">`XMLHttpRequest`</SwmToken> instance is created and opened with the HTTP method and URL from the config. The timeout is set here as well.

```javascript
    let request = new XMLHttpRequest();

    request.open(_config.method.toUpperCase(), _config.url, true);

    // Set the request timeout in MS
    request.timeout = _config.timeout;

    function onloadend() {
      if (!request) {
        return;
      }
      // Prepare the response
      const responseHeaders = AxiosHeaders.from(
        'getAllResponseHeaders' in request && request.getAllResponseHeaders()
      );
      const responseData = !responseType || responseType === 'text' || responseType === 'json' ?
        request.responseText : request.response;
      const response = {
        data: responseData,
        status: request.status,
        statusText: request.statusText,
        headers: responseHeaders,
        config,
        request
      };
```

---

</SwmSnippet>

<SwmSnippet path="/lib/adapters/xhr.js" line="133">

---

Headers are normalized and set on the request, with special handling to remove <SwmToken path="lib/adapters/xhr.js" pos="133:5:7" line-data="    // Remove Content-Type if data is undefined">`Content-Type`</SwmToken> if no data is provided. Credentials and <SwmToken path="lib/adapters/xhr.js" pos="148:5:5" line-data="    // Add responseType to request if needed">`responseType`</SwmToken> are also configured based on the config.

```javascript
    // Remove Content-Type if data is undefined
    requestData === undefined && requestHeaders.setContentType(null);

    // Add headers to the request
    if ('setRequestHeader' in request) {
      utils.forEach(requestHeaders.toJSON(), function setRequestHeader(val, key) {
        request.setRequestHeader(key, val);
      });
    }

    // Add withCredentials to request if needed
    if (!utils.isUndefined(_config.withCredentials)) {
      request.withCredentials = !!_config.withCredentials;
    }

    // Add responseType to request if needed
    if (responseType && responseType !== 'json') {
      request.responseType = _config.responseType;
    }
```

---

</SwmSnippet>

# response and error handling

<SwmSnippet path="/lib/adapters/xhr.js" line="37">

---

The adapter defines an onloadend handler to process the response once the request finishes. It extracts response headers, data (handling different response types), and constructs a response object with status and config references. This object is passed to the settle function, which resolves or rejects the Promise based on HTTP status.

```javascript
    // Set the request timeout in MS
    request.timeout = _config.timeout;

    function onloadend() {
      if (!request) {
        return;
      }
      // Prepare the response
      const responseHeaders = AxiosHeaders.from(
        'getAllResponseHeaders' in request && request.getAllResponseHeaders()
      );
      const responseData = !responseType || responseType === 'text' || responseType === 'json' ?
        request.responseText : request.response;
      const response = {
        data: responseData,
        status: request.status,
        statusText: request.statusText,
        headers: responseHeaders,
        config,
        request
      };

      settle(function _resolve(value) {
        resolve(value);
        done();
      }, function _reject(err) {
        reject(err);
        done();
      }, response);
```

---

</SwmSnippet>

<SwmSnippet path="/lib/adapters/xhr.js" line="71">

---

To support browsers without onloadend, it falls back to using onreadystatechange and defers calling onloadend to the next tick to avoid premature handling.

```javascript
    if ('onloadend' in request) {
      // Use onloadend if available
      request.onloadend = onloadend;
    } else {
      // Listen for ready state to emulate onloadend
      request.onreadystatechange = function handleLoad() {
        if (!request || request.readyState !== 4) {
          return;
        }

        // The request errored out and we didn't get a response, this will be
        // handled by onerror instead
        // With one exception: request that using file: protocol, most browsers
        // will return status as 0 even though it's a successful request
        if (request.status === 0 && !(request.responseURL && request.responseURL.indexOf('file:') === 0)) {
          return;
        }
        // readystate handler is calling before onerror or ontimeout handlers,
        // so we should call onloadend on the next 'tick'
        setTimeout(onloadend);
      };
    }
```

---

</SwmSnippet>

<SwmSnippet path="/lib/adapters/xhr.js" line="94">

---

Error handlers for abort, network errors, and timeouts reject the Promise with appropriate <SwmToken path="lib/adapters/xhr.js" pos="100:5:5" line-data="      reject(new AxiosError(&#39;Request aborted&#39;, AxiosError.ECONNABORTED, config, request));">`AxiosError`</SwmToken> instances, cleaning up the request reference to avoid memory leaks.

```javascript
    // Handle browser request cancellation (as opposed to a manual cancellation)
    request.onabort = function handleAbort() {
      if (!request) {
        return;
      }

      reject(new AxiosError('Request aborted', AxiosError.ECONNABORTED, config, request));

      // Clean up request
      request = null;
    };

    // Handle low level network errors
    request.onerror = function handleError() {
      // Real errors are hidden from us by the browser
      // onerror should only fire if it's a network error
      reject(new AxiosError('Network Error', AxiosError.ERR_NETWORK, config, request));

      // Clean up request
      request = null;
    };

    // Handle timeout
    request.ontimeout = function handleTimeout() {
      let timeoutErrorMessage = _config.timeout ? 'timeout of ' + _config.timeout + 'ms exceeded' : 'timeout exceeded';
      const transitional = _config.transitional || transitionalDefaults;
      if (_config.timeoutErrorMessage) {
        timeoutErrorMessage = _config.timeoutErrorMessage;
      }
      reject(new AxiosError(
        timeoutErrorMessage,
        transitional.clarifyTimeoutError ? AxiosError.ETIMEDOUT : AxiosError.ECONNABORTED,
        config,
        request));

      // Clean up request
      request = null;
    };
```

---

</SwmSnippet>

# cancellation and progress support

<SwmSnippet path="/lib/adapters/xhr.js" line="168">

---

Cancellation is handled by subscribing to <SwmToken path="lib/adapters/xhr.js" pos="168:6:6" line-data="    if (_config.cancelToken || _config.signal) {">`cancelToken`</SwmToken> or signal events. When cancellation triggers, the request is aborted, the Promise rejected with a <SwmToken path="lib/adapters/xhr.js" pos="175:16:16" line-data="        reject(!cancel || cancel.type ? new CanceledError(null, config, request) : cancel);">`CanceledError`</SwmToken>, and the request cleaned up. This ensures that manual or external cancellations are respected.

```javascript
    if (_config.cancelToken || _config.signal) {
      // Handle cancellation
      // eslint-disable-next-line func-names
      onCanceled = cancel => {
        if (!request) {
          return;
        }
        reject(!cancel || cancel.type ? new CanceledError(null, config, request) : cancel);
        request.abort();
        request = null;
      };

      _config.cancelToken && _config.cancelToken.subscribe(onCanceled);
      if (_config.signal) {
        _config.signal.aborted ? onCanceled() : _config.signal.addEventListener('abort', onCanceled);
      }
    }
```

---

</SwmSnippet>

<SwmSnippet path="/lib/adapters/xhr.js" line="153">

---

Progress events for upload and download are throttled and attached if callbacks are provided. This uses a helper to reduce event frequency and flush events properly, improving performance and UX during large transfers.

```javascript
    // Handle progress if needed
    if (onDownloadProgress) {
      ([downloadThrottled, flushDownload] = progressEventReducer(onDownloadProgress, true));
      request.addEventListener('progress', downloadThrottled);
    }

    // Not all browsers support upload events
    if (onUploadProgress && request.upload) {
      ([uploadThrottled, flushUpload] = progressEventReducer(onUploadProgress));

      request.upload.addEventListener('progress', uploadThrottled);

      request.upload.addEventListener('loadend', flushUpload);
    }
```

---

</SwmSnippet>

# compatibility and protocol validation

<SwmSnippet path="/lib/adapters/xhr.js" line="186">

---

Before sending, the adapter parses the URL protocol and rejects unsupported protocols early. This prevents invalid requests and aligns with platform capabilities.

```javascript
    const protocol = parseProtocol(_config.url);

    if (protocol && platform.protocols.indexOf(protocol) === -1) {
      reject(new AxiosError('Unsupported protocol ' + protocol + ':', AxiosError.ERR_BAD_REQUEST, config));
      return;
    }
```

---

</SwmSnippet>

<SwmSnippet path="/lib/adapters/xhr.js" line="194">

---

Finally, the request is sent with the prepared data or null if none exists.

```javascript
    // Send the request
    request.send(requestData || null);
  });
}
```

---

</SwmSnippet>

# cleanup

<SwmSnippet path="/lib/adapters/xhr.js" line="24">

---

After the response settles or errors, the done function flushes any remaining progress events and unsubscribes cancellation listeners to avoid leaks. The request reference is nulled after completion or error to free resources.

```javascript
    function done() {
      flushUpload && flushUpload(); // flush events
      flushDownload && flushDownload(); // flush events

      _config.cancelToken && _config.cancelToken.unsubscribe(onCanceled);

      _config.signal && _config.signal.removeEventListener('abort', onCanceled);
    }
```

---

</SwmSnippet>

<SwmSnippet path="/lib/adapters/xhr.js" line="67">

---

&nbsp;

```javascript
      // Clean up request
      request = null;
    }
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBYXhpb3MlM0ElM0FTd2ltbS1EZW1v" repo-name="axios"><sup>Powered by [Swimm](/)</sup></SwmMeta>
