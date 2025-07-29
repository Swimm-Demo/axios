---
title: The GithubAPI class
---
# What is <SwmToken path="bin/GithubAPI.js" pos="9:6:6" line-data="export default class GithubAPI {">`GithubAPI`</SwmToken>

This document covers the <SwmToken path="bin/GithubAPI.js" pos="9:6:6" line-data="export default class GithubAPI {">`GithubAPI`</SwmToken> class in <SwmPath>[bin/GithubAPI.js](bin/GithubAPI.js)</SwmPath>, focusing on:

1. What <SwmToken path="bin/GithubAPI.js" pos="9:6:6" line-data="export default class GithubAPI {">`GithubAPI`</SwmToken> is and its purpose
2. Detailed explanations of all variables and functions defined in <SwmToken path="bin/GithubAPI.js" pos="9:6:6" line-data="export default class GithubAPI {">`GithubAPI`</SwmToken>

# What is <SwmToken path="bin/GithubAPI.js" pos="9:6:6" line-data="export default class GithubAPI {">`GithubAPI`</SwmToken>

<SwmToken path="bin/GithubAPI.js" pos="9:6:6" line-data="export default class GithubAPI {">`GithubAPI`</SwmToken> is a class designed to provide a convenient interface for interacting with the GitHub REST API for a specific repository. It encapsulates common GitHub operations such as managing issues, pull requests, comments, labels, releases, and tags, allowing other parts of the codebase to perform these actions with simple method calls. The class is initialized with a repository owner and name, and it uses an Axios instance configured for the target repository.

<SwmSnippet path="/bin/GithubAPI.js" line="26">

---

The function <SwmToken path="bin/GithubAPI.js" pos="26:3:3" line-data="  async createComment(issue, body) {">`createComment`</SwmToken> creates a new comment on a specified issue by sending a POST request to the GitHub API with the comment body.

```javascript
  async createComment(issue, body) {
    return (await this.axios.post(`/issues/${issue}/comments`, {body})).data;
  }
```

---

</SwmSnippet>

<SwmSnippet path="/bin/GithubAPI.js" line="30">

---

The function <SwmToken path="bin/GithubAPI.js" pos="30:3:3" line-data="  async getComments(issue, {desc = false, per_page= 100, page = 1} = {}) {">`getComments`</SwmToken> retrieves a list of comments for a given issue, supporting pagination and sorting direction.

```javascript
  async getComments(issue, {desc = false, per_page= 100, page = 1} = {}) {
    return (await this.axios.get(`/issues/${issue}/comments`, {params: {direction: desc ? 'desc' : 'asc', per_page, page}})).data;
  }
```

---

</SwmSnippet>

<SwmSnippet path="/bin/GithubAPI.js" line="34">

---

The function <SwmToken path="bin/GithubAPI.js" pos="34:3:3" line-data="  async getComment(id) {">`getComment`</SwmToken> fetches a single comment by its unique identifier from the GitHub API.

```javascript
  async getComment(id) {
    return (await this.axios.get(`/issues/comments/${id}`)).data;
  }
```

---

</SwmSnippet>

<SwmSnippet path="/bin/GithubAPI.js" line="38">

---

The function <SwmToken path="bin/GithubAPI.js" pos="38:3:3" line-data="  async updateComment(id, body) {">`updateComment`</SwmToken> updates the content of an existing comment by sending a PATCH request with the new body.

```javascript
  async updateComment(id, body) {
    return (await this.axios.patch(`/issues/comments/${id}`, {body})).data;
  }
```

---

</SwmSnippet>

<SwmSnippet path="/bin/GithubAPI.js" line="42">

---

The function <SwmToken path="bin/GithubAPI.js" pos="42:3:3" line-data="  async appendLabels(issue, labels) {">`appendLabels`</SwmToken> adds one or more labels to a specified issue by posting the labels array to the GitHub API.

```javascript
  async appendLabels(issue, labels) {
    return (await this.axios.post(`/issues/${issue}/labels`, {labels})).data;
  }
```

---

</SwmSnippet>

<SwmSnippet path="/bin/GithubAPI.js" line="46">

---

The function <SwmToken path="bin/GithubAPI.js" pos="46:3:3" line-data="  async getUser(user) {">`getUser`</SwmToken> retrieves public information about a GitHub user by their username.

```javascript
  async getUser(user) {
    return (await githubAxios.get(`/users/${user}`)).data;
  }
```

---

</SwmSnippet>

<SwmSnippet path="/bin/GithubAPI.js" line="50">

---

The function <SwmToken path="bin/GithubAPI.js" pos="50:3:3" line-data="  async isCollaborator(user) {">`isCollaborator`</SwmToken> checks if a user is a collaborator on the repository by making a GET request and returns true if the response status is 204.

```javascript
  async isCollaborator(user) {
    try {
      return (await this.axios.get(`/collaborators/${user}`)).status === 204;
    } catch (e) {

    }
  }
```

