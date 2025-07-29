---
title: Axios Sandbox Client HTML UI
---
# introduction

This document explains the design and implementation of the Axios sandbox client HTML UI. It answers these main questions:

1. How is the UI structured to allow input of HTTP request details?
2. How does the UI handle different HTTP methods and their data requirements?
3. How are user inputs validated and persisted?
4. How is the Axios request constructed and executed from the UI?

# ui structure and layout

The UI is built as a simple HTML page styled with Bootstrap and custom CSS for layout and responsiveness. It divides the interface into two main sections side-by-side: input controls on the left and output display on the right. The input section contains fields for URL, HTTP method, parameters, data, and headers, plus a submit button. The output section shows the raw request, response, and any error messages.

<SwmSnippet path="/sandbox/client.html" line="1">

---

This layout is defined in the HTML structure and CSS grid styles, enabling a clear separation of concerns and easy user interaction.

```html
<!doctype html>
<html>
<head>
  <title>AXIOS | Sandbox</title>
  <link rel="stylesheet" type="text/css" href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css"/>
  <style type="text/css">
    pre {
      min-height: 39px;
      overflow: auto;
    }
    .header{
      display: flex;
      flex-direction: row;
    }
    .box{
      display: grid;
      grid-template-columns: 1fr 1fr;
      grid-gap: 25px;
    }
    .well {
      max-width: 400px;
    }
    @media screen and (max-width: 1000px) {
      .box {
        grid-template-columns: 1fr;
      }
    }
  </style>
</head>
<body class="container">
<div class="header">
  <img src="https://axios-http.com/assets/logo.svg" alt="axios" width="100" height="60">
  <h1> &nbsp;| Sandbox</h1>
</div>

<div class="box">
  <div class="well">
    <h3>Input</h3>
    <form role="form" onsubmit="return false;">
      <div class="form-group">
        <label for="url">URL</label>
        <input id="url" type="url" class="form-control" placeholder="/api"/>
      </div>
      <div class="form-group">
        <label for="method">Method</label>
        <select id="method" class="form-control">
          <option value="GET">GET</option>
          <option value="POST">POST</option>
          <option value="PUT">PUT</option>
          <option value="DELETE">DELETE</option>
          <option value="HEAD">HEAD</option>
          <option value="PATCH">PATCH</option>
        </select>
      </div>
      <div class="form-group">
        <label for="params">Params</label>
        <textarea id="params" class="form-control" placeholder='{"foo": "bar", "baz": 123.45}'></textarea>
      </div>
      <div class="form-group" style="display: none;">
        <label for="data">Data</label>
        <textarea id="data" class="form-control" placeholder='{"foo": "bar", "baz": 123.45}'></textarea>
      </div>
      <div class="form-group">
        <label for="headers">Headers</label>
        <textarea id="headers" class="form-control" placeholder='{"X-Requested-With": "XMLHttpRequest"}'></textarea>
      </div>
      <button id="submit" type="submit" class="btn btn-primary">Send Request</button>
    </form>
  </div>

  <div class="response">
    <div class="well">
      <h3>Request</h3>
      <pre id="request">No Data</pre>
    </div>

    <div class="well">
      <h3>Response</h3>
      <pre id="response">No Data</pre>
    </div>

    <div class="well">
      <h3>Error</h3>
      <pre id="error">None</pre>
    </div>
  </div>
</div>
```

---

</SwmSnippet>

# handling http methods and input fields

The UI distinguishes between HTTP methods that accept a request body (PATCH, POST, PUT) and those that do not. This affects which input fields are shown:

- For methods that accept data, the "Params" field is hidden and the "Data" field is shown.
- For other methods, the "Params" field is visible and the "Data" field is hidden.

<SwmSnippet path="/sandbox/client.html" line="102">

---

This dynamic toggling is controlled by the <SwmToken path="sandbox/client.html" pos="145:3:3" line-data="    function syncParamsAndData() {">`syncParamsAndData`</SwmToken> function, which runs on method change to keep the UI consistent with the selected HTTP method.

```html
    function acceptsData(method) {
      return ['PATCH', 'POST', 'PUT'].indexOf(method) > -1;
    }
```

---

</SwmSnippet>

<SwmSnippet path="/sandbox/client.html" line="145">

---

&nbsp;

```html
    function syncParamsAndData() {
      switch (method.value) {
        case 'PATCH':
        case 'POST':
        case 'PUT':
          params.parentNode.style.display = 'none';
          data.parentNode.style.display = '';
          break;
        default:
          params.parentNode.style.display = '';
          data.parentNode.style.display = 'none';
          break;
      }
    }
```

