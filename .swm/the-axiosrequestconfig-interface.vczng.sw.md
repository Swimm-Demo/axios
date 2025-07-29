---
title: The AxiosRequestConfig interface
---
# Intro

This document covers the interface <SwmToken path="index.d.ts" pos="318:4:4" line-data="export interface AxiosRequestConfig&lt;D = any&gt; {">`AxiosRequestConfig`</SwmToken> as defined in <SwmPath>[index.d.ts](index.d.ts)</SwmPath>. We will address:

1. What <SwmToken path="index.d.ts" pos="318:4:4" line-data="export interface AxiosRequestConfig&lt;D = any&gt; {">`AxiosRequestConfig`</SwmToken> is and its purpose
2. All variables defined in <SwmToken path="index.d.ts" pos="318:4:4" line-data="export interface AxiosRequestConfig&lt;D = any&gt; {">`AxiosRequestConfig`</SwmToken>, with code citations for each

# What is <SwmToken path="index.d.ts" pos="318:4:4" line-data="export interface AxiosRequestConfig&lt;D = any&gt; {">`AxiosRequestConfig`</SwmToken>

<SwmToken path="index.d.ts" pos="318:4:4" line-data="export interface AxiosRequestConfig&lt;D = any&gt; {">`AxiosRequestConfig`</SwmToken> is a <SwmToken path="index.d.ts" pos="1:2:2" line-data="// TypeScript Version: 4.7">`TypeScript`</SwmToken> interface that defines the shape of the configuration object used when making HTTP requests with Axios. It allows developers to specify request details such as the URL, HTTP method, headers, query parameters, request body, timeout, authentication, and many other options. This interface is central to customizing and controlling the behavior of Axios requests, ensuring flexibility and type safety for both browser and Node.js environments.

# Variables and functions

Below are the variables defined in <SwmToken path="index.d.ts" pos="318:4:4" line-data="export interface AxiosRequestConfig&lt;D = any&gt; {">`AxiosRequestConfig`</SwmToken>, each with its purpose and code citation.

<SwmSnippet path="/index.d.ts" line="319">

---

The variable <SwmToken path="index.d.ts" pos="319:1:1" line-data="  url?: string;">`url`</SwmToken> specifies the request URL.

```typescript
  url?: string;
```

---

</SwmSnippet>

<SwmSnippet path="/index.d.ts" line="320">

---

The variable <SwmToken path="index.d.ts" pos="320:1:1" line-data="  method?: Method | string;">`method`</SwmToken> defines the HTTP method to be used for the request, such as GET, POST, PUT, etc.

```typescript
  method?: Method | string;
```

---

</SwmSnippet>

<SwmSnippet path="/index.d.ts" line="321">

---

The variable <SwmToken path="index.d.ts" pos="321:1:1" line-data="  baseURL?: string;">`baseURL`</SwmToken> sets a base URL that will be prepended to the request URL unless the URL is absolute.

```typescript
  baseURL?: string;
```

---

</SwmSnippet>

<SwmSnippet path="/index.d.ts" line="322">

---

The variable <SwmToken path="index.d.ts" pos="322:1:1" line-data="  allowAbsoluteUrls?: boolean;">`allowAbsoluteUrls`</SwmToken> determines whether absolute URLs are allowed in the request.

```typescript
  allowAbsoluteUrls?: boolean;
```

---

</SwmSnippet>

<SwmSnippet path="/index.d.ts" line="323">

---

The variable <SwmToken path="index.d.ts" pos="323:1:1" line-data="  transformRequest?: AxiosRequestTransformer | AxiosRequestTransformer[];">`transformRequest`</SwmToken> allows you to specify one or more functions to modify the request data before it is sent to the server.

```typescript
  transformRequest?: AxiosRequestTransformer | AxiosRequestTransformer[];
```

---

</SwmSnippet>

<SwmSnippet path="/index.d.ts" line="324">

---

The variable <SwmToken path="index.d.ts" pos="324:1:1" line-data="  transformResponse?: AxiosResponseTransformer | AxiosResponseTransformer[];">`transformResponse`</SwmToken> allows you to specify one or more functions to modify the response data before it is passed to then/catch.

```typescript
  transformResponse?: AxiosResponseTransformer | AxiosResponseTransformer[];
```

---

</SwmSnippet>

<SwmSnippet path="/index.d.ts" line="325">

---

The variable <SwmToken path="index.d.ts" pos="325:1:1" line-data="  headers?: (RawAxiosRequestHeaders &amp; MethodsHeaders) | AxiosHeaders;">`headers`</SwmToken> allows you to set custom headers for the request.

