---
title: Karma Test Runner Configuration
---
# introduction

This document explains the configuration of the Karma test runner used in the axios project. It answers these questions:

1. How does the configuration handle different browser environments, including Sauce Labs and local runs?
2. How are custom browser launchers defined and selected?
3. What are the key settings for test files, preprocessors, and reporters?
4. How are timeouts and logging configured to improve test stability?

# handling browser environments and custom launchers

The configuration dynamically sets up browsers based on environment variables. If Sauce Labs credentials are present, it defines a set of custom launchers for various browsers and platforms using the <SwmToken path="karma.conf.cjs" pos="12:2:2" line-data="function createCustomLauncher(browser, version, platform) {">`createCustomLauncher`</SwmToken> function. This function specifies the base as <SwmToken path="karma.conf.cjs" pos="14:5:5" line-data="    base: &#39;SauceLabs&#39;,">`SauceLabs`</SwmToken> and sets browser name, version, and platform, enabling tests to run on Sauce Labs' cloud infrastructure.

The script checks environment variables like <SwmToken path="karma.conf.cjs" pos="31:2:2" line-data="      &#39;SAUCE_CHROME&#39;,">`SAUCE_CHROME`</SwmToken>, <SwmToken path="karma.conf.cjs" pos="32:2:2" line-data="      &#39;SAUCE_FIREFOX&#39;,">`SAUCE_FIREFOX`</SwmToken>, etc., to decide which browsers to enable. If none are specified, it defaults to running all supported browsers. This selective enabling helps control test scope and resource usage.

<SwmSnippet path="/karma.conf.cjs" line="9">

---

If Sauce Labs credentials are missing but the tests run on Travis CI pull requests, it falls back to Firefox only, since encrypted variables are unavailable in <SwmToken path="karma.conf.cjs" pos="127:28:28" line-data="      &#39;Cannot run on Sauce Labs as encrypted environment variables are not available to PRs. &#39; +">`PRs`</SwmToken>. On <SwmToken path="karma.conf.cjs" pos="132:12:12" line-data="    console.log(&#39;Running ci on GitHub Actions.&#39;);">`GitHub`</SwmToken> Actions, it uses headless Firefox and Chrome. Otherwise, it defaults to local Chrome. This logic ensures tests run appropriately in different CI and local environments.

