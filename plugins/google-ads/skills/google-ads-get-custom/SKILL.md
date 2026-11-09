---
name: google-ads-get-custom
description: Run a custom GAQL query against the Google Ads account when the standard export in data/ lacks the needed fields, date ranges, aggregation levels, or segments. Saves the results to data/custom/ as markdown.
---

# Fetch custom Google Ads data

Runs a custom GAQL query to fetch data that isn't in the standard export.

## When to use

Use this when the data in `data/account/` and `data/performance/` isn't enough and you need:
- Different date ranges
- Metrics not included in the standard export
- Data at different aggregation levels
- Specific segments or filters

If `config.json` doesn't exist in the current folder, tell the user to download it from [claudeppc.ai](https://claudeppc.ai) and stop.

## Shell and network notes

- Run every command from the folder that contains `config.json`.
- On Windows (PowerShell), use the PowerShell commands. On macOS or Linux, use the bash commands. On Windows always call `curl.exe`, never `curl`.
- The request can take up to 230 seconds. If your command tool has a timeout setting, set it to at least 240 seconds.
- The request needs network access to `api.claudeppc.ai`. If the sandbox blocks it, ask for permission to run the command with network access. If that isn't possible, tell the user to enable network access as described in the plugin README, then stop.

## Steps

1. Write a valid GAQL query for the data you need, and pick a short query name using only letters, digits, and underscores (for example `campaign_7d`, `keyword_device`). The name becomes the output file name.

2. Save the GAQL query as plain text in `tmp/query.txt` (create the `tmp` folder if needed). The query goes in a file so quotes in GAQL don't break the shell command.

3. Run the query, replacing `{query_name}`:

PowerShell:
```powershell
curl.exe -s --max-time 230 -o tmp/response.zip -D tmp/response_headers.txt -X POST "https://api.claudeppc.ai/api/cli/google-ads/get-custom-data?pluginVersion=2.1.0&host=openai" -F "config=@config.json" -F "name={query_name}" -F "query=<tmp/query.txt"
```

bash:
```bash
curl -s --max-time 230 -o tmp/response.zip -D tmp/response_headers.txt -X POST "https://api.claudeppc.ai/api/cli/google-ads/get-custom-data?pluginVersion=2.1.0&host=openai" -F "config=@config.json" -F "name={query_name}" -F "query=<tmp/query.txt"
```

4. Read `tmp/response_headers.txt`:
   - Check the HTTP status on the last line that starts with `HTTP/`. If it is not `200`, the body in `tmp/response.zip` is a JSON error (usually a GAQL error), not a zip. Read it, fix the query, and try again, or show the error to the user.
   - If there is a line starting with `X-Plugin-Update:`, show its value to the user as a notice.

   Then delete `tmp/response_headers.txt` and `tmp/query.txt`.

5. Extract and clean up. The zip contains `custom/<query_name>.md`, so extract it into `data`:

PowerShell:
```powershell
New-Item -ItemType Directory -Force data | Out-Null; Expand-Archive -Path tmp/response.zip -DestinationPath data -Force; Remove-Item tmp/response.zip
```

bash:
```bash
mkdir -p data && unzip -o tmp/response.zip -d data && rm tmp/response.zip
```

6. Read `data/custom/<query_name>.md` and present the data to the user.

## Example

Query name: `top_keywords`

GAQL (`tmp/query.txt`):
```sql
SELECT ad_group_criterion.keyword.text, metrics.conversions FROM keyword_view WHERE metrics.conversions > 0 ORDER BY metrics.conversions DESC LIMIT 50
```

## Notes

- Monetary values in GAQL results are in micros (divide by 1,000,000 for currency).
- Results are saved as markdown tables.
- Multiple custom queries can coexist in `data/custom/`.