```typescript
  headers?: (RawAxiosRequestHeaders & MethodsHeaders) | AxiosHeaders;
```

---

</SwmSnippet>

<SwmSnippet path="/index.d.ts" line="326">

---

The variable <SwmToken path="index.d.ts" pos="326:1:1" line-data="  params?: any;">`params`</SwmToken> is used to specify URL parameters to be sent with the request.

```typescript
  params?: any;
```

---

</SwmSnippet>

<SwmSnippet path="/index.d.ts" line="327">

---

The variable <SwmToken path="index.d.ts" pos="327:1:1" line-data="  paramsSerializer?: ParamsSerializerOptions | CustomParamsSerializer;">`paramsSerializer`</SwmToken> allows you to define a custom function or options for serializing query parameters.

```typescript
  paramsSerializer?: ParamsSerializerOptions | CustomParamsSerializer;
```

---

</SwmSnippet>

<SwmSnippet path="/index.d.ts" line="328">

---

The variable <SwmToken path="index.d.ts" pos="328:1:1" line-data="  data?: D;">`data`</SwmToken> contains the request payload to be sent as the body of the request.

```typescript
  data?: D;
```

---

</SwmSnippet>

<SwmSnippet path="/index.d.ts" line="329">

---

The variable <SwmToken path="index.d.ts" pos="329:1:1" line-data="  timeout?: Milliseconds;">`timeout`</SwmToken> sets the number of milliseconds before the request times out.

```typescript
  timeout?: Milliseconds;
```

---

</SwmSnippet>

<SwmSnippet path="/index.d.ts" line="330">

---

The variable <SwmToken path="index.d.ts" pos="330:1:1" line-data="  timeoutErrorMessage?: string;">`timeoutErrorMessage`</SwmToken> allows you to specify a custom error message for request timeouts.

```typescript
  timeoutErrorMessage?: string;
```

---

</SwmSnippet>

<SwmSnippet path="/index.d.ts" line="331">

---

The variable <SwmToken path="index.d.ts" pos="331:1:1" line-data="  withCredentials?: boolean;">`withCredentials`</SwmToken> indicates whether or not cross-site Access-Control requests should be made using credentials.

```typescript
  withCredentials?: boolean;
```

---

</SwmSnippet>

<SwmSnippet path="/index.d.ts" line="332">

---

The variable <SwmToken path="index.d.ts" pos="332:1:1" line-data="  adapter?: AxiosAdapterConfig | AxiosAdapterConfig[];">`adapter`</SwmToken> allows you to specify a custom adapter or an array of adapters for handling the request.

```typescript
  adapter?: AxiosAdapterConfig | AxiosAdapterConfig[];
```

---

</SwmSnippet>

<SwmSnippet path="/index.d.ts" line="333">

---

The variable <SwmToken path="index.d.ts" pos="333:1:1" line-data="  auth?: AxiosBasicCredentials;">`auth`</SwmToken> provides HTTP Basic authentication credentials.

```typescript
  auth?: AxiosBasicCredentials;
```

---

</SwmSnippet>

<SwmSnippet path="/index.d.ts" line="334">

---

The variable <SwmToken path="index.d.ts" pos="334:1:1" line-data="  responseType?: ResponseType;">`responseType`</SwmToken> specifies the type of data that the server will respond with, such as 'json', 'blob', or 'arraybuffer'.

```typescript
  responseType?: ResponseType;
```

---

</SwmSnippet>

<SwmSnippet path="/index.d.ts" line="335">

---

The variable <SwmToken path="index.d.ts" pos="335:1:1" line-data="  responseEncoding?: responseEncoding | string;">`responseEncoding`</SwmToken> sets the encoding to use for the response data.

```typescript
  responseEncoding?: responseEncoding | string;
```

---

</SwmSnippet>

<SwmSnippet path="/index.d.ts" line="336">

---

The variable <SwmToken path="index.d.ts" pos="336:1:1" line-data="  xsrfCookieName?: string;">`xsrfCookieName`</SwmToken> defines the name of the cookie to use as a value for XSRF token.

```typescript
  xsrfCookieName?: string;
```

---

</SwmSnippet>

<SwmSnippet path="/index.d.ts" line="337">

---

The variable <SwmToken path="index.d.ts" pos="337:1:1" line-data="  xsrfHeaderName?: string;">`xsrfHeaderName`</SwmToken> defines the name of the HTTP header to use for the XSRF token.

```typescript
  xsrfHeaderName?: string;
```

---

</SwmSnippet>

<SwmSnippet path="/index.d.ts" line="338">

---

The variable <SwmToken path="index.d.ts" pos="338:1:1" line-data="  onUploadProgress?: (progressEvent: AxiosProgressEvent) =&gt; void;">`onUploadProgress`</SwmToken> is a callback function that handles progress events for uploads.

