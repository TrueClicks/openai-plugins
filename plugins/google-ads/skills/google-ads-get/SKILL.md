---
name: google-ads-get
description: Download Google Ads account structure and performance data from the TrueClicks API into the current folder. Use only when the user explicitly asks to download or refresh their Google Ads data.
---

# Download Google Ads data

Downloads the Google Ads account linked to `config.json` into this folder as markdown files.

## Shell and network notes

- Run every command from the folder that contains `config.json`.
- On Windows (PowerShell), use the PowerShell commands. On macOS or Linux, use the bash commands. On Windows always call `curl.exe`, never `curl` (in Windows PowerShell `curl` is an alias for `Invoke-WebRequest`).
- Each request can take up to 230 seconds. If your command tool has a timeout setting, set it to at least 240 seconds.
- The request needs network access to `api.claudeppc.ai`. If the sandbox blocks it, ask for permission to run the command with network access. If that isn't possible, tell the user to enable network access as described in the plugin README, then stop.

## Step 1: Check configuration

Check whether `config.json` exists in the current folder.

- If it does **not exist**, or contains placeholder values (`YOUR-TOKEN-HERE`, `1234567890`): tell the user to download their `config.json` from [claudeppc.ai](https://claudeppc.ai) and put it in this folder. **Stop here.**
- If it exists with real values, continue to Step 2.

## Step 2: Ask what to download

Ask the user both questions in one message, then **stop and wait for the answer**. Don't guess.

**Question 1: What data do you want to download?**
1. Structure + performance 30d (recommended): full account structure plus last 30 days compared with the previous 30 days
2. Structure only: account settings, campaigns, ad groups, ads, keywords, targeting, extensions
3. Performance 30d: last 30 days compared with the previous 30 days
4. Performance 7d: last 7 days compared with the previous 7 days
5. Performance monthly: this month, last month, and 2 months ago

**Question 2: Which entities should be included?**
1. Active only (recommended): only enabled campaigns, ad groups, ads, and keywords
2. Include paused: also include paused campaigns, ad groups, ads, and keywords

## Step 3: Run the download

From the answers:
- Set `{scope}` to one of `structure+30d`, `structure`, `30d`, `7d`, `monthly`.
- Set `{includePaused}` to `false` (active only) or `true` (include paused).

For `structure+30d`, run **two downloads in sequence**: first `structure`, then `30d`. Follow the download and extract steps below for each one. Show the plugin update notice (if any) only once.

### Download

PowerShell:
```powershell
New-Item -ItemType Directory -Force tmp | Out-Null; curl.exe -s --max-time 230 -o tmp/response.zip -D tmp/response_headers.txt -X POST "https://api.claudeppc.ai/api/cli/google-ads/get-data?includePaused={includePaused}&scope={scope}&pluginVersion=2.1.0&host=openai" -F "config=@config.json"
```

bash:
```bash
mkdir -p tmp && curl -s --max-time 230 -o tmp/response.zip -D tmp/response_headers.txt -X POST "https://api.claudeppc.ai/api/cli/google-ads/get-data?includePaused={includePaused}&scope={scope}&pluginVersion=2.1.0&host=openai" -F "config=@config.json"
```

Read `tmp/response_headers.txt`:
- Check the HTTP status on the last line that starts with `HTTP/`. If it is not `200`, the body in `tmp/response.zip` is a JSON error, not a zip. Show that error to the user, suggest checking `config.json`, delete the `tmp` folder, and **stop**.
- If there is a line starting with `X-Plugin-Update:`, show its value to the user as a notice.

Then delete `tmp/response_headers.txt`.

### Extract

**For `structure`**, wipe the existing structure data and the workspace guide, then extract:

PowerShell:
```powershell
Remove-Item -Recurse -Force -ErrorAction SilentlyContinue data/account, docs, AGENTS.md; Expand-Archive -Path tmp/response.zip -DestinationPath . -Force; Remove-Item tmp/response.zip
```

bash:
```bash
rm -rf data/account docs AGENTS.md && unzip -o tmp/response.zip && rm tmp/response.zip
```

**For `30d`, `7d`, or `monthly`**, wipe only that performance dataset, then extract:

PowerShell:
```powershell
Remove-Item -Recurse -Force -ErrorAction SilentlyContinue data/performance/{scope}; Expand-Archive -Path tmp/response.zip -DestinationPath . -Force; Remove-Item tmp/response.zip
```

bash:
```bash
rm -rf data/performance/{scope} && unzip -o tmp/response.zip && rm tmp/response.zip
```

## Step 4: Load the workspace guide

Every download includes `AGENTS.md`, the workspace guide that explains how the data is organized and which skills to use. Read it so you have the workspace context for the rest of the conversation.

## Step 5: Summarize the results

**For `structure+30d`:**
1. Read `data/account/account_summary.md` to load the account context.
2. Give the user a brief summary of what was downloaded (account name, number of campaigns, plus 30-day performance data).
3. Say whether paused entities were included.

**For `structure`:**
1. Read `data/account/account_summary.md` to load the account context.
2. Give the user a brief summary of what was downloaded (account name, number of campaigns).
3. Say whether paused entities were included.
4. If the `data/performance/` folder doesn't exist, ask the user whether they also want 30-day performance data. It enables performance-based analysis and auditing.

**For performance datasets (`30d`, `7d`, `monthly`):**
1. List the periods covered (shown in the file headers as `- Periods: ...`).
2. Mention the number of campaigns with performance data.
3. Remind the user that performance values use `→` to separate periods (newest first).
