---
title: Defaults Configuration in Core Library
---
# Overview of Defaults

Defaults is a fundamental configuration object within the core library that establishes the baseline settings applied to every HTTP request. It ensures consistent behavior by defining standard properties such as adapters, transformers, timeout values, headers, and validation functions.

# Data Transformation in Defaults

The properties `transformRequest` and `transformResponse` are arrays of functions responsible for processing data at different stages of the HTTP lifecycle. `transformRequest` functions prepare and serialize data before it is sent, handling formats like JSON, FormData, and URLSearchParams. Conversely, `transformResponse` functions parse and transform incoming response data, including JSON parsing with configurable options to control strictness and error handling.

# Transitional Flags for Compatibility

Within Defaults, the `transitional` property contains flags that manage behaviors related to JSON parsing and error handling. These flags, such as `silentJSONParsing` and `forcedJSONParsing`, provide backward compatibility and allow gradual adoption of new features, offering flexibility in how responses are processed and errors are handled.

# Environment-Specific Compatibility

Defaults include environment-specific classes like `FormData` and `Blob` to maintain compatibility across different platforms, such as browsers and Node.js. This design ensures that the HTTP client operates seamlessly regardless of the runtime environment.

# Headers Configuration

The Defaults object preconfigures HTTP headers with common values applicable to all methods, including a default `Accept` header. The `Content-Type` header is initially undefined, allowing customization per request. Additionally, each HTTP method (e.g., GET, POST, PUT) has its own header object initialized as empty, enabling method-specific header customization.

# Usage in Creating Default Instance

In the file <SwmPath>[lib/axios.js](lib/axios.js)</SwmPath>, the default axios instance is created by invoking `createInstance` with the Defaults object. This approach ensures that every request made through this instance inherits the base configuration, including adapters, transformers, headers, and timeout settings, thereby maintaining consistent behavior across all HTTP requests.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBYXhpb3MlM0ElM0FTd2ltbS1EZW1v" repo-name="axios"><sup>Powered by [Swimm](/)</sup></SwmMeta>
