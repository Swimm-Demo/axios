---
title: Stream Cookies Utilities Overview
---
# Overview of Stream Cookies Utils

Stream Cookies Utils are a set of helper functions designed to manage HTTP cookies within environments that support the standard browser API. These utilities simplify cookie operations by abstracting the direct manipulation of the browser's document.cookie interface.

# Core Functionality

The utilities provide three primary methods: write, read, and remove. The write method creates a properly formatted cookie string, including optional attributes such as expiration date, path, domain, and secure flag, and assigns it to document.cookie to store the cookie. The read method retrieves a cookie's value by searching for the cookie name within document.cookie using a regular expression and returns the decoded value if found. The remove method deletes a cookie by setting its expiration date to a past time, prompting the browser to remove it.

# Handling Non-Standard Environments

In environments lacking standard browser cookie support, such as web workers or React Native, these utility methods are implemented as no-operations or return null. This design choice ensures that the absence of cookie support does not cause errors or unexpected behavior in such contexts.

# Practical Example

For example, when the remove method is called with a cookie name, it internally invokes the write method with the same name, an empty value, and an expiration date set to one day in the past. This effectively instructs the browser to delete the specified cookie.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBYXhpb3MlM0ElM0FTd2ltbS1EZW1v" repo-name="axios"><sup>Powered by [Swimm](/)</sup></SwmMeta>
