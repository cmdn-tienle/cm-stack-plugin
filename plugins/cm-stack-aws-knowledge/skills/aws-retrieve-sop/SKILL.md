---
name: aws-retrieve-sop
description: >
  Retrieve complete execution plans for AWS Standard Operating Procedures (SOPs) via the AWS Knowledge MCP server.
  Use when the user needs step-by-step AWS operational procedures, asks about AWS agent SOPs, wants a
  complete execution plan for an AWS workflow, or needs operational guidance for AWS tasks like deploying
  applications, setting up networking, configuring security, etc. Also trigger on "AWS SOP", "AWS standard
  operating procedure", "AWS execution plan", "AWS workflow steps", "how to deploy to AWS step by step",
  "AWS setup guide". IMPORTANT: Always search for SOPs first using aws-search-docs with topic "agent_sops"
  to discover the correct sop_name before using this skill.
---

# AWS Agent SOP Retrieval

Retrieve complete step-by-step execution plans for AWS Standard Operating Procedures via the AWS Knowledge MCP server at `https://knowledge-mcp.global.api.aws`.

## Overview

This skill wraps the `aws___retrieve_agent_sop` MCP tool. It retrieves expert-level SOPs with detailed, tested, production-hardened steps for common AWS workflows. SOPs include security best practices and error handling.

## CRITICAL PREREQUISITE

**You MUST call `aws-search-docs` with topic `agent_sops` BEFORE using this skill.** SOP names are unpredictable identifiers that can only be discovered through search results. Guessing or fabricating an `sop_name` will fail.

## When to Use

- User needs step-by-step guidance for an AWS task
- User asks "how do I deploy X to AWS"
- User wants a complete execution plan for an AWS workflow
- User asks about AWS SOPs or operational procedures
- User says "AWS setup guide", "deploy step by step", "AWS SOP"

## When NOT to Use

- User wants to search for documentation → use `aws-search-docs`
- User doesn't know the SOP name yet → search first with `aws-search-docs` topic `agent_sops`
- User wants general AWS info → use `aws-research`

## Workflow

```mermaid
flowchart TB
    start([User needs AWS operational guidance]) --> sopname{Know the exact sop_name?}
    sopname -->|No| search[Use aws-search-docs with topic agent_sops]
    search --> find[Find sop_name from search results]
    find --> sopname
    sopname -->|Yes| call[Call aws___retrieve_agent_sop via curl]
    call --> success{Success?}
    success -->|No - invalid name| search
    success -->|Yes| present[Present the complete SOP]
    present --> followup{Need more help?}
    followup -->|Read related docs| readdocs[Use aws-read-docs]
    followup -->|Done| done([End])
    readdocs --> done
```

## Implementation

### Step 1: Discover the SOP Name (Required)

If the user doesn't know the exact SOP name, search first:

```bash
curl -s -X POST https://knowledge-mcp.global.api.aws \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "tools/call",
    "params": {
      "name": "aws___search_documentation",
      "arguments": {
        "search_phrase": "<USER'S TASK DESCRIPTION>",
        "topics": ["agent_sops"],
        "limit": 5
      }
    }
  }' | python3 -c "import sys,json; d=json.load(sys.stdin); r=json.loads(d['result']['content'][0]['text']); print(json.dumps(r, indent=2))"
```

Look for results that contain a `sop_name` field. This is the exact identifier you need.

### Step 2: Retrieve the SOP via Curl

Copy the `sop_name` value exactly — do NOT modify, paraphrase, or guess it:

```bash
curl -s -X POST https://knowledge-mcp.global.api.aws \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "tools/call",
    "params": {
      "name": "aws___retrieve_agent_sop",
      "arguments": {
        "sop_name": "<EXACT_SOP_NAME_FROM_SEARCH>"
      }
    }
  }' | python3 -c "import sys,json; d=json.load(sys.stdin); r=json.loads(d['result']['content'][0]['text']); print(json.dumps(r, indent=2))"
```

### Step 3: Present the SOP

The response contains a complete execution plan with:
- Step-by-step instructions
- Security best practices
- Error handling guidance
- Constraints and requirements

Present the SOP clearly, following the original structure. Emphasize that the steps are production-hardened and must be followed exactly.

### Step 4: Offer Follow-up

- Read related documentation → use `aws-read-docs`
- Search for more SOPs → use `aws-search-docs` with `agent_sops` topic
- Check service availability for a region → use `aws-check-availability`

## Parameter Reference

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `sop_name` | string | Yes | Exact SOP name from search results. Must be copied verbatim from the `sop_name` field. |

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Calling this tool without searching first | ALWAYS search with `agent_sops` topic first to discover the correct name |
| Guessing or modifying the SOP name | Copy the `sop_name` value exactly from search results — no modifications |
| Using the result title instead of `sop_name` | Use only the `sop_name` field value, not the title |
| Getting an error and retrying with the same name | Re-search to get the correct name — the error means the name is wrong |
