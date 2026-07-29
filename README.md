# AWS Feature Relevance Notification System — PoC

Automated system that monitors AWS "What's New" announcements, scores their relevance
against customer workload profiles using Amazon Bedrock (Claude), and delivers enriched
Slack notifications with benefit categorization and adoption recommendations.

## Architecture

```
EventBridge (10 min) → RSS Lambda → DynamoDB (dedup) → Step Functions
                                                              │
                                          ┌───────────────────┤
                                          ▼                   ▼
                                   Bedrock KB Retrieve   Free-text Profile
                                          │                   │
                                          └─────────┬─────────┘
                                                    ▼
                                          Claude Scoring (relevance + category)
                                                    │
                                              Score ≥ 40?
                                              /        \
                                            Yes         No → Skip
                                             │
                                             ▼
                                    Claude Enrichment (notification content)
                                             │
                                             ▼
                                    Slack Webhook Notification
```

## Components

| Component | Purpose |
|-----------|---------|
| `lambdas/rss-ingestion/` | Polls AWS RSS feed, deduplicates, matches to workload services |
| `lambdas/relevance-scorer/` | Retrieves workload context (KB + free-text), invokes Claude for scoring |
| `lambdas/slack-notification/` | Formats and sends enriched Slack Block Kit messages |
| `lambdas/workload-manager/` | API for adding/updating workload profiles (free-text + services) |
| `step-functions/` | State machine definition (ASL) |
| `sample-data/` | Sample workload profiles for demo |
| `infrastructure.yaml` | CloudFormation template |

## Benefit Categories

| Category | Tag | What it captures |
|----------|-----|-----------------|
| Cost Optimization | 💰 | Reduces spend, better pricing |
| Performance | ⚡ | Lower latency, higher throughput |
| Security & Compliance | 🔒 | New controls, compliance, encryption |
| Operational Excellence | 🔧 | Simplifies management, observability |
| Reliability | 🛡️ | Higher availability, better DR |
| Technical Debt Reduction | 🧹 | Deprecation replacements, modernization |
| Simplification | ✨ | Replaces complex workarounds |
| New Capability | 🚀 | Unlocks previously impossible patterns |

## Deployment

```bash
# Deploy infrastructure
aws cloudformation deploy \
  --template-file infrastructure.yaml \
  --stack-name feature-relevance-poc \
  --capabilities CAPABILITY_NAMED_IAM \
  --region us-west-2 \
  --parameter-overrides SlackWebhookUrl="https://hooks.slack.com/..."

# Add a workload profile
aws lambda invoke --function-name WorkloadManagerFn \
  --payload '{"action":"create","profile":{...}}' \
  --region us-west-2 /tmp/out.json

# Test with a sample announcement
aws lambda invoke --function-name RSSIngestionFn \
  --payload '{"test_mode":true}' \
  --region us-west-2 /tmp/out.json
```

## Region: us-west-2
## Model: us.anthropic.claude-sonnet-4-6
