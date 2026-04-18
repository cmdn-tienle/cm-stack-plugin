---
name: aws-check-availability
description: >
  Check AWS resource and service availability across regions via the AWS Knowledge MCP server.
  Use when the user asks if an AWS service is available in a specific region, needs to compare service
  availability across regions, asks about regional support for AWS products, APIs, or CloudFormation
  resources, or wants to plan multi-region deployments. Also trigger on "is AWS X available in region Y",
  "AWS service availability", "which regions support AWS Lambda/ECS/Fargate/etc", "AWS regional
  availability", "compare AWS regions".
---

# AWS Regional Availability Checker

Check AWS resource availability across regions via the AWS Knowledge MCP server at `https://knowledge-mcp.global.api.aws`.

## Overview

This skill wraps the `aws___get_regional_availability` MCP tool. It checks whether AWS products (services/features), API operations, or CloudFormation resource types are available in specific regions. Supports comparing availability across multiple regions.

## When to Use

- User asks "is service X available in region Y?"
- User wants to compare service availability across regions
- User needs to verify CloudFormation support in a region
- User asks about API availability by region
- User is planning multi-region deployments
- User says "AWS regional availability", "compare regions", "which regions support..."

## When NOT to Use

- User wants a list of all regions → use `aws-list-regions`
- User wants AWS documentation → use `aws-search-docs`
- User wants general AWS information → use `aws-research`

## Workflow

```mermaid
flowchart TB
    start([User asks about regional availability]) --> type{Determine resource type}
    type -->|product| product[Check service/feature availability]
    type -->|api| api[Check API operation availability]
    type -->|cfn| cfn[Check CloudFormation resource support]
    type -->|Unsure| infer[Infer type from context]
    infer --> type
    product --> call[Call aws___get_regional_availability via curl]
    api --> call
    cfn --> call
    call --> present[Present availability by region]
    present --> paginated{More results?}
    paginated -->|Yes| next[Use next_token for next page]
    next --> call
    paginated -->|No| done([End])
```

## Implementation

### Step 1: Determine Resource Type

| Resource Type | Value | Use When | Filter Format |
|---------------|-------|----------|---------------|
| **Product** | `product` | Checking AWS services or features | `["AWS Lambda", "Amazon S3"]` |
| **API** | `api` | Checking SDK/API operations | `["IAM+GetSSHPublicKey", "EC2"]` |
| **CloudFormation** | `cfn` | Checking CFN resource types | `["AWS::EC2::Instance"]` |

### Step 2: Identify Regions (Optional)

- Up to 10 regions per call
- If omitted, checks all regions (may require pagination)
- Use `aws-list-regions` first if user doesn't know region codes

### Step 3: Execute via Curl

**Check a specific product in specific regions:**
```bash
curl -s -X POST https://knowledge-mcp.global.api.aws \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "tools/call",
    "params": {
      "name": "aws___get_regional_availability",
      "arguments": {
        "resource_type": "product",
        "regions": ["us-east-1", "eu-west-1", "ap-southeast-1"],
        "filters": ["AWS Lambda"]
      }
    }
  }' | python3 -c "import sys,json; d=json.load(sys.stdin); r=json.loads(d['result']['content'][0]['text']); print(json.dumps(r, indent=2))"
```

**Check API availability in a single region:**
```bash
curl -s -X POST https://knowledge-mcp.global.api.aws \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "tools/call",
    "params": {
      "name": "aws___get_regional_availability",
      "arguments": {
        "resource_type": "api",
        "regions": ["us-east-1"],
        "filters": ["Lambda+Invoke", "S3+GetObject"]
      }
    }
  }' | python3 -c "import sys,json; d=json.load(sys.stdin); r=json.loads(d['result']['content'][0]['text']); print(json.dumps(r, indent=2))"
```

**Check CloudFormation support:**
```bash
curl -s -X POST https://knowledge-mcp.global.api.aws \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "tools/call",
    "params": {
      "name": "aws___get_regional_availability",
      "arguments": {
        "resource_type": "cfn",
        "regions": ["us-east-1"],
        "filters": ["AWS::Lambda::Function"]
      }
    }
  }' | python3 -c "import sys,json; d=json.load(sys.stdin); r=json.loads(d['result']['content'][0]['text']); print(json.dumps(r, indent=2))"
```

### Step 4: Handle Pagination

For single-region queries without filters, results may be paginated. Use `next_token` from the previous response:

```bash
curl -s -X POST https://knowledge-mcp.global.api.aws \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "tools/call",
    "params": {
      "name": "aws___get_regional_availability",
      "arguments": {
        "resource_type": "product",
        "regions": ["us-east-1"],
        "next_token": "<NEXT_TOKEN_FROM_PREVIOUS_RESPONSE>"
      }
    }
  }' | python3 -c "import sys,json; d=json.load(sys.stdin); r=json.loads(d['result']['content'][0]['text']); print(json.dumps(r, indent=2))"
```

### Step 5: Present Results

Status values: `isAvailableIn` | `isNotAvailableIn` | `isPlannedIn` | `Not Found`

**Single region**: Flat structure
```
AWS Lambda: isAvailableIn
```

**Multiple regions**: Nested by region
```
AWS Lambda:
  us-east-1: isAvailableIn
  eu-west-1: isAvailableIn
  ap-south-1: isNotAvailableIn
```

## Parameter Reference

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `resource_type` | string | Yes | One of: `product`, `api`, `cfn` |
| `regions` | string[] | No | Region codes to check. Max 10. |
| `filters` | string[] | No | Specific resources to check. Required for multi-region. |
| `next_token` | string | No | Pagination token. Only for single-region, no filters. |

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Using wrong filter format for resource type | Products: `["AWS Lambda"]`, APIs: `["EC2"]` or `["Lambda+Invoke"]`, CFN: `["AWS::EC2::Instance"]` |
| Querying many regions without filters (multi-region) | Filters are required when querying multiple regions |
| More than 10 regions in a single call | Split into multiple calls of max 10 regions each |
| Not handling pagination for large results | Check for `next_token` in response and make follow-up calls |
