---
title: Helper Functions for Parsing in Axios CLI
---
# introduction

This document explains the main ideas behind the helper functions used for parsing in the Axios CLI. These functions are designed to extract structured information from text inputs, which is essential for processing CLI data or documentation sections.

We will cover:

1. How the code extracts multiple matches from a text using a callback.
2. How it identifies and parses specific sections in a markdown-like text.
3. How it extracts version numbers from raw version strings.

# extracting multiple matches with a callback

The function that handles this is designed to repeatedly apply a regular expression to a text input and invoke a callback for each match found. This approach avoids returning large arrays and instead processes matches on the fly, which can be more memory-efficient and flexible for different use cases.

<SwmSnippet path="/bin/helpers/parser.js" line="1">

---

This function is a foundational utility that other parsing functions build upon to handle complex text extraction scenarios.

```javascript
export const matchAll = (text, regexp, cb) => {
  let match;
  while((match = regexp.exec(text))) {
    cb(match);
  }
}

export const parseSection = (body, name, cb) => {
  matchAll(body, new RegExp(`^(#+)\\s+${name}?(.*?)^\\1\\s+\\w+`, 'gims'), cb);
}
```

---

</SwmSnippet>

# parsing named sections in markdown-like text

Building on the multiple match extractor, the code defines a function to parse sections identified by headers in a markdown-style format. It uses a dynamic regular expression that looks for headers with a specific name and captures the content until the next header of the same level.

<SwmSnippet path="/bin/helpers/parser.js" line="1">

---

This allows the CLI to isolate and process specific parts of documentation or input files, such as changelog sections or configuration blocks, by their header names.

```javascript
export const matchAll = (text, regexp, cb) => {
  let match;
  while((match = regexp.exec(text))) {
    cb(match);
  }
}

export const parseSection = (body, name, cb) => {
  matchAll(body, new RegExp(`^(#+)\\s+${name}?(.*?)^\\1\\s+\\w+`, 'gims'), cb);
}
```

---

</SwmSnippet>

# extracting semantic version numbers

Another helper function focuses on parsing version strings. It uses a regular expression to capture major, minor, and patch numbers from a version string that may or may not start with a "v" character.

<SwmSnippet path="/bin/helpers/parser.js" line="12">

---

This function simplifies version handling by providing a consistent way to extract numeric version components from various raw version formats.

```javascript
export const parseVersion = (rawVersion) => /^v?(\d+).(\d+).(\d+)/.exec(rawVersion);
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBYXhpb3MlM0ElM0FTd2ltbS1EZW1v" repo-name="axios"><sup>Powered by [Swimm](/)</sup></SwmMeta>
