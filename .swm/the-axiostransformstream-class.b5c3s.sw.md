---
title: The AxiosTransformStream class
---
This document covers the <SwmToken path="lib/helpers/AxiosTransformStream.js" pos="8:2:2" line-data="class AxiosTransformStream extends stream.Transform{">`AxiosTransformStream`</SwmToken> class. We'll address:

1. What <SwmToken path="lib/helpers/AxiosTransformStream.js" pos="8:2:2" line-data="class AxiosTransformStream extends stream.Transform{">`AxiosTransformStream`</SwmToken> is and its purpose
2. The key functions \_read and \_transform
3. All variables and functions defined in <SwmToken path="lib/helpers/AxiosTransformStream.js" pos="8:2:2" line-data="class AxiosTransformStream extends stream.Transform{">`AxiosTransformStream`</SwmToken>

# What is <SwmToken path="lib/helpers/AxiosTransformStream.js" pos="8:2:2" line-data="class AxiosTransformStream extends stream.Transform{">`AxiosTransformStream`</SwmToken>

<SwmToken path="lib/helpers/AxiosTransformStream.js" pos="8:2:2" line-data="class AxiosTransformStream extends stream.Transform{">`AxiosTransformStream`</SwmToken> is a custom <SwmToken path="lib/helpers/AxiosTransformStream.js" pos="8:6:8" line-data="class AxiosTransformStream extends stream.Transform{">`stream.Transform`</SwmToken> subclass designed to control and monitor the flow of data through streams, particularly for throttling upload and download rates in HTTP requests. It is used internally by Axios to provide rate limiting and progress reporting for data streams, making it possible to control how quickly data is read from or written to a stream. This is especially useful for managing bandwidth usage and providing progress updates during large file transfers.

<SwmSnippet path="/lib/helpers/AxiosTransformStream.js" line="47">

---

The \_read function is an override of the <SwmToken path="lib/helpers/AxiosTransformStream.js" pos="8:6:8" line-data="class AxiosTransformStream extends stream.Transform{">`stream.Transform`</SwmToken> \_read method. It checks if there is a pending <SwmToken path="lib/helpers/AxiosTransformStream.js" pos="50:6:6" line-data="    if (internals.onReadCallback) {">`onReadCallback`</SwmToken> in the internal state and, if so, executes it before delegating to the parent class's \_read. This mechanism is used to resume reading from the stream when backpressure is relieved or when a chunk has finished processing.

```javascript
  _read(size) {
    const internals = this[kInternals];

    if (internals.onReadCallback) {
      internals.onReadCallback();
    }

    return super._read(size);
  }
```

---

</SwmSnippet>

<SwmSnippet path="/lib/helpers/AxiosTransformStream.js" line="57">

---

The \_transform function is the core of <SwmToken path="lib/helpers/AxiosTransformStream.js" pos="8:2:2" line-data="class AxiosTransformStream extends stream.Transform{">`AxiosTransformStream`</SwmToken>. It processes incoming data chunks, applies rate limiting based on the configured <SwmToken path="lib/helpers/AxiosTransformStream.js" pos="59:3:3" line-data="    const maxRate = internals.maxRate;">`maxRate`</SwmToken>, and emits progress events. It uses helper functions to split large chunks, delay processing to enforce bandwidth limits, and push data downstream. The function ensures that data is processed in manageable pieces and that the stream does not exceed the specified rate.

