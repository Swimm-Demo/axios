---
title: DataURI Protocol Helper
---
# Overview of the DataURI Protocol Helper

The DataURI Protocol helper is a utility designed to parse data URIs, which are URLs embedding data directly within them, into usable data formats such as Buffer or Blob. This functionality enables the application to process embedded data without requiring additional network requests, enhancing efficiency and flexibility.

# How the DataURI Protocol Helper Works

The helper begins by extracting the protocol from the URI string using a protocol parsing function to confirm that the URI uses the 'data' protocol. Once verified, it removes the protocol prefix to isolate the data portion of the URI. It then applies a regular expression to parse the data URI into its components: the MIME type, encoding type (such as base64), and the actual data payload. Depending on the encoding, the data payload is decoded appropriately—base64 decoding if indicated, or URL decoding otherwise—and converted into a Buffer. If requested and supported by the environment, the Buffer is wrapped into a Blob object with the correct MIME type.

# Error Handling in the DataURI Protocol Helper

The helper includes robust error handling to ensure reliability. If the URI does not conform to the expected data URI format or the protocol is unsupported, it throws an AxiosError indicating the invalid URL or unsupported protocol. Additionally, if Blob support is requested but not available in the environment, it throws an error specifying that Blob is not supported.

# Key Functions: fromDataURI and parseProtocol

Two primary functions underpin the DataURI Protocol helper: `fromDataURI` and `parseProtocol`. The `parseProtocol` function extracts the protocol segment from a URL string using a regular expression, returning the protocol or an empty string if none is found. This function is crucial for verifying that the URI uses the 'data' protocol before further processing. The `fromDataURI` function handles the main parsing logic: it first uses `parseProtocol` to confirm the protocol, then removes the protocol prefix, and applies a regular expression to extract the MIME type, encoding, and data payload. It decodes the payload into a Buffer and, if requested and supported, wraps it into a Blob with the correct MIME type. Errors are thrown if the URI format is invalid or if Blob support is missing when required.

# Usage in HTTP Adapter

Within the HTTP adapter's `dispatchHttpRequest` function, the DataURI Protocol helper is utilized to convert data URI URLs into Buffer or Blob objects based on the response type. For instance, when the response type is 'blob', the helper converts the data URI into a Blob using the environment's Blob constructor. This enables the response to be handled as a binary large object, facilitating operations that require binary data handling.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBYXhpb3MlM0ElM0FTd2ltbS1EZW1v" repo-name="axios"><sup>Powered by [Swimm](/)</sup></SwmMeta>
