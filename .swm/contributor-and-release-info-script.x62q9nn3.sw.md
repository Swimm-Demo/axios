---
title: Contributor and Release Info Script
---
# introduction

This document explains the design and implementation of a script that generates contributor and release information for the Axios repository. It covers:

1. How the script fetches and caches GitHub commit, user, and issue data.
2. How it processes and deduplicates contributor data.
3. How it generates release info and renders contributor and PR lists using templates.
4. The parameters and usage of the main exported functions.

# fetching and caching GitHub data

<SwmSnippet path="/bin/contributors.js" line="20">

---

The script uses axios to query GitHub's API for commit, user, and issue information. To avoid redundant requests, it caches results keyed by commit SHA, user email, or issue ID. This caching is implemented with closures holding cache objects, as seen in the functions that fetch commit info (

```javascript
const getUserFromCommit = ((commitCache) => async (sha) => {
  try {
    if(commitCache[sha] !== undefined) {
      return commitCache[sha];
    }

    console.log(colorize()`fetch github commit info (${sha})`);
```

---

</SwmSnippet>

<SwmSnippet path="/bin/contributors.js" line="35">

---

to

```javascript
  } catch (err) {
    return commitCache[sha] = null;
  }
```

---

</SwmSnippet>

<SwmSnippet path="/bin/contributors.js" line="38">

---

), issue info (

```javascript
})({});

const getIssueById = ((cache) => async (id) => {
  if(cache[id] !== undefined) {
    return cache[id];
  }

  try {
    const {data} = await axios.get(`https://api.github.com/repos/axios/axios/issues/${id}`);
```

---

</SwmSnippet>

<SwmSnippet path="/bin/contributors.js" line="48">

---

to

```javascript
    return cache[id] = data;
  } catch (err) {
    return null;
  }
```

---

</SwmSnippet>

<SwmSnippet path="/bin/contributors.js" line="52">

---

), and user info (

```javascript
})({});

const getUserInfo = ((userCache) => async (userEntry) => {
  const {email, commits} = userEntry;

  if (userCache[email] !== undefined) {
    return userCache[email];
  }
```

---

</SwmSnippet>

<SwmSnippet path="/bin/contributors.js" line="61">

---

to

```javascript
  console.log(colorize()`fetch github user info [${userEntry.name}]`);

  return userCache[email] = {
    ...userEntry,
    ...await getUserFromCommit(commits[0].hash)
  }
```

---

</SwmSnippet>

).

Caching improves performance and reduces API rate limits by returning cached data when available. The commit fetcher also extracts a small avatar URL variant for display.

# processing and deduplicating contributors

<SwmSnippet path="/bin/contributors.js" line="134">

---

Contributor data is aggregated from release commits, fixes, and merges. Each commit contributes author info, commit hashes, PR references, and stats like insertions and deletions (

```javascript
    for (const {hash, author, email, insertions, deletions} of commits) {
      const entry = authors[email] = (authors[email] || {
        name: author,
        prs: [],
        email,
        commits: [],
        insertions: 0, deletions: 0
      });
```

---

</SwmSnippet>

<SwmSnippet path="/bin/contributors.js" line="155">

---

to

```javascript
      entry.github = entry.login ? `https://github.com/${encodeURIComponent(entry.login)}` : '';

      entry.insertions += insertions;
      entry.deletions += deletions;
      entry.points = entry.insertions + entry.deletions;
    }
```

---

</SwmSnippet>

).

<SwmSnippet path="/bin/contributors.js" line="67">

---

To handle contributors with multiple emails or GitHub logins, the script deduplicates authors by merging stats and metadata (

```javascript
})({});

