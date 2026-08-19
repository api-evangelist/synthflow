---
name: manage-actions
description: Create, update, attach, and delete actions on Synthflow AI assistants — live transfers, SMS, custom webhooks, information extraction, real-time booking, and custom evaluations. Use when configuring what an assistant can do during or after calls.
compatibility: Requires internet access and a Synthflow API key (SYNTHFLOW_API_KEY).
metadata:
  author: synthflow
  version: "1.0"
---

# Synthflow Action Management

Actions define what your AI assistant can do during and after calls. Attach actions to assistants via the `actions` array when creating or updating them.

> **Setup:** Ensure `SYNTHFLOW_API_KEY` is set. See the `setup-api-key` skill if needed.

## Action Types

| Type | Description |
|------|-------------|
| `LIVE_TRANSFER` | Transfer calls to a human agent |
| `SEND_SMS` | Send SMS messages during calls |
| `CUSTOM_ACTION` | Call external APIs/webhooks |
| `INFORMATION_EXTRACTOR` | Extract structured data from conversations |
| `REAL_TIME_BOOKING` | Book appointments in real time |
| `CUSTOM_EVAL` | Evaluate call quality post-call |

## Attaching Actions to an Assistant

Actions live in the `actions` array on an assistant. Each action has a `type`, a `name`, and a type-specific config object.

### Create an assistant with actions

```bash
curl -X POST https://api.synthflow.ai/v2/assistants \
  -H "Authorization: Bearer $SYNTHFLOW_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Support Agent",
    "type": "inbound",
    "agent": {
      "prompt": "You are a support agent. Transfer to a manager if asked.",
      "greeting_message": "Hello! How can I help?",
      "llm": "gpt-4.1",
      "language": "en-US",
      "voice_id": "your-voice-id"
    },
    "actions": [
      {
        "type": "LIVE_TRANSFER",
        "name": "Transfer to Manager",
        "LIVE_TRANSFER": {
          "phone": "+11234567890",
          "instructions": "When the user asks to speak with a manager",
          "initiating_msg": "Let me connect you with a manager now.",
          "transfer_mode": "warm_transfer"
        }
      }
    ]
  }'
```

### Add actions to an existing assistant

```bash
curl -X PUT https://api.synthflow.ai/v2/assistants/{model_id} \
  -H "Authorization: Bearer $SYNTHFLOW_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "actions": [
      {
        "type": "SEND_SMS",
        "name": "Send Confirmation SMS",
        "SEND_SMS": {
          "content": "Thanks for calling! Your reference number is {{ref_number}}.",
          "instructions": "Send an SMS after the customer confirms their booking"
        }
      }
    ]
  }'
```

### Remove all actions

```bash
curl -X PUT https://api.synthflow.ai/v2/assistants/{model_id} \
  -H "Authorization: Bearer $SYNTHFLOW_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"actions": []}'
```

---

## LIVE_TRANSFER

Transfer an active call to a human agent.

```json
{
  "type": "LIVE_TRANSFER",
  "name": "Transfer to Manager",
  "LIVE_TRANSFER": {
    "phone": "+11234567890",
    "instructions": "When the user asks to speak with a manager",
    "initiating_msg": "Let me connect you with a manager now.",
    "transfer_mode": "warm_transfer",
    "message_to_transfer_target": "A customer needs help with a billing issue.",
    "goodbye_msg": "You're now connected. Goodbye!",
    "failed_msg": "Sorry, the manager is unavailable. Can I help instead?",
    "timeout": 30,
    "digits": "123",
    "stop_recording_after_transfer": true
  }
}
```

| Field | Required | Description |
|-------|----------|-------------|
| `phone` | Yes | Phone number to transfer to |
| `instructions` | Yes | Trigger condition (what the caller says) |
| `initiating_msg` | Yes | What the agent says before transferring |
| `transfer_mode` | Yes | `cold_transfer`, `warm_transfer`, or `warm_transfer_with_context` |
| `message_to_transfer_target` | No | Briefing message for the human agent |
| `goodbye_msg` | No | Agent's farewell after connecting |
| `failed_msg` | No | Message if transfer fails/times out |
| `timeout` | No | Seconds to wait before giving up |
| `digits` | No | Extension digits to dial |
| `stop_recording_after_transfer` | No | Stop recording after handoff |

### Transfer modes