---

</SwmSnippet>

<SwmSnippet path="/sandbox/client.html" line="198">

---

&nbsp;

```html
    method.onchange = function () {
      localStorage.setItem('method', method.value);
      syncParamsAndData();
    };
```

---

</SwmSnippet>

# input validation and local storage syncing

User inputs for URL, params, data, and headers are parsed as JSON where applicable. If parsing fails, an error message is displayed. This prevents malformed JSON from causing runtime errors.

The UI also persists all input fields in <SwmToken path="sandbox/client.html" pos="138:7:7" line-data="      url.value = localStorage.getItem(&#39;url&#39;) || &#39;/api&#39;;">`localStorage`</SwmToken>, so user entries survive page reloads. This syncing happens on input changes and on page load, restoring previous values for convenience.

<SwmSnippet path="/sandbox/client.html" line="110">

---

These mechanisms ensure input correctness and improve user experience by remembering their last inputs.

```html
    function getParams() {
      try {
        return params.value.length === 0 ? null : JSON.parse(params.value);
      } catch (e) {
        error.textContent = "Invalid JSON in Params";
        return null;
      }
    }

    function getData() {
      try {
        return data.value.length === 0 ? null : JSON.parse(data.value);
      } catch (e) {
        error.textContent = "Invalid JSON in Data";
        return null;
      }
    }

    function getHeaders() {
      try {
        return headers.value.length === 0 ? null : JSON.parse(headers.value);
      } catch (e) {
        error.textContent = "Invalid JSON in Headers";
        return null;
      }
    }

    function syncWithLocalStorage() {
      url.value = localStorage.getItem('url') || '/api';
      method.value = localStorage.getItem('method') || 'GET';
      params.value = localStorage.getItem('params') || '';
      data.value = localStorage.getItem('data') || '';
      headers.value = localStorage.getItem('headers') || '';
    }
```

---

</SwmSnippet>

<SwmSnippet path="/sandbox/client.html" line="194">

---

&nbsp;

```html
    url.onchange = function () {
      localStorage.setItem('url', url.value);
    };
```

---

</SwmSnippet>

<SwmSnippet path="/sandbox/client.html" line="203">

---

&nbsp;

```html
    params.onchange = function () {
      localStorage.setItem('params', params.value);
    };

    data.onchange = function () {
      localStorage.setItem('data', data.value);
    };

    headers.onchange = function () {
      localStorage.setItem('headers', headers.value);
    };

    syncWithLocalStorage();
    syncParamsAndData();
  })();
```

---

</SwmSnippet>

# constructing and sending the axios request

When the user clicks "Send Request," the UI gathers all inputs, validates the URL, and builds an options object for Axios. It conditionally assigns <SwmToken path="sandbox/client.html" pos="56:7:7" line-data="        &lt;label for=&quot;params&quot;&gt;Params&lt;/label&gt;">`params`</SwmToken> or <SwmToken path="sandbox/client.html" pos="60:7:7" line-data="        &lt;label for=&quot;data&quot;&gt;Data&lt;/label&gt;">`data`</SwmToken> based on the HTTP method. The request details are displayed in the "Request" panel as formatted JSON.

The Axios call is then made with these options. On success, the response data is shown in the "Response" panel and errors are cleared. On failure, the error message or response error data is displayed in the "Error" panel, and the response panel indicates an error occurred.

<SwmSnippet path="/sandbox/client.html" line="160">

---

This flow provides immediate feedback on the request lifecycle and results.

```html
    submit.onclick = function (event) {
      event.preventDefault();

      if (url.value === '') {
        error.textContent = 'Please enter a valid URL';
        return;
      }

      var options = {
        url: getUrl(),
        params: !acceptsData(method.value) ? getParams() : undefined,
        data: acceptsData(method.value) ? getData() : undefined,
        method: method.value,
        headers: getHeaders()
      };

      request.textContent = JSON.stringify(options, null, 2);

      axios(options)
        .then(function (res) {
          response.innerHTML = JSON.stringify(res.data, null, 2);
          error.textContent = "None";
        })
        .catch(function (err) {
          if (err.response) {
            error.textContent = JSON.stringify(err.response.data, null, 2);
            response.innerHTML = "Error in Response";
          } else {
            error.textContent = err.message;
            response.innerHTML = "No Response Data";
          }
        });
    };
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBYXhpb3MlM0ElM0FTd2ltbS1EZW1v" repo-name="axios"><sup>Powered by [Swimm](/)</sup></SwmMeta>