```typescript
  onUploadProgress?: (progressEvent: AxiosProgressEvent) => void;
```

---

</SwmSnippet>

<SwmSnippet path="/index.d.ts" line="339">

---

The variable <SwmToken path="index.d.ts" pos="339:1:1" line-data="  onDownloadProgress?: (progressEvent: AxiosProgressEvent) =&gt; void;">`onDownloadProgress`</SwmToken> is a callback function that handles progress events for downloads.

```typescript
  onDownloadProgress?: (progressEvent: AxiosProgressEvent) => void;
```

---

</SwmSnippet>

<SwmSnippet path="/index.d.ts" line="340">

---

The variable <SwmToken path="index.d.ts" pos="340:1:1" line-data="  maxContentLength?: number;">`maxContentLength`</SwmToken> sets the maximum size of the HTTP response content in bytes.

```typescript
  maxContentLength?: number;
```

---

</SwmSnippet>

<SwmSnippet path="/index.d.ts" line="341">

---

The variable <SwmToken path="index.d.ts" pos="341:1:1" line-data="  validateStatus?: ((status: number) =&gt; boolean) | null;">`validateStatus`</SwmToken> is a function that determines whether the response status code is valid for resolving the promise.

```typescript
  validateStatus?: ((status: number) => boolean) | null;
```

---

</SwmSnippet>

<SwmSnippet path="/index.d.ts" line="342">

---

The variable <SwmToken path="index.d.ts" pos="342:1:1" line-data="  maxBodyLength?: number;">`maxBodyLength`</SwmToken> sets the maximum size of the request body in bytes.

```typescript
  maxBodyLength?: number;
```

---

</SwmSnippet>

<SwmSnippet path="/index.d.ts" line="343">

---

The variable <SwmToken path="index.d.ts" pos="343:1:1" line-data="  maxRedirects?: number;">`maxRedirects`</SwmToken> specifies the maximum number of redirects to follow.

```typescript
  maxRedirects?: number;
```

---

</SwmSnippet>

<SwmSnippet path="/index.d.ts" line="344">

---

The variable <SwmToken path="index.d.ts" pos="344:1:1" line-data="  maxRate?: number | [MaxUploadRate, MaxDownloadRate];">`maxRate`</SwmToken> sets the maximum upload or download rate, either as a single number or as an array for upload and download rates.

```typescript
  maxRate?: number | [MaxUploadRate, MaxDownloadRate];
```

---

</SwmSnippet>

<SwmSnippet path="/index.d.ts" line="345">

---

The variable <SwmToken path="index.d.ts" pos="345:1:1" line-data="  beforeRedirect?: (options: Record&lt;string, any&gt;, responseDetails: {headers: Record&lt;string, string&gt;, statusCode: HttpStatusCode}) =&gt; void;">`beforeRedirect`</SwmToken> is a callback function invoked before a redirect is followed.

```typescript
  beforeRedirect?: (options: Record<string, any>, responseDetails: {headers: Record<string, string>, statusCode: HttpStatusCode}) => void;
```

---

</SwmSnippet>

<SwmSnippet path="/index.d.ts" line="346">

---

The variable <SwmToken path="index.d.ts" pos="346:1:1" line-data="  socketPath?: string | null;">`socketPath`</SwmToken> specifies the UNIX Socket to use for the request, if any.

```typescript
  socketPath?: string | null;
```

---

</SwmSnippet>

<SwmSnippet path="/index.d.ts" line="347">

---

The variable <SwmToken path="index.d.ts" pos="347:1:1" line-data="  transport?: any;">`transport`</SwmToken> allows you to specify a custom transport mechanism for the request.

```typescript
  transport?: any;
```

---

</SwmSnippet>

<SwmSnippet path="/index.d.ts" line="348">

---

The variable <SwmToken path="index.d.ts" pos="348:1:1" line-data="  httpAgent?: any;">`httpAgent`</SwmToken> allows you to specify a custom HTTP agent for Node.js requests.

```typescript
  httpAgent?: any;
```

---

</SwmSnippet>

<SwmSnippet path="/index.d.ts" line="349">

---

The variable <SwmToken path="index.d.ts" pos="349:1:1" line-data="  httpsAgent?: any;">`httpsAgent`</SwmToken> allows you to specify a custom HTTPS agent for Node.js requests.

```typescript
  httpsAgent?: any;
```

---

</SwmSnippet>

# Usage

## <SwmToken path="index.d.ts" pos="318:4:4" line-data="export interface AxiosRequestConfig&lt;D = any&gt; {">`AxiosRequestConfig`</SwmToken> in Type Aliases

