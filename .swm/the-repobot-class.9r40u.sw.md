---
title: The RepoBot class
---
# What is <SwmToken path="bin/RepoBot.js" pos="20:2:2" line-data="class RepoBot {">`RepoBot`</SwmToken>

This document covers the following aspects of <SwmToken path="bin/RepoBot.js" pos="20:2:2" line-data="class RepoBot {">`RepoBot`</SwmToken>:

1. What <SwmToken path="bin/RepoBot.js" pos="20:2:2" line-data="class RepoBot {">`RepoBot`</SwmToken> is and its purpose
2. Variables and functions defined in <SwmToken path="bin/RepoBot.js" pos="20:2:2" line-data="class RepoBot {">`RepoBot`</SwmToken>, with detailed explanations for each

# What is <SwmToken path="bin/RepoBot.js" pos="20:2:2" line-data="class RepoBot {">`RepoBot`</SwmToken>

<SwmToken path="bin/RepoBot.js" pos="20:2:2" line-data="class RepoBot {">`RepoBot`</SwmToken> is a class defined in <SwmPath>[bin/RepoBot.js](bin/RepoBot.js)</SwmPath> that automates interactions with GitHub pull requests and releases. It is designed to streamline the process of labeling, commenting, and notifying contributors about published pull requests. <SwmToken path="bin/RepoBot.js" pos="20:2:2" line-data="class RepoBot {">`RepoBot`</SwmToken> leverages GitHub's API to perform these actions, making it useful for maintaining release workflows and automating communication in repositories.

<SwmSnippet path="/bin/RepoBot.js" line="38">

---

The function <SwmToken path="bin/RepoBot.js" pos="38:3:3" line-data="  async addComment(targetId, message) {">`addComment`</SwmToken> is responsible for adding a comment to a specific GitHub issue or pull request. It delegates the actual comment creation to the <SwmToken path="bin/RepoBot.js" pos="39:7:7" line-data="    return this.github.createComment(targetId, message);">`createComment`</SwmToken> method of the GitHub API wrapper, passing the target ID and message.

```javascript
  async addComment(targetId, message) {
    return this.github.createComment(targetId, message);
  }
```

---

</SwmSnippet>

<SwmSnippet path="/bin/RepoBot.js" line="42">

---

The function <SwmToken path="bin/RepoBot.js" pos="42:3:3" line-data="  async notifyPRPublished(id, tag) {">`notifyPRPublished`</SwmToken> manages the process of labeling a pull request as published and posting a release comment if appropriate. It checks if the pull request is merged, applies the release tag as a label, skips automated or collaborator <SwmToken path="bin/RepoBot.js" pos="109:18:18" line-data="    console.log(colorize()`Found ${merges.length} PRs in ${tag}:`);">`PRs`</SwmToken>, and ensures that duplicate comments are not posted. If all conditions are met, it generates a comment using a Handlebars template and posts it to the pull request.

```javascript
  async notifyPRPublished(id, tag) {
    let pr;

    try {
      pr = await this.github.getPR(id);
    } catch (err) {
      if(err.response?.status === 404) {
        throw new Error(`PR #${id} not found (404)`);
      }

      throw err;
    }

    tag = normalizeTag(tag);

    const {merged, labels, user: {login, type}} = pr;

    const isBot = type === 'Bot';

    if (!merged) {
      return false
    }

    await this.github.appendLabels(id, [tag]);

    if (isBot || labels.find(({name}) => name === 'automated pr') || (skipCollaboratorPRs && await this.github.isCollaborator(login))) {
      return false;
    }

    const comments = await this.github.getComments(id, {desc: true});

    const comment = comments.find(
      ({body, user}) => user.login === GITHUB_BOT_LOGIN && body.indexOf('published in') >= 0
    )

    if (comment) {
      console.log(colorize()`Release comment [${comment.html_url}] already exists in #${pr.id}`);
      return false;
    }

    const author = await this.github.getUser(login);

    author.isBot = isBot;

    const message = await this.constructor.renderTemplate(this.templates.published, {
      id,
      author,
      release: {
        tag,
        url: `https://github.com/${this.owner}/${this.repo}/releases/tag/${tag}`
      }
    });

    return await this.addComment(id, message);
  }
