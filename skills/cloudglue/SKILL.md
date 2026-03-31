---
name: cloudglue
description: Turn video into LLM-ready data with the Cloudglue JS SDK — upload, describe, extract, search, and chat over video collections using the @cloudglue/cloudglue-js package
license: Apache-2.0
metadata:
  author: cloudglue
  version: "1.0"
  repository: https://github.com/cloudglue/skills
---

# Cloudglue SDK Guide

Build AI-powered video applications with Cloudglue. This skill teaches you how to find current documentation, verify API signatures, and use the Cloudglue JS SDK to upload, describe, extract, search, and chat over video content.

## Critical: Do not trust internal knowledge

Everything you know about Cloudglue is likely outdated or wrong. Never rely on memory. Always verify against current documentation.

Your training data may contain obsolete APIs, deprecated patterns, and incorrect usage. Cloudglue evolves rapidly — APIs change between versions, method signatures shift, and patterns get refactored. Always check the embedded docs or source code for the installed SDK version before writing any code.

## Finding Documentation

Use this 3-tier hierarchy — start with tier 1 and fall back as needed:

### Tier 1: Embedded Docs (most reliable, version-locked)

```
node_modules/@cloudglue/cloudglue-js/docs/
```

Read these first. They contain exact method signatures, parameters, code snippets, and types for the installed SDK version. See [references/embedded-docs.md](references/embedded-docs.md) for a file listing and decision tree.

### Tier 2: Source Code

```
node_modules/@cloudglue/cloudglue-js/src/api/*.api.ts
```

Each `Enhanced*Api` class wraps a generated client with user-friendly methods. Read the source when embedded docs lack detail on edge cases or internal behavior.

### Tier 3: Remote Docs

```
https://docs.cloudglue.dev/llms.txt
```

Use when you need conceptual deep dives, data connector setup, or features not yet in the installed SDK version. See [references/remote-docs.md](references/remote-docs.md) for navigation.

## Quick Start

```bash
npm install @cloudglue/cloudglue-js
```

```typescript
import { Cloudglue } from '@cloudglue/cloudglue-js';

const client = new Cloudglue({
  apiKey: process.env.CLOUDGLUE_API_KEY,  // keys start with 'cg-'
});
```

## Core Mental Model

```
Files (upload video/audio)
  → Processing (describe, extract, face detection)
    → Collections (group processed files by type)
      → Querying (chat, search, deep search, responses API)
```

### Files
Upload video/audio files or provide URLs. Files go through processing (pending → processing → completed). Use `waitForReady()` to poll until done.

### Describe
Generate multimodal descriptions: speech transcription, visual scene descriptions, scene text (OCR), audio descriptions, summaries. This is the primary way to analyze video content. **Note:** `transcribe` is deprecated — use `describe` instead.

### Extract
Pull structured data from videos using custom prompts and JSON schemas. Supports segment-level (default) or video-level extraction.

### Collections
Group videos for querying. Each collection has a type that determines what data is generated:

| Type | What it produces | Supports |
|------|-----------------|----------|
| `media-descriptions` | Full multimodal descriptions (recommended) | Chat, Search, Responses, Deep Search |
| `rich-transcripts` | Speech with visual context **(deprecated)** | Chat, Search, Deep Search |
| `entities` | Structured extracted data | Responses API (entity-backed, nimbus-002-preview) |
| `face-analysis` | Face detection data | Face queries |

See [references/collection-types.md](references/collection-types.md) for detailed comparison and selection guidance.

### Chat
Q&A over collections using `nimbus-001`. Simple synchronous API. For streaming, background jobs, or function calling, use the Responses API instead.

### Responses API
Next-generation query interface (OpenAI Responses-compatible). Supports streaming, background jobs, function calling (tools), and multiple knowledge base sources (collections, files, or default index). Models: `nimbus-001` (fast), `nimbus-002-preview` (reasoning, entity-backed).