<SwmToken path="index.d.ts" pos="318:4:4" line-data="export interface AxiosRequestConfig&lt;D = any&gt; {">`AxiosRequestConfig`</SwmToken> is used as a base type for <SwmToken path="index.d.ts" pos="368:4:4" line-data="export type RawAxiosRequestConfig&lt;D = any&gt; = AxiosRequestConfig&lt;D&gt;;">`RawAxiosRequestConfig`</SwmToken>, which acts as an alias. This shows that <SwmToken path="index.d.ts" pos="368:4:4" line-data="export type RawAxiosRequestConfig&lt;D = any&gt; = AxiosRequestConfig&lt;D&gt;;">`RawAxiosRequestConfig`</SwmToken> inherits all properties of <SwmToken path="index.d.ts" pos="318:4:4" line-data="export interface AxiosRequestConfig&lt;D = any&gt; {">`AxiosRequestConfig`</SwmToken>, allowing it to be used interchangeably where request configuration is needed.

## <SwmToken path="index.d.ts" pos="318:4:4" line-data="export interface AxiosRequestConfig&lt;D = any&gt; {">`AxiosRequestConfig`</SwmToken> in Internal Request Configuration

<SwmToken path="index.d.ts" pos="107:5:5" line-data="  (this: InternalAxiosRequestConfig, data: any, headers: AxiosRequestHeaders): any;">`InternalAxiosRequestConfig`</SwmToken> extends <SwmToken path="index.d.ts" pos="318:4:4" line-data="export interface AxiosRequestConfig&lt;D = any&gt; {">`AxiosRequestConfig`</SwmToken> by adding a mandatory headers property of type <SwmToken path="index.d.ts" pos="92:4:4" line-data="export type AxiosRequestHeaders = RawAxiosRequestHeaders &amp; AxiosHeaders;">`AxiosRequestHeaders`</SwmToken>. This indicates that internally, the request configuration always includes headers, ensuring consistent header handling during request processing.

## <SwmToken path="index.d.ts" pos="318:4:4" line-data="export interface AxiosRequestConfig&lt;D = any&gt; {">`AxiosRequestConfig`</SwmToken> in Defaults Interfaces

<SwmToken path="index.d.ts" pos="388:4:4" line-data="export interface AxiosDefaults&lt;D = any&gt; extends Omit&lt;AxiosRequestConfig&lt;D&gt;, &#39;headers&#39;&gt; {">`AxiosDefaults`</SwmToken> and <SwmToken path="index.d.ts" pos="392:4:4" line-data="export interface CreateAxiosDefaults&lt;D = any&gt; extends Omit&lt;AxiosRequestConfig&lt;D&gt;, &#39;headers&#39;&gt; {">`CreateAxiosDefaults`</SwmToken> interfaces extend <SwmToken path="index.d.ts" pos="318:4:4" line-data="export interface AxiosRequestConfig&lt;D = any&gt; {">`AxiosRequestConfig`</SwmToken> while omitting the headers property, which they redefine with more specific types. <SwmToken path="index.d.ts" pos="388:4:4" line-data="export interface AxiosDefaults&lt;D = any&gt; extends Omit&lt;AxiosRequestConfig&lt;D&gt;, &#39;headers&#39;&gt; {">`AxiosDefaults`</SwmToken> uses <SwmToken path="index.d.ts" pos="374:4:4" line-data="export interface HeadersDefaults {">`HeadersDefaults`</SwmToken> for headers, while <SwmToken path="index.d.ts" pos="392:4:4" line-data="export interface CreateAxiosDefaults&lt;D = any&gt; extends Omit&lt;AxiosRequestConfig&lt;D&gt;, &#39;headers&#39;&gt; {">`CreateAxiosDefaults`</SwmToken> allows headers to be <SwmToken path="index.d.ts" pos="325:5:5" line-data="  headers?: (RawAxiosRequestHeaders &amp; MethodsHeaders) | AxiosHeaders;">`RawAxiosRequestHeaders`</SwmToken>, <SwmToken path="index.d.ts" pos="325:14:14" line-data="  headers?: (RawAxiosRequestHeaders &amp; MethodsHeaders) | AxiosHeaders;">`AxiosHeaders`</SwmToken>, or a partial <SwmToken path="index.d.ts" pos="374:4:4" line-data="export interface HeadersDefaults {">`HeadersDefaults`</SwmToken>. This design supports flexible default configuration of requests, especially for headers.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBYXhpb3MlM0ElM0FTd2ltbS1EZW1v" repo-name="axios"><sup>Powered by [Swimm](/)</sup></SwmMeta>
