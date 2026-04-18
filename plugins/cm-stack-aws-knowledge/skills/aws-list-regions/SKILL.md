---
name: aws-list-regions
description: >
  List all AWS regions with their identifiers and display names via the AWS Knowledge MCP server.
  Use when the user asks about available AWS regions, needs to know AWS region codes, asks "what regions
  does AWS support", needs region information for deployment planning, or wants to verify a region code.
  Also trigger on "AWS regions list", "show me AWS regions", "which AWS regions are available",
  "AWS region codes", "all AWS regions".
---

# AWS Region Listing

List all AWS regions with identifiers and names via the AWS Knowledge MCP server at `https://knowledge-mcp.global.api.aws`.

## Overview

This skill wraps the `aws___list_regions` MCP tool. It returns a complete list of all AWS regions including their region codes and human-readable names. No parameters required.

## When to Use

- User asks "what regions does AWS support?"
- User needs AWS region codes for deployment or configuration
- User wants to verify a region code is valid
- User is planning multi-region infrastructure
- User says "list AWS regions", "show me all regions", "AWS region codes"

## When NOT to Use

- User wants to check if a specific service is available in a region → use `aws-check-availability`
- User wants AWS documentation → use `aws-search-docs`

## Workflow

```mermaid
flowchart TB
    start([User asks about AWS regions]) --> call[Call aws___list_regions via curl]
    call --> present[Present regions as formatted table]
    present --> followup{Need service availability?}
    followup -->|Yes| avail[Use aws-check-availability]
    followup -->|No| done([End])
    avail --> done
```

## Implementation

### Step 1: Call the API

```bash
curl -s -X POST https://knowledge-mcp.global.api.aws \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "tools/call",
    "params": {
      "name": "aws___list_regions",
      "arguments": {}
    }
  }' | python3 -c "import sys,json; d=json.load(sys.stdin); r=json.loads(d['result']['content'][0]['text']); print(json.dumps(r, indent=2))"
```

### Step 2: Present Results

Format the response as a table with two columns:

| Region Code | Region Name |
|-------------|-------------|
| us-east-1 | US East (N. Virginia) |
| us-west-2 | US West (Oregon) |
| eu-west-1 | Europe (Ireland) |
| ... | ... |

Each result includes:
- `region_id`: The unique region code (e.g., `us-east-1`)
- `region_long_name`: The human-friendly name (e.g., `US East (N. Virginia)`)

### Step 3: Offer Follow-up

- Check service availability in specific regions → use `aws-check-availability`
- Search for documentation about a specific region → use `aws-search-docs`

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Hardcoding region lists instead of calling this tool | Always call the API for the current region list — AWS adds new regions regularly |
| Not suggesting availability check after listing regions | If the user is planning deployments, suggest checking service availability |