```javascript
  _transform(chunk, encoding, callback) {
    const internals = this[kInternals];
    const maxRate = internals.maxRate;

    const readableHighWaterMark = this.readableHighWaterMark;

    const timeWindow = internals.timeWindow;

    const divider = 1000 / timeWindow;
    const bytesThreshold = (maxRate / divider);
    const minChunkSize = internals.minChunkSize !== false ? Math.max(internals.minChunkSize, bytesThreshold * 0.01) : 0;

    const pushChunk = (_chunk, _callback) => {
      const bytes = Buffer.byteLength(_chunk);
      internals.bytesSeen += bytes;
      internals.bytes += bytes;

      internals.isCaptured && this.emit('progress', internals.bytesSeen);

      if (this.push(_chunk)) {
        process.nextTick(_callback);
      } else {
        internals.onReadCallback = () => {
          internals.onReadCallback = null;
          process.nextTick(_callback);
        };
      }
    }

    const transformChunk = (_chunk, _callback) => {
      const chunkSize = Buffer.byteLength(_chunk);
      let chunkRemainder = null;
      let maxChunkSize = readableHighWaterMark;
      let bytesLeft;
      let passed = 0;

      if (maxRate) {
        const now = Date.now();

        if (!internals.ts || (passed = (now - internals.ts)) >= timeWindow) {
          internals.ts = now;
          bytesLeft = bytesThreshold - internals.bytes;
          internals.bytes = bytesLeft < 0 ? -bytesLeft : 0;
          passed = 0;
        }

        bytesLeft = bytesThreshold - internals.bytes;
      }

      if (maxRate) {
        if (bytesLeft <= 0) {
          // next time window
          return setTimeout(() => {
            _callback(null, _chunk);
          }, timeWindow - passed);
        }

        if (bytesLeft < maxChunkSize) {
          maxChunkSize = bytesLeft;
        }
      }

      if (maxChunkSize && chunkSize > maxChunkSize && (chunkSize - maxChunkSize) > minChunkSize) {
        chunkRemainder = _chunk.subarray(maxChunkSize);
        _chunk = _chunk.subarray(0, maxChunkSize);
      }

      pushChunk(_chunk, chunkRemainder ? () => {
        process.nextTick(_callback, null, chunkRemainder);
      } : _callback);
    };

    transformChunk(chunk, function transformNextChunk(err, _chunk) {
      if (err) {
        return callback(err);
      }

      if (_chunk) {
        transformChunk(_chunk, transformNextChunk);
      } else {
        callback(null);
      }
    });
  }
```

---

</SwmSnippet>

<SwmSnippet path="/lib/helpers/AxiosTransformStream.js" line="6">

---

<SwmToken path="lib/helpers/AxiosTransformStream.js" pos="6:2:2" line-data="const kInternals = Symbol(&#39;internals&#39;);">`kInternals`</SwmToken> is a Symbol used as a private key to store internal state for each <SwmToken path="lib/helpers/AxiosTransformStream.js" pos="8:2:2" line-data="class AxiosTransformStream extends stream.Transform{">`AxiosTransformStream`</SwmToken> instance. This helps encapsulate stream-specific data and avoid property name collisions.

```javascript
const kInternals = Symbol('internals');
```

---

</SwmSnippet>

<SwmSnippet path="/lib/helpers/AxiosTransformStream.js" line="25">

---

The internals object holds the stream's configuration and runtime state, including <SwmToken path="lib/helpers/AxiosTransformStream.js" pos="26:1:1" line-data="      timeWindow: options.timeWindow,">`timeWindow`</SwmToken>, <SwmToken path="lib/helpers/AxiosTransformStream.js" pos="27:1:1" line-data="      chunkSize: options.chunkSize,">`chunkSize`</SwmToken>, <SwmToken path="lib/helpers/AxiosTransformStream.js" pos="28:1:1" line-data="      maxRate: options.maxRate,">`maxRate`</SwmToken>, <SwmToken path="lib/helpers/AxiosTransformStream.js" pos="29:1:1" line-data="      minChunkSize: options.minChunkSize,">`minChunkSize`</SwmToken>, <SwmToken path="lib/helpers/AxiosTransformStream.js" pos="30:1:1" line-data="      bytesSeen: 0,">`bytesSeen`</SwmToken>, <SwmToken path="lib/helpers/AxiosTransformStream.js" pos="31:1:1" line-data="      isCaptured: false,">`isCaptured`</SwmToken>, <SwmToken path="lib/helpers/AxiosTransformStream.js" pos="32:1:1" line-data="      notifiedBytesLoaded: 0,">`notifiedBytesLoaded`</SwmToken>, ts (timestamp), bytes, and <SwmToken path="lib/helpers/AxiosTransformStream.js" pos="35:1:1" line-data="      onReadCallback: null">`onReadCallback`</SwmToken>. This object is used throughout the class to manage throttling, progress, and chunk handling.

