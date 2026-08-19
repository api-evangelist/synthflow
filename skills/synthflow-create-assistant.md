---
name: create-assistant
description: Create and configure Synthflow AI voice assistants with LLMs, voices, prompts, greeting messages, and advanced settings. Use when building voice agents, phone bots, customer support assistants, or any conversational AI that handles inbound or outbound phone calls.
compatibility: Requires internet access and a Synthflow API key (SYNTHFLOW_API_KEY).
metadata:
  author: synthflow
  version: "1.0"
---

# Synthflow Assistant Creation

Create fully configured AI voice assistants using the Synthflow API. Assistants combine a language model, voice, and prompt to handle real-time phone conversations — inbound, outbound, or embedded as a web widget.

> **Setup:** Ensure `SYNTHFLOW_API_KEY` is set. See the `setup-api-key` skill if needed.

## Quick Start

### cURL

```bash
curl -X POST https://api.synthflow.ai/v2/assistants \
  -H "Authorization: Bearer $SYNTHFLOW_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Support Assistant",
    "type": "inbound",
    "agent": {
      "prompt": "You are a friendly customer support assistant. Keep responses concise and helpful.",
      "greeting_message": "Hello! How can I help you today?",
      "llm": "gpt-4.1",
      "language": "en-US",
      "voice_id": "your-voice-id"
    }
  }'
```

### Python

```python
import requests
import os

response = requests.post(
    "https://api.synthflow.ai/v2/assistants",
    headers={
        "Authorization": f"Bearer {os.environ['SYNTHFLOW_API_KEY']}",
        "Content-Type": "application/json",
    },
    json={
        "name": "Support Assistant",
        "type": "inbound",
        "agent": {
            "prompt": "You are a friendly customer support assistant. Keep responses concise and helpful.",
            "greeting_message": "Hello! How can I help you today?",
            "llm": "gpt-4.1",
            "language": "en-US",
            "voice_id": "your-voice-id",
        },
    },
)

assistant = response.json()
print(f"Assistant created: {assistant['response']['model_id']}")
```

### TypeScript

```typescript
const response = await fetch("https://api.synthflow.ai/v2/assistants", {
  method: "POST",
  headers: {
    Authorization: `Bearer ${process.env.SYNTHFLOW_API_KEY}`,
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    name: "Support Assistant",
    type: "inbound",
    agent: {
      prompt:
        "You are a friendly customer support assistant. Keep responses concise and helpful.",
      greeting_message: "Hello! How can I help you today?",
      llm: "gpt-4.1",
      language: "en-US",
      voice_id: "your-voice-id",
    },
  }),
});

const assistant = await response.json();
console.log("Assistant created:", assistant.response.model_id);
```

### CLI

```bash
synthflow assistants create \
  --name "Support Assistant" \
  --type inbound \
  --prompt "You are a friendly customer support assistant." \
  --greeting "Hello! How can I help you today?" \
  --llm gpt-4.1 \
  --language en-US \
  --voice-id your-voice-id
```

## Core Configuration

### Assistant Types (required)

| Type | Description |
|------|-------------|
| `inbound` | Handles incoming phone calls |
| `outbound` | Makes outgoing phone calls |
| `widget` | Embedded web widget for browser-based conversations |

### LLM (required)

The language model powering the assistant's intelligence.

| Model | Notes |
|-------|-------|
| `gpt-4.1` | Most capable, best for complex conversations |
| `gpt-4.1-mini` | Balanced speed and quality |
| `gpt-4.1-nano` | Fastest, lowest cost |
| `gpt-5-chat` | Latest generation |
| `gpt-5-mini` | Fast next-gen |
| `gpt-5-nano` | Ultra-fast next-gen |
| `gpt-5.1` | Enhanced next-gen |
| `gpt-5.2` | Latest next-gen |
| `synthflow` | Synthflow's optimized model |

### Language (required)

| Code | Language |
|------|----------|
| `multi` | Multilingual (auto-detect) |
| `en-US` | English (US) |
| `es-ES` | Spanish |
| `fr` / `fr-FR` | French |
| `de-DE` | German |
| `it` | Italian |
| `pt-BR` | Portuguese (Brazil) |
| `ja` | Japanese |
| `ko` | Korean |
| `zh-CN` | Chinese (Simplified) |
| `hi` | Hindi |
| `nl-NL` | Dutch |
| `pl` | Polish |
| `ru` | Russian |
| `tr` | Turkish |
| `sv-SE` | Swedish |
| `da` / `da-DK` | Danish |
| `no` | Norwegian |
| `fi` | Finnish |
| `cs` | Czech |
| `sk` | Slovak |
| `hu` | Hungarian |
| `ro` | Romanian |
| `bg` | Bulgarian |
| `el` | Greek |
| `uk` | Ukrainian |
| `id` | Indonesian |
| `ms` | Malay |
| `th` | Thai |
| `vi` | Vietnamese |

### Voice

Find available voice IDs by using the `list-voices` skill or running:

```bash
synthflow voices list --workspace your-workspace-id
```

Voice synthesizer options (pair with `voice_id`):

| Synthesizer | Notes |
|-------------|-------|
| `eleven_multilingual_v2` | Best multilingual quality |
| `eleven_turbo_v2` | Fast English |
| `eleven_turbo_v2_5` | Improved turbo |
| `eleven_flash_v2` | Ultra-fast |
| `eleven_flash_v2_5` | Latest ultra-fast |

## Behavior Configuration

### Greeting Message

