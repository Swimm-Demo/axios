---
title: Pull Request Size Report Script
---
# introduction

This document explains the design and usage of a script that generates a report on pull request file sizes. The main questions answered here are:

1. How does the script gather file size and history data?
2. How is the report content generated and formatted?
3. What inputs does the script accept and how is it run?

# gathering file size and history data

The core of the script is a function that takes a list of files and collects detailed stats for each. It uses file system operations to get the raw size and reads the file content to calculate the gzip-compressed size. It also fetches the commit history for each file blob, which helps track size changes over time. This data is organized into an object keyed by file path, including human-readable size history strings.

<SwmSnippet path="/bin/pr.js" line="10">

---

This approach provides a comprehensive view of file size evolution, useful for monitoring pull request impact on bundle size or codebase weight.

```javascript
  for(const [name, file] of Object.entries(files)) {
    const commits = await getBlobHistory(file);

    stat[file] = {
      name,
      size: (await fs.stat(file)).size,
      path: file,
      gzip: await gzipSize(String(await fs.readFile(file))),
      commits,
      history: commits.map(({tag, size}) => `${prettyBytes(size)} (${tag})`).join(' ← ')
    }
  }
```

---

</SwmSnippet>

# generating the report content

Once the file stats are collected, another function compiles this data into a formatted report. It uses Handlebars templates to produce a readable output, registering a helper to convert byte counts into human-friendly strings. The template file path can be customized, allowing flexible report layouts.

<SwmSnippet path="/bin/pr.js" line="26">

---

This separation of data gathering and presentation keeps the script modular and easy to adapt.

```javascript
const generateBody = async ({files, template = './templates/pr.hbs'} = {}) => {
  const data = {
    files: await generateFileReport(files)
  };

  Handlebars.registerHelper('filesize', (bytes)=> prettyBytes(bytes));

  return Handlebars.compile(String(await fs.readFile(template)))(data);
}
```

---

</SwmSnippet>

# script parameters and execution

The script expects an object mapping descriptive names to file paths. This input defines which files to analyze and how they are labeled in the report. The example usage shows how to generate a report for two browser build files, printing the result to the console.

<SwmSnippet path="/bin/pr.js" line="36">

---

To run the script, you invoke it with the desired files object. It outputs the formatted report string, which can be redirected or used in pull request comments.

```javascript
console.log(await generateBody({
  files: {
    'Browser build (UMD)' : './dist/axios.min.js',
    'Browser build (ESM)' : './dist/esm/axios.min.js',
  }
}));
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBYXhpb3MlM0ElM0FTd2ltbS1EZW1v" repo-name="axios"><sup>Powered by [Swimm](/)</sup></SwmMeta>
