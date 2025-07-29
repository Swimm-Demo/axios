---
title: The Axios class
---
This document covers the Axios class as implemented in <SwmPath>[lib/core/Axios.js](lib/core/Axios.js)</SwmPath>. It will address:

1. What Axios is and its purpose in the codebase
2. The variables and functions defined in the Axios class, with a breakdown of each key method and property.

# What is Axios

The Axios class in <SwmPath>[lib/core/Axios.js](lib/core/Axios.js)</SwmPath> is the core implementation of the Axios HTTP client. It encapsulates the logic for creating HTTP requests, managing configuration defaults, and handling request and response interceptors. This class serves as the foundation for Axios instances, enabling features such as request transformation, error handling, and flexible configuration merging.

<SwmSnippet path="/lib/core/Axios.js" line="38">

---

The function <SwmToken path="lib/core/Axios.js" pos="38:3:3" line-data="  async request(configOrUrl, config) {">`request`</SwmToken> is the main entry point for dispatching HTTP requests. It accepts either a configuration object or a URL and an optional config, merges them with the instance defaults, and delegates the actual request logic to the internal <SwmToken path="lib/core/Axios.js" pos="40:7:7" line-data="      return await this._request(configOrUrl, config);">`_request`</SwmToken> method. It also includes enhanced error handling to improve stack trace clarity when exceptions occur.

```javascript
  async request(configOrUrl, config) {
    try {
      return await this._request(configOrUrl, config);
    } catch (err) {
      if (err instanceof Error) {
        let dummy = {};

        Error.captureStackTrace ? Error.captureStackTrace(dummy) : (dummy = new Error());

        // slice off the Error: ... line
        const stack = dummy.stack ? dummy.stack.replace(/^.+\n/, '') : '';
        try {
          if (!err.stack) {
            err.stack = stack;
            // match without the 2 top stack lines
          } else if (stack && !String(err.stack).endsWith(stack.replace(/^.+\n.+\n/, ''))) {
            err.stack += '\n' + stack
          }
        } catch (e) {
          // ignore the case where "stack" is an un-writable property
        }
      }

      throw err;
    }
  }
```

---

</SwmSnippet>

<SwmSnippet path="/lib/core/Axios.js" line="65">

---

The function <SwmToken path="lib/core/Axios.js" pos="65:1:1" line-data="  _request(configOrUrl, config) {">`_request`</SwmToken> implements the core logic for preparing and executing HTTP requests. It merges the provided configuration with the instance defaults, validates transitional and serializer options, sets up headers, and manages the interceptor chains for both requests and responses. Depending on whether interceptors are synchronous, it either chains promises or executes interceptors directly before dispatching the request. The method ensures that all configuration and transformation steps are applied before the request is sent and after the response is received.

```javascript
  _request(configOrUrl, config) {
    /*eslint no-param-reassign:0*/
    // Allow for axios('example/url'[, config]) a la fetch API
    if (typeof configOrUrl === 'string') {
      config = config || {};
      config.url = configOrUrl;
    } else {
      config = configOrUrl || {};
    }

    config = mergeConfig(this.defaults, config);

    const {transitional, paramsSerializer, headers} = config;

    if (transitional !== undefined) {
      validator.assertOptions(transitional, {
        silentJSONParsing: validators.transitional(validators.boolean),
        forcedJSONParsing: validators.transitional(validators.boolean),
        clarifyTimeoutError: validators.transitional(validators.boolean)
      }, false);
    }

    if (paramsSerializer != null) {
      if (utils.isFunction(paramsSerializer)) {
        config.paramsSerializer = {
          serialize: paramsSerializer
        }
      } else {
        validator.assertOptions(paramsSerializer, {
          encode: validators.function,
          serialize: validators.function
        }, true);
      }
    }

    // Set config.allowAbsoluteUrls
    if (config.allowAbsoluteUrls !== undefined) {
      // do nothing
    } else if (this.defaults.allowAbsoluteUrls !== undefined) {
      config.allowAbsoluteUrls = this.defaults.allowAbsoluteUrls;
    } else {
      config.allowAbsoluteUrls = true;
    }

    validator.assertOptions(config, {
      baseUrl: validators.spelling('baseURL'),
      withXsrfToken: validators.spelling('withXSRFToken')
    }, true);

    // Set config.method
    config.method = (config.method || this.defaults.method || 'get').toLowerCase();

    // Flatten headers
    let contextHeaders = headers && utils.merge(
      headers.common,
      headers[config.method]
    );

    headers && utils.forEach(
      ['delete', 'get', 'head', 'post', 'put', 'patch', 'common'],
      (method) => {
        delete headers[method];
      }
    );

    config.headers = AxiosHeaders.concat(contextHeaders, headers);

    // filter out skipped interceptors
    const requestInterceptorChain = [];
    let synchronousRequestInterceptors = true;
    this.interceptors.request.forEach(function unshiftRequestInterceptors(interceptor) {
      if (typeof interceptor.runWhen === 'function' && interceptor.runWhen(config) === false) {
        return;
      }

      synchronousRequestInterceptors = synchronousRequestInterceptors && interceptor.synchronous;

      requestInterceptorChain.unshift(interceptor.fulfilled, interceptor.rejected);
    });

    const responseInterceptorChain = [];
    this.interceptors.response.forEach(function pushResponseInterceptors(interceptor) {
      responseInterceptorChain.push(interceptor.fulfilled, interceptor.rejected);
    });

    let promise;
    let i = 0;
    let len;

    if (!synchronousRequestInterceptors) {
      const chain = [dispatchRequest.bind(this), undefined];
      chain.unshift(...requestInterceptorChain);
      chain.push(...responseInterceptorChain);
      len = chain.length;

      promise = Promise.resolve(config);

      while (i < len) {
        promise = promise.then(chain[i++], chain[i++]);
      }

      return promise;
    }

    len = requestInterceptorChain.length;

    let newConfig = config;

    i = 0;

    while (i < len) {
      const onFulfilled = requestInterceptorChain[i++];
      const onRejected = requestInterceptorChain[i++];
      try {
        newConfig = onFulfilled(newConfig);
      } catch (error) {
        onRejected.call(this, error);
        break;
      }
    }

    try {
      promise = dispatchRequest.call(this, newConfig);
    } catch (error) {
      return Promise.reject(error);
    }

    i = 0;
    len = responseInterceptorChain.length;

    while (i < len) {
      promise = promise.then(responseInterceptorChain[i++], responseInterceptorChain[i++]);
    }

    return promise;
  }
```

