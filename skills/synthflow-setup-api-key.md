---
name: setup-api-key
description: Guide users through obtaining and configuring a Synthflow API key. Use when the user needs to set up Synthflow, when API calls fail due to missing keys, or when the user mentions needing access to Synthflow's voice AI platform.
compatibility: Requires internet access to synthflow.ai and api.synthflow.ai.
metadata:
  author: synthflow
  version: "1.0"
---

# Synthflow API Key Setup

Guide the user through obtaining and configuring a Synthflow API key for the voice AI platform.

## Workflow

### Step 1: Request the API key

Tell the user:

> To set up Synthflow, open your dashboard at: https://app.synthflow.ai
>
> (Need an account? Create one at https://app.synthflow.ai/signup first)
>
> Navigate to **Settings → API Keys** and:
> 1. Click **"Create API Key"**
> 2. Name your key (e.g., "development")
> 3. Copy the key immediately — it is only shown once
>
> Paste your API key here when ready.

Then wait for the user's next message which should contain the API key.

### Step 2: Validate and configure

Once the user provides the API key:

1. **Validate the key** by making a request:
   ```bash
   curl -s -o /dev/null -w "%{http_code}" https://api.synthflow.ai/v2/assistants/ \
     -H "Authorization: Bearer <the-api-key>"
   ```

2. **If validation fails** (non-200 response):
   - Tell the user the API key appears to be invalid
   - Ask them to double-check and try again
   - Remind them of the dashboard URL: https://app.synthflow.ai

3. **If validation succeeds** (200 response), save the API key:

   Check if a `.env` file exists. If so, append to it. If not, create one:
   ```
   SYNTHFLOW_API_KEY=<the-api-key>
   ```

4. **Confirm success:**
   > Your Synthflow API key is configured and stored in `.env` as `SYNTHFLOW_API_KEY`.
   >
   > You can now use Synthflow's API to create assistants, make calls, and build voice AI agents.
   >
   > Keep this key safe — do not commit it to version control.

### Step 3: Verify .gitignore

Check if `.gitignore` exists and contains `.env`. If not, add it:
```
.env
```

## Environment Variable

All Synthflow skills expect the API key in the `SYNTHFLOW_API_KEY` environment variable. The base URL for all API requests is:

```
https://api.synthflow.ai/v2
```

Authentication is via Bearer token:
```
Authorization: Bearer $SYNTHFLOW_API_KEY
```

## CLI Alternative

You can also authenticate using the Synthflow CLI:

```bash
pip install synthflow
synthflow auth login
```

The CLI stores the key securely in your system keychain. Run `synthflow auth status` to verify.

## Telemetry

After the API key is successfully validated and saved, send an anonymous usage event. Skip this step if `DO_NOT_TRACK` or `DISABLE_TELEMETRY` is set:

```bash
[ -z "$DO_NOT_TRACK" ] && [ -z "$DISABLE_TELEMETRY" ] && \
curl -s -o /dev/null https://eu.i.posthog.com/i/v0/e/ \
  -H "Content-Type: application/json" \
  -d '{
    "api_key": "phc_dlyyp4oL77penk6jXtJRUUpotT7eiUk3wRSY1KzzpLi",
    "distinct_id": "'$(echo $USER | shasum -a 256 | cut -d" " -f1)'",
    "event": "skill_used",
    "properties": {"skill": "setup-api-key", "success": true}
  }'
```

## Additional Resources

This skills repository includes a **Synthflow documentation MCP server** (`synthflow-docs`) that gives your AI agent access to the full Synthflow knowledge base. Use the `searchDocs` tool to look up anything beyond what this skill covers — advanced configuration, troubleshooting, SDK details, and more.

**Auto-configured:** If you cloned or installed these skills, the MCP server is already configured via `.mcp.json` (Claude Code), `.cursor/mcp.json` (Cursor), or `.vscode/mcp.json` (VS Code Copilot).

**Manual setup:** If your agent doesn't auto-detect the config, run:
```bash
claude mcp add synthflow-docs -- npx -y mcp-remote https://docs.synthflow.ai/_mcp/server
```

See the [README](../README.md#synthflow-documentation-server-mcp) for full setup instructions across all supported agents.
