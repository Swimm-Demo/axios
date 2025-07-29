---
title: FormData Streams Helper Overview
---
# Introduction to FormData Streams Helper

FormData Streams Helper is a utility within the codebase that transforms a FormData instance into a readable stream compatible with HTTP multipart/form-data requests. This approach enables efficient transmission of form data, including files and binary content, by streaming rather than buffering the entire payload in memory.

# How FormData Streams Helper Works

The helper processes each entry in the FormData by wrapping key-value pairs into instances of the FormDataPart class. Each FormDataPart is responsible for encoding its headers and content, supporting both string and binary data types. Proper multipart formatting is ensured by appending carriage return and line feed sequences. The stream is constructed by sequentially concatenating boundary markers, encoded parts, and a closing footer boundary.

# Implementation Details

The core function, `formDataToStream`, generates a unique boundary string and calculates the total content length by summing the sizes of all parts and boundaries. It returns a Node.js readable stream created from an asynchronous generator that yields each boundary and encoded part in order. The `FormDataPart` class's `encode` method asynchronously yields headers and content, utilizing a helper function to read Blob-like objects in a way that supports various environments and data types.

# Integration in HTTP Requests

Within the HTTP adapter, when the request data is identified as a FormData instance, `formDataToStream` converts it into a stream. The helper also sets appropriate multipart headers, including `Content-Type` with the boundary and `Content-Length` if available. This integration allows Axios to send multipart form data efficiently and correctly over HTTP.

# Example Usage

An example from the HTTP adapter demonstrates the conversion of FormData to a stream and the updating of headers: `data = formDataToStream(data, (formHeaders) => { headers.set(formHeaders); });`. This shows how the helper is used internally before dispatching the HTTP request.

# Key Functions

Two primary functions underpin the FormData Streams Helper: `formDataToStream` and the `encode` method of the `FormDataPart` class. The former manages the overall stream creation and header calculation, while the latter handles the encoding of individual form data parts, ensuring proper formatting and streaming of both string and binary data.

# Use Cases and Benefits

This helper is essential for enabling Axios to handle multipart form data uploads in both browser and Node.js environments. By streaming data and automatically managing headers, it supports efficient, standards-compliant uploads of files and form fields without requiring the entire payload to be loaded into memory.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBYXhpb3MlM0ElM0FTd2ltbS1EZW1v" repo-name="axios"><sup>Powered by [Swimm](/)</sup></SwmMeta>
