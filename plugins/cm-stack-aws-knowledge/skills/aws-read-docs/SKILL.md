---
name: aws-read-docs
description: >
  Fetch and read full AWS documentation pages converted to markdown format via the AWS Knowledge MCP server.
  Use when the user has a specific AWS documentation URL and wants to read the full content, needs detailed
  documentation from a URL found via search, or asks to "read this AWS doc" or "open this AWS page".
  Also trigger when the user pastes an AWS documentation URL and asks about its contents, says "show me this
  AWS page", or wants the full documentation for a specific AWS topic URL.
---

# AWS Documentation Reader

Fetch and convert AWS documentation pages to readable markdown via the AWS Knowledge MCP server at `https://knowledge-mcp.global.api.aws`.

## Overview

This skill wraps the `aws___read_documentation` MCP tool. It retrieves AWS documentation pages and converts them to markdown format. Supports batching multiple URLs, pagination for long documents, and includes a Table of Contents for navigation.

## When to Use

- User has a specific AWS documentation URL to read
- User wants full content from a URL found via `aws-search-docs`
- User pastes an AWS doc URL and asks about its contents
- User says "read this AWS page", "show me this doc", "open this URL"
- User needs to read multiple AWS doc pages at once

## When NOT to Use

- User wants to search for AWS docs (no specific URL) → use `aws-search-docs`
- User wants related content recommendations → use `aws-recommend`
- User wants a comprehensive research answer → use `aws-research`
- URL is not from an allowed domain

## Allowed URL Domains

- `docs.aws.amazon.com`
- `aws.amazon.com` (except `/marketplace`)
- `repost.aws/knowledge-center`
- `docs.amplify.aws`
- `ui.docs.amplify.aws`
- `github.com/aws-cloudformation/aws-cloudformation-templates`
- `github.com/aws-samples/aws-cdk-examples`
- `github.com/aws-samples/generative-ai-cdk-constructs-samples`
- `github.com/aws-samples/serverless-patterns`
- `github.com/awsdocs/aws-cdk-guide`
- `github.com/awslabs/aws-solutions-constructs`
- `github.com/cdklabs/cdk-nag`
- `constructs.dev/packages/*`

## Workflow

```mermaid
flowchart TB
    start([User provides AWS doc URL]) --> validate{URL from allowed domain?}
    validate -->|No| error[Inform user of domain restrictions]
    validate -->|Yes| build[Build request with URL and options]
    build --> call[Call aws___read_documentation via curl]
    call --> parse[Parse and present markdown content]
    parse --> truncated{Content truncated?}
    truncated -->|Yes| paginate[Use start_index to continue reading]
    paginate --> call
    truncated -->|No| followup{Follow-up?}
    followup -->|Get recommendations| rec[Use aws-recommend]
    followup -->|Search more| search[Use aws-search-docs]
    followup -->|Done| done([End])
    rec --> done
    search --> done
    error --> done
```

## Implementation

### Step 1: Identify URL(s)

Get the AWS documentation URL from:
- User directly provides a URL
- Previous `aws-search-docs` results
- User references a specific AWS documentation page

### Step 2: Build the Request

Each request object has:
- `url` (required): The documentation URL
- `max_length` (optional): Max characters to return. Default 10000. Use smaller values for TOC/overview.
- `start_index` (optional): Starting character position. Default 0. Use for pagination.

**Single URL example:**
```bash
curl -s -X POST https://knowledge-mcp.global.api.aws \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "tools/call",
    "params": {
      "name": "aws___read_documentation",
      "arguments": {
        "requests": [
          {
            "url": "https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-management.html",
            "max_length": 10000
          }
        ]
      }
    }
  }' | python3 -c "import sys,json; d=json.load(sys.stdin); r=json.loads(d['result']['content'][0]['text']); print(json.dumps(r, indent=2))"
```

**Batch multiple URLs (2-5 for optimal performance):**
```bash
curl -s -X POST https://knowledge-mcp.global.api.aws \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "tools/call",
    "params": {
      "name": "aws___read_documentation",
      "arguments": {
        "requests": [
          {
            "url": "https://docs.aws.amazon.com/AmazonS3/latest/userguide/access-management.html",
            "max_length": 5000
          },
          {
            "url": "https://repost.aws/knowledge-center/ec2-instance-connection-troubleshooting"
          }
        ]
      }
    }
  }' | python3 -c "import sys,json; d=json.load(sys.stdin); r=json.loads(d['result']['content'][0]['text']); print(json.dumps(r, indent=2))"
```

### Step 3: Handle Response

Each result includes:
- `status`: "SUCCESS" or "ERROR"
- `content`: The markdown text
- `total_length`: Full document length in characters
- `start_index` / `end_index`: Character range returned
- `truncated`: Whether there's more content
- `redirected_url`: If the page was redirected

### Step 4: Handle Long Documents

If `truncated` is true, the response includes a Table of Contents with character positions. Options:

1. **Continue reading**: Set `start_index` to the previous `end_index`
2. **Jump to section**: Use TOC character positions to jump directly
3. **Stop early**: If you found what you need, stop reading

**Example — Jump to a specific section:**
```bash
# If TOC shows "Using a logging library (char 3331-6016)"
curl -s -X POST https://knowledge-mcp.global.api.aws \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "tools/call",
    "params": {
      "name": "aws___read_documentation",
      "arguments": {
        "requests": [
          {
            "url": "https://docs.aws.amazon.com/lambda/latest/dg/python-logging.html",
            "start_index": 3331,
            "max_length": 3000
          }
        ]
      }
    }
  }' | python3 -c "import sys,json; d=json.load(sys.stdin); r=json.loads(d['result']['content'][0]['text']); print(json.dumps(r, indent=2))"
```

## Parameter Reference

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `requests` | array | Yes | List of request objects. Batch 2-5 for efficiency. |
| `requests[].url` | string | Yes | AWS documentation URL. Must be from allowed domains. |
| `requests[].max_length` | integer | No | Max characters to return. Default 10000. |
| `requests[].start_index` | integer | No | Starting character position. Default 0. Use for pagination. |

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Using a non-allowed URL domain | Only use URLs from the allowed list above |
| Fetching entire huge documents at once | Start with a small `max_length` to get TOC, then jump to relevant sections |
| Not batching when reading multiple pages | Use 2-5 requests in a single call for better performance |
| Ignoring truncated responses | Check `truncated` field and use `start_index` to continue reading |
