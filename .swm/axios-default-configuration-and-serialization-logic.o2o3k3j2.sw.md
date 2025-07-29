---
title: Axios Default Configuration and Serialization Logic
---
# introduction

This document explains key design decisions and implementation details in <SwmPath>[lib/defaults/index.js](lib/defaults/index.js)</SwmPath> related to axios's default configuration and data serialization. We will cover:

1. How axios safely stringifies request data to avoid errors.
2. The logic behind transforming request data based on its type and content-type headers.
3. How response data is parsed, especially JSON responses.
4. The default headers and environment settings that axios uses.

# safe stringification of request data

Axios needs to convert request data into a string format when sending JSON or other payloads. The function responsible for this tries to parse the input string first to check if it is valid JSON. If parsing succeeds, it returns the trimmed original string to avoid unnecessary re-stringification. If parsing fails due to syntax errors, it falls back to stringifying the raw value. This approach prevents double stringification and handles invalid JSON gracefully.

<SwmSnippet path="/lib/defaults/index.js" line="11">

---

This logic is implemented in the <SwmToken path="lib/defaults/index.js" pos="21:2:2" line-data="function stringifySafely(rawValue, parser, encoder) {">`stringifySafely`</SwmToken> function, which accepts optional parser and encoder functions to allow customization. It ensures that data sent as JSON is correctly formatted without causing runtime errors.

```javascript
/**
 * It takes a string, tries to parse it, and if it fails, it returns the stringified version
 * of the input
 *
 * @param {any} rawValue - The value to be stringified.
 * @param {Function} parser - A function that parses a string into a JavaScript object.
 * @param {Function} encoder - A function that takes a value and returns a string.
 *
 * @returns {string} A stringified version of the rawValue.
 */
function stringifySafely(rawValue, parser, encoder) {
  if (utils.isString(rawValue)) {
    try {
      (parser || JSON.parse)(rawValue);
      return utils.trim(rawValue);
    } catch (e) {
      if (e.name !== 'SyntaxError') {
        throw e;
      }
    }
  }

  return (encoder || JSON.stringify)(rawValue);
}
```

---

</SwmSnippet>

# transforming request data based on type and headers

Axios applies a series of transformations to request data before sending it. This is done in the <SwmToken path="lib/defaults/index.js" pos="42:1:1" line-data="  transformRequest: [function transformRequest(data, headers) {">`transformRequest`</SwmToken> array, which contains a function that inspects the data type and content-type header to decide how to serialize the data.

Key points in this logic:

- If the data is an HTML form element, it converts it to <SwmToken path="lib/defaults/index.js" pos="48:7:7" line-data="      data = new FormData(data);">`FormData`</SwmToken>.
- If the data is <SwmToken path="lib/defaults/index.js" pos="48:7:7" line-data="      data = new FormData(data);">`FormData`</SwmToken> and the content-type is JSON, it converts <SwmToken path="lib/defaults/index.js" pos="48:7:7" line-data="      data = new FormData(data);">`FormData`</SwmToken> to a JSON object string.
- For binary types like ArrayBuffer, Buffer, Stream, File, Blob, or ReadableStream, it returns the data as-is.
- If the data is a URLSearchParams instance, it sets the content-type to <SwmToken path="lib/defaults/index.js" pos="70:6:14" line-data="      headers.setContentType(&#39;application/x-www-form-urlencoded;charset=utf-8&#39;, false);">`application/x-www-form-urlencoded`</SwmToken> and serializes it accordingly.
- If the content-type is <SwmToken path="lib/defaults/index.js" pos="70:6:14" line-data="      headers.setContentType(&#39;application/x-www-form-urlencoded;charset=utf-8&#39;, false);">`application/x-www-form-urlencoded`</SwmToken> and the data is an object, it serializes it using a URL-encoded form serializer.
- If the content-type is <SwmToken path="lib/defaults/index.js" pos="81:24:28" line-data="      if ((isFileList = utils.isFileList(data)) || contentType.indexOf(&#39;multipart/form-data&#39;) &gt; -1) {">`multipart/form-data`</SwmToken> or the data is a FileList, it converts the data to <SwmToken path="lib/defaults/index.js" pos="48:7:7" line-data="      data = new FormData(data);">`FormData`</SwmToken>.
- For JSON content or object payloads, it sets the content-type to <SwmToken path="lib/defaults/index.js" pos="44:12:14" line-data="    const hasJSONContentType = contentType.indexOf(&#39;application/json&#39;) &gt; -1;">`application/json`</SwmToken> and safely stringifies the data.

<SwmSnippet path="/lib/defaults/index.js" line="36">

---

This layered approach ensures that axios handles various data formats correctly and sets appropriate headers automatically.

