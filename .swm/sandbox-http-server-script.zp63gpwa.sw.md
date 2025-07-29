---
title: Sandbox HTTP Server Script
---
# introduction

This document explains the sandbox HTTP server script. It answers these questions:

1. What is the purpose of this server script?
2. How does it handle different request paths and serve files or API responses?
3. How does it process incoming data and errors?
4. How is the server started and how does it handle port conflicts?

# purpose and basic setup

The script creates a simple HTTP server for local testing and development. It serves static files and provides a basic API endpoint to echo request details. This helps test axios requests against a controlled environment.

The server is created using Node.js's built-in <SwmToken path="sandbox/server.js" pos="4:2:2" line-data="import http from &#39;http&#39;;">`http`</SwmToken> module. It listens on port 3000 by default.

# request handling and routing

The server listens for incoming requests and parses the URL path. It logs each request with its method and path for debugging.

If the path is `/`, it redirects internally to <SwmPath>[examples/all/index.html](examples/all/index.html)</SwmPath> to serve the main page.

Static files are served based on the path:

- <SwmPath>[examples/all/index.html](examples/all/index.html)</SwmPath> serves <SwmPath>[sandbox/client.html](sandbox/client.html)</SwmPath> as HTML.
- <SwmPath>[dist/axios.js](dist/axios.js)</SwmPath> serves the built axios JavaScript bundle.
- <SwmPath>[dist/axios.js.map](dist/axios.js.map)</SwmPath> serves the source map for axios.

<SwmSnippet path="/sandbox/server.js" line="1">

---

This is done by streaming the file contents directly to the response with the correct content type header. This avoids loading entire files into memory.

```javascript
import fs from 'fs';
import url from 'url';
import path from 'path';
import http from 'http';
let server;

function pipeFileToResponse(res, file, type) {
  if (type) {
    res.writeHead(200, {
      'Content-Type': type
    });
  }

  fs.createReadStream(path.join(path.resolve() ,'sandbox', file)).pipe(res);
}

server = http.createServer(function (req, res) {
  req.setEncoding('utf8');

  const parsed = url.parse(req.url, true);
  let pathname = parsed.pathname;

  console.log('[' + new Date() + ']', req.method, pathname);

  if (pathname === '/') {
    pathname = '/index.html';
  }

  if (pathname === '/index.html') {
    pipeFileToResponse(res, './client.html', 'text/html');
  } else if (pathname === '/axios.js') {
    pipeFileToResponse(res, '../dist/axios.js', 'text/javascript');
  } else if (pathname === '/axios.js.map') {
    pipeFileToResponse(res, '../dist/axios.js.map', 'text/javascript');
  } else if (pathname === '/api') {
    let status;
    let result;
    let data = '';
```

---

</SwmSnippet>

# api endpoint and data processing

Requests to <SwmToken path="sandbox/server.js" pos="35:13:14" line-data="  } else if (pathname === &#39;/api&#39;) {">`/api`</SwmToken> are handled differently. The server collects incoming data chunks and concatenates them.

Once all data is received, it tries to parse the data as JSON. If parsing succeeds, it responds with a JSON object containing:

- The original request URL
- The parsed data (if any)
- The HTTP method
- The request headers

If JSON parsing fails, it responds with a 400 status and an error message.

<SwmSnippet path="/sandbox/server.js" line="40">

---

This allows testing axios's ability to send JSON data and receive JSON responses.

```javascript
    req.on('data', function (chunk) {
      data += chunk;
    });

    req.on('end', function () {
      try {
        status = 200;
        result = {
          url: req.url,
          data: data ? JSON.parse(data) : undefined,
          method: req.method,
          headers: req.headers
        };
      } catch (e) {
        console.error('Error:', e.message);
        status = 400;
        result = {
          error: e.message
        };
      }

      res.writeHead(status, {
        'Content-Type': 'application/json'
      });
      res.end(JSON.stringify(result));
    });
```

---

</SwmSnippet>

# handling unknown paths and errors

Any request path not matched by the above routes returns a 404 response with a simple HTML message.

<SwmSnippet path="/sandbox/server.js" line="66">

---

The server also listens for errors on startup. If the port 3000 is already in use, it logs a message and closes the server to avoid crashing.

```javascript
  } else {
    res.writeHead(404);
    res.end('<h1>404 Not Found</h1>');
  }
});

const PORT = 3000;

server.listen(PORT, console.log(`Listening on localhost:${PORT}...`));
server.on('error', (error) => {
  if (error.code === 'EADDRINUSE') {
    console.log(`Address localhost:${PORT} in use please retry when the port is available!`);
    server.close();
  }
});
```

---

</SwmSnippet>

# running the server

To run the server, execute the script with Node.js:

```
node sandbox/server.js
```

It will start listening on <SwmToken path="sandbox/server.js" pos="74:16:16" line-data="server.listen(PORT, console.log(`Listening on localhost:${PORT}...`));">`localhost`</SwmToken>`:`<SwmToken path="sandbox/server.js" pos="72:6:6" line-data="const PORT = 3000;">`3000`</SwmToken>. Open a browser or use axios to make requests to this address.

If the port is busy, the script will notify you and exit gracefully.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBYXhpb3MlM0ElM0FTd2ltbS1EZW1v" repo-name="axios"><sup>Powered by [Swimm](/)</sup></SwmMeta>