---

</SwmSnippet>

<SwmSnippet path="/bin/GithubAPI.js" line="58">

---

The function <SwmToken path="bin/GithubAPI.js" pos="58:3:3" line-data="  async deleteLabel(issue, label) {">`deleteLabel`</SwmToken> removes a specific label from an issue by sending a DELETE request to the GitHub API.

```javascript
  async deleteLabel(issue, label) {
    return (await this.axios.delete(`/issues/${issue}/labels/${label}`)).data;
  }
```

---

</SwmSnippet>

<SwmSnippet path="/bin/GithubAPI.js" line="62">

---

The function <SwmToken path="bin/GithubAPI.js" pos="62:3:3" line-data="  async getIssue(issue) {">`getIssue`</SwmToken> fetches detailed information about a specific issue in the repository.

```javascript
  async getIssue(issue) {
    return (await this.axios.get(`/issues/${issue}`)).data;
  }
```

---

</SwmSnippet>

<SwmSnippet path="/bin/GithubAPI.js" line="66">

---

The function <SwmToken path="bin/GithubAPI.js" pos="66:3:3" line-data="  async getPR(issue) {">`getPR`</SwmToken> retrieves information about a pull request by its number.

```javascript
  async getPR(issue) {
    return (await this.axios.get(`/pulls/${issue}`)).data;
  }
```

---

</SwmSnippet>

<SwmSnippet path="/bin/GithubAPI.js" line="70">

---

The function <SwmToken path="bin/GithubAPI.js" pos="70:3:3" line-data="  async getIssues({state= &#39;open&#39;, labels, sort = &#39;created&#39;, desc = false, per_page = 100, page = 1}) {">`getIssues`</SwmToken> lists issues in the repository, supporting filters such as state, labels, sorting, and pagination.

```javascript
  async getIssues({state= 'open', labels, sort = 'created', desc = false, per_page = 100, page = 1}) {
    return (await this.axios.get(`/issues`, {params: {state, labels, sort, direction: desc ? 'desc' : 'asc', per_page, page}})).data;
  }
```

---

</SwmSnippet>

<SwmSnippet path="/bin/GithubAPI.js" line="74">

---

The function <SwmToken path="bin/GithubAPI.js" pos="74:3:3" line-data="  async updateIssue(issue, data) {">`updateIssue`</SwmToken> updates properties of an issue, such as title, body, or state, by sending a PATCH request with the update data.

```javascript
  async updateIssue(issue, data) {
    return (await this.axios.patch(`/issues/${issue}`, data)).data;
  }
```

---

</SwmSnippet>

<SwmSnippet path="/bin/GithubAPI.js" line="78">

---

The function <SwmToken path="bin/GithubAPI.js" pos="78:3:3" line-data="  async closeIssue(issue) {">`closeIssue`</SwmToken> closes an issue by updating its state to 'closed' using the <SwmToken path="bin/GithubAPI.js" pos="79:5:5" line-data="    return this.updateIssue(issue, {">`updateIssue`</SwmToken> method.

```javascript
  async closeIssue(issue) {
    return this.updateIssue(issue, {
      state: "closed"
    })
  }
```

---

</SwmSnippet>

<SwmSnippet path="/bin/GithubAPI.js" line="84">

---

The function <SwmToken path="bin/GithubAPI.js" pos="84:3:3" line-data="  async getReleases({per_page = 30, page= 1} = {}) {">`getReleases`</SwmToken> retrieves a paginated list of releases for the repository.

```javascript
  async getReleases({per_page = 30, page= 1} = {}) {
    return (await this.axios.get(`/releases`, {params: {per_page, page}})).data;
  }
```

---

</SwmSnippet>

<SwmSnippet path="/bin/GithubAPI.js" line="88">

---

The function <SwmToken path="bin/GithubAPI.js" pos="88:3:3" line-data="  async getRelease(release = &#39;latest&#39;) {">`getRelease`</SwmToken> fetches information about a specific release, either by tag or by release ID, using the <SwmToken path="bin/GithubAPI.js" pos="89:12:12" line-data="    return (await this.axios.get(parseVersion(release) ? `/releases/tags/${release}` : `/releases/${release}`)).data;">`parseVersion`</SwmToken> helper to determine the request path.

```javascript
  async getRelease(release = 'latest') {
    return (await this.axios.get(parseVersion(release) ? `/releases/tags/${release}` : `/releases/${release}`)).data;
  }
```

---

</SwmSnippet>

<SwmSnippet path="/bin/GithubAPI.js" line="92">

---

The function <SwmToken path="bin/GithubAPI.js" pos="92:3:3" line-data="  async getTags({per_page = 30, page= 1} = {}) {">`getTags`</SwmToken> lists tags in the repository, supporting pagination.