const deduplicate = (authors) => {
  const loginsMap = {};
  const combined= {};

  const assign = (a, b) => {
    const {insertions, deletions, points, ...rest} = b;
```

---

</SwmSnippet>

<SwmSnippet path="/bin/contributors.js" line="95">

---

to

```javascript
  return combined;
}
```

---

</SwmSnippet>

). This ensures each contributor is represented once with combined contributions.

# generating release info

<SwmSnippet path="/bin/contributors.js" line="98">

---

The core function <SwmToken path="bin/contributors.js" pos="98:2:2" line-data="const getReleaseInfo = ((releaseCache) =&gt; async (tag) =&gt; {">`getReleaseInfo`</SwmToken> (

```javascript
const getReleaseInfo = ((releaseCache) => async (tag) => {
  if(releaseCache[tag] !== undefined) {
    return releaseCache[tag];
  }

  const isUnreleasedTag = !tag;

  const version = 'v' + tag.replace(/^v/, '');
```

---

</SwmSnippet>

<SwmSnippet path="/bin/contributors.js" line="174">

---

to

```javascript
  releaseCache[tag] = release;

  return release;
```

---

</SwmSnippet>

) generates detailed release data for a given tag or the unreleased state. It runs the <SwmToken path="bin/contributors.js" pos="108:4:6" line-data="    `npx auto-changelog --unreleased-only --stdout --commit-limit false --template json` :">`auto-changelog`</SwmToken> CLI tool with appropriate parameters to get commit and PR data in JSON format.

It then processes commits and merges to build the authors list, enriches author info with GitHub data, marks bots, and sorts contributors by contribution points. The release object also includes all commits and merges.

Caching is used here as well to avoid regenerating release info for the same tag multiple times.

# rendering contributors and PR lists

Two rendering functions produce formatted output from Handlebars templates:

<SwmSnippet path="/bin/contributors.js" line="177">

---

- <SwmToken path="bin/contributors.js" pos="179:2:2" line-data="const renderContributorsList = async (tag, template) =&gt; {">`renderContributorsList`</SwmToken>`(`<SwmToken path="bin/contributors.js" pos="98:16:16" line-data="const getReleaseInfo = ((releaseCache) =&gt; async (tag) =&gt; {">`tag`</SwmToken>`, `<SwmToken path="bin/contributors.js" pos="179:12:12" line-data="const renderContributorsList = async (tag, template) =&gt; {">`template`</SwmToken>`)` (

```javascript
})({});

const renderContributorsList = async (tag, template) => {
  const release = await getReleaseInfo(tag);

  const compile = Handlebars.compile(String(await fs.readFile(template)))

  const content = compile(release);
```

---

</SwmSnippet>

<SwmSnippet path="/bin/contributors.js" line="186">

---

to

```javascript
  return removeExtraLineBreaks(cleanTemplate(content));
}
```

---

</SwmSnippet>

) loads release info for the tag, compiles the given template, and returns cleaned-up rendered content listing contributors.

<SwmSnippet path="/bin/contributors.js" line="189">

---

- <SwmToken path="bin/contributors.js" pos="189:2:2" line-data="const renderPRsList = async (tag, template, {comments_threshold= 5, awesome_threshold= 5, label = &#39;add_to_changelog&#39;} = {}) =&gt; {">`renderPRsList`</SwmToken>`(`<SwmToken path="bin/contributors.js" pos="98:16:16" line-data="const getReleaseInfo = ((releaseCache) =&gt; async (tag) =&gt; {">`tag`</SwmToken>`, `<SwmToken path="bin/contributors.js" pos="179:12:12" line-data="const renderContributorsList = async (tag, template) =&gt; {">`template`</SwmToken>`, options)` (

```javascript
const renderPRsList = async (tag, template, {comments_threshold= 5, awesome_threshold= 5, label = 'add_to_changelog'} = {}) => {
  const release = await getReleaseInfo(tag);

  const prs = {};

  for(const merge of release.merges) {
    const pr = await getIssueById(merge.id);
```

---

</SwmSnippet>

<SwmSnippet path="/bin/contributors.js" line="224">

---

to

```javascript
  const content = compile(release);

  return removeExtraLineBreaks(cleanTemplate(content));
}
```

---

</SwmSnippet>

) loads release info, filters PRs by label and thresholds for comments and reactions, extracts changelog messages from PR bodies, and renders the PR list using the template.

Both functions clean extra line breaks and whitespace from the output for neat formatting.

# utility and exports

<SwmSnippet path="/bin/contributors.js" line="229">

---

The script also provides <SwmToken path="bin/contributors.js" pos="229:2:2" line-data="const getTagRef = async (tag) =&gt; {">`getTagRef`</SwmToken>`(`<SwmToken path="bin/contributors.js" pos="98:16:16" line-data="const getReleaseInfo = ((releaseCache) =&gt; async (tag) =&gt; {">`tag`</SwmToken>`)` (

```javascript
const getTagRef = async (tag) => {
  try {
    return (await exec(`git show-ref --tags "refs/tags/${tag}"`)).stdout.split(' ')[0];
  } catch(e) {
  }
}

export {
  renderContributorsList,
  getReleaseInfo,
  renderPRsList,
  getTagRef
}
```

---

</SwmSnippet>

) to get the git commit SHA for a tag by running a git command.

All main functions are exported for use in other scripts or build processes:

- <SwmToken path="bin/contributors.js" pos="179:2:2" line-data="const renderContributorsList = async (tag, template) =&gt; {">`renderContributorsList`</SwmToken>
- <SwmToken path="bin/contributors.js" pos="98:2:2" line-data="const getReleaseInfo = ((releaseCache) =&gt; async (tag) =&gt; {">`getReleaseInfo`</SwmToken>
- <SwmToken path="bin/contributors.js" pos="189:2:2" line-data="const renderPRsList = async (tag, template, {comments_threshold= 5, awesome_threshold= 5, label = &#39;add_to_changelog&#39;} = {}) =&gt; {">`renderPRsList`</SwmToken>
- <SwmToken path="bin/contributors.js" pos="229:2:2" line-data="const getTagRef = async (tag) =&gt; {">`getTagRef`</SwmToken>

# parameters and running the script

- <SwmToken path="bin/contributors.js" pos="179:2:2" line-data="const renderContributorsList = async (tag, template) =&gt; {">`renderContributorsList`</SwmToken>`(`<SwmToken path="bin/contributors.js" pos="98:16:16" line-data="const getReleaseInfo = ((releaseCache) =&gt; async (tag) =&gt; {">`tag`</SwmToken>`, templatePath)`:

  - <SwmToken path="bin/contributors.js" pos="98:16:16" line-data="const getReleaseInfo = ((releaseCache) =&gt; async (tag) =&gt; {">`tag`</SwmToken>: Git tag string (e.g., "v1.2.3") or empty for unreleased.
  - `templatePath`: path to a Handlebars template file for contributors.\
    Returns a Promise resolving to the rendered contributors list string.

- <SwmToken path="bin/contributors.js" pos="189:2:2" line-data="const renderPRsList = async (tag, template, {comments_threshold= 5, awesome_threshold= 5, label = &#39;add_to_changelog&#39;} = {}) =&gt; {">`renderPRsList`</SwmToken>`(`<SwmToken path="bin/contributors.js" pos="98:16:16" line-data="const getReleaseInfo = ((releaseCache) =&gt; async (tag) =&gt; {">`tag`</SwmToken>`, templatePath, options)`:

  - <SwmToken path="bin/contributors.js" pos="98:16:16" line-data="const getReleaseInfo = ((releaseCache) =&gt; async (tag) =&gt; {">`tag`</SwmToken>: Git tag string or empty for unreleased.
  - `templatePath`: path to a Handlebars template file for PRs.
  - `options` (optional): object with keys:
    - <SwmToken path="bin/contributors.js" pos="189:16:16" line-data="const renderPRsList = async (tag, template, {comments_threshold= 5, awesome_threshold= 5, label = &#39;add_to_changelog&#39;} = {}) =&gt; {">`comments_threshold`</SwmToken> (default 5) - minimum comments to mark PR as "hot"
    - <SwmToken path="bin/contributors.js" pos="189:22:22" line-data="const renderPRsList = async (tag, template, {comments_threshold= 5, awesome_threshold= 5, label = &#39;add_to_changelog&#39;} = {}) =&gt; {">`awesome_threshold`</SwmToken> (default 5) - minimum reaction points to mark PR as "awesome"
    - <SwmToken path="bin/contributors.js" pos="189:28:28" line-data="const renderPRsList = async (tag, template, {comments_threshold= 5, awesome_threshold= 5, label = &#39;add_to_changelog&#39;} = {}) =&gt; {">`label`</SwmToken> (default <SwmToken path="bin/contributors.js" pos="189:33:33" line-data="const renderPRsList = async (tag, template, {comments_threshold= 5, awesome_threshold= 5, label = &#39;add_to_changelog&#39;} = {}) =&gt; {">`add_to_changelog`</SwmToken>) - GitHub label to filter PRs\
      Returns a Promise resolving to the rendered PR list string.

- <SwmToken path="bin/contributors.js" pos="180:9:12" line-data="  const release = await getReleaseInfo(tag);">`getReleaseInfo(tag)`</SwmToken>:\
  Returns a Promise resolving to the release info object for the tag.

- <SwmToken path="bin/contributors.js" pos="229:2:2" line-data="const getTagRef = async (tag) =&gt; {">`getTagRef`</SwmToken>`(`<SwmToken path="bin/contributors.js" pos="98:16:16" line-data="const getReleaseInfo = ((releaseCache) =&gt; async (tag) =&gt; {">`tag`</SwmToken>`)`:\
  Returns a Promise resolving to the git commit SHA for the tag.

To run the script, invoke these exported functions from a Node.js environment. The script depends on <SwmToken path="bin/contributors.js" pos="108:2:6" line-data="    `npx auto-changelog --unreleased-only --stdout --commit-limit false --template json` :">`npx auto-changelog`</SwmToken> being available and requires GitHub API access for data fetching. Templates must be provided as Handlebars files.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBYXhpb3MlM0ElM0FTd2ltbS1EZW1v" repo-name="axios"><sup>Powered by [Swimm](/)</sup></SwmMeta>