```

---

</SwmSnippet>

<SwmSnippet path="/bin/RepoBot.js" line="98">

---

The function <SwmToken path="bin/RepoBot.js" pos="98:3:3" line-data="  async notifyPublishedPRs(tag) {">`notifyPublishedPRs`</SwmToken> iterates over all pull requests associated with a given release tag. For each pull request, it attempts to notify it as published by calling <SwmToken path="bin/RepoBot.js" pos="116:11:11" line-data="        const result = await this.notifyPRPublished(pr.id, tag);">`notifyPRPublished`</SwmToken>. It logs the outcome for each PR, handling errors gracefully and continuing with the next PR in the list.

```javascript
  async notifyPublishedPRs(tag) {
    tag = normalizeTag(tag);

    const release = await getReleaseInfo(tag);

    if (!release) {
      throw Error(colorize()`Can't get release info for ${tag}`);
    }

    const {merges} = release;

    console.log(colorize()`Found ${merges.length} PRs in ${tag}:`);

    let i = 0;

    for (const pr of merges) {
      try {
        console.log(colorize()`${i++}) Notify PR #${pr.id}`)
        const result = await this.notifyPRPublished(pr.id, tag);
        console.log('✔️', result ? 'Label, comment' : 'Label');
      } catch (err) {
        console.warn(colorize('green', 'red')`❌ Failed notify PR ${pr.id}: ${err.message}`);
      }
    }
  }
```

---

</SwmSnippet>

<SwmSnippet path="/bin/RepoBot.js" line="124">

---

The static function <SwmToken path="bin/RepoBot.js" pos="124:5:5" line-data="  static async renderTemplate(template, data) {">`renderTemplate`</SwmToken> compiles and renders a Handlebars template using data provided at runtime. It reads the template file asynchronously and applies the data to generate the final message, which is typically used for release comments.

```javascript
  static async renderTemplate(template, data) {
    return Handlebars.compile(String(await fs.readFile(template)))(data);
  }
```

---

</SwmSnippet>

# Usage

## <SwmToken path="bin/RepoBot.js" pos="20:2:2" line-data="class RepoBot {">`RepoBot`</SwmToken> Usage in <SwmPath>[bin/actions/notify_published.js](bin/actions/notify_published.js)</SwmPath>

In the file <SwmPath>[bin/actions/notify_published.js](bin/actions/notify_published.js)</SwmPath>, an instance of <SwmToken path="bin/RepoBot.js" pos="20:2:2" line-data="class RepoBot {">`RepoBot`</SwmToken> is created to handle notifications for published pull requests. The bot's method <SwmToken path="bin/RepoBot.js" pos="98:3:3" line-data="  async notifyPublishedPRs(tag) {">`notifyPublishedPRs`</SwmToken> is called with a tag string to trigger the notification process. This demonstrates how <SwmToken path="bin/RepoBot.js" pos="20:2:2" line-data="class RepoBot {">`RepoBot`</SwmToken> is used to automate communication related to repository publishing events.

## <SwmToken path="bin/RepoBot.js" pos="20:2:2" line-data="class RepoBot {">`RepoBot`</SwmToken> Export in <SwmPath>[bin/RepoBot.js](bin/RepoBot.js)</SwmPath>

The class <SwmToken path="bin/RepoBot.js" pos="20:2:2" line-data="class RepoBot {">`RepoBot`</SwmToken> is defined and exported as the default export in <SwmPath>[bin/RepoBot.js](bin/RepoBot.js)</SwmPath>. This allows other parts of the codebase to import and instantiate <SwmToken path="bin/RepoBot.js" pos="20:2:2" line-data="class RepoBot {">`RepoBot`</SwmToken> for various automation tasks related to repository management.

&nbsp;

*This is an auto-generated document by Swimm 🌊 and has not yet been verified by a human*

<SwmMeta version="3.0.0" repo-id="Z2l0aHViJTNBJTNBYXhpb3MlM0ElM0FTd2ltbS1EZW1v" repo-name="axios"><sup>Powered by [Swimm](/)</sup></SwmMeta>