```cjs
var resolve = require('@rollup/plugin-node-resolve').default;
var commonjs = require('@rollup/plugin-commonjs');

function createCustomLauncher(browser, version, platform) {
  return {
    base: 'SauceLabs',
    browserName: browser,
    version: version,
    platform: platform
  };
}

module.exports = function(config) {
  var customLaunchers = {};
  var browsers = process.env.Browsers && process.env.Browsers.split(',');
  var sauceLabs;

  if (process.env.SAUCE_USERNAME || process.env.SAUCE_ACCESS_KEY) {
    customLaunchers = {};

    var runAll = true;
    var options = [
      'SAUCE_CHROME',
      'SAUCE_FIREFOX',
      'SAUCE_SAFARI',
      'SAUCE_OPERA',
      'SAUCE_IE',
      'SAUCE_EDGE',
      'SAUCE_IOS',
      'SAUCE_ANDROID'
    ];

    options.forEach(function(opt) {
      if (process.env[opt]) {
        runAll = false;
      }
    });

    // Chrome
    if (runAll || process.env.SAUCE_CHROME) {
      customLaunchers.SL_Chrome = createCustomLauncher('chrome');
      // customLaunchers.SL_ChromeDev = createCustomLauncher('chrome', 'dev');
      // customLaunchers.SL_ChromeBeta = createCustomLauncher('chrome', 'beta');
    }

    // Firefox
    if (runAll || process.env.SAUCE_FIREFOX) {
      //customLaunchers.SL_Firefox = createCustomLauncher('firefox');
      // customLaunchers.SL_FirefoxDev = createCustomLauncher('firefox', 'dev');
      // customLaunchers.SL_FirefoxBeta = createCustomLauncher('firefox', 'beta');
    }

    // Safari
    if (runAll || process.env.SAUCE_SAFARI) {
      // customLaunchers.SL_Safari7 = createCustomLauncher('safari', 7);
      // customLaunchers.SL_Safari8 = createCustomLauncher('safari', 8);
      customLaunchers.SL_Safari9 = createCustomLauncher(
        'safari',
        9.0,
        'OS X 10.11'
      );
      customLaunchers.SL_Safari10 = createCustomLauncher(
        'safari',
        '10.1',
        'macOS 10.12'
      );
      customLaunchers.SL_Safari11 = createCustomLauncher(
        'safari',
        '11.1',
        'macOS 10.13'
      );
    }

    // Opera
    if (runAll || process.env.SAUCE_OPERA) {
      // TODO The available versions of Opera are too old and lack basic APIs
      // customLaunchers.SL_Opera11 = createCustomLauncher('opera', 11, 'Windows XP');
      // customLaunchers.SL_Opera12 = createCustomLauncher('opera', 12, 'Windows 7');
    }

    // IE
    if (runAll || process.env.SAUCE_IE) {
      customLaunchers.SL_IE11 = createCustomLauncher('internet explorer', 11, 'Windows 8.1');
    }

    // Edge
    if (runAll || process.env.SAUCE_EDGE) {
      customLaunchers.SL_Edge = createCustomLauncher('microsoftedge', null, 'Windows 10');
    }

    // IOS
    if (runAll || process.env.SAUCE_IOS) {
      // TODO IOS7 capture always timesout
      // customLaunchers.SL_IOS7 = createCustomLauncher('iphone', '7.1', 'OS X 10.10');
      // TODO Mobile browsers are causing failures, possibly from too many concurrent VMs
      // customLaunchers.SL_IOS8 = createCustomLauncher('iphone', '8.4', 'OS X 10.10');
      // customLaunchers.SL_IOS9 = createCustomLauncher('iphone', '9.2', 'OS X 10.10');
    }

    // Android
    if (runAll || process.env.SAUCE_ANDROID) {
      // TODO Mobile browsers are causing failures, possibly from too many concurrent VMs
      // customLaunchers.SL_Android4 = createCustomLauncher('android', '4.4', 'Linux');
      // customLaunchers.SL_Android5 = createCustomLauncher('android', '5.1', 'Linux');
    }

    browsers = Object.keys(customLaunchers);

    sauceLabs = {
      recordScreenshots: false,
      connectOptions: {
        // port: 5757,
        logfile: 'sauce_connect.log'
      },
      public: 'public'
    };
  } else if (process.env.TRAVIS_PULL_REQUEST && process.env.TRAVIS_PULL_REQUEST !== 'false') {
    console.log(
      'Cannot run on Sauce Labs as encrypted environment variables are not available to PRs. ' +
      'Running on Travis.'
    );
    browsers = ['Firefox'];
  } else if (process.env.GITHUB_ACTIONS === 'true') {
    console.log('Running ci on GitHub Actions.');
    browsers = ['FirefoxHeadless', 'ChromeHeadless'];
  } else {
    browsers = browsers || ['Chrome'];
    console.log(`Running ${browsers} locally since SAUCE_USERNAME and SAUCE_ACCESS_KEY environment variables are not set.`);
  }
```

---

</SwmSnippet>

# test files, preprocessors, and frameworks

The configuration specifies Jasmine and Sinon as the testing frameworks, with <SwmToken path="karma.conf.cjs" pos="146:6:8" line-data="    frameworks: [&#39;jasmine-ajax&#39;, &#39;jasmine&#39;, &#39;sinon&#39;],">`jasmine-ajax`</SwmToken> for mocking Ajax requests. It loads helper scripts and all spec files under <SwmPath>[test/specs/](test/specs/)</SwmPath>, without watching them for changes.

<SwmSnippet path="/karma.conf.cjs" line="144">

---

Preprocessing uses Rollup with plugins for resolving node modules and converting CommonJS modules to ES modules. The output format is an immediately-invoked function expression (IIFE) named <SwmToken path="karma.conf.cjs" pos="174:5:5" line-data="        name: &#39;_axios&#39;,">`_axios`</SwmToken> with inline source maps. This bundling prepares test files for browser execution.

```cjs
    // frameworks to use
    // available frameworks: https://npmjs.org/browse/keyword/karma-adapter
    frameworks: ['jasmine-ajax', 'jasmine', 'sinon'],
```

---

</SwmSnippet>

<SwmSnippet path="/karma.conf.cjs" line="149">

---

&nbsp;

```cjs
    // list of files / patterns to load in the browser
    files: [
      {pattern: 'test/specs/__helpers.js', watched: false},
      {pattern: 'test/specs/**/*.spec.js', watched: false}
    ],
```

---

</SwmSnippet>

<SwmSnippet path="/karma.conf.cjs" line="160">

---

&nbsp;

```cjs
    // preprocess matching files before serving them to the browser
    // available preprocessors: https://npmjs.org/browse/keyword/karma-preprocessor
    preprocessors: {
      'test/specs/__helpers.js': ['rollup'],
      'test/specs/**/*.spec.js': ['rollup']
    },

    rollupPreprocessor: {
      plugins: [
        resolve({browser: true}),
        commonjs()
      ],
      output: {
        format: 'iife',
        name: '_axios',
        sourcemap: 'inline'
      }
    },
```

---

</SwmSnippet>