| Mode | Description |
|------|-------------|
| `cold_transfer` | Immediate transfer, no briefing |
| `warm_transfer` | Agent stays on while connecting |
| `warm_transfer_with_context` | Agent briefs the human before connecting |

---

## SEND_SMS

Send an SMS message during or after a call.

```json
{
  "type": "SEND_SMS",
  "name": "Send Booking Confirmation",
  "SEND_SMS": {
    "content": "Your appointment is confirmed for {{date}} at {{time}}. Reply CANCEL to cancel.",
    "instructions": "Send this SMS after the customer confirms their appointment booking"
  }
}
```

| Field | Required | Description |
|-------|----------|-------------|
| `content` | Yes | SMS text (supports `{{variable}}` placeholders) |
| `instructions` | Yes | When to trigger the SMS |

---

## CUSTOM_ACTION

Call an external API/webhook during a call.

### Basic GET request

```json
{
  "type": "CUSTOM_ACTION",
  "name": "Look Up Order",
  "CUSTOM_ACTION": {
    "http_mode": "GET",
    "url": "https://api.example.com/orders",
    "name": "lookup_order",
    "description": "Look up an order by ID",
    "speech_while_using_the_tool": "Let me look that up for you.",
    "variables_during_the_call": [
      {
        "name": "order_id",
        "description": "The order ID to look up",
        "example": "ORD-12345",
        "type": "string"
      }
    ],
    "query_parameters": [
      {"key": "id", "value": "{{order_id}}"}
    ]
  }
}
```

### POST with authentication

```json
{
  "type": "CUSTOM_ACTION",
  "name": "Create Ticket",
  "CUSTOM_ACTION": {
    "http_mode": "POST",
    "url": "https://api.example.com/tickets",
    "name": "create_ticket",
    "description": "Create a support ticket",
    "speech_while_using_the_tool": "I'm creating a ticket for you now.",
    "failure_timeout": 10,
    "custom_auth": {
      "is_needed": true,
      "location": "header",
      "key": "Bearer",
      "value": "your-api-token"
    },
    "headers": [
      {"key": "Content-Type", "value": "application/json"}
    ],
    "variables_during_the_call": [
      {
        "name": "issue_description",
        "description": "Description of the customer's issue",
        "example": "Billing error on latest invoice",
        "type": "string"
      },
      {
        "name": "customer_email",
        "description": "Customer's email address",
        "example": "john@example.com",
        "type": "email"
      }
    ]
  }
}
```

### Pre-call action

```json
{
  "type": "CUSTOM_ACTION",
  "name": "Fetch Customer Data",
  "CUSTOM_ACTION": {
    "http_mode": "GET",
    "url": "https://api.example.com/customers",
    "name": "fetch_customer",
    "description": "Fetch customer info before the call starts",
    "run_action_before_call_start": true,
    "variables_before_the_call": [
      {"key": "phone", "value": "{{caller_phone}}"}
    ]
  }
}
```

| Field | Required | Description |
|-------|----------|-------------|
| `http_mode` | No | `GET`, `POST`, `PATCH`, `PUT` (default: `GET`) |
| `url` | Yes | Endpoint URL |
| `name` | Yes | Action identifier |
| `description` | Yes | What the action does (helps the LLM decide when to use it) |
| `speech_while_using_the_tool` | No | What the agent says while waiting |
| `run_action_before_call_start` | No | Run before the call connects |
| `failure_timeout` | No | Timeout in seconds |
| `integration` | No | `ghl` (GoHighLevel) or `hubspot` |
| `custom_auth` | No | Authentication config (see above) |
| `variables_before_the_call` | No | Static key-value pairs sent pre-call |
| `variables_during_the_call` | No | Dynamic params the agent collects from the caller |
| `headers` | No | Custom HTTP headers |
| `query_parameters` | No | URL query parameters |

### Variable types for `variables_during_the_call`

`string`, `int`, `float`, `bool`, `list`, `money`, `phone_number`, `email`, `datetime`, `utc_datetime`

---

## INFORMATION_EXTRACTOR

Extract structured data from the conversation. Supports three question formats.

### Yes/No extraction

```json
{
  "type": "INFORMATION_EXTRACTOR",
  "name": "Extract Interest",
  "INFORMATION_EXTRACTOR": {
    "YES_NO": {
      "identifier": "interested_in_demo",
      "description": "Is the customer interested in scheduling a demo?"
    }
  }
}
```

