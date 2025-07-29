---
title: Progress Throttle Utilities in HTTP Helpers
---
# Overview of Progress Throttle Utils

Progress Throttle Utils in Helpers consist of utility functions designed to manage the throttling of progress events during HTTP requests and responses. Their primary role is to control the frequency of progress updates, preventing excessive event emissions that could degrade system performance or overwhelm the user interface.

# Core Throttling Mechanism

At the heart of these utilities is the `throttle` function. This function restricts how often a given callback can be executed by tracking the timestamp of the last invocation. If a subsequent call occurs before a specified threshold time has elapsed, the function schedules the callback to run after the remaining delay. This approach ensures that rapid, consecutive calls are batched and handled efficiently.

# Role of progressEventDecorator

Building upon the `throttle` function, the `progressEventDecorator` wraps progress event handlers to apply throttling based on the total content length and the amount of data loaded. It returns two elements: a throttled progress event handler that emits controlled progress updates, and a flush function that can be called to immediately trigger any pending updates. This design allows for precise and efficient management of progress events during data transfer.

# Integration with HTTP Adapter

These utilities are integrated within the HTTP adapter, specifically in functions like `dispatchHttpRequest` and `handleResponse`. For example, in `dispatchHttpRequest`, the `progressEventDecorator` is used with the content length and a throttled function to create a throttled handler. This handler manages upload progress events, ensuring updates are emitted at a controlled frequency, which enhances both performance and user experience.

# Example Usage in dispatchHttpRequest

Within the `dispatchHttpRequest` function, the `progressEventDecorator` is invoked with the total content length and a throttled callback function. This produces a throttled progress event handler that is then used to report upload progress. By controlling the rate of progress event emissions, this mechanism prevents UI flooding and optimizes resource usage during HTTP data uploads.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBYXhpb3MlM0ElM0FTd2ltbS1EZW1v" repo-name="axios"><sup>Powered by [Swimm](/)</sup></SwmMeta>
