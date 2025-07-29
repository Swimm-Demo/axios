---
title: Script to Update Sponsors Block in README.md
---
# introduction

This document explains the script that updates the sponsors block in the <SwmPath>[README.md](README.md)</SwmPath> file. It answers these questions:

1. How does the script fetch sponsor data reliably?
2. How does it identify and replace the sponsors block in the README?
3. How does it handle cases when the sponsors block is already up to date or missing?
4. What are the script's inputs and outputs, and how is it executed?

# fetching sponsor data with retries

<SwmSnippet path="/bin/sponsors.js" line="12">

---

The script uses a custom axios instance with a specific <SwmToken path="bin/sponsors.js" pos="8:2:4" line-data="    &quot;User-Agent&quot;: &#39;Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/126.0.0.0 Safari/537.36&#39;">`User-Agent`</SwmToken> header to fetch sponsor data from a URL. To handle transient network errors, it implements a retry mechanism with exponential backoff. The function attempts the HTTP GET request up to a maximum number of retries (default 3). If a request fails, it waits for an increasing delay before retrying. This ensures more robust data fetching without failing immediately on temporary issues.

```javascript
const getWithRetry = (url, retries = 3) => {
  let counter = 0;
  const doRequest = async () => {
    try {
      return await axios.get(url)
    } catch (err) {
      if (counter++ >= retries) {
        throw err;
      }
      await new Promise(resolve => setTimeout(resolve, counter ** counter * 1000));
      return doRequest();
    }
  }
```

---

</SwmSnippet>

# locating and updating the sponsors block in <SwmPath>[README.md](README.md)</SwmPath>

The core function reads the <SwmPath>[README.md](README.md)</SwmPath> file and looks for a specific marker string that indicates where the sponsors block should be injected or updated. If the marker is found, it slices the file content at that point. Then it fetches the latest sponsor content from the given URL.

<SwmSnippet path="/bin/sponsors.js" line="29">

---

If the current content before the marker differs from the fetched sponsor content, it overwrites the README with the new sponsor content plus the remaining original content after the marker. This approach replaces only the sponsors block without affecting the rest of the README.

```javascript
const updateReadmeSponsors = async (url, path, marker = '<!--<div>marker</div>-->') => {
  let fileContent = (await fs.readFile(path)).toString();

  const index = fileContent.indexOf(marker);

  if(index >= 0) {
    const readmeContent = fileContent.slice(index);

    let {data: sponsorContent} = await getWithRetry(url);
    sponsorContent += '\n';

    const currentSponsorContent = fileContent.slice(0, index);

    if (currentSponsorContent !== sponsorContent) {
      console.log(colorize()`Sponsor block in [${path}] is outdated`);
      await fs.writeFile(path, sponsorContent + readmeContent);
      return sponsorContent;
    } else {
      console.log(colorize()`Sponsor block in [${path}] is up to date`);
    }
```

---

</SwmSnippet>

# handling missing marker and unchanged content

<SwmSnippet path="/bin/sponsors.js" line="49">

---

If the marker is not found in the README, the script logs a warning and does not modify the file. If the sponsor content is already up to date (matches the current content before the marker), it logs that no update is needed and leaves the file untouched. This prevents unnecessary file writes.

```javascript
  } else {
    console.warn(colorize()`Can not find marker (${marker}) in ${path} to inject sponsor block`);
  }
```

---

</SwmSnippet>

<SwmSnippet path="/bin/sponsors.js" line="42">

---

&nbsp;

```javascript
    if (currentSponsorContent !== sponsorContent) {
      console.log(colorize()`Sponsor block in [${path}] is outdated`);
      await fs.writeFile(path, sponsorContent + readmeContent);
      return sponsorContent;
    } else {
      console.log(colorize()`Sponsor block in [${path}] is up to date`);
    }
```

---

</SwmSnippet>

# script parameters and execution

The script takes three parameters:

- The URL to fetch the sponsors markdown content (default example: '<https://axios-http.com/data/sponsors.md>')
- The path to the README file to update (default <SwmPath>[README.md](README.md)</SwmPath>)
- The marker string in the README that identifies where to inject the sponsors block (default '<!--<div>marker</div>-->')

<SwmSnippet path="/bin/sponsors.js" line="56">

---

It is executed as an immediately invoked async function that calls the update function with the URL and README path. After updating, it writes a GitHub Actions output variable indicating whether the README was changed. If updated, it also saves the new sponsors content to a temporary file for further use.

```javascript
(async(url) => {
  const newContent = await updateReadmeSponsors(url, './README.md');

  await exec(`echo "changed=${newContent ? 'true' : 'false'}" >> $GITHUB_OUTPUT`);
  if (newContent !== false) {
    await fs.mkdir('./temp').catch(() => {});
    await fs.writeFile('./temp/sponsors.md', newContent);
  }
})('https://axios-http.com/data/sponsors.md');
```

---

</SwmSnippet>

# how to run the script

Run the script with Node.js from the command line. It expects the <SwmPath>[README.md](README.md)</SwmPath> file in the current directory and fetches sponsor data from the hardcoded URL unless modified. It logs status messages about updates or warnings. The script is designed to be used in CI environments like GitHub Actions but can be run locally as well.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBYXhpb3MlM0ElM0FTd2ltbS1EZW1v" repo-name="axios"><sup>Powered by [Swimm](/)</sup></SwmMeta>
