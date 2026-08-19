---
name: create-eval
description: Create custom evaluations for Synthflow AI agents to measure call quality, track business outcomes, and score agent performance. Use when defining success metrics, building scorecards, or configuring post-call analysis.
compatibility: Requires internet access and a Synthflow API key (SYNTHFLOW_API_KEY).
metadata:
  author: synthflow
  version: "1.0"
---

# Synthflow Custom Evaluations

Define custom evaluations to measure your AI agent's performance after every call. Evaluations run automatically post-call and score the conversation against your criteria.

Custom evaluations are configured as **actions** on an assistant. They are part of the assistant's `actions` array and evaluate each completed call.

> **Setup:** Ensure `SYNTHFLOW_API_KEY` is set. See the `setup-api-key` skill if needed.

## Quick Start — Add a Custom Eval to an Assistant

Custom evaluations are added when creating or updating an assistant via the `actions` array.

### cURL

```bash
curl -X PUT https://api.synthflow.ai/v2/assistants/{model_id} \
  -H "Authorization: Bearer $SYNTHFLOW_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "actions": [
      {
        "type": "CUSTOM_EVAL",
        "name": "Appointment Booking Check",
        "CUSTOM_EVAL": {
          "question": {
            "identifier": "appointment_booked",
            "text": "Did the agent successfully book an appointment for the caller?",
            "category": "pass_fail",
            "expected_result": "true"
          }
        }
      }
    ]
  }'
```

### Python

```python
import requests
import os

headers = {
    "Authorization": f"Bearer {os.environ['SYNTHFLOW_API_KEY']}",
    "Content-Type": "application/json",
}

# Add a custom eval to an existing assistant
response = requests.put(
    f"https://api.synthflow.ai/v2/assistants/{model_id}",
    headers=headers,
    json={
        "actions": [
            {
                "type": "CUSTOM_EVAL",
                "name": "Appointment Booking Check",
                "CUSTOM_EVAL": {
                    "question": {
                        "identifier": "appointment_booked",
                        "text": "Did the agent successfully book an appointment?",
                        "category": "pass_fail",
                        "expected_result": "true",
                    }
                },
            }
        ]
    },
)
print(f"Updated: {response.json()}")
```

### TypeScript

