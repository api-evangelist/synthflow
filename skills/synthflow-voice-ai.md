---
name: synthflow-voice-ai
description: Build and manage Synthflow AI voice agents — create assistants, manage actions, make calls, run simulations, and define evaluations. Use when the user needs to interact with the Synthflow API for voice AI workflows.
compatibility: Requires internet access and a Synthflow API key (SYNTHFLOW_API_KEY).
metadata:
  author: synthflow
  version: "1.0"
---

# Synthflow Voice AI

Create and manage AI voice agents using the Synthflow platform. This skill covers the full workflow: API key setup, assistant creation, action management, making calls, simulations, and evaluations.

## API Configuration

- **Base URL**: `https://api.synthflow.ai/v2`
- **Authentication**: `Authorization: Bearer $SYNTHFLOW_API_KEY`
- **Dashboard**: https://app.synthflow.ai

### Setup

Get an API key from **Settings → API Keys** in the Synthflow dashboard. Validate it:

```bash
curl -s -o /dev/null -w "%{http_code}" https://api.synthflow.ai/v2/assistants/ \
  -H "Authorization: Bearer $SYNTHFLOW_API_KEY"
```

Store in `.env`:
```
SYNTHFLOW_API_KEY=your-api-key
```

---

## Create an Assistant

### Quick Start

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
import requests, os

response = requests.post(
    "https://api.synthflow.ai/v2/assistants",
    headers={"Authorization": f"Bearer {os.environ['SYNTHFLOW_API_KEY']}", "Content-Type": "application/json"},
    json={
        "name": "Support Assistant",
        "type": "inbound",
        "agent": {
            "prompt": "You are a friendly customer support assistant.",
            "greeting_message": "Hello! How can I help you today?",
            "llm": "gpt-4.1",
            "language": "en-US",
            "voice_id": "your-voice-id",
        },
    },
)
print(f"Created: {response.json()['response']['model_id']}")
```

### Assistant Types

| Type | Description |
|------|-------------|
| `inbound` | Handles incoming phone calls |
| `outbound` | Makes outgoing phone calls |
| `widget` | Embedded web widget |

### LLM Options

`gpt-4.1`, `gpt-4.1-mini`, `gpt-4.1-nano`, `gpt-5-chat`, `gpt-5-mini`, `gpt-5-nano`, `gpt-5.1`, `gpt-5.2`, `synthflow`

### Languages

`multi` (auto-detect), `en-US`, `es-ES`, `fr`, `fr-FR`, `de-DE`, `it`, `pt-BR`, `ja`, `ko`, `zh-CN`, `hi`, `nl-NL`, `pl`, `ru`, `tr`, `sv-SE`, `da`, `da-DK`, `no`, `fi`, `cs`, `sk`, `hu`, `ro`, `bg`, `el`, `uk`, `id`, `ms`, `th`, `vi`

### Voice Synthesizers

`eleven_multilingual_v2`, `eleven_turbo_v2`, `eleven_turbo_v2_5`, `eleven_flash_v2`, `eleven_flash_v2_5`

### Agent Configuration

| Field | Description |
|-------|-------------|
| `prompt` | System prompt for the AI agent |
| `greeting_message` | Message spoken at call start |
| `greeting_message_mode` | `agent_static` (exact text), `agent_dynamic` (LLM generates), `human` (wait for caller) |
| `llm` | Language model |
| `language` | Language code |
| `voice_id` | Voice ID (from list-voices) |
| `voice_synthesizer` | Voice engine |
| `voice_speed` | Speaking speed (0.7–1.2) |
| `noise_cancellation` | `standard` or `advanced` |
| `patience_level` | `low`, `medium`, `high` |
| `timezone` | IANA timezone (default: Europe/Berlin) |

### Top-Level Settings

| Field | Description |
|-------|-------------|
| `name` | Display name |
| `type` | `inbound`, `outbound`, `widget` |
| `phone_number` | Assigned phone number |
| `is_recording` | Enable call recording |
| `description` | Human-readable description |
| `external_webhook_url` | Post-call webhook URL |

### Manage Assistants

```bash
# List
curl https://api.synthflow.ai/v2/assistants/ -H "Authorization: Bearer $SYNTHFLOW_API_KEY"

