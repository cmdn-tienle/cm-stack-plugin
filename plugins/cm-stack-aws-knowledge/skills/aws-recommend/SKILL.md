---
name: aws-recommend
description: >
  Get content recommendations for AWS documentation pages via the AWS Knowledge MCP server.
  Use when the user is reading an AWS doc and wants related content, asks "what else should I read
  about this", wants to explore related AWS services, needs to find similar documentation, or wants
  to discover popular/new pages for an AWS service. Also trigger on "AWS recommendations", "related
  AWS docs", "similar AWS pages", "what else to read about AWS".
---

# AWS Content Recommendations

Get related content recommendations for AWS documentation pages via the AWS Knowledge MCP server at `https://knowledge-mcp.global.api.aws`.

## Overview

This skill wraps the `aws___recommend` MCP tool. Given an AWS documentation URL, it returns four categories of recommendations: Highly Rated, New, Similar, and Journey (commonly viewed next). Useful for exploring a service or finding related content.

## When to Use

- User is reading an AWS doc and wants related content
- User asks "what else should I read about this topic"
- User wants to explore a new AWS service
- User asks for popular or important pages for an AWS service
- User wants to find newly released features for a service
- User says "recommend AWS docs", "related AWS pages", "similar content"

## When NOT to Use

- User wants to search for docs (no specific URL) → use `aws-search-docs`
- User wants to read a specific doc page → use `aws-read-docs`
- User asks about regional availability → use `aws-check-availability`

## Workflow

```mermaid
flowchart TB
    start([User wants related AWS content]) --> url{Has a doc URL?}
    url -->|No| search[Suggest using aws-search-docs to find a URL first]
    url -->|Yes| validate{URL from docs.aws.amazon.com?}
    validate -->|No| error[URL must be from docs.aws.amazon.com]
    validate -->|Yes| call[Call aws___recommend via curl]
    call --> categorize[Present recommendations by category]
    categorize --> followup{Read a recommendation?}
    followup -->|Yes| readdocs[Use aws-read-docs]
    followup -->|No| done([End])
    readdocs --> done
    search --> done
    error --> done
```

## Implementation

### Step 1: Identify the URL

Get the AWS documentation URL from:
- User directly provides a URL
- Previous `aws-search-docs` results
- Previous `aws-read-docs` content

**Important**: The URL must be from `docs.aws.amazon.com`.

### Step 2: Get Recommendations via Curl

```bash
curl -s -X POST https://knowledge-mcp.global.api.aws \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "tools/call",
    "params": {
      "name": "aws___recommend",
      "arguments": {
        "url": "<AWS_DOC_URL>"
      }
    }
  }' | python3 -c "import sys,json; d=json.load(sys.stdin); r=json.loads(d['result']['content'][0]['text']); print(json.dumps(r, indent=2))"
```

**Example:**
```bash
curl -s -X POST https://knowledge-mcp.global.api.aws \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "tools/call",
    "params": {
      "name": "aws___recommend",
      "arguments": {
        "url": "https://docs.aws.amazon.com/lambda/latest/dg/welcome.html"
      }
    }
  }' | python3 -c "import sys,json; d=json.load(sys.stdin); r=json.loads(d['result']['content'][0]['text']); print(json.dumps(r, indent=2))"
```

### Step 3: Present Results by Category

Organize recommendations into the four categories:

| Category | Description |
|----------|-------------|
| **Highly Rated** | Popular pages within the same AWS service |
| **New** | Recently added pages — useful for finding new features |
| **Similar** | Pages covering similar topics |
| **Journey** | Pages commonly viewed next by other users |

Each recommendation includes: `url`, `title`, and `context` (brief description).

### Step 4: Offer Follow-up

- Read any recommended page → use `aws-read-docs`
- Search for more docs on a topic → use `aws-search-docs`

## Parameter Reference

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `url` | string | Yes | URL of an AWS documentation page. Must be from `docs.aws.amazon.com`. |

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Using a non-docs.aws.amazon.com URL | This tool only works with docs.aws.amazon.com URLs |
| Not leveraging the "New" category for discovering features | Use a service's welcome page URL to find newly released content |
| Recommending without context | Explain why each recommendation is relevant to the user's question |
