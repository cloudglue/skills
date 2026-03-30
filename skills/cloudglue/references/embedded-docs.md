# Embedded Docs Guide

The CloudGlue JS SDK ships documentation in `node_modules/@cloudglue/cloudglue-js/docs/`. These docs are version-locked to the installed SDK and are the most reliable reference.

## File Listing

| File | What it covers |
|------|---------------|
| `overview.md` | What CloudGlue is, mental model, SDK architecture, namespace reference table |
| `getting-started.md` | Installation, client setup, env vars, imports, error handling basics |
| `files.md` | Upload, list, get, update, delete files; segments; thumbnails; waitForReady |
| `collections.md` | Create collections, add videos, list/get processed data (entities, descriptions, transcripts) |
| `describe.md` | Multimodal video descriptions — modalities, response formats, waitForReady |
| `extract.md` | Structured data extraction — prompts, schemas, segment vs video level |
| `chat.md` | Chat completions over collections with nimbus-001 |
| `responses.md` | Responses API — streaming, knowledge bases, function calling, background jobs |
| `search.md` | Semantic search — scopes, filters, FilterOperator |
| `deep-search.md` | Agentic retrieval — streaming, knowledge bases, background jobs |
| `data-connectors.md` | Browse files in S3, Dropbox, Google Drive, Zoom, etc. |
| `advanced.md` | Face detection, face match, segmentations, webhooks, tags, shareable |
| `types.md` | Key TypeScript types, interfaces, enums |
| `errors.md` | CloudglueError, common HTTP errors, polling timeouts, streaming issues |

## Which File to Read

**Starting a new project?**
→ Read `overview.md` then `getting-started.md`

**Uploading and processing video?**
→ Read `files.md`, then `describe.md` or `extract.md` depending on your goal

**Building a searchable video collection?**
→ Read `collections.md`, then `search.md` or `chat.md`

**Implementing streaming Q&A?**
→ Read `responses.md` (preferred) or `chat.md` (simpler, no streaming)

**Looking up a specific type or interface?**
→ Read `types.md`

**Debugging an error?**
→ Read `errors.md`

**Using face detection or other advanced features?**
→ Read `advanced.md`

## Fallback to Source Code

If embedded docs lack detail, read the source directly:

```
node_modules/@cloudglue/cloudglue-js/src/api/*.api.ts
```

Key files:
- `files.api.ts` — `EnhancedFilesApi` with upload, listing, segmentation, thumbnails
- `collections.api.ts` — `EnhancedCollectionsApi` with collection CRUD and data retrieval
- `describe.api.ts` — `EnhancedDescribeApi` with describe job lifecycle
- `extract.api.ts` — `EnhancedExtractApi` with extraction job lifecycle
- `chat-completion.api.ts` — `EnhancedChatApi` with chat completions
- `response.api.ts` — `EnhancedResponseApi` with streaming, knowledge bases, tools
- `search.api.ts` — `EnhancedSearchApi` with semantic search
- `deep-search.api.ts` — `EnhancedDeepSearchApi` with agentic retrieval
- `data-connectors.api.ts` — `EnhancedDataConnectorsApi` for external sources
- `face-detection.api.ts` — `EnhancedFaceDetectionApi`
- `face-match.api.ts` — `EnhancedFaceMatchApi`

The main client orchestration is in `src/client.ts` and types are in `src/types.ts`.
