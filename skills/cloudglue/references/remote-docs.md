# Remote Docs Guide

Use remote documentation when embedded docs lack detail, you need conceptual deep dives, or you're working with features not yet in the installed SDK version.

## LLM-Friendly Entry Point

```
https://docs.cloudglue.dev/llms.txt
```

This provides an agent-optimized overview of Cloudglue's capabilities, documentation structure, and key links.

## Documentation Site Structure

### Getting Started
- `https://docs.cloudglue.dev/introduction` — Overview and value proposition
- `https://docs.cloudglue.dev/getting-started/setup` — Account setup and API keys
- `https://docs.cloudglue.dev/getting-started/quickstart-*` — Language-specific quickstarts
- `https://docs.cloudglue.dev/getting-started/webhooks` — Webhook configuration
- `https://docs.cloudglue.dev/getting-started/rate-limits` — Rate limits by plan tier
- `https://docs.cloudglue.dev/getting-started/mcp-server` — MCP server for AI tools

### Core Concepts
- `https://docs.cloudglue.dev/core-concepts/files` — File management deep dive
- `https://docs.cloudglue.dev/core-concepts/segments` — Video segmentation concepts
- `https://docs.cloudglue.dev/core-concepts/entities` — Entity extraction concepts
- `https://docs.cloudglue.dev/core-concepts/media-descriptions` — Description modalities
- `https://docs.cloudglue.dev/core-concepts/collections` — Collection types and configuration
- `https://docs.cloudglue.dev/core-concepts/operations` — Processing operations

### Deep Dives
- `https://docs.cloudglue.dev/deep-dives/extraction-guide` — Detailed extraction patterns
- `https://docs.cloudglue.dev/deep-dives/description-guide` — Description configuration
- `https://docs.cloudglue.dev/deep-dives/chat-completion` — Chat API details
- `https://docs.cloudglue.dev/deep-dives/responses-api` — Responses API guide

### Connect Your Data
- `https://docs.cloudglue.dev/connect-your-data/overview` — Data connector overview
- Individual connector docs: S3, Dropbox, Google Drive, GCS, Zoom, Gong, Recall

### API Reference
- `https://docs.cloudglue.dev/api-reference/introduction` — API reference overview
- Endpoint docs organized by feature (Files, Collections, Chat, Search, etc.)

### SDKs
- `https://docs.cloudglue.dev/sdks/overview` — SDK overview
- `https://docs.cloudglue.dev/sdks/javascript` — JavaScript SDK guide
- `https://docs.cloudglue.dev/sdks/python` — Python SDK guide

## When to Use Remote Docs

| Scenario | What to read |
|----------|-------------|
| Understanding rate limits for your plan | Getting Started → Rate Limits |
| Setting up a data connector (S3, Zoom, etc.) | Connect Your Data → specific connector |
| Deep dive on extraction prompt engineering | Deep Dives → Extraction Guide |
| Understanding collection type tradeoffs | Core Concepts → Collections |
| Responses API advanced patterns | Deep Dives → Responses API |
| Checking latest API changes | Changelog (linked from docs site) |
| MCP server integration | Getting Started → MCP Server |