# reporters, ports, and logging

<SwmSnippet path="/karma.conf.cjs" line="180">

---

The test results reporter is set to 'progress' only, with code coverage disabled due to CI issues. The web server listens on port 9876. Logging level is INFO, and colors are enabled for readability.

```cjs
    // test results reporter to use
    // possible values: 'dots', 'progress'
    // available reporters: https://npmjs.org/browse/keyword/karma-reporter
    // Disable code coverage, as it's breaking CI:
    // reporters: ['dots', 'coverage', 'saucelabs'],
    reporters: ['progress'],
```

---

</SwmSnippet>

<SwmSnippet path="/karma.conf.cjs" line="188">

---

&nbsp;

```cjs
    // web server port
    port: 9876,
```

---

</SwmSnippet>

<SwmSnippet path="/karma.conf.cjs" line="199">

---

&nbsp;

```cjs
    // enable / disable colors in the output (reporters and logs)
    colors: true,
```

---

</SwmSnippet>

<SwmSnippet path="/karma.conf.cjs" line="203">

---

&nbsp;

```cjs
    // level of logging
    // possible values: config.LOG_DISABLE || config.LOG_ERROR || config.LOG_WARN || config.LOG_INFO || config.LOG_DEBUG
    logLevel: config.LOG_INFO,
```

---

</SwmSnippet>

# timeouts and stability settings

Timeouts are increased to prevent flaky test failures caused by slow browser connections or inactivity. The capture timeout is set to 4 minutes, browser disconnect timeout to 10 seconds, and browser no activity timeout also to 4 minutes. The browser disconnect tolerance allows one disconnect before failing.

<SwmSnippet path="/karma.conf.cjs" line="192">

---

Watching files for changes is disabled (<SwmToken path="karma.conf.cjs" pos="209:1:4" line-data="    autoWatch: false,">`autoWatch: false`</SwmToken>), and the test runner does not exit after a single run (<SwmToken path="karma.conf.cjs" pos="219:1:4" line-data="    singleRun: false,">`singleRun: false`</SwmToken>), which suits continuous testing scenarios.

```cjs
    // Increase timeouts to prevent the issue with disconnected tests (https://goo.gl/nstA69)
    captureTimeout: 4 * 60 * 1000,
    browserDisconnectTimeout: 10000,
    browserDisconnectTolerance: 1,
    browserNoActivityTimeout: 4 * 60 * 1000,
```

---

</SwmSnippet>

<SwmSnippet path="/karma.conf.cjs" line="208">

---

&nbsp;

```cjs
    // enable / disable watching file and executing tests whenever any file changes
    autoWatch: false,
```

---

</SwmSnippet>

<SwmSnippet path="/karma.conf.cjs" line="217">

---

&nbsp;

```cjs
    // Continuous Integration mode
    // if true, Karma captures browsers, runs the tests and exits
    singleRun: false,
```

---

</SwmSnippet>

# additional configuration

The configuration includes Webpack settings for development mode with caching and inline source maps, excluding the HTTP adapter from bundling. Webpack server stats are colored for clarity.

Coverage reporting is configured to output lcov reports into the <SwmToken path="karma.conf.cjs" pos="243:5:6" line-data="      dir: &#39;coverage/&#39;,">`coverage/`</SwmToken> directory.

<SwmSnippet path="/karma.conf.cjs" line="221">

---

Sauce Labs options disable screenshot recording and specify connection options like the log file.

```cjs
    // Webpack config
    webpack: {
      mode: 'development',
      cache: true,
      devtool: 'inline-source-map',
      externals: [
        {
          './adapters/http': 'var undefined'
        }
      ]
    },

    webpackServer: {
      stats: {
        colors: true
      }
    },
```

---

</SwmSnippet>

<SwmSnippet path="/karma.conf.cjs" line="240">

---

&nbsp;

```cjs
    // Coverage reporting
    coverageReporter: {
      type: 'lcov',
      dir: 'coverage/',
      subdir: '.'
    },
```

---

</SwmSnippet>

<SwmSnippet path="/karma.conf.cjs" line="115">

---

&nbsp;

```cjs
    browsers = Object.keys(customLaunchers);

    sauceLabs = {
      recordScreenshots: false,
      connectOptions: {
        // port: 5757,
        logfile: 'sauce_connect.log'
      },
      public: 'public'
    };
```

---

</SwmSnippet>

<SwmSnippet path="/karma.conf.cjs" line="247">

---

&nbsp;

```cjs
    sauceLabs: sauceLabs,
    customLaunchers: customLaunchers
  });
};
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBYXhpb3MlM0ElM0FTd2ltbS1EZW1v" repo-name="axios"><sup>Powered by [Swimm](/)</sup></SwmMeta>
