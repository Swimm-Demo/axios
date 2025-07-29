---
title: Overview
---
Axios is a promise-based HTTP client that allows you to make HTTP requests from both the browser and Node.js, handling responses and supporting features like interceptors, cancellation, and automatic data serialization.

# Main Components

### TypeScript Declarations (<SwmPath>[index.d.cts](index.d.cts)</SwmPath>)

TypeScript declarations in Axios serve to specify the types and interfaces for HTTP requests, responses, headers, errors, and configuration options, ensuring robust type checking and autocompletion support for developers using Axios.

- **Classes**
  - <SwmLink doc-title="The axiosrequestconfig interface">[The axiosrequestconfig interface](/.swm/the-axiosrequestconfig-interface.0exuu.sw.md)</SwmLink>

### CLI Utilities (<SwmPath>[bin/](bin/)</SwmPath>)

CLI utilities are command-line scripts that support automation and operational tasks for repository maintenance and interaction with external services.

- **Inject contributors list**
  - <SwmLink doc-title="Inject contributors and prs sections into changelog">[Inject contributors and prs sections into changelog](/.swm/inject-contributors-and-prs-sections-into-changelog.dotng1ki.sw.md)</SwmLink>
- **Repo**
  - <SwmLink doc-title="Git utilities for blob size history and tags">[Git utilities for blob size history and tags](/.swm/git-utilities-for-blob-size-history-and-tags.smg2fizh.sw.md)</SwmLink>
- **Sponsors**
  - <SwmLink doc-title="Script to update sponsors block in readmemd">[Script to update sponsors block in readmemd](/.swm/script-to-update-sponsors-block-in-readmemd.tsgqfq03.sw.md)</SwmLink>
- **Pr**
  - <SwmLink doc-title="Pull request size report script">[Pull request size report script](/.swm/pull-request-size-report-script.1nxxse3u.sw.md)</SwmLink>
- **Helpers**
  - <SwmLink doc-title="Cli utilities helpers">[Cli utilities helpers](/.swm/cli-utilities-helpers.bb05dgqh.sw.md)</SwmLink>
  - **Colorize**
    - <SwmLink doc-title="Colorize helper function for cli output">[Colorize helper function for cli output](/.swm/colorize-helper-function-for-cli-output.c81siwld.sw.md)</SwmLink>
  - **Parser**
    - <SwmLink doc-title="Helper functions for parsing in axios cli">[Helper functions for parsing in axios cli](/.swm/helper-functions-for-parsing-in-axios-cli.16t6lc2p.sw.md)</SwmLink>
- **Contributors**
  - <SwmLink doc-title="Contributor and release info script">[Contributor and release info script](/.swm/contributor-and-release-info-script.x62q9nn3.sw.md)</SwmLink>
- **Repo bot**
  - **Classes**
    - <SwmLink doc-title="The repobot class">[The repobot class](/.swm/the-repobot-class.9r40u.sw.md)</SwmLink>
- **Githubapi**
  - **Classes**
    - <SwmLink doc-title="The githubapi class">[The githubapi class](/.swm/the-githubapi-class.jcxzd.sw.md)</SwmLink>

### TypeScript Types (<SwmPath>[index.d.ts](index.d.ts)</SwmPath>)

TypeScript types serve as the foundation for defining and enforcing the shape and behavior of HTTP requests and responses, headers, errors, and configuration within the HTTP client, ensuring robust and maintainable code.

- **Classes**
  - <SwmLink doc-title="The axiosrequestconfig interface">[The axiosrequestconfig interface](/.swm/the-axiosrequestconfig-interface.vczng.sw.md)</SwmLink>

### Examples (<SwmPath>[sandbox/](sandbox/)</SwmPath>)

Examples are included to allow manual testing and to demonstrate how to run and use the HTTP client in various contexts, supporting both browser and terminal execution.

- **Server**
  - <SwmLink doc-title="Sandbox http server script">[Sandbox http server script](/.swm/sandbox-http-server-script.zp63gpwa.sw.md)</SwmLink>
- **Client**
  - <SwmLink doc-title="Axios sandbox client html ui">[Axios sandbox client html ui](/.swm/axios-sandbox-client-html-ui.ml62ftxv.sw.md)</SwmLink>

### Core Library (<SwmPath>[lib/](lib/)</SwmPath>)

- <SwmLink doc-title="Core library overview">[Core library overview](/.swm/core-library-overview.21o7lrom.sw.md)</SwmLink>
- **Cancel**
  - <SwmLink doc-title="Cancellation mechanism in core library">[Cancellation mechanism in core library](/.swm/cancellation-mechanism-in-core-library.walc0r4h.sw.md)</SwmLink>
  - **Cancel token**
    - **Classes**
      - <SwmLink doc-title="The canceltoken class">[The canceltoken class](/.swm/the-canceltoken-class.azdiu.sw.md)</SwmLink>
- **Utils**
  - <SwmLink doc-title="Axios utility functions">[Axios utility functions](/.swm/axios-utility-functions.2e323lfo.sw.md)</SwmLink>