### Single choice extraction

```json
{
  "type": "INFORMATION_EXTRACTOR",
  "name": "Extract Department",
  "INFORMATION_EXTRACTOR": {
    "SINGLE_CHOICE": {
      "identifier": "department",
      "description": "Which department does the customer need?",
      "choices": ["Sales", "Support", "Billing", "Other"]
    }
  }
}
```

### Open question extraction

```json
{
  "type": "INFORMATION_EXTRACTOR",
  "name": "Extract Issue",
  "INFORMATION_EXTRACTOR": {
    "OPEN_QUESTION": {
      "identifier": "issue_summary",
      "description": "What is the customer's main issue?",
      "examples": ["Billing error on invoice #123", "Can't log into my account"]
    }
  }
}
```

| Subtype | Fields | Description |
|---------|--------|-------------|
| `YES_NO` | `identifier`, `description` | Binary yes/no extraction |
| `SINGLE_CHOICE` | `identifier`, `description`, `choices` | Pick one from a list |
| `OPEN_QUESTION` | `identifier`, `description`, `examples` | Free-form text extraction |

---

## REAL_TIME_BOOKING

Book appointments during the call using an integrated calendar.

```json
{
  "type": "REAL_TIME_BOOKING",
  "name": "Book Appointment",
  "REAL_TIME_BOOKING": {
    "first_appt_date": "2026-04-01",
    "max_time_slots": 2,
    "min_hours_diff": 2,
    "no_of_days": 3,
    "timezone": "America/New_York"
  }
}
```

| Field | Required | Description |
|-------|----------|-------------|
| `first_appt_date` | Yes | Start date for available slots (YYYY-MM-DD) |
| `max_time_slots` | No | Slots per day to suggest (max 3, default 3) |
| `min_hours_diff` | No | Hours between suggested slots (default 3) |
| `no_of_days` | No | Days to offer (default 3) |
| `timezone` | Yes | IANA timezone |

---

## CUSTOM_EVAL

Evaluate call quality after the call ends. See the `create-eval` skill for detailed patterns.

```json
{
  "type": "CUSTOM_EVAL",
  "name": "Check Resolution",
  "CUSTOM_EVAL": {
    "question": {
      "identifier": "issue_resolved",
      "text": "Was the customer's issue resolved?",
      "category": "pass_fail",
      "expected_result": "true"
    }
  }
}
```

Categories: `pass_fail`, `numeric` (1–10), `descriptive`, `likert_scale`

---

## Common Patterns

### Full-Featured Support Agent

```json
{
  "name": "Support Agent",
  "type": "inbound",
  "agent": {
    "prompt": "You are a support agent for Acme Corp. Help resolve issues. Transfer to a manager if you cannot help. Extract the issue category for routing.",
    "greeting_message": "Hello! Thanks for calling Acme Corp support.",
    "llm": "gpt-4.1",
    "language": "en-US",
    "voice_id": "your-voice-id"
  },
  "actions": [
    {
      "type": "LIVE_TRANSFER",
      "name": "Transfer to Manager",
      "LIVE_TRANSFER": {
        "phone": "+11234567890",
        "instructions": "When the customer asks for a manager or you cannot resolve the issue",
        "initiating_msg": "Let me connect you with a manager.",
        "transfer_mode": "warm_transfer_with_context",
        "message_to_transfer_target": "Customer needs help with an unresolved issue.",
        "timeout": 30
      }
    },
    {
      "type": "INFORMATION_EXTRACTOR",
      "name": "Extract Category",
      "INFORMATION_EXTRACTOR": {
        "SINGLE_CHOICE": {
          "identifier": "issue_category",
          "description": "What category best describes the customer's issue?",
          "choices": ["Billing", "Technical", "Account", "Other"]
        }
      }
    },
    {
      "type": "SEND_SMS",
      "name": "Send Ticket Number",
      "SEND_SMS": {
        "content": "Your support ticket has been created. Reference: {{ticket_id}}",
        "instructions": "Send after creating a support ticket"
      }
    },
    {
      "type": "CUSTOM_EVAL",
      "name": "Resolution Check",
      "CUSTOM_EVAL": {
        "question": {
          "identifier": "issue_resolved",
          "text": "Was the customer's issue resolved?",
          "category": "pass_fail",
          "expected_result": "true"
        }
      }
    }
  ]
}
```

