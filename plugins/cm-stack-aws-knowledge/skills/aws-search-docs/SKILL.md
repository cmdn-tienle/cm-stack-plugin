---
name: aws-search-docs
description: >
  Search across all AWS documentation, agent SOPs, and technical content using the AWS Knowledge MCP server.
  Use when the user asks about AWS services, needs AWS documentation, wants to look up AWS features,
  asks "how to" questions about AWS, needs troubleshooting guidance for AWS, or mentions specific AWS
  services like EC2, S3, Lambda, RDS, DynamoDB, ECS, EKS, CloudFormation, CDK, Amplify, or any other
  AWS product. Also trigger on phrases like "search AWS docs", "AWS documentation for", "how do I use AWS",
  "AWS best practices", "AWS reference", "AWS troubleshooting", "look up AWS", "find AWS docs".
---

# AWS Documentation Search

Search AWS documentation, SOPs, and technical content via the AWS Knowledge MCP server at `https://knowledge-mcp.global.api.aws`.

## Overview

This skill wraps the `aws___search_documentation` MCP tool. It sends a JSON-RPC 2.0 request via curl to search across all AWS documentation with optional topic-based filtering for targeted results.

## When to Use

- User asks about any AWS service or feature (Lambda, S3, EC2, RDS, DynamoDB, etc.)
- User needs AWS CLI, SDK, or API usage examples
- User asks "how do I..." questions about AWS
- User needs troubleshooting help for AWS errors
- User asks about AWS architecture, patterns, or best practices
- User mentions AWS CDK, CloudFormation, or Amplify
- User says "search AWS docs", "look up AWS", "find AWS documentation"

## When NOT to Use

- User has a specific AWS doc URL and wants to read it → use `aws-read-docs`
- User wants related content for a doc they're reading → use `aws-recommend`
- User wants a comprehensive answer with full doc content → use `aws-research`
- Non-AWS questions

## Workflow

```mermaid
flowchart TB
    start([User asks about AWS]) --> extract[Extract search phrase from query]
    extract --> topics{Determine relevant topics}
    topics --> call[Call aws___search_documentation via curl]
    call --> present[Present results with titles, URLs, and context]
    present --> followup{Follow-up?}
    followup -->|Read a result| readdocs[Use aws-read-docs]
    followup -->|Refine search| extract
    followup -->|Done| done([End])
    readdocs --> done
```

## Implementation

### Step 1: Extract Search Phrase

Analyze the user's question and extract a concise, specific search phrase. Use service names and specific terms for best results.

**Good search phrases:**
- "S3 bucket versioning configuration"
- "Lambda environment variables Python SDK"
- "DynamoDB GSI query patterns"
- "AccessDenied error S3 GetObject"

**Bad search phrases:**
- "versioning" (too vague)
- "environment variables" (missing AWS context)

### Step 2: Determine Relevant Topics (Optional)

Select up to 3 topics from the table below to narrow results. If unsure, omit topics (defaults to "general").

| Topic | Use When | Example Queries |
|-------|----------|----------------|
| `reference_documentation` | API methods, SDK code, CLI commands | "boto3 S3 upload_file", "Lambda InvokeFunction API" |
| `current_awareness` | New features, announcements, releases | "Lambda new features 2024", "what's new in ECS" |
| `troubleshooting` | Error messages, debugging, problems | "AccessDenied S3", "Lambda timeout error" |
| `amplify_docs` | Amplify framework questions | "Amplify Auth React", "Amplify Storage Flutter" |
| `cdk_docs` | CDK concepts, APIs, CLI, getting started | "CDK stack props Python", "cdk deploy command" |
| `cdk_constructs` | CDK code examples, patterns, constructs | "Lambda function CDK Python example" |
| `cloudformation` | CloudFormation templates, SAM | "DynamoDB CloudFormation template" |
| `agent_sops` | Step-by-step AWS operational procedures | "set up VPC peering", "deploy Lambda with API Gateway" |
| `general` | Architecture, blogs, guides, best practices | "Lambda best practices", "S3 architecture patterns" |

### Step 3: Execute Search via Curl

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
        "search_phrase": "<SEARCH_PHRASE>",
        "topics": ["<TOPIC_1>", "<TOPIC_2>"],
        "limit": 5
      }
    }
  }' | python3 -c "import sys,json; d=json.load(sys.stdin); r=json.loads(d['result']['content'][0]['text']); print(json.dumps(r, indent=2))"
```

**Minimal example (no topic filter):**
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
        "search_phrase": "AWS Lambda best practices"
      }
    }
  }' | python3 -c "import sys,json; d=json.load(sys.stdin); r=json.loads(d['result']['content'][0]['text']); print(json.dumps(r, indent=2))"
```

### Step 4: Present Results

Parse and present results in a readable format:

- **rank_order**: Relevance (lower = more relevant)
- **title**: Page title
- **url**: Direct documentation link
- **context**: Summary excerpt

### Step 5: Offer Follow-up

- Read a specific result → use `aws-read-docs` skill
- Refine the search with different terms or topics
- Get recommendations for a result URL → use `aws-recommend` skill
- If SOP results appear, retrieve full SOP → use `aws-retrieve-sop` skill

## Parameter Reference

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `search_phrase` | string | Yes | Natural language search query. Be specific with service names. |
| `topics` | string[] | No | Filter by topic category. Max 3 values. Defaults to `["general"]`. |
| `limit` | integer | No | Maximum results per topic. Default is 5. |

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Using vague search phrases like "AWS" | Include service name and specific feature, e.g., "S3 bucket encryption" |
| Using more than 3 topics | Pick the 3 most relevant. Multiple topics broaden results. |
| Using general knowledge instead of searching | Always search for AWS topics — the tool has up-to-date information. |
| Not including framework/language for Amplify/CDK queries | Include framework (React, Next.js) or language (Python, TypeScript) in the search phrase. |