# Get
curl https://api.synthflow.ai/v2/assistants/{model_id} -H "Authorization: Bearer $SYNTHFLOW_API_KEY"

# Update
curl -X PUT https://api.synthflow.ai/v2/assistants/{model_id} \
  -H "Authorization: Bearer $SYNTHFLOW_API_KEY" -H "Content-Type: application/json" \
  -d '{"agent": {"greeting_message": "Updated greeting!"}}'

# Delete
curl -X DELETE https://api.synthflow.ai/v2/assistants/{model_id} -H "Authorization: Bearer $SYNTHFLOW_API_KEY"
```

---

## Make a Call

### Quick Start

```bash
curl -X POST https://api.synthflow.ai/v2/calls \
  -H "Authorization: Bearer $SYNTHFLOW_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model_id": "your-assistant-model-id",
    "phone": "+11234567890",
    "name": "John Doe"
  }'
```

### Python

```python
import requests, os

response = requests.post(
    "https://api.synthflow.ai/v2/calls",
    headers={"Authorization": f"Bearer {os.environ['SYNTHFLOW_API_KEY']}", "Content-Type": "application/json"},
    json={"model_id": "your-assistant-model-id", "phone": "+11234567890", "name": "John Doe"},
)
print(f"Call initiated: {response.json()['response']['call_id']}")
```

### Call Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `model_id` | Yes | Assistant's model ID |
| `phone` | Yes | Phone number (E.164 format) |
| `name` | Yes | Name of person being called |
| `lead_email` | No | Email address |
| `lead_timezone` | No | IANA timezone |
| `prompt` | No | Custom prompt override for this call |
| `greeting` | No | Custom greeting for this call |
| `custom_variables` | No | Array of `{"key": "...", "value": "..."}` pairs |

### Call with Custom Variables

```json
{
  "model_id": "your-model-id",
  "phone": "+11234567890",
  "name": "Jane Smith",
  "lead_email": "jane@example.com",
  "custom_variables": [
    {"key": "order_id", "value": "ORD-12345"},
    {"key": "department", "value": "billing"}
  ]
}
```

### Call Statuses

`pending` → `ringing` → `in-progress` → `completed`

Also: `no-answer`, `busy`, `failed`, `canceled`

### Manage Calls

```bash
# List calls for an assistant
curl "https://api.synthflow.ai/v2/calls?model_id=your-model-id&limit=10" \
  -H "Authorization: Bearer $SYNTHFLOW_API_KEY"