---

</SwmSnippet>

<SwmSnippet path="/lib/core/Axios.js" line="202">

---

The function <SwmToken path="lib/core/Axios.js" pos="202:1:1" line-data="  getUri(config) {">`getUri`</SwmToken> generates a full request URL based on the merged configuration. It combines the base URL, request URL, and query parameters, applying any custom parameter serialization if specified. This is useful for inspecting or logging the final URL that would be used for a request without actually sending it.

```javascript
  getUri(config) {
    config = mergeConfig(this.defaults, config);
    const fullPath = buildFullPath(config.baseURL, config.url, config.allowAbsoluteUrls);
    return buildURL(fullPath, config.params, config.paramsSerializer);
  }
```

---

</SwmSnippet>

<SwmSnippet path="/lib/core/Axios.js" line="22">

---

The variable <SwmToken path="lib/core/Axios.js" pos="23:3:3" line-data="    this.defaults = instanceConfig || {};">`defaults`</SwmToken> holds the default configuration for the Axios instance. This includes settings such as base URL, headers, and other request options that are applied to every request unless overridden.

```javascript
  constructor(instanceConfig) {
    this.defaults = instanceConfig || {};
```

---

</SwmSnippet>

<SwmSnippet path="/lib/core/Axios.js" line="24">

---

The variable <SwmToken path="lib/core/Axios.js" pos="24:3:3" line-data="    this.interceptors = {">`interceptors`</SwmToken> is an object containing two <SwmToken path="lib/core/Axios.js" pos="25:6:6" line-data="      request: new InterceptorManager(),">`InterceptorManager`</SwmToken> instances: one for request interceptors and one for response interceptors. These allow users to register functions that can modify requests before they are sent or responses before they are returned to the caller.

```javascript
    this.interceptors = {
      request: new InterceptorManager(),
      response: new InterceptorManager()
    };
```

---

</SwmSnippet>

# Usage

## Axios Instance Creation

The Axios class is instantiated through a factory function that creates a new Axios instance with a default configuration. This instance is then bound to the Axios prototype's request method, allowing the instance to make HTTP requests using the configured defaults. The instance also inherits all prototype methods from Axios, enabling it to support various HTTP operations.

## HTTP Method Aliases

Axios provides convenient aliases for common HTTP methods such as 'get', 'delete', 'head', and 'options'. These aliases are implemented as prototype methods on the Axios class, which internally call the generic request method with the appropriate HTTP method and URL. This design simplifies making requests by allowing direct calls like axios.get(url) instead of manually specifying the method each time.

## Form Data Support

In addition to standard HTTP methods, Axios also supports form submissions through method variants suffixed with 'Form'. These methods are generated dynamically and allow sending form data easily by setting the appropriate content type and handling serialization automatically. This feature enhances Axios's flexibility in handling different types of HTTP requests.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBYXhpb3MlM0ElM0FTd2ltbS1EZW1v" repo-name="axios"><sup>Powered by [Swimm](/)</sup></SwmMeta>
