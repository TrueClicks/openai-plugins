# Google Ads plugin for Codex and ChatGPT

Free Google Ads account management plugin by [TrueClicks](https://www.trueclicks.com). Download account data, run custom GAQL queries, and validate and execute changes. It works in Codex (CLI and the ChatGPT desktop app) and in ChatGPT Work running locally in the desktop app.

The skills run shell commands (`curl`) against the TrueClicks API and read and write files in a local account folder. They don't work in ChatGPT Chat on the web or mobile, which can't access local files.

For full documentation, visit [claudeppc.ai](https://claudeppc.ai).

## Installation

Add the TrueClicks marketplace once:

```text
codex plugin marketplace add TrueClicks/openai-plugins
```

Then install the plugin:

- **Codex CLI:** run `codex`, enter `/plugins`, open the **TrueClicks** tab, install **Google Ads**, and start a new session.
- **ChatGPT desktop app:** open the **Plugins** tab, find **Google Ads** under the TrueClicks source, select the plus button, and start a new chat.

## Setup

### 1. Create config.json

Go to [claudeppc.ai](https://claudeppc.ai) and click **Get GPT Token** to sign in with your Google account, select your MCC and Google Ads account, then download the `config.json` file.

Each config file is tied to a single Google Ads account. Managing multiple accounts? Create a separate config for each one.

`config.json` contains your access token. Don't commit it or share it.

### 2. Put config.json in a folder

Create a folder for the Google Ads account and move the downloaded `config.json` into it. This folder is your working directory.

### 3. Allow network access to api.claudeppc.ai

The skills call `https://api.claudeppc.ai`. Sandboxed commands may not be allowed to reach the network.

- **Codex CLI:** approve the network request when Codex asks, or allow network access for sandboxed commands in `~/.codex/config.toml`:

  ```toml
  [sandbox_workspace_write]
  network_access = true
  ```

- **ChatGPT Work:** in **Settings > Data controls > Work network access**, turn on **Allow public internet access**. On a Business or Enterprise workspace, your admin may need to allow it.

### 4. Start managing your account

Open Codex in that folder (or start a ChatGPT Work task in **Local** mode on that folder) and run the `google-ads-get` skill to download your account data.

Every download writes an `AGENTS.md` workspace guide into the folder, so later sessions know how the data is organized and which skills to use.

## Skills

| Skill | Codex | ChatGPT | Description |
|-------|-------|---------|-------------|
| google-ads-get | `$google-ads-get` | `@google-ads-get` | Download account structure and performance data |
| google-ads-get-custom | `$google-ads-get-custom` | `@google-ads-get-custom` | Run a custom GAQL query against the account |
| google-ads-validate-changes | `$google-ads-validate-changes` | `@google-ads-validate-changes` | Validate pending changes before applying them (dry-run) |
| google-ads-execute-changes | `$google-ads-execute-changes` | `@google-ads-execute-changes` | Apply validated changes to the live account |

`google-ads-get-custom` can also run on its own when the model needs data that isn't in the standard export. The other three run only when you ask for them.

## Updating

```text
codex plugin marketplace upgrade trueclicks
```

When a newer version is available, the skills show a notice after each API call.

## FAQ

For frequently asked questions and troubleshooting, visit [claudeppc.ai](https://claudeppc.ai).
