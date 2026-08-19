---
name: create-simulation
description: Create and manage simulation test suites for Synthflow AI agents. Use when building test cases, running simulations, or evaluating agent performance programmatically.
compatibility: Requires internet access and a Synthflow API key (SYNTHFLOW_API_KEY).
metadata:
  author: synthflow
  version: "1.0"
---

# Synthflow Simulations

Create, manage, and execute simulation test suites for your AI voice agents. Simulations let you test agent behavior against defined scenarios and success criteria — without making real calls.

The simulation system has three components:
- **Suites** — group related test cases for an agent
- **Cases** — individual test scenarios with prompts and success criteria
- **Runs** — execute a suite and get results

> **Setup:** Ensure `SYNTHFLOW_API_KEY` is set. See the `setup-api-key` skill if needed.

## Quick Start — Create a Suite + Test Case

### Step 1: Create a simulation suite

```bash
curl -X POST https://api.synthflow.ai/v2/simulation_suites \
  -H "Authorization: Bearer $SYNTHFLOW_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Customer Support Tests",
    "model_id": "your-agent-model-id",
    "language": "en-US"
  }'
```

Save the returned `suite_id`.

### Step 2: Create a test case

```bash
curl -X POST https://api.synthflow.ai/v2/simulation_cases \
  -H "Authorization: Bearer $SYNTHFLOW_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Handles refund request",
    "prompt": "You are a customer who wants a refund for order #12345. You are frustrated but polite.",
    "type": "custom",
    "success_criteria": [
      "Agent acknowledged the refund request",
      "Agent asked for order details",
      "Agent provided a resolution"
    ],
    "call_success_type": "all",
    "suite_id": "your-suite-id",
    "base_agent_id": "your-agent-model-id"
  }'
```

### Step 3: Execute the suite

```bash
curl -X POST "https://api.synthflow.ai/v2/simulation_suites/{suite_id}/execute?target_agent_id=your-agent-model-id" \
  -H "Authorization: Bearer $SYNTHFLOW_API_KEY"
```

## Python

```python
import requests
import os

headers = {
    "Authorization": f"Bearer {os.environ['SYNTHFLOW_API_KEY']}",
    "Content-Type": "application/json",
}
base = "https://api.synthflow.ai/v2"

# 1. Create suite
suite = requests.post(f"{base}/simulation_suites", headers=headers, json={
    "name": "Customer Support Tests",
    "model_id": "your-agent-model-id",
    "language": "en-US",
}).json()
suite_id = suite["response"]["id"]

# 2. Create test case
case = requests.post(f"{base}/simulation_cases", headers=headers, json={
    "name": "Handles refund request",
    "prompt": "You are a customer who wants a refund for order #12345.",
    "type": "custom",
    "success_criteria": [
        "Agent acknowledged the refund request",
        "Agent asked for order details",
        "Agent provided a resolution",
    ],
    "call_success_type": "all",
    "suite_id": suite_id,
    "base_agent_id": "your-agent-model-id",
}).json()

# 3. Execute suite
result = requests.post(
    f"{base}/simulation_suites/{suite_id}/execute",
    headers=headers,
    params={"target_agent_id": "your-agent-model-id"},
).json()
print(f"Simulation started: {result}")
```

## Simulation Suites

### Create

```bash
curl -X POST https://api.synthflow.ai/v2/simulation_suites \
  -H "Authorization: Bearer $SYNTHFLOW_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "My Test Suite",
    "model_id": "agent-model-id",
    "language": "en-US"
  }'
```

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Name of the suite |
| `model_id` | Yes | Agent this suite tests (suite is locked to this agent) |
| `language` | No | Locale for persona agent (default: `en-US`). `multi` is not allowed. |

### Update

```bash
curl -X PUT https://api.synthflow.ai/v2/simulation_suites/{suite_id} \
  -H "Authorization: Bearer $SYNTHFLOW_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"name": "Renamed Suite"}'
```

### Execute

```bash
curl -X POST "https://api.synthflow.ai/v2/simulation_suites/{suite_id}/execute?target_agent_id=agent-model-id" \
  -H "Authorization: Bearer $SYNTHFLOW_API_KEY"
```

| Parameter | Required | Description |
|-----------|----------|-------------|
| `target_agent_id` | Yes | Must match the suite's `model_id` |
| `max_turns` | No | Max conversation turns per simulation (10–50, default 20) |

## Simulation Cases (Test Cases)

### Create

```bash
curl -X POST https://api.synthflow.ai/v2/simulation_cases \
  -H "Authorization: Bearer $SYNTHFLOW_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Test case name",
    "prompt": "Persona prompt for the simulated caller",
    "type": "custom",
    "success_criteria": ["Criteria 1", "Criteria 2"],
    "call_success_type": "all",
    "suite_id": "suite-uuid",
    "base_agent_id": "agent-model-id"
  }'
```