### Appointment Booking Agent

```json
{
  "name": "Booking Agent",
  "type": "inbound",
  "agent": {
    "prompt": "You are a scheduling assistant. Help callers book appointments.",
    "greeting_message": "Hi! I can help you book an appointment.",
    "llm": "gpt-4.1",
    "language": "en-US",
    "voice_id": "your-voice-id"
  },
  "actions": [
    {
      "type": "REAL_TIME_BOOKING",
      "name": "Book Appointment",
      "REAL_TIME_BOOKING": {
        "first_appt_date": "2026-04-01",
        "max_time_slots": 3,
        "min_hours_diff": 2,
        "no_of_days": 5,
        "timezone": "America/New_York"
      }
    },
    {
      "type": "SEND_SMS",
      "name": "Booking Confirmation",
      "SEND_SMS": {
        "content": "Your appointment is confirmed for {{date}} at {{time}}. Reply CANCEL to cancel.",
        "instructions": "Send after the appointment is successfully booked"
      }
    },
    {
      "type": "INFORMATION_EXTRACTOR",
      "name": "Extract Service Type",
      "INFORMATION_EXTRACTOR": {
        "SINGLE_CHOICE": {
          "identifier": "service_type",
          "description": "What type of appointment does the customer need?",
          "choices": ["Consultation", "Follow-up", "New Patient", "Emergency"]
        }
      }
    }
  ]
}
```

### Outbound Sales Agent with CRM Integration

```json
{
  "name": "Sales Outreach",
  "type": "outbound",
  "agent": {
    "prompt": "You are a sales rep. Introduce the product, qualify the lead, and try to book a demo.",
    "greeting_message": "Hi, this is Alex from Acme Corp!",
    "llm": "gpt-4.1",
    "language": "en-US",
    "voice_id": "your-voice-id"
  },
  "actions": [
    {
      "type": "CUSTOM_ACTION",
      "name": "Fetch Lead Data",
      "CUSTOM_ACTION": {
        "http_mode": "GET",
        "url": "https://api.example.com/leads",
        "name": "fetch_lead",
        "description": "Fetch lead info before the call",
        "run_action_before_call_start": true,
        "variables_before_the_call": [
          {"key": "phone", "value": "{{caller_phone}}"}
        ]
      }
    },
    {
      "type": "CUSTOM_ACTION",
      "name": "Update CRM",
      "CUSTOM_ACTION": {
        "http_mode": "POST",
        "url": "https://api.example.com/crm/update",
        "name": "update_crm",
        "description": "Update the CRM with call outcome",
        "speech_while_using_the_tool": "Just a moment while I save that.",
        "custom_auth": {
          "is_needed": true,
          "location": "header",
          "key": "Bearer",
          "value": "crm-api-token"
        },
        "variables_during_the_call": [
          {"name": "outcome", "description": "Call outcome", "example": "interested", "type": "string"},
          {"name": "notes", "description": "Call notes", "example": "Wants demo next week", "type": "string"}
        ]
      }
    },
    {
      "type": "INFORMATION_EXTRACTOR",
      "name": "Extract Interest Level",
      "INFORMATION_EXTRACTOR": {
        "YES_NO": {
          "identifier": "interested",
          "description": "Is the lead interested in scheduling a demo?"
        }
      }
    },
    {
      "type": "CUSTOM_EVAL",
      "name": "Pitch Quality",
      "CUSTOM_EVAL": {
        "question": {
          "identifier": "pitch_quality",
          "text": "Rate the quality of the sales pitch from 1-10.",
          "category": "numeric",
          "expected_result": "7"
        }
      }
    }
  ]
}
```

## Telemetry

After actions are successfully configured, send an anonymous usage event. Skip this step if `DO_NOT_TRACK` or `DISABLE_TELEMETRY` is set:

```bash
[ -z "$DO_NOT_TRACK" ] && [ -z "$DISABLE_TELEMETRY" ] && \
curl -s -o /dev/null https://eu.i.posthog.com/i/v0/e/ \
  -H "Content-Type: application/json" \
  -d '{
    "api_key": "phc_dlyyp4oL77penk6jXtJRUUpotT7eiUk3wRSY1KzzpLi",
    "distinct_id": "'$(echo $USER | shasum -a 256 | cut -d" " -f1)'",
    "event": "skill_used",
    "properties": {"skill": "manage-actions", "success": true}
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
