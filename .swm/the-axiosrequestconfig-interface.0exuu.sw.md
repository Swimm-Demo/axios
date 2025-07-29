---
title: The AxiosRequestConfig interface
---
# What is <SwmToken path="index.d.cts" pos="128:8:8" line-data="  constructor(config?: axios.AxiosRequestConfig);">`AxiosRequestConfig`</SwmToken>

This document will cover:

1. What <SwmToken path="index.d.cts" pos="128:8:8" line-data="  constructor(config?: axios.AxiosRequestConfig);">`AxiosRequestConfig`</SwmToken> is and its purpose in the codebase
2. All variables and functions defined in <SwmToken path="index.d.cts" pos="128:8:8" line-data="  constructor(config?: axios.AxiosRequestConfig);">`AxiosRequestConfig`</SwmToken>, with code citations for each

# What is <SwmToken path="index.d.cts" pos="128:8:8" line-data="  constructor(config?: axios.AxiosRequestConfig);">`AxiosRequestConfig`</SwmToken>

<SwmToken path="index.d.cts" pos="128:8:8" line-data="  constructor(config?: axios.AxiosRequestConfig);">`AxiosRequestConfig`</SwmToken> is an interface defined in the codebase that specifies the shape of the configuration object used when making HTTP requests with Axios. It allows developers to customize various aspects of a request, such as the URL, HTTP method, headers, query parameters, request body, timeout, authentication, and more. This interface is central to how Axios enables flexible and powerful HTTP request handling in both browser and Node.js environments.

# Variables and functions

<SwmToken path="index.d.cts" pos="128:8:8" line-data="  constructor(config?: axios.AxiosRequestConfig);">`AxiosRequestConfig`</SwmToken> defines a set of optional properties that control the behavior of HTTP requests. Each property is designed to configure a specific aspect of the request lifecycle.

<SwmSnippet path="/index.d.cts" line="386">

---

The variable <SwmToken path="index.d.cts" pos="386:1:1" line-data="    url?: string;">`url`</SwmToken> specifies the request URL.

```cts
    url?: string;
```

---

</SwmSnippet>

<SwmSnippet path="/index.d.cts" line="387">

---

The variable <SwmToken path="index.d.cts" pos="387:1:1" line-data="    method?: Method | string;">`method`</SwmToken> defines the HTTP method to be used for the request, such as GET, POST, PUT, etc.

```cts
    method?: Method | string;
```

---

</SwmSnippet>

<SwmSnippet path="/index.d.cts" line="388">

---

The variable <SwmToken path="index.d.cts" pos="388:1:1" line-data="    baseURL?: string;">`baseURL`</SwmToken> sets a base URL that will be prepended to the request URL unless the URL is absolute.

```cts
    baseURL?: string;
```

---

</SwmSnippet>

<SwmSnippet path="/index.d.cts" line="389">

---

The variable <SwmToken path="index.d.cts" pos="389:1:1" line-data="    allowAbsoluteUrls?: boolean;">`allowAbsoluteUrls`</SwmToken> determines whether absolute URLs are permitted.

```cts
    allowAbsoluteUrls?: boolean;
```

---

</SwmSnippet>

<SwmSnippet path="/index.d.cts" line="390">

---

The variable <SwmToken path="index.d.cts" pos="390:1:1" line-data="    transformRequest?: AxiosRequestTransformer | AxiosRequestTransformer[];">`transformRequest`</SwmToken> allows you to specify one or more functions to modify the request data before it is sent.

```cts
    transformRequest?: AxiosRequestTransformer | AxiosRequestTransformer[];
```

---

</SwmSnippet>

<SwmSnippet path="/index.d.cts" line="391">

---

The variable <SwmToken path="index.d.cts" pos="391:1:1" line-data="    transformResponse?: AxiosResponseTransformer | AxiosResponseTransformer[];">`transformResponse`</SwmToken> allows you to specify one or more functions to modify the response data before it is returned to the caller.

