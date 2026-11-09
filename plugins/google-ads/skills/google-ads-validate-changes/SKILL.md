---
name: google-ads-validate-changes
description: Validate the pending changes in pending_changes.json against the Google Ads API without executing them (dry-run). Use only when the user explicitly asks to validate their pending changes.
---

# Validate pending Google Ads changes

Validates the changes in `pending_changes.json` against the Google Ads API without executing them (dry-run). Nothing changes in the account.

## Shell and network notes

- Run every command from the folder that contains `config.json` and `pending_changes.json`.
- On Windows (PowerShell), use the PowerShell command. On macOS or Linux, use the bash command. On Windows always call `curl.exe`, never `curl`.
- The request can take up to 230 seconds. If your command tool has a timeout setting, set it to at least 240 seconds.
- The request needs network access to `api.claudeppc.ai`. If the sandbox blocks it, ask for permission to run the command with network access. If that isn't possible, tell the user to enable network access as described in the plugin README, then stop.

## Steps

1. Check that `config.json` and `pending_changes.json` both exist. If either is missing, tell the user and stop. Read `pending_changes.json` and show a brief summary (number of operations by type).

2. Send both files to the API:

PowerShell:
```powershell
New-Item -ItemType Directory -Force tmp | Out-Null; curl.exe -s --max-time 230 -D tmp/response_headers.txt -X POST "https://api.claudeppc.ai/api/cli/google-ads/validate-changes?pluginVersion=2.1.0&host=openai" -F "config=@config.json" -F "pendingChanges=@pending_changes.json"
```

bash:
```bash
mkdir -p tmp && curl -s --max-time 230 -D tmp/response_headers.txt -X POST "https://api.claudeppc.ai/api/cli/google-ads/validate-changes?pluginVersion=2.1.0&host=openai" -F "config=@config.json" -F "pendingChanges=@pending_changes.json"
```

3. Read `tmp/response_headers.txt`. If there is a line starting with `X-Plugin-Update:`, show its value to the user as a notice. Then delete `tmp/response_headers.txt`.

4. After the API responds:
   - If validation succeeded: confirm the changes are ready, and tell the user they can apply them with `$google-ads-execute-changes` (in ChatGPT: `@google-ads-execute-changes`).
   - If validation failed: show the errors, explain what went wrong, and suggest fixes to `pending_changes.json`. The format is specified in `docs/PENDING_CHANGES.md`.