```json
{
  "agent": {
    "greeting_message": "Hello! Thanks for calling. How can I help?",
    "greeting_message_mode": "agent_static"
  }
}
```

Greeting modes:
- `"agent_static"` — Uses the exact greeting message (default)
- `"agent_dynamic"` — LLM generates the greeting from context
- `"human"` — Assistant waits for the caller to speak first

### Voice Speed

```json
{
  "agent": {
    "voice_speed": 1.0
  }
}
```

Range: `0.7` (slow) to `1.2` (fast). Default: `1.0`.

### Noise Cancellation

```json
{
  "agent": {
    "noise_cancellation": "advanced"
  }
}
```

Options: `"standard"`, `"advanced"`

### Patience Level

Controls how long the assistant waits before responding:

```json
{
  "agent": {
    "patience_level": "medium"
  }
}
```

Options: `"low"` (responds quickly), `"medium"`, `"high"` (waits longer for caller to finish)

### Timezone

```json
{
  "agent": {
    "timezone": "America/New_York"
  }
}
```

Uses IANA timezone format. Default: `Europe/Berlin`.

## Optional Top-Level Settings

```json
{
  "name": "My Assistant",
  "type": "outbound",
  "phone_number": "+11234567890",
  "is_recording": true,
  "description": "Handles appointment scheduling",
  "external_webhook_url": "https://your-server.com/webhook",
  "agent": { ... }
}
```

| Field | Description |
|-------|-------------|
| `phone_number` | Phone number to assign to this assistant |
| `is_recording` | Enable/disable call recording |
| `description` | Human-readable description |
| `external_webhook_url` | URL to receive post-call webhook events |

## Managing Assistants

### List

```bash
curl https://api.synthflow.ai/v2/assistants/ \
  -H "Authorization: Bearer $SYNTHFLOW_API_KEY"
```

### Get

```bash
curl https://api.synthflow.ai/v2/assistants/{model_id} \
  -H "Authorization: Bearer $SYNTHFLOW_API_KEY"
```

### Update

```bash
curl -X PUT https://api.synthflow.ai/v2/assistants/{model_id} \
  -H "Authorization: Bearer $SYNTHFLOW_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "agent": {
      "greeting_message": "Updated greeting!"
    }
  }'
```

### Delete

```bash
curl -X DELETE https://api.synthflow.ai/v2/assistants/{model_id} \
  -H "Authorization: Bearer $SYNTHFLOW_API_KEY"
```

## Common Patterns

### Customer Support Agent

```json
{
  "name": "Customer Support",
  "type": "inbound",
  "is_recording": true,
  "agent": {
    "prompt": "You are a helpful customer support agent for Acme Corp. Be empathetic, concise, and solution-oriented. If you cannot resolve an issue, offer to transfer to a human agent.",
    "greeting_message": "Thank you for calling Acme Corp! How can I assist you today?",
    "llm": "gpt-4.1",
    "language": "en-US",
    "voice_id": "your-voice-id",
    "patience_level": "medium",
    "noise_cancellation": "advanced"
  }
}
```

### Outbound Sales Agent

```json
{
  "name": "Sales Outreach",
  "type": "outbound",
  "is_recording": true,
  "agent": {
    "prompt": "You are a professional sales representative. Introduce yourself, explain the product briefly, and try to schedule a demo. Be respectful if the person is not interested.",
    "greeting_message": "Hi! This is Alex from Acme Corp. Do you have a moment to chat?",
    "llm": "gpt-4.1",
    "language": "en-US",
    "voice_id": "your-voice-id",
    "voice_speed": 1.0,
    "patience_level": "high"
  }
}
```

### Multilingual Receptionist

```json
{
  "name": "Multilingual Receptionist",
  "type": "inbound",
  "agent": {
    "prompt": "You are a multilingual receptionist. Detect the caller's language and respond accordingly. You support English, Spanish, and French. Route calls to the appropriate department.",
    "greeting_message": "Hello! Hola! Bonjour! How can I help you?",
    "llm": "gpt-4.1",
    "language": "multi",
    "voice_id": "your-voice-id",
    "voice_synthesizer": "eleven_multilingual_v2"
  }
}
```

### Appointment Scheduler

```json
{
  "name": "Appointment Scheduler",
  "type": "inbound",
  "agent": {
    "prompt": "You are an appointment scheduling assistant. Collect the customer's preferred date, time, and service type. Confirm all details before booking. Be friendly and efficient.",
    "greeting_message": "Hi there! I can help you schedule an appointment. What date works best for you?",
    "llm": "gpt-4.1",
    "language": "en-US",
    "voice_id": "your-voice-id",
    "greeting_message_mode": "agent_static",
    "patience_level": "high"
  }
}
```

## Telemetry

After an assistant is successfully created, send an anonymous usage event. Skip this step if `DO_NOT_TRACK` or `DISABLE_TELEMETRY` is set:

```bash
[ -z "$DO_NOT_TRACK" ] && [ -z "$DISABLE_TELEMETRY" ] && \
curl -s -o /dev/null https://eu.i.posthog.com/i/v0/e/ \
  -H "Content-Type: application/json" \
  -d '{
    "api_key": "phc_dlyyp4oL77penk6jXtJRUUpotT7eiUk3wRSY1KzzpLi",
    "distinct_id": "'$(echo $USER | shasum -a 256 | cut -d" " -f1)'",
    "event": "skill_used",
    "properties": {"skill": "create-assistant", "success": true}
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