```cts
    transformResponse?: AxiosResponseTransformer | AxiosResponseTransformer[];
```

---

</SwmSnippet>

<SwmSnippet path="/index.d.cts" line="392">

---

The variable <SwmToken path="index.d.cts" pos="392:1:1" line-data="    headers?: (RawAxiosRequestHeaders &amp; MethodsHeaders) | AxiosHeaders;">`headers`</SwmToken> is used to set custom HTTP headers for the request.

```cts
    headers?: (RawAxiosRequestHeaders & MethodsHeaders) | AxiosHeaders;
```

---

</SwmSnippet>

<SwmSnippet path="/index.d.cts" line="393">

---

The variable <SwmToken path="index.d.cts" pos="393:1:1" line-data="    params?: any;">`params`</SwmToken> allows you to specify URL query parameters to be appended to the request URL.

```cts
    params?: any;
```

---

</SwmSnippet>

<SwmSnippet path="/index.d.cts" line="394">

---

The variable <SwmToken path="index.d.cts" pos="394:1:1" line-data="    paramsSerializer?: ParamsSerializerOptions | CustomParamsSerializer;">`paramsSerializer`</SwmToken> lets you define a custom function or options for serializing query parameters.

```cts
    paramsSerializer?: ParamsSerializerOptions | CustomParamsSerializer;
```

---

</SwmSnippet>

<SwmSnippet path="/index.d.cts" line="395">

---

The variable <SwmToken path="index.d.cts" pos="395:1:1" line-data="    data?: D;">`data`</SwmToken> contains the request payload, typically used with methods like POST or PUT.

```cts
    data?: D;
```

---

</SwmSnippet>

<SwmSnippet path="/index.d.cts" line="396">

---

The variable <SwmToken path="index.d.cts" pos="396:1:1" line-data="    timeout?: Milliseconds;">`timeout`</SwmToken> sets the number of milliseconds before the request times out.

```cts
    timeout?: Milliseconds;
```

---

</SwmSnippet>

<SwmSnippet path="/index.d.cts" line="397">

---

The variable <SwmToken path="index.d.cts" pos="397:1:1" line-data="    timeoutErrorMessage?: string;">`timeoutErrorMessage`</SwmToken> allows you to specify a custom error message when a timeout occurs.

```cts
    timeoutErrorMessage?: string;
```

---

</SwmSnippet>

<SwmSnippet path="/index.d.cts" line="398">

---

The variable <SwmToken path="index.d.cts" pos="398:1:1" line-data="    withCredentials?: boolean;">`withCredentials`</SwmToken> indicates whether or not cross-site Access-Control requests should be made using credentials.

```cts
    withCredentials?: boolean;
```

---

</SwmSnippet>

<SwmSnippet path="/index.d.cts" line="399">

---

The variable <SwmToken path="index.d.cts" pos="399:1:1" line-data="    adapter?: AxiosAdapterConfig | AxiosAdapterConfig[];">`adapter`</SwmToken> allows you to specify a custom adapter or an array of adapters for handling requests.

```cts
    adapter?: AxiosAdapterConfig | AxiosAdapterConfig[];
```

---

</SwmSnippet>

<SwmSnippet path="/index.d.cts" line="400">

---

The variable <SwmToken path="index.d.cts" pos="400:1:1" line-data="    auth?: AxiosBasicCredentials;">`auth`</SwmToken> provides HTTP Basic authentication credentials.

```cts
    auth?: AxiosBasicCredentials;
```

---

</SwmSnippet>

<SwmSnippet path="/index.d.cts" line="401">

---

The variable <SwmToken path="index.d.cts" pos="401:1:1" line-data="    responseType?: ResponseType;">`responseType`</SwmToken> sets the type of data that the server will respond with, such as 'json', 'blob', or 'text'.

```cts
    responseType?: ResponseType;
```

---

</SwmSnippet>

