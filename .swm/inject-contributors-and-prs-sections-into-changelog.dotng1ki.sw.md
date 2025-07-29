---
title: Inject Contributors and PRs Sections into Changelog
---
# introduction

This document explains how the script in <SwmPath>[bin/injectContributorsList.js](bin/injectContributorsList.js)</SwmPath> works to inject contributors and pull request (PR) sections into the changelog file. We will cover:

1. How the script identifies where to inject content in the changelog.
2. How it decides whether to inject or skip sections.
3. How the injection is performed and saved back to the file.
4. How to run the script and what parameters it uses.

# identifying injection points in the changelog

The script reads the changelog file (default <SwmPath>[CHANGELOG.md](CHANGELOG.md)</SwmPath>) and uses a regular expression to find release headers, which look like markdown headings with tags (e.g., `# [v1.2.3]`). It processes the changelog content between these headers to check if the target section (contributors or <SwmToken path="bin/injectContributorsList.js" pos="69:2:2" line-data="  &#39;PRs&#39;,">`PRs`</SwmToken>) already exists.

<SwmSnippet path="/bin/injectContributorsList.js" line="18">

---

This is done by reading the file content and applying a regex to find headers and slice the content accordingly. The regex and slicing logic are crucial to correctly locate where to inject new sections.

```javascript
  const content = String(await fs.readFile(infile));
  const headerRE = /^#+\s+\[([-_\d.\w]+)].+?$/mig;

  let tag;
  let index = 0;
  let isFirstTag = true;
```

---

</SwmSnippet>

# deciding when and what to inject

For each release section found, the script checks if the contributors or <SwmToken path="bin/injectContributorsList.js" pos="69:2:2" line-data="  &#39;PRs&#39;,">`PRs`</SwmToken> section is already present using a section-specific regex. If the section is missing, it decides whether to inject content based on whether the release tag exists in the git repository (using <SwmToken path="bin/injectContributorsList.js" pos="39:15:15" line-data="        const target = isFirstTag &amp;&amp; (!await getTagRef(currentTag)) ? &#39;&#39; : currentTag;">`getTagRef`</SwmToken>) and whether it is the first tag processed (to handle unreleased versions).

<SwmSnippet path="/bin/injectContributorsList.js" line="25">

---

This logic ensures that the script does not duplicate sections and handles unreleased or missing tags gracefully.

```javascript
  const newContent = await asyncReplace(content, headerRE, async (match, nextTag, offset) => {
    const releaseContent = content.slice(index, offset);

    const hasSection = contributorsRE.test(releaseContent);

    const currentTag = tag;

    tag = nextTag;
    index = offset + match.length;

    if(currentTag) {
      if (hasSection) {
        console.log(colorize()`[${currentTag}]: ✓ OK`);
      } else {
        const target = isFirstTag && (!await getTagRef(currentTag)) ? '' : currentTag;

        console.log(colorize()`[${currentTag}]: ❌ MISSED` + (!target ? ' (UNRELEASED)' : ''));

        isFirstTag = false;
```

---

</SwmSnippet>

# injecting content and updating the changelog

When injection is needed, the script calls an injector function specific to the section type (contributors or <SwmToken path="bin/injectContributorsList.js" pos="69:2:2" line-data="  &#39;PRs&#39;,">`PRs`</SwmToken>). This function generates the markdown content to insert, using templates and data fetched from git or other sources.

The generated section is then prepended before the release header in the changelog. The script logs the rendered section for verification and finally writes the updated changelog content back to the file.

<SwmSnippet path="/bin/injectContributorsList.js" line="45">

---

This approach cleanly inserts the new sections without disrupting existing content.

```javascript
        console.log(`Generating section...`);

        const section = await injector(target);

        if (!section) {
          return match;
        }

        console.log(colorize()`\nRENDERED SECTION [${name}] for [${currentTag}]:`);
        console.log('-------------BEGIN--------------\n');
        console.log(section);
        console.log('--------------END---------------\n');

        return section + '\n' + match;
      }
    }

    return match;
  });

  await fs.writeFile(infile, newContent);
}
```

---

</SwmSnippet>

# running the script and parameters

The script exports an async function <SwmToken path="bin/injectContributorsList.js" pos="13:2:2" line-data="const injectSection = async (name, contributorsRE, injector, infile = &#39;../CHANGELOG.md&#39;) =&gt; {">`injectSection`</SwmToken> that takes:

- name: the section name (e.g., <SwmToken path="bin/injectContributorsList.js" pos="69:2:2" line-data="  &#39;PRs&#39;,">`PRs`</SwmToken> or 'contributors')
- <SwmToken path="bin/injectContributorsList.js" pos="13:12:12" line-data="const injectSection = async (name, contributorsRE, injector, infile = &#39;../CHANGELOG.md&#39;) =&gt; {">`contributorsRE`</SwmToken>: a regex to detect if the section exists
- injector: a function that generates the section content given a tag
- infile: optional path to the changelog file (default <SwmPath>[CHANGELOG.md](CHANGELOG.md)</SwmPath>)

At the bottom of the script, <SwmToken path="bin/injectContributorsList.js" pos="13:2:2" line-data="const injectSection = async (name, contributorsRE, injector, infile = &#39;../CHANGELOG.md&#39;) =&gt; {">`injectSection`</SwmToken> is called twice: once for <SwmToken path="bin/injectContributorsList.js" pos="69:2:2" line-data="  &#39;PRs&#39;,">`PRs`</SwmToken> and once for contributors, each with their own regex and injector function. The <SwmToken path="bin/injectContributorsList.js" pos="69:2:2" line-data="  &#39;PRs&#39;,">`PRs`</SwmToken> injector skips injection if the tag exists, while the contributors injector always runs.

<SwmSnippet path="/bin/injectContributorsList.js" line="10">

---

To run the script, execute it with Node.js in the bin directory. It will update the changelog file in place by injecting missing contributors and <SwmToken path="bin/injectContributorsList.js" pos="69:2:2" line-data="  &#39;PRs&#39;,">`PRs`</SwmToken> sections.

```javascript
const CONTRIBUTORS_TEMPLATE = path.resolve(__dirname, '../templates/contributors.hbs');
const PRS_TEMPLATE = path.resolve(__dirname, '../templates/prs.hbs');

const injectSection = async (name, contributorsRE, injector, infile = '../CHANGELOG.md') => {
  console.log(colorize()`Checking ${name} sections in ${infile}`);

  infile = path.resolve(__dirname, infile);
```

---

</SwmSnippet>

<SwmSnippet path="/bin/injectContributorsList.js" line="68">

---

&nbsp;

```javascript
await injectSection(
  'PRs',
  /^\s*### PRs/mi,
  (tag) => tag ? '' : renderPRsList(tag, PRS_TEMPLATE, {awesome_threshold: 5, comments_threshold: 7}),
);

await injectSection(
  'contributors',
  /^\s*### Contributors/mi,
  (tag) => renderContributorsList(tag, CONTRIBUTORS_TEMPLATE)
);
```

---

</SwmSnippet>

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBYXhpb3MlM0ElM0FTd2ltbS1EZW1v" repo-name="axios"><sup>Powered by [Swimm](/)</sup></SwmMeta>