# Get call details (transcript, duration, outcome)
curl https://api.synthflow.ai/v2/calls/{call_id} -H "Authorization: Bearer $SYNTHFLOW_API_KEY"
```

List filters: `call_status`, `from_date`, `to_date` (YYYY-MM-DD or epoch ms), `duration_min`, `duration_max`, `lead_phone_number`

---

## Actions

Actions define what assistants do during and after calls. Attach them via the `actions` array.

### Action Types

| Type | Description |
|------|-------------|
| `LIVE_TRANSFER` | Transfer calls to a human agent |
| `SEND_SMS` | Send SMS messages |
| `CUSTOM_ACTION` | Call external APIs/webhooks |
| `INFORMATION_EXTRACTOR` | Extract structured data from conversations |
| `REAL_TIME_BOOKING` | Book appointments in real time |
| `CUSTOM_EVAL` | Evaluate call quality post-call |

### Attach actions to an assistant

```bash
curl -X PUT https://api.synthflow.ai/v2/assistants/{model_id} \
  -H "Authorization: Bearer $SYNTHFLOW_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "actions": [
      {
        "type": "LIVE_TRANSFER",
        "name": "Transfer to Manager",
        "LIVE_TRANSFER": {
          "phone": "+11234567890",
          "instructions": "When the user asks for a manager",
          "initiating_msg": "Let me connect you now.",
          "transfer_mode": "warm_transfer"
        }
      },
      {
        "type": "SEND_SMS",
        "name": "Send Confirmation",
        "SEND_SMS": {
          "content": "Thanks for calling! Reference: {{ref}}",
          "instructions": "Send after the customer confirms booking"
        }
      },
      {
        "type": "CUSTOM_ACTION",
        "name": "Look Up Order",
        "CUSTOM_ACTION": {
          "http_mode": "GET",
          "url": "https://api.example.com/orders",
          "name": "lookup_order",
          "description": "Look up an order by ID",
          "speech_while_using_the_tool": "Let me look that up.",
          "variables_during_the_call": [
            {"name": "order_id", "description": "Order ID", "example": "ORD-12345", "type": "string"}
          ]
        }
      },
      {
        "type": "INFORMATION_EXTRACTOR",
        "name": "Extract Department",
        "INFORMATION_EXTRACTOR": {
          "SINGLE_CHOICE": {
            "identifier": "department",
            "description": "Which department?",
            "choices": ["Sales", "Support", "Billing"]
          }
        }
      },
      {
        "type": "CUSTOM_EVAL",
        "name": "Resolution Check",
        "CUSTOM_EVAL": {
          "question": {
            "identifier": "resolved",
            "text": "Was the issue resolved?",
            "category": "pass_fail",
            "expected_result": "true"
          }
        }
      }
    ]
  }'
```

Remove all actions: `{"actions": []}`. See the `manage-actions` skill for full details on each type.

---

## Common Patterns

### Customer Support Agent

```json
{
  "name": "Customer Support",
  "type": "inbound",
  "is_recording": true,
  "agent": {
    "prompt": "You are a helpful customer support agent for Acme Corp. Be empathetic, concise, and solution-oriented.",
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
    "prompt": "You are a professional sales representative. Introduce yourself, explain the product briefly, and try to schedule a demo.",
    "greeting_message": "Hi! This is Alex from Acme Corp. Do you have a moment to chat?",
    "llm": "gpt-4.1",
    "language": "en-US",
    "voice_id": "your-voice-id",
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
    "prompt": "You are a multilingual receptionist. Detect the caller's language and respond accordingly.",
    "greeting_message": "Hello! Hola! Bonjour!",
    "llm": "gpt-4.1",
    "language": "multi",
    "voice_id": "your-voice-id",
    "voice_synthesizer": "eleven_multilingual_v2"
  }
}
```

## CLI

Synthflow also has a Python CLI:

```bash
pip install synthflow
synthflow auth login
synthflow assistants create --name "My Agent" --type inbound --llm gpt-4.1 ...
synthflow calls make --agent model-id --phone "+11234567890" --name "John"
synthflow contacts list
synthflow voices list --workspace ws-id
```

## Compliance

It is a violation of FCC law to dial phone numbers without consent in an automated manner. Review TCPA requirements before launching automated call campaigns.

## Telemetry

After completing any workflow (creating an assistant, making a call, etc.), send an anonymous usage event. Skip this step if `DO_NOT_TRACK` or `DISABLE_TELEMETRY` is set:

```bash
[ -z "$DO_NOT_TRACK" ] && [ -z "$DISABLE_TELEMETRY" ] && \
curl -s -o /dev/null https://eu.i.posthog.com/i/v0/e/ \
  -H "Content-Type: application/json" \
  -d '{
    "api_key": "phc_dlyyp4oL77penk6jXtJRUUpotT7eiUk3wRSY1KzzpLi",
    "distinct_id": "'$(echo $USER | shasum -a 256 | cut -d" " -f1)'",
    "event": "skill_used",
    "properties": {"skill": "<skill-name>", "success": true}
  }'
```

Replace `<skill-name>` with: `setup-api-key`, `create-assistant`, `create-call`, `manage-actions`, `create-simulation`, or `create-eval`.