```javascript
  async getTags({per_page = 30, page= 1} = {}) {
    return (await this.axios.get(`/tags`, {params: {per_page, page}})).data;
  }
```

---

</SwmSnippet>

<SwmSnippet path="/bin/GithubAPI.js" line="96">

---

The function <SwmToken path="bin/GithubAPI.js" pos="96:3:3" line-data="  async reopenIssue(issue) {">`reopenIssue`</SwmToken> reopens a closed issue by updating its state to 'open' using the <SwmToken path="bin/GithubAPI.js" pos="97:5:5" line-data="    return this.updateIssue(issue, {">`updateIssue`</SwmToken> method.

```javascript
  async reopenIssue(issue) {
    return this.updateIssue(issue, {
      state: "open"
    })
  }
```

---

</SwmSnippet>

<SwmSnippet path="/bin/GithubAPI.js" line="102">

---

The static function <SwmToken path="bin/GithubAPI.js" pos="102:5:5" line-data="  static async getTagRef(tag) {">`getTagRef`</SwmToken> retrieves the git reference hash for a given tag by executing a git command.

```javascript
  static async getTagRef(tag) {
    try {
      return (await exec(`git show-ref --tags "refs/tags/${tag}"`)).stdout.split(' ')[0];
    } catch (e) {
    }
  }
```

---

</SwmSnippet>

<SwmSnippet path="/bin/GithubAPI.js" line="109">

---

The static function <SwmToken path="bin/GithubAPI.js" pos="109:5:5" line-data="  static async getLatestTag() {">`getLatestTag`</SwmToken> finds the most recent tag in the repository by executing a git command and parsing the output.

```javascript
  static async getLatestTag() {
    try{
      const {stdout} = await exec(`git for-each-ref refs/tags --sort=-taggerdate --format='%(refname)' --count=1`);

      return stdout.split('/').pop();
    } catch (e) {}
  }
```

---

</SwmSnippet>

<SwmSnippet path="/bin/GithubAPI.js" line="117">

---

The static function <SwmToken path="bin/GithubAPI.js" pos="117:3:3" line-data="  static normalizeTag(tag){">`normalizeTag`</SwmToken> ensures a tag string is prefixed with 'v', removing any existing 'v' prefix before adding it.

```javascript
  static normalizeTag(tag){
    return tag ? 'v' + tag.replace(/^v/, '') : '';
  }
```

---

</SwmSnippet>

# Usage

## <SwmToken path="bin/GithubAPI.js" pos="9:6:6" line-data="export default class GithubAPI {">`GithubAPI`</SwmToken> in <SwmPath>[bin/GithubAPI.js](bin/GithubAPI.js)</SwmPath>

In <SwmPath>[bin/GithubAPI.js](bin/GithubAPI.js)</SwmPath>, the <SwmToken path="bin/GithubAPI.js" pos="9:6:6" line-data="export default class GithubAPI {">`GithubAPI`</SwmToken> class prototype methods <SwmToken path="bin/GithubAPI.js" pos="46:3:3" line-data="  async getUser(user) {">`getUser`</SwmToken> and <SwmToken path="bin/GithubAPI.js" pos="50:3:3" line-data="  async isCollaborator(user) {">`isCollaborator`</SwmToken> are enhanced with memoization to optimize repeated calls by caching their promise results. This improves performance when these methods are called multiple times with the same arguments.

## <SwmToken path="bin/GithubAPI.js" pos="9:6:6" line-data="export default class GithubAPI {">`GithubAPI`</SwmToken> in <SwmPath>[bin/RepoBot.js](bin/RepoBot.js)</SwmPath>

In <SwmPath>[bin/RepoBot.js](bin/RepoBot.js)</SwmPath>, an instance of <SwmToken path="bin/GithubAPI.js" pos="9:6:6" line-data="export default class GithubAPI {">`GithubAPI`</SwmToken> is created or passed as a parameter to manage repository-specific operations. The instance is assigned to the 'github' property, and its 'owner' and 'repo' properties are used to set corresponding values in the RepoBot class, linking the bot's functionality to a specific GitHub repository.

## <SwmToken path="bin/GithubAPI.js" pos="9:6:6" line-data="export default class GithubAPI {">`GithubAPI`</SwmToken> in <SwmPath>[bin/api.js](bin/api.js)</SwmPath>

In <SwmPath>[bin/api.js](bin/api.js)</SwmPath>, a new <SwmToken path="bin/GithubAPI.js" pos="9:6:6" line-data="export default class GithubAPI {">`GithubAPI`</SwmToken> instance is created with the parameters 'axios' for both owner and repository names, and this instance is exported as the default export. This setup provides a ready-to-use API client configured for the 'axios/axios' repository, facilitating GitHub interactions related to this project.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBYXhpb3MlM0ElM0FTd2ltbS1EZW1v" repo-name="axios"><sup>Powered by [Swimm](/)</sup></SwmMeta>
