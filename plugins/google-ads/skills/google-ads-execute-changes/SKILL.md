---
name: google-ads-execute-changes
description: Apply the pending changes in pending_changes.json to the live Google Ads account. Use only when the user explicitly asks to execute their pending changes, and only after they confirm.
---

# Execute pending Google Ads changes

Applies the changes in `pending_changes.json` to the **live** Google Ads account.

## User confirmation is required

This makes real changes in a live account. **Never run the execute command without an explicit confirmation from the user in this conversation.**

- After showing the summary (step 1), ask: "This will apply all pending changes to your live Google Ads account. Do you want to execute these changes? (yes / no)"
- Then **stop and wait for the user's reply.** Don't run the command in the same turn as the question.
- Continue only if the user clearly answers yes. For anything else, abort and don't run the command.
- A confirmation given for an earlier `pending_changes.json`, or earlier in the conversation before the file changed, doesn't count. Ask again.

## Shell and network notes

- Run every command from the folder that contains `config.json` and `pending_changes.json`.
- On Windows (PowerShell), use the PowerShell command. On macOS or Linux, use the bash command. On Windows always call `curl.exe`, never `curl`.
- The request can take up to 230 seconds. If your command tool has a timeout setting, set it to at least 240 seconds.
- The request needs network access to `api.claudeppc.ai`. If the sandbox blocks it, ask for permission to run the command with network access. If that isn't possible, tell the user to enable network access as described in the plugin README, then stop.

## Steps

1. Check that `config.json` and `pending_changes.json` both exist. If either is missing, tell the user and stop. Read `pending_changes.json` and show the user a summary:
   - Total number of operations
   - Breakdown by operation type
   - Estimated budget impact (if applicable)

2. **Ask for confirmation** as described above, and wait. If the user doesn't confirm, abort.

3. If the user confirms, send both files to the API:

PowerShell:
```powershell
New-Item -ItemType Directory -Force tmp | Out-Null; curl.exe -s --max-time 230 -D tmp/response_headers.txt -X POST "https://api.claudeppc.ai/api/cli/google-ads/execute-changes?pluginVersion=2.1.0&host=openai" -F "config=@config.json" -F "pendingChanges=@pending_changes.json"
```

bash:
```bash
mkdir -p tmp && curl -s --max-time 230 -D tmp/response_headers.txt -X POST "https://api.claudeppc.ai/api/cli/google-ads/execute-changes?pluginVersion=2.1.0&host=openai" -F "config=@config.json" -F "pendingChanges=@pending_changes.json"
```

   Run this command **once**. If it times out or the connection drops, don't retry automatically: the changes may already be applied. Tell the user and recommend running `$google-ads-get` (in ChatGPT: `@google-ads-get`) to check the account state first.

4. Read `tmp/response_headers.txt`. If there is a line starting with `X-Plugin-Update:`, show its value to the user as a notice. Then delete `tmp/response_headers.txt`.

5. If the API call succeeded, archive the results:
   - Create the `change-history/` folder if it doesn't exist.
   - Save `pending_changes.json` with added `status`, `executed_at`, and `execution_results` fields to `change-history/changes-{YYYYMMDD-HHmm}.json`.
   - Delete `pending_changes.json`.

6. After the API responds:
   - If everything succeeded: summarize what was applied, give the change history file path, and recommend running `$google-ads-get` (in ChatGPT: `@google-ads-get`) to refresh the local data.
   - If it partially failed: explain which operations succeeded and which failed, and suggest next steps.
   - If it completely failed: show the errors and suggest fixes.