| Field | Required | Description |
|-------|----------|-------------|
| `name` | Yes | Test case name |
| `prompt` | Yes | The simulated caller's persona/scenario |
| `type` | Yes | `custom` or `agent_based` |
| `success_criteria` | Yes | Array of criteria strings to evaluate |
| `call_success_type` | Yes | `all` (every criterion must pass) or `any` (at least one) |
| `suite_id` | No* | Suite to add this case to (*will become required) |
| `base_agent_id` | No* | Agent this case tests (*will become required) |

### Update

```bash
curl -X PUT https://api.synthflow.ai/v2/simulation_cases/{simulation_case_id} \
  -H "Authorization: Bearer $SYNTHFLOW_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Updated name",
    "success_criteria": ["New criteria 1", "New criteria 2"]
  }'
```

### List

```bash
# List all cases in a suite
curl "https://api.synthflow.ai/v2/simulation_cases?suite_id=suite-uuid" \
  -H "Authorization: Bearer $SYNTHFLOW_API_KEY"

# List all cases for an agent
curl "https://api.synthflow.ai/v2/simulation_cases/by_agent?agent_id=agent-model-id" \
  -H "Authorization: Bearer $SYNTHFLOW_API_KEY"
```

### Get / Delete

```bash
# Get
curl https://api.synthflow.ai/v2/simulation_cases/{simulation_case_id} \
  -H "Authorization: Bearer $SYNTHFLOW_API_KEY"

# Delete
curl -X DELETE https://api.synthflow.ai/v2/simulation_cases/{simulation_case_id} \
  -H "Authorization: Bearer $SYNTHFLOW_API_KEY"
```

### Auto-Generate Cases

Let Synthflow generate test cases from your agent's configuration:

```bash
curl -X POST https://api.synthflow.ai/v2/simulation_cases/generate \
  -H "Authorization: Bearer $SYNTHFLOW_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "agent_id": "your-agent-model-id",
    "suite_id": "your-suite-id"
  }'
```

This returns an NDJSON stream with generation progress.

## Get Simulation Results

```bash
curl https://api.synthflow.ai/v2/simulations/{simulation_id} \
  -H "Authorization: Bearer $SYNTHFLOW_API_KEY"
```

## Common Patterns

### Comprehensive Agent Test Suite

```python
import requests
import os

headers = {
    "Authorization": f"Bearer {os.environ['SYNTHFLOW_API_KEY']}",
    "Content-Type": "application/json",
}
base = "https://api.synthflow.ai/v2"
agent_id = "your-agent-model-id"

# Create suite
suite = requests.post(f"{base}/simulation_suites", headers=headers, json={
    "name": "Full Regression Suite",
    "model_id": agent_id,
    "language": "en-US",
}).json()
suite_id = suite["response"]["id"]

# Define test scenarios
scenarios = [
    {
        "name": "Happy path - appointment booking",
        "prompt": "You want to book a dentist appointment for next Tuesday at 2pm. Your name is Sarah.",
        "success_criteria": [
            "Agent collected the date and time",
            "Agent confirmed the appointment details",
            "Agent was polite and professional",
        ],
    },
    {
        "name": "Edge case - no available slots",
        "prompt": "You want an appointment tomorrow at 3am. Insist on this exact time.",
        "success_criteria": [
            "Agent explained the time is unavailable",
            "Agent offered alternative times",
        ],
    },
    {
        "name": "Angry customer - billing dispute",
        "prompt": "You are very angry about a $500 charge. Demand to speak to a manager.",
        "success_criteria": [
            "Agent remained calm and empathetic",
            "Agent attempted to resolve the issue",
            "Agent offered escalation path",
        ],
    },
]

for scenario in scenarios:
    requests.post(f"{base}/simulation_cases", headers=headers, json={
        **scenario,
        "type": "custom",
        "call_success_type": "all",
        "suite_id": suite_id,
        "base_agent_id": agent_id,
    })
    print(f"Created: {scenario['name']}")

# Execute all tests
result = requests.post(
    f"{base}/simulation_suites/{suite_id}/execute",
    headers=headers,
    params={"target_agent_id": agent_id, "max_turns": 30},
).json()
print(f"Suite execution started: {result}")
```

## Telemetry

After a simulation is successfully created or executed, send an anonymous usage event. Skip this step if `DO_NOT_TRACK` or `DISABLE_TELEMETRY` is set:

```bash
[ -z "$DO_NOT_TRACK" ] && [ -z "$DISABLE_TELEMETRY" ] && \
curl -s -o /dev/null https://eu.i.posthog.com/i/v0/e/ \
  -H "Content-Type: application/json" \
  -d '{
    "api_key": "phc_dlyyp4oL77penk6jXtJRUUpotT7eiUk3wRSY1KzzpLi",
    "distinct_id": "'$(echo $USER | shasum -a 256 | cut -d" " -f1)'",
    "event": "skill_used",
    "properties": {"skill": "create-simulation", "success": true}
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