```javascript
    const internals = this[kInternals] = {
      timeWindow: options.timeWindow,
      chunkSize: options.chunkSize,
      maxRate: options.maxRate,
      minChunkSize: options.minChunkSize,
      bytesSeen: 0,
      isCaptured: false,
      notifiedBytesLoaded: 0,
      ts: Date.now(),
      bytes: 0,
      onReadCallback: null
    };
```

---

</SwmSnippet>

<SwmSnippet path="/lib/helpers/AxiosTransformStream.js" line="69">

---

<SwmToken path="lib/helpers/AxiosTransformStream.js" pos="69:3:3" line-data="    const pushChunk = (_chunk, _callback) =&gt; {">`pushChunk`</SwmToken> is a helper function inside \_transform that pushes a chunk downstream, updates byte counters, emits progress events if needed, and manages backpressure by deferring the callback if the stream buffer is full.

```javascript
    const pushChunk = (_chunk, _callback) => {
      const bytes = Buffer.byteLength(_chunk);
      internals.bytesSeen += bytes;
      internals.bytes += bytes;

      internals.isCaptured && this.emit('progress', internals.bytesSeen);

      if (this.push(_chunk)) {
        process.nextTick(_callback);
      } else {
        internals.onReadCallback = () => {
          internals.onReadCallback = null;
          process.nextTick(_callback);
        };
      }
    }
```

---

</SwmSnippet>

<SwmSnippet path="/lib/helpers/AxiosTransformStream.js" line="86">

---

<SwmToken path="lib/helpers/AxiosTransformStream.js" pos="86:3:3" line-data="    const transformChunk = (_chunk, _callback) =&gt; {">`transformChunk`</SwmToken> is a helper function inside \_transform that handles splitting large chunks, enforcing rate limits, and recursively processing any remaining data. It ensures that each chunk respects the configured bandwidth and chunk size constraints.

```javascript
    const transformChunk = (_chunk, _callback) => {
      const chunkSize = Buffer.byteLength(_chunk);
      let chunkRemainder = null;
      let maxChunkSize = readableHighWaterMark;
      let bytesLeft;
      let passed = 0;

      if (maxRate) {
        const now = Date.now();

        if (!internals.ts || (passed = (now - internals.ts)) >= timeWindow) {
          internals.ts = now;
          bytesLeft = bytesThreshold - internals.bytes;
          internals.bytes = bytesLeft < 0 ? -bytesLeft : 0;
          passed = 0;
        }

        bytesLeft = bytesThreshold - internals.bytes;
      }

      if (maxRate) {
        if (bytesLeft <= 0) {
          // next time window
          return setTimeout(() => {
            _callback(null, _chunk);
          }, timeWindow - passed);
        }

        if (bytesLeft < maxChunkSize) {
          maxChunkSize = bytesLeft;
        }
      }

      if (maxChunkSize && chunkSize > maxChunkSize && (chunkSize - maxChunkSize) > minChunkSize) {
        chunkRemainder = _chunk.subarray(maxChunkSize);
        _chunk = _chunk.subarray(0, maxChunkSize);
      }

      pushChunk(_chunk, chunkRemainder ? () => {
        process.nextTick(_callback, null, chunkRemainder);
      } : _callback);
    };
```

---

</SwmSnippet>

<SwmSnippet path="/lib/helpers/AxiosTransformStream.js" line="91">

---

The variable passed is used within <SwmToken path="lib/helpers/AxiosTransformStream.js" pos="86:3:3" line-data="    const transformChunk = (_chunk, _callback) =&gt; {">`transformChunk`</SwmToken> to track the elapsed time in the current time window, which is essential for enforcing rate limits.