```typescript
const response = await fetch(
  `https://api.synthflow.ai/v2/assistants/${modelId}`,
  {
    method: "PUT",
    headers: {
      Authorization: `Bearer ${process.env.SYNTHFLOW_API_KEY}`,
      "Content-Type": "application/json",
    },
    body: JSON.stringify({
      actions: [
        {
          type: "CUSTOM_EVAL",
          name: "Appointment Booking Check",
          CUSTOM_EVAL: {
            question: {
              identifier: "appointment_booked",
              text: "Did the agent successfully book an appointment?",
              category: "pass_fail",
              expected_result: "true",
            },
          },
        },
      ],
    }),
  }
);
```

## Evaluation Categories

| Category | Description | `expected_result` format |
|----------|-------------|--------------------------|
| `pass_fail` | Binary pass/fail | `"true"` or `"false"` |
| `numeric` | Score from 1–10 | `"1"` through `"10"` |
| `descriptive` | Qualitative assessment | Free text (e.g., `"Excellent"`) |
| `likert_scale` | Agreement-based scale | Scale value |

## Evaluation Fields

Each custom eval action has this structure:

```json
{
  "type": "CUSTOM_EVAL",
  "name": "Human-readable action name",
  "CUSTOM_EVAL": {
    "question": {
      "identifier": "unique_eval_id",
      "text": "The evaluation prompt — what to check",
      "category": "pass_fail",
      "expected_result": "true"
    }
  }
}
```

| Field | Description |
|-------|-------------|
| `identifier` | Unique name for this eval (used in results) |
| `text` | The evaluation question/prompt — analyzed against the call transcript |
| `category` | Scoring type: `pass_fail`, `numeric`, `descriptive`, `likert_scale` |
| `expected_result` | The desired outcome |

## Multiple Evaluations

Add several evaluations to score different aspects of each call:

```json
{
  "actions": [
    {
      "type": "CUSTOM_EVAL",
      "name": "Booking Success",
      "CUSTOM_EVAL": {
        "question": {
          "identifier": "appointment_booked",
          "text": "Did the agent successfully book an appointment for the caller?",
          "category": "pass_fail",
          "expected_result": "true"
        }
      }
    },
    {
      "type": "CUSTOM_EVAL",
      "name": "Customer Satisfaction",
      "CUSTOM_EVAL": {
        "question": {
          "identifier": "customer_satisfaction",
          "text": "Rate the customer's apparent satisfaction with the call from 1-10.",
          "category": "numeric",
          "expected_result": "8"
        }
      }
    },
    {
      "type": "CUSTOM_EVAL",
      "name": "Tone Assessment",
      "CUSTOM_EVAL": {
        "question": {
          "identifier": "agent_tone",
          "text": "Describe the agent's tone throughout the conversation.",
          "category": "descriptive",
          "expected_result": "Professional and empathetic"
        }
      }
    }
  ]
}
```

## Common Patterns

### Sales Call Scorecard

```json
{
  "actions": [
    {
      "type": "CUSTOM_EVAL",
      "name": "Intro Quality",
      "CUSTOM_EVAL": {
        "question": {
          "identifier": "proper_introduction",
          "text": "Did the agent introduce themselves and the company clearly?",
          "category": "pass_fail",
          "expected_result": "true"
        }
      }
    },
    {
      "type": "CUSTOM_EVAL",
      "name": "Pain Points Identified",
      "CUSTOM_EVAL": {
        "question": {
          "identifier": "pain_points",
          "text": "Did the agent identify at least one customer pain point?",
          "category": "pass_fail",
          "expected_result": "true"
        }
      }
    },
    {
      "type": "CUSTOM_EVAL",
      "name": "Demo Scheduled",
      "CUSTOM_EVAL": {
        "question": {
          "identifier": "demo_scheduled",
          "text": "Did the agent successfully schedule a demo or follow-up?",
          "category": "pass_fail",
          "expected_result": "true"
        }
      }
    },
    {
      "type": "CUSTOM_EVAL",
      "name": "Overall Pitch Quality",
      "CUSTOM_EVAL": {
        "question": {
          "identifier": "pitch_quality",
          "text": "Rate the overall quality of the sales pitch from 1-10.",
          "category": "numeric",
          "expected_result": "7"
        }
      }
    }
  ]
}
```

### Support Quality Metrics

```json
{
  "actions": [
    {
      "type": "CUSTOM_EVAL",
      "name": "Issue Resolved",
      "CUSTOM_EVAL": {
        "question": {
          "identifier": "issue_resolved",
          "text": "Was the customer's issue fully resolved during the call?",
          "category": "pass_fail",
          "expected_result": "true"
        }
      }
    },
    {
      "type": "CUSTOM_EVAL",
      "name": "Empathy Score",
      "CUSTOM_EVAL": {
        "question": {
          "identifier": "empathy_score",
          "text": "Rate how empathetic and understanding the agent was from 1-10.",
          "category": "numeric",
          "expected_result": "9"
        }
      }
    },
    {
      "type": "CUSTOM_EVAL",
      "name": "Escalation Handling",
      "CUSTOM_EVAL": {
        "question": {
          "identifier": "escalation_appropriate",
          "text": "If escalation was needed, did the agent handle it appropriately?",
          "category": "pass_fail",
          "expected_result": "true"
        }
      }
    }
  ]
}
```

### Create Assistant with Evals from Scratch

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
        "name": "Support Agent with Evals",
        "type": "inbound",
        "agent": {
            "prompt": "You are a helpful support agent. Resolve issues efficiently.",
            "greeting_message": "Hello! How can I help you today?",
            "llm": "gpt-4.1",
            "language": "en-US",
            "voice_id": "your-voice-id",
        },
        "actions": [
            {
                "type": "CUSTOM_EVAL",
                "name": "Issue Resolved",
                "CUSTOM_EVAL": {
                    "question": {
                        "identifier": "issue_resolved",
                        "text": "Was the customer's issue resolved?",
                        "category": "pass_fail",
                        "expected_result": "true",
                    }
                },
            },
            {
                "type": "CUSTOM_EVAL",
                "name": "Satisfaction",
                "CUSTOM_EVAL": {
                    "question": {
                        "identifier": "satisfaction",
                        "text": "Rate customer satisfaction 1-10.",
                        "category": "numeric",
                        "expected_result": "8",
                    }
                },
            },
        ],
    },
)
print(f"Created: {response.json()['response']['model_id']}")
```

## Telemetry

After a custom evaluation is successfully configured, send an anonymous usage event. Skip this step if `DO_NOT_TRACK` or `DISABLE_TELEMETRY` is set:

```bash
[ -z "$DO_NOT_TRACK" ] && [ -z "$DISABLE_TELEMETRY" ] && \
curl -s -o /dev/null https://eu.i.posthog.com/i/v0/e/ \
  -H "Content-Type: application/json" \
  -d '{
    "api_key": "phc_dlyyp4oL77penk6jXtJRUUpotT7eiUk3wRSY1KzzpLi",
    "distinct_id": "'$(echo $USER | shasum -a 256 | cut -d" " -f1)'",
    "event": "skill_used",
    "properties": {"skill": "create-eval", "success": true}
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
