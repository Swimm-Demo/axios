---
title: 'Git Utilities for Blob Size, History, and Tags'
---
# introduction

This document explains the main ideas behind the git utility functions implemented in <SwmPath>[bin/repo.js](bin/repo.js)</SwmPath>. These utilities help interact with git objects to retrieve information about file sizes, commit history, and tags.

We will cover:

1. How the blob size of a file at a specific commit is retrieved.
2. How commit history with tags is extracted for a file.
3. How tags matching a pattern are listed and sorted.

# getting blob size from git objects

The function to get the blob size uses the git plumbing command <SwmToken path="bin/repo.js" pos="8:2:9" line-data="    `git cat-file -s ${sha}:${filepath}`">`git cat-file -s`</SwmToken> which returns the size of the object identified by a commit SHA and file path. This is wrapped in a promise-based exec call to run the shell command asynchronously. The function returns the size as a number or 0 if the size is not found.

<SwmSnippet path="/bin/repo.js" line="1">

---

This approach avoids loading the entire file content and directly queries git's internal object database for size, making it efficient for large repositories or files.

```javascript
import util from "util";
import cp from "child_process";

export const exec = util.promisify(cp.exec);

export const getBlobSize = async (filepath, sha ='HEAD') => {
  const size = (await exec(
    `git cat-file -s ${sha}:${filepath}`
  )).stdout;

  return size ? +size : 0;
}
```

---

</SwmSnippet>

# extracting commit history with tags for a file

To get the commit history related to a file, the function runs <SwmToken path="bin/repo.js" pos="16:2:4" line-data="    `git log --max-count=${maxCount} --no-walk --tags=v* --oneline --format=%H%d -- ${filepath}`">`git log`</SwmToken> with options to limit the number of commits, filter by tags matching <SwmToken path="bin/repo.js" pos="16:23:24" line-data="    `git log --max-count=${maxCount} --no-walk --tags=v* --oneline --format=%H%d -- ${filepath}`">`v*`</SwmToken>, and format the output to include commit SHA and tag names. It then parses this output using a regular expression to extract the SHA and tag for each commit.

For each matched commit, it calls the blob size function to get the file size at that commit. This ties together commit metadata with file size information, useful for tracking file changes across tagged releases.

<SwmSnippet path="/bin/repo.js" line="14">

---

The use of <SwmToken path="bin/repo.js" pos="16:15:24" line-data="    `git log --max-count=${maxCount} --no-walk --tags=v* --oneline --format=%H%d -- ${filepath}`">`--no-walk --tags=v*`</SwmToken> ensures only commits referenced by tags matching the pattern are considered, focusing on tagged releases rather than all commits.

```javascript
export const getBlobHistory = async (filepath, maxCount= 5) => {
  const log = (await exec(
    `git log --max-count=${maxCount} --no-walk --tags=v* --oneline --format=%H%d -- ${filepath}`
  )).stdout;

  const commits = [];

  let match;

  const regexp = /^(\w+) \(tag: (v?[.\d]+)\)$/gm;

  while((match = regexp.exec(log))) {
    commits.push({
      sha: match[1],
      tag: match[2],
      size: await getBlobSize(filepath, match[1])
    })
  }

  return commits;
}
```

---

</SwmSnippet>

# listing tags with pattern and sorting

The tag listing function runs <SwmToken path="bin/repo.js" pos="38:2:7" line-data="    `git tag -l ${pattern} --sort=${sort}`">`git tag -l`</SwmToken> with a pattern and sort order. It returns the tags as an array by splitting the command output on newlines.

<SwmSnippet path="/bin/repo.js" line="36">

---

This provides a simple way to retrieve tags matching a version pattern and sort them semantically (e.g., descending by version number), which is helpful for version management and release automation.

```javascript
export const getTags = async (pattern = 'v*', sort = '-v:refname') => {
  const log = (await exec(
    `git tag -l ${pattern} --sort=${sort}`
  )).stdout;

  return log.split(/\r?\n/);
}
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBYXhpb3MlM0ElM0FTd2ltbS1EZW1v" repo-name="axios"><sup>Powered by [Swimm](/)</sup></SwmMeta>