```javascript
      let passed = 0;

      if (maxRate) {
        const now = Date.now();

        if (!internals.ts || (passed = (now - internals.ts)) >= timeWindow) {
          internals.ts = now;
          bytesLeft = bytesThreshold - internals.bytes;
          internals.bytes = bytesLeft < 0 ? -bytesLeft : 0;
          passed = 0;
        }
```

---

</SwmSnippet>

<SwmSnippet path="/lib/helpers/AxiosTransformStream.js" line="31">

---

<SwmToken path="lib/helpers/AxiosTransformStream.js" pos="31:1:1" line-data="      isCaptured: false,">`isCaptured`</SwmToken> is a flag in the internals object that tracks whether a 'progress' event listener has been attached. This ensures that progress events are only emitted when needed.

```javascript
      isCaptured: false,
```

---

</SwmSnippet>

<SwmSnippet path="/lib/helpers/AxiosTransformStream.js" line="30">

---

<SwmToken path="lib/helpers/AxiosTransformStream.js" pos="30:1:1" line-data="      bytesSeen: 0,">`bytesSeen`</SwmToken> is a counter in the internals object that tracks the total number of bytes processed by the stream. It is used for progress reporting.

```javascript
      bytesSeen: 0,
```

---

</SwmSnippet>

<SwmSnippet path="/lib/helpers/AxiosTransformStream.js" line="34">

---

bytes is a counter in the internals object that tracks the number of bytes processed in the current time window, which is used for rate limiting.

```javascript
      bytes: 0,
```

---

</SwmSnippet>

# Usage

## <SwmToken path="lib/helpers/AxiosTransformStream.js" pos="8:2:2" line-data="class AxiosTransformStream extends stream.Transform{">`AxiosTransformStream`</SwmToken>

<SwmToken path="lib/helpers/AxiosTransformStream.js" pos="8:2:2" line-data="class AxiosTransformStream extends stream.Transform{">`AxiosTransformStream`</SwmToken> is utilized in the HTTP adapter to manage streaming data with rate limiting capabilities. For example, during data upload, the stream pipeline incorporates <SwmToken path="lib/helpers/AxiosTransformStream.js" pos="8:2:2" line-data="class AxiosTransformStream extends stream.Transform{">`AxiosTransformStream`</SwmToken> to enforce a maximum upload rate, ensuring that data is sent at a controlled speed. Similarly, during data download, <SwmToken path="lib/helpers/AxiosTransformStream.js" pos="8:2:2" line-data="class AxiosTransformStream extends stream.Transform{">`AxiosTransformStream`</SwmToken> is instantiated with a maximum download rate to regulate the speed at which response data is received. This usage helps in scenarios where bandwidth throttling or progress monitoring is required.

## Upload Rate Limiting

When uploading data, the HTTP adapter converts the data into a readable stream and pipes it through <SwmToken path="lib/helpers/AxiosTransformStream.js" pos="8:2:2" line-data="class AxiosTransformStream extends stream.Transform{">`AxiosTransformStream`</SwmToken>. The <SwmToken path="lib/helpers/AxiosTransformStream.js" pos="28:1:1" line-data="      maxRate: options.maxRate,">`maxRate`</SwmToken> option is set based on the configured maximum upload rate, which is converted to a finite number. This setup allows the upload process to be throttled, preventing excessive bandwidth consumption and enabling smoother progress tracking.

## Download Rate Limiting

For downloading, if either a download progress callback or a maximum download rate is specified, the HTTP adapter creates an instance of <SwmToken path="lib/helpers/AxiosTransformStream.js" pos="8:2:2" line-data="class AxiosTransformStream extends stream.Transform{">`AxiosTransformStream`</SwmToken> with the <SwmToken path="lib/helpers/AxiosTransformStream.js" pos="28:1:1" line-data="      maxRate: options.maxRate,">`maxRate`</SwmToken> option set accordingly. This transform stream is then used to control the flow of incoming data, allowing the client to limit the download speed and handle progress events effectively.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBYXhpb3MlM0ElM0FTd2ltbS1EZW1v" repo-name="axios"><sup>Powered by [Swimm](/)</sup></SwmMeta>