- **Adapters**
  - **Xhr**
    - <SwmLink doc-title="Xhr adapter implementation for axios">[Xhr adapter implementation for axios](/.swm/xhr-adapter-implementation-for-axios.qv565w38.sw.md)</SwmLink>
  - **Http**
    - **Flows**
      - <SwmLink doc-title="Sending an http request and handling the response">[Sending an http request and handling the response](/.swm/sending-an-http-request-and-handling-the-response.2lu8nlcx.sw.md)</SwmLink>
      - <SwmLink doc-title="Sending and managing an http request">[Sending and managing an http request](/.swm/sending-and-managing-an-http-request.0mp4rcg9.sw.md)</SwmLink>
- **Helpers**
  - <SwmLink doc-title="Core library helpers overview">[Core library helpers overview](/.swm/core-library-helpers-overview.8e9zxwlr.sw.md)</SwmLink>
  - **FormData Streams**
    - <SwmLink doc-title="Formdata streams helper overview">[Formdata streams helper overview](/.swm/formdata-streams-helper-overview.lzbsxcrm.sw.md)</SwmLink>
  - **Stream Cookies Utils**
    - <SwmLink doc-title="Stream cookies utilities overview">[Stream cookies utilities overview](/.swm/stream-cookies-utilities-overview.ut4pw65t.sw.md)</SwmLink>
    - **Track stream**
      - <SwmLink doc-title="Stream chunking and progress tracking helpers">[Stream chunking and progress tracking helpers](/.swm/stream-chunking-and-progress-tracking-helpers.u6xfhhwe.sw.md)</SwmLink>
  - **DataURI Protocol**
    - <SwmLink doc-title="Datauri protocol helper">[Datauri protocol helper](/.swm/datauri-protocol-helper.1jweg17y.sw.md)</SwmLink>
  - **Progress Throttle Utils**
    - <SwmLink doc-title="Progress throttle utilities in http helpers">[Progress throttle utilities in http helpers](/.swm/progress-throttle-utilities-in-http-helpers.1omnsblw.sw.md)</SwmLink>
    - **Throttle**
      - <SwmLink doc-title="Throttle function utility">[Throttle function utility](/.swm/throttle-function-utility.j6lqdrhq.sw.md)</SwmLink>
  - **To form data**
    - **Flows**
      - <SwmLink doc-title="Preparing form data from objects">[Preparing form data from objects](/.swm/preparing-form-data-from-objects.hh7aeuw5.sw.md)</SwmLink>
  - **Validator**
    - <SwmLink doc-title="Validator helpers and option assertion utilities">[Validator helpers and option assertion utilities](/.swm/validator-helpers-and-option-assertion-utilities.cuoq9tcn.sw.md)</SwmLink>
  - **Form data tojson**
    - <SwmLink doc-title="Formdata to json conversion helper">[Formdata to json conversion helper](/.swm/formdata-to-json-conversion-helper.49bh48nf.sw.md)</SwmLink>
  - **Axios transform stream**
    - **Classes**
      - <SwmLink doc-title="The axiostransformstream class">[The axiostransformstream class](/.swm/the-axiostransformstream-class.b5c3s.sw.md)</SwmLink>
- **Defaults**
  - <SwmLink doc-title="Defaults configuration in core library">[Defaults configuration in core library](/.swm/defaults-configuration-in-core-library.5e92ygx1.sw.md)</SwmLink>
  - **Index**
    - <SwmLink doc-title="Axios default configuration and serialization logic">[Axios default configuration and serialization logic](/.swm/axios-default-configuration-and-serialization-logic.o2o3k3j2.sw.md)</SwmLink>
- **Core**
  - **Axios**
    - **Classes**
      - <SwmLink doc-title="The axios class">[The axios class](/.swm/the-axios-class.tj77q.sw.md)</SwmLink>
  - **Axios headers**
    - **Classes**
      - <SwmLink doc-title="The axiosheaders class">[The axiosheaders class](/.swm/the-axiosheaders-class.8bzen.sw.md)</SwmLink>
  - **Dispatch request**
    - <SwmLink doc-title="Axios request dispatch flow">[Axios request dispatch flow](/.swm/axios-request-dispatch-flow.dckmpulv.sw.md)</SwmLink>
  - **Interceptor manager**
    - **Classes**
      - <SwmLink doc-title="The interceptormanager class">[The interceptormanager class](/.swm/the-interceptormanager-class.hf6sf.sw.md)</SwmLink>

### Test Config (<SwmPath>[karma.conf.cjs](karma.conf.cjs)</SwmPath>)

Test Config sets up the automated testing environment for Axios, enabling cross-browser testing with customizable launchers and integration with CI services to ensure code quality and compatibility.

- <SwmLink doc-title="Karma test runner configuration">[Karma test runner configuration](/.swm/karma-test-runner-configuration.cps66di9.sw.md)</SwmLink>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBYXhpb3MlM0ElM0FTd2ltbS1EZW1v" repo-name="axios"><sup>Powered by [Swimm](/)</sup></SwmMeta>
