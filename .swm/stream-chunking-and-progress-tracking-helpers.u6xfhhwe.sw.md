---
title: Stream Chunking and Progress Tracking Helpers
---
# introduction

This document explains the implementation of stream chunking and progress tracking helpers in <SwmPath>[lib/helpers/trackStream.js](lib/helpers/trackStream.js)</SwmPath>. We will cover:

1. How the code splits large data chunks into smaller pieces.
2. How it reads from different types of streams uniformly.
3. How it tracks progress and signals completion during streaming.

# chunking large data

<SwmSnippet path="/lib/helpers/trackStream.js" line="2">

---

The function that breaks down large chunks into smaller ones is a generator that yields slices of the original chunk. If the chunk is smaller than the requested chunk size, it yields it as is. Otherwise, it iterates through the chunk, slicing it into pieces of the specified size and yielding each piece. This approach avoids loading the entire chunk into memory at once and allows processing data in manageable parts.

```javascript
export const streamChunk = function* (chunk, chunkSize) {
  let len = chunk.byteLength;

  if (!chunkSize || len < chunkSize) {
    yield chunk;
    return;
  }

  let pos = 0;
  let end;

  while (pos < len) {
    end = pos + chunkSize;
    yield chunk.slice(pos, end);
    pos = end;
  }
}
```

---

</SwmSnippet>

# reading from streams uniformly

<SwmSnippet path="/lib/helpers/trackStream.js" line="20">

---

To handle different stream types, the code uses an async generator that reads bytes from the input. If the input already supports async iteration, it yields chunks directly. Otherwise, it uses the stream’s reader interface to read chunks asynchronously until done. This abstraction lets the rest of the code work with any stream-like input without worrying about its specific interface.

```javascript
export const readBytes = async function* (iterable, chunkSize) {
  for await (const chunk of readStream(iterable)) {
    yield* streamChunk(chunk, chunkSize);
  }
}

const readStream = async function* (stream) {
  if (stream[Symbol.asyncIterator]) {
    yield* stream;
    return;
  }

  const reader = stream.getReader();
  try {
    for (;;) {
      const {done, value} = await reader.read();
      if (done) {
        break;
      }
      yield value;
    }
  } finally {
    await reader.cancel();
  }
}
```

---

</SwmSnippet>

# tracking progress and signaling finish

The main function wraps the reading and chunking logic into a new <SwmToken path="lib/helpers/trackStream.js" pos="58:5:5" line-data="  return new ReadableStream({">`ReadableStream`</SwmToken>. It creates an iterator from the byte reader with chunking applied. It keeps track of the total bytes processed and calls an <SwmToken path="lib/helpers/trackStream.js" pos="46:15:15" line-data="export const trackStream = (stream, chunkSize, onProgress, onFinish) =&gt; {">`onProgress`</SwmToken> callback with the updated count after each chunk. When the stream ends or is canceled, it calls an <SwmToken path="lib/helpers/trackStream.js" pos="46:18:18" line-data="export const trackStream = (stream, chunkSize, onProgress, onFinish) =&gt; {">`onFinish`</SwmToken> callback once, ensuring it only triggers once even if multiple finish signals occur.

Inside the <SwmToken path="lib/helpers/trackStream.js" pos="58:5:5" line-data="  return new ReadableStream({">`ReadableStream`</SwmToken>’s pull method, it awaits the next chunk from the iterator. If done, it closes the stream and calls <SwmToken path="lib/helpers/trackStream.js" pos="46:18:18" line-data="export const trackStream = (stream, chunkSize, onProgress, onFinish) =&gt; {">`onFinish`</SwmToken>. Otherwise, it updates the byte count, calls <SwmToken path="lib/helpers/trackStream.js" pos="46:15:15" line-data="export const trackStream = (stream, chunkSize, onProgress, onFinish) =&gt; {">`onProgress`</SwmToken>, and enqueues the chunk for downstream consumption. The cancel method also triggers <SwmToken path="lib/helpers/trackStream.js" pos="46:18:18" line-data="export const trackStream = (stream, chunkSize, onProgress, onFinish) =&gt; {">`onFinish`</SwmToken> and attempts to clean up the iterator.

<SwmSnippet path="/lib/helpers/trackStream.js" line="46">

---

The stream is created with a small <SwmToken path="lib/helpers/trackStream.js" pos="85:1:1" line-data="    highWaterMark: 2">`highWaterMark`</SwmToken> to control backpressure and avoid buffering too much data.

```javascript
export const trackStream = (stream, chunkSize, onProgress, onFinish) => {
  const iterator = readBytes(stream, chunkSize);

  let bytes = 0;
  let done;
  let _onFinish = (e) => {
    if (!done) {
      done = true;
      onFinish && onFinish(e);
    }
  }

  return new ReadableStream({
    async pull(controller) {
      try {
        const {done, value} = await iterator.next();

        if (done) {
         _onFinish();
          controller.close();
          return;
        }

        let len = value.byteLength;
        if (onProgress) {
          let loadedBytes = bytes += len;
          onProgress(loadedBytes);
        }
        controller.enqueue(new Uint8Array(value));
      } catch (err) {
        _onFinish(err);
        throw err;
      }
    },
    cancel(reason) {
      _onFinish(reason);
      return iterator.return();
    }
  }, {
    highWaterMark: 2
  })
}
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBYXhpb3MlM0ElM0FTd2ltbS1EZW1v" repo-name="axios"><sup>Powered by [Swimm](/)</sup></SwmMeta>
