---
title: Validator Helpers and Option Assertion Utilities
---
# introduction

This document explains the main design decisions behind the validator helpers and option assertion utilities in <SwmPath>[lib/helpers/validator.js](lib/helpers/validator.js)</SwmPath>. We will cover:

1. How basic type validators are implemented and why.
2. How transitional options are handled with warnings and errors.
3. How options are asserted against a schema to enforce correct types and detect unknown keys.

# basic type validators

The file defines a set of simple validators for primitive JavaScript types like object, boolean, number, function, string, and symbol. Each validator checks if a value matches the expected type and returns true if it does. If not, it returns a string describing the expected type with correct grammar ("a" vs "an"). This approach lets the validators be used both as boolean checks and as error message generators.

<SwmSnippet path="/lib/helpers/validator.js" line="1">

---

This design keeps the validators lightweight and reusable across the codebase for consistent type checking.

```javascript
'use strict';

import {VERSION} from '../env/data.js';
import AxiosError from '../core/AxiosError.js';

const validators = {};

// eslint-disable-next-line func-names
['object', 'boolean', 'number', 'function', 'string', 'symbol'].forEach((type, i) => {
  validators[type] = function validator(thing) {
    return typeof thing === type || 'a' + (i < 1 ? 'n ' : ' ') + type;
  };
});
```

---

</SwmSnippet>

# transitional option validator

The transitional validator is a higher-order function that wraps another validator or disables an option entirely. It supports three cases:

- If the validator is false, it throws an error indicating the option was removed, including the version it was removed in.
- If the option is deprecated (a version is provided), it logs a warning once per option to inform users about deprecation.
- Otherwise, it delegates validation to the wrapped validator or returns true if none is provided.

<SwmSnippet path="/lib/helpers/validator.js" line="15">

---

This mechanism helps manage options that are being phased out or changed, providing clear feedback to developers about deprecated or removed options without breaking existing code immediately.

```javascript
const deprecatedWarnings = {};

/**
 * Transitional option validator
 *
 * @param {function|boolean?} validator - set to false if the transitional option has been removed
 * @param {string?} version - deprecated version / removed since version
 * @param {string?} message - some message with additional info
 *
 * @returns {function}
 */
validators.transitional = function transitional(validator, version, message) {
  function formatMessage(opt, desc) {
    return '[Axios v' + VERSION + '] Transitional option \'' + opt + '\'' + desc + (message ? '. ' + message : '');
  }

  // eslint-disable-next-line func-names
  return (value, opt, opts) => {
    if (validator === false) {
      throw new AxiosError(
        formatMessage(opt, ' has been removed' + (version ? ' in ' + version : '')),
        AxiosError.ERR_DEPRECATED
      );
    }

    if (version && !deprecatedWarnings[opt]) {
      deprecatedWarnings[opt] = true;
      // eslint-disable-next-line no-console
      console.warn(
        formatMessage(
          opt,
          ' has been deprecated since v' + version + ' and will be removed in the near future'
        )
      );
    }

    return validator ? validator(value, opt, opts) : true;
  };
};
```

---

</SwmSnippet>

# option assertion against schema

The <SwmToken path="lib/helpers/validator.js" pos="73:2:2" line-data="function assertOptions(options, schema, allowUnknown) {">`assertOptions`</SwmToken> function enforces that an options object conforms to a given schema of validators. It checks:

- The options argument is an object.
- Each key in options has a corresponding validator in the schema.
- Each option value passes its validator or throws an error with a descriptive message.
- Unknown options cause an error unless explicitly allowed.

<SwmSnippet path="/lib/helpers/validator.js" line="63">

---

This function centralizes option validation, ensuring that only expected and correctly typed options are accepted. It improves robustness by catching configuration errors early and providing clear error messages.

```javascript
/**
 * Assert object's properties type
 *
 * @param {object} options
 * @param {object} schema
 * @param {boolean?} allowUnknown
 *
 * @returns {object}
 */

function assertOptions(options, schema, allowUnknown) {
  if (typeof options !== 'object') {
    throw new AxiosError('options must be an object', AxiosError.ERR_BAD_OPTION_VALUE);
  }
  const keys = Object.keys(options);
  let i = keys.length;
  while (i-- > 0) {
    const opt = keys[i];
    const validator = schema[opt];
    if (validator) {
      const value = options[opt];
      const result = value === undefined || validator(value, opt, options);
      if (result !== true) {
        throw new AxiosError('option ' + opt + ' must be ' + result, AxiosError.ERR_BAD_OPTION_VALUE);
      }
      continue;
    }
    if (allowUnknown !== true) {
      throw new AxiosError('Unknown option ' + opt, AxiosError.ERR_BAD_OPTION);
    }
  }
}
```

---

</SwmSnippet>

# export structure

<SwmSnippet path="/lib/helpers/validator.js" line="96">

---

The file exports the <SwmToken path="lib/helpers/validator.js" pos="97:1:1" line-data="  assertOptions,">`assertOptions`</SwmToken> function and the validators object as the default export, making them available for use throughout the Axios codebase.

```javascript
export default {
  assertOptions,
  validators
};
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBYXhpb3MlM0ElM0FTd2ltbS1EZW1v" repo-name="axios"><sup>Powered by [Swimm](/)</sup></SwmMeta>