<SwmSnippet path="/index.d.cts" line="402">

---

The variable <SwmToken path="index.d.cts" pos="402:1:1" line-data="    responseEncoding?: responseEncoding | string;">`responseEncoding`</SwmToken> specifies the encoding to use for the response.

```cts
    responseEncoding?: responseEncoding | string;
```

---

</SwmSnippet>

<SwmSnippet path="/index.d.cts" line="403">

---

The variable <SwmToken path="index.d.cts" pos="403:1:1" line-data="    xsrfCookieName?: string;">`xsrfCookieName`</SwmToken> sets the name of the cookie to use as a value for XSRF token.

```cts
    xsrfCookieName?: string;
```

---

</SwmSnippet>

<SwmSnippet path="/index.d.cts" line="404">

---

The variable <SwmToken path="index.d.cts" pos="404:1:1" line-data="    xsrfHeaderName?: string;">`xsrfHeaderName`</SwmToken> sets the name of the HTTP header that carries the XSRF token value.

```cts
    xsrfHeaderName?: string;
```

---

</SwmSnippet>

<SwmSnippet path="/index.d.cts" line="405">

---

The variable <SwmToken path="index.d.cts" pos="405:1:1" line-data="    onUploadProgress?: (progressEvent: AxiosProgressEvent) =&gt; void;">`onUploadProgress`</SwmToken> is a callback function that is called periodically with upload progress events.

```cts
    onUploadProgress?: (progressEvent: AxiosProgressEvent) => void;
```

---

</SwmSnippet>

<SwmSnippet path="/index.d.cts" line="406">

---

The variable <SwmToken path="index.d.cts" pos="406:1:1" line-data="    onDownloadProgress?: (progressEvent: AxiosProgressEvent) =&gt; void;">`onDownloadProgress`</SwmToken> is a callback function that is called periodically with download progress events.

```cts
    onDownloadProgress?: (progressEvent: AxiosProgressEvent) => void;
```

---

</SwmSnippet>

<SwmSnippet path="/index.d.cts" line="407">

---

The variable <SwmToken path="index.d.cts" pos="407:1:1" line-data="    maxContentLength?: number;">`maxContentLength`</SwmToken> sets the maximum size of the HTTP response content in bytes.

```cts
    maxContentLength?: number;
```

---

</SwmSnippet>

<SwmSnippet path="/index.d.cts" line="408">

---

The variable <SwmToken path="index.d.cts" pos="408:1:1" line-data="    validateStatus?: ((status: number) =&gt; boolean) | null;">`validateStatus`</SwmToken> is a function that determines whether the response status code is considered valid.

```cts
    validateStatus?: ((status: number) => boolean) | null;
```

---

</SwmSnippet>

<SwmSnippet path="/index.d.cts" line="409">

---

The variable <SwmToken path="index.d.cts" pos="409:1:1" line-data="    maxBodyLength?: number;">`maxBodyLength`</SwmToken> sets the maximum size of the request body in bytes.

```cts
    maxBodyLength?: number;
```

---

</SwmSnippet>

<SwmSnippet path="/index.d.cts" line="410">

---

The variable <SwmToken path="index.d.cts" pos="410:1:1" line-data="    maxRedirects?: number;">`maxRedirects`</SwmToken> specifies the maximum number of redirects to follow.

```cts
    maxRedirects?: number;
```

---

</SwmSnippet>

<SwmSnippet path="/index.d.cts" line="411">

---

The variable <SwmToken path="index.d.cts" pos="411:1:1" line-data="    maxRate?: number | [MaxUploadRate, MaxDownloadRate];">`maxRate`</SwmToken> sets the maximum upload and download rate in bytes per second.

```cts
    maxRate?: number | [MaxUploadRate, MaxDownloadRate];
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBYXhpb3MlM0ElM0FTd2ltbS1EZW1v" repo-name="axios"><sup>Powered by [Swimm](/)</sup></SwmMeta>
