---
title: Throttle Function Utility
---
# introduction

This document explains the design and implementation of the throttle utility in <SwmPath>[lib/helpers/throttle.js](lib/helpers/throttle.js)</SwmPath>. It answers these questions:

1. Why do we need a throttle function in axios?
2. How does the throttle function control the invocation rate of another function?
3. What mechanisms are used to handle calls that happen too frequently?

# why throttle is needed

In axios, certain operations like event handling or repeated calls can happen too often, causing performance issues or redundant processing. The throttle utility limits how often a function can run per unit time, ensuring that it doesn't execute more frequently than a specified rate. This helps reduce unnecessary work and improves efficiency.

# how throttle controls invocation rate

The throttle function takes two parameters: the function to throttle (fn) and the frequency (freq) in calls per second. It calculates a threshold interval (in milliseconds) between allowed calls. When the returned throttled function is called, it checks how much time has passed since the last invocation. If enough time has passed (greater than or equal to the threshold), it immediately calls the original function with the current arguments.

<SwmSnippet path="/lib/helpers/throttle.js" line="1">

---

If not enough time has passed, it schedules the function to be called later with the most recent arguments, ensuring the function is not called more often than the frequency limit.

```javascript
/**
 * Throttle decorator
 * @param {Function} fn
 * @param {Number} freq
 * @return {Function}
 */
function throttle(fn, freq) {
  let timestamp = 0;
  let threshold = 1000 / freq;
  let lastArgs;
  let timer;
```

---

</SwmSnippet>

# handling calls that happen too frequently

When calls come in faster than allowed, the throttle function stores the latest arguments and sets a timer to invoke the function once the threshold time has elapsed. If a timer is already set, it doesn't create a new one, preventing multiple queued calls.

The invoke helper clears any existing timer, resets the stored arguments, updates the timestamp, and calls the original function with the provided arguments.

<SwmSnippet path="/lib/helpers/throttle.js" line="13">

---

Additionally, a flush method is provided to immediately invoke the function with the last stored arguments if any are pending. This can be useful to force execution before the timer fires.

```javascript
  const invoke = (args, now = Date.now()) => {
    timestamp = now;
    lastArgs = null;
    if (timer) {
      clearTimeout(timer);
      timer = null;
    }
    fn(...args);
  }

  const throttled = (...args) => {
    const now = Date.now();
    const passed = now - timestamp;
    if ( passed >= threshold) {
      invoke(args, now);
    } else {
      lastArgs = args;
      if (!timer) {
        timer = setTimeout(() => {
          timer = null;
          invoke(lastArgs)
        }, threshold - passed);
      }
    }
  }

  const flush = () => lastArgs && invoke(lastArgs);

  return [throttled, flush];
}
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBYXhpb3MlM0ElM0FTd2ltbS1EZW1v" repo-name="axios"><sup>Powered by [Swimm](/)</sup></SwmMeta>