### Search
Semantic search across collections at `segment` scope (find specific moments) or `file` scope (find relevant videos). Supports metadata and property filters.

### Deep Search
Agentic retrieval — runs multiple search passes with LLM reasoning to find and synthesize answers. Supports streaming and the same knowledge base sources as the Responses API.

## API Quick Reference

| Namespace | Key Methods | Notes |
|-----------|-------------|-------|
| `client.files` | `uploadFile`, `listFiles`, `getFile`, `waitForReady` | File param is `globalThis.File` |
| `client.collections` | `createCollection`, `addMedia`, `addMediaByUrl`, `waitForReady` | `addVideo`/`addVideoByUrl` are deprecated |
| `client.describe` | `createDescribe`, `waitForReady`, `getDescribe` | Replaces `transcribe` |
| `client.extract` | `createExtract`, `waitForReady`, `getExtract` | Prompt + schema for structured data |
| `client.chat` | `createCompletion` | Requires videos in collections |
| `client.responses` | `createResponse`, `createStreamingResponse`, `waitForReady` | Preferred over chat for new projects |
| `client.search` | `searchContent` | Segment or file scope |
| `client.deepSearch` | `createDeepSearch`, `createStreamingDeepSearch`, `waitForReady` | Agentic multi-pass search |
| `client.dataConnectors` | `list`, `listFiles` | S3, Dropbox, Google Drive, Zoom, etc. |
| `client.faceDetection` | `createFaceDetection`, `waitForReady` | Detect faces in video |
| `client.faceMatch` | `createFaceMatch`, `waitForReady` | Match faces with source image |

## Key Patterns

### Async Job Polling
All processing jobs use `waitForReady()`:
```typescript
const result = await client.describe.waitForReady(jobId, {
  pollingInterval: 5000,  // ms between polls (default)
  maxAttempts: 36,        // max attempts (default, = 3 min)
});
```

### Knowledge Base Sources
The Responses API and Deep Search share three knowledge base types:
```typescript
{ source: 'collections', collections: ['col_id'] }  // query specific collections
{ source: 'files', files: ['file_id'] }              // query specific files
{ source: 'default' }                                // files with use_in_default_index
```

### Streaming
Responses API and Deep Search support SSE streaming:
```typescript
const stream = await client.responses.createStreamingResponse({ ... });
for await (const event of stream) {
  if (event.type === 'response.output_text.delta') {
    process.stdout.write(event.delta);
  }
}
```

### Filters
Search and list operations accept metadata/property filters:
```typescript
import { FilterOperator } from '@cloudglue/cloudglue-js';
filter: {
  metadata: [{ path: 'key', operator: FilterOperator.Equal, valueText: 'value' }],
}
```

## Common Gotchas

- **`transcribe` is deprecated** — use `describe` for all transcription/description needs
- **`rich-transcripts` collection type is deprecated** — use `media-descriptions` (the default) instead; it provides full multimodal output including speech
- **Chat requires collections** — videos must be added to a collection before querying with chat/search
- **`nimbus-002-preview` for entity-backed** — entity-backed knowledge in Responses API requires this model
- **File uploads use FormData** — the SDK handles this internally via axios, not Zodios
- **`waitForReady` defaults** — 5s interval × 36 attempts = 3 min max; increase for large videos
- **Streaming requires Node 18+** — uses `ReadableStream` / `fetch` APIs
- **`addVideo`/`addVideoByUrl` are deprecated** — use `addMedia`/`addMediaByUrl`

## References

- [Embedded docs guide](references/embedded-docs.md) — how to find and use version-locked SDK docs
- [Remote docs guide](references/remote-docs.md) — navigating the online documentation
- [Common errors](references/common-errors.md) — troubleshooting guide
- [Core workflows](references/core-workflows.md) — end-to-end code patterns
- [Collection types](references/collection-types.md) — choosing the right collection type