```javascript
const defaults = {

  transitional: transitionalDefaults,

  adapter: ['xhr', 'http', 'fetch'],

  transformRequest: [function transformRequest(data, headers) {
    const contentType = headers.getContentType() || '';
    const hasJSONContentType = contentType.indexOf('application/json') > -1;
    const isObjectPayload = utils.isObject(data);

    if (isObjectPayload && utils.isHTMLForm(data)) {
      data = new FormData(data);
    }

    const isFormData = utils.isFormData(data);

    if (isFormData) {
      return hasJSONContentType ? JSON.stringify(formDataToJSON(data)) : data;
    }

    if (utils.isArrayBuffer(data) ||
      utils.isBuffer(data) ||
      utils.isStream(data) ||
      utils.isFile(data) ||
      utils.isBlob(data) ||
      utils.isReadableStream(data)
    ) {
      return data;
    }
    if (utils.isArrayBufferView(data)) {
      return data.buffer;
    }
    if (utils.isURLSearchParams(data)) {
      headers.setContentType('application/x-www-form-urlencoded;charset=utf-8', false);
      return data.toString();
    }

    let isFileList;

    if (isObjectPayload) {
      if (contentType.indexOf('application/x-www-form-urlencoded') > -1) {
        return toURLEncodedForm(data, this.formSerializer).toString();
      }

      if ((isFileList = utils.isFileList(data)) || contentType.indexOf('multipart/form-data') > -1) {
        const _FormData = this.env && this.env.FormData;

        return toFormData(
          isFileList ? {'files[]': data} : data,
          _FormData && new _FormData(),
          this.formSerializer
        );
      }
    }

    if (isObjectPayload || hasJSONContentType ) {
      headers.setContentType('application/json', false);
      return stringifySafely(data);
    }

    return data;
  }],
```

---

</SwmSnippet>

# parsing response data with JSON handling

Axios also transforms response data. The <SwmToken path="lib/defaults/index.js" pos="100:1:1" line-data="  transformResponse: [function transformResponse(data) {">`transformResponse`</SwmToken> function checks if the response data is a string and if JSON parsing is forced or requested by the <SwmToken path="lib/defaults/index.js" pos="103:9:9" line-data="    const JSONRequested = this.responseType === &#39;json&#39;;">`responseType`</SwmToken>.

- If so, it attempts to parse the string as JSON.
- If parsing fails and strict JSON parsing is enabled, it throws an <SwmToken path="lib/defaults/index.js" pos="118:3:3" line-data="            throw AxiosError.from(e, AxiosError.ERR_BAD_RESPONSE, this, null, this.response);">`AxiosError`</SwmToken> with a bad response code.
- If parsing is silent or not strict, it returns the raw data.

<SwmSnippet path="/lib/defaults/index.js" line="100">

---

This design allows axios to automatically parse JSON responses while providing options to control error handling behavior.

```javascript
  transformResponse: [function transformResponse(data) {
    const transitional = this.transitional || defaults.transitional;
    const forcedJSONParsing = transitional && transitional.forcedJSONParsing;
    const JSONRequested = this.responseType === 'json';

    if (utils.isResponse(data) || utils.isReadableStream(data)) {
      return data;
    }

    if (data && utils.isString(data) && ((forcedJSONParsing && !this.responseType) || JSONRequested)) {
      const silentJSONParsing = transitional && transitional.silentJSONParsing;
      const strictJSONParsing = !silentJSONParsing && JSONRequested;

      try {
        return JSON.parse(data);
      } catch (e) {
        if (strictJSONParsing) {
          if (e.name === 'SyntaxError') {
            throw AxiosError.from(e, AxiosError.ERR_BAD_RESPONSE, this, null, this.response);
          }
          throw e;
        }
      }
    }

    return data;
  }],
```

---

</SwmSnippet>

# default headers and environment settings

Axios sets default headers and environment variables to standardize requests:

- The common headers accept JSON, plain text, or any content type, with content-type initially undefined.
- Method-specific headers (delete, get, head, post, put, patch) are initialized as empty objects.
- The environment object exposes platform-specific classes like <SwmToken path="lib/defaults/index.js" pos="48:7:7" line-data="      data = new FormData(data);">`FormData`</SwmToken> and Blob, allowing axios to work in different environments (browser, Node.js).

<SwmSnippet path="/lib/defaults/index.js" line="140">

---

These defaults provide a baseline configuration that can be customized per request or globally.

```javascript
  env: {
    FormData: platform.classes.FormData,
    Blob: platform.classes.Blob
  },
```

---

</SwmSnippet>

<SwmSnippet path="/lib/defaults/index.js" line="149">

---

&nbsp;

```javascript
  headers: {
    common: {
      'Accept': 'application/json, text/plain, */*',
      'Content-Type': undefined
    }
  }
};

utils.forEach(['delete', 'get', 'head', 'post', 'put', 'patch'], (method) => {
  defaults.headers[method] = {};
});

export default defaults;
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBYXhpb3MlM0ElM0FTd2ltbS1EZW1v" repo-name="axios"><sup>Powered by [Swimm](/)</sup></SwmMeta>
