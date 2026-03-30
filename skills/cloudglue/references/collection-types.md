# Collection Types

Collections group videos for batch processing and querying. The collection type determines what data is generated for each video and which query APIs are available.

## Type Comparison

| Type | Data Generated | Chat | Search | Responses | Deep Search |
|------|---------------|------|--------|-----------|-------------|
| `media-descriptions` | Speech + visual + scene text + audio descriptions (recommended) | Yes | Yes | Yes | Yes |
| `rich-transcripts` | Speech transcription with visual context **(deprecated)** | Yes | Yes | Yes | Yes |
| `entities` | Structured data via extraction prompts/schemas | No | No | Yes (entity-backed) | No |
| `face-analysis` | Face detection data per video | No | No | No | No |

## media-descriptions (default)

The most versatile type. Generates full multimodal descriptions including speech transcription, visual scene descriptions, scene text (OCR), and audio descriptions.

**Best for:** General-purpose video understanding, Q&A, search, content analysis.

```typescript
const collection = await client.collections.createCollection({
  name: 'Meeting Recordings',
  collection_type: 'media-descriptions',
  // Optional: customize what modalities to generate
  // describe_config: {
  //   enable_speech: true,
  //   enable_visual_scene_description: true,
  //   enable_scene_text: true,
  //   enable_audio_description: true,
  // },
});
```

Retrieve data:
```typescript
const descriptions = await client.collections.getMediaDescriptions(collectionId, fileId, {
  response_format: 'markdown',
  include_word_timestamps: true,
  include_chapters: true,
});
```

## rich-transcripts (deprecated)

> **Deprecated:** Use `media-descriptions` instead. It provides full multimodal output including speech transcription, and supports all query APIs.

Focused on speech transcription with some visual context. Lighter processing than media-descriptions.

**Best for:** Legacy use only. For new collections, use `media-descriptions`.

```typescript
const collection = await client.collections.createCollection({
  name: 'Podcast Episodes',
  collection_type: 'rich-transcripts',
});
```

Retrieve data:
```typescript
const transcripts = await client.collections.getTranscripts(collectionId, fileId, {
  response_format: 'json',
  modalities: ['speech', 'visual_scene_description'],
  include_word_timestamps: true,
});
```

## entities

Generates structured extracted data using prompts and schemas. Does NOT support chat or search — designed for use with the Responses API's entity-backed knowledge feature (requires `nimbus-002-preview`).

**Best for:** Structured data extraction (product catalogs, action items, CRM data from calls).

Example entity schema (for meeting analysis):
```json
{
  "meeting_summary": "a paragraph describing summary",
  "action_items": ["list of action items"],
  "participants": [{"email": "participant email", "name": "full name"}]
}
```

```typescript
const collection = await client.collections.createCollection({
  name: 'Meeting Insights',
  collection_type: 'entities',
  // extract_config: {
  //   prompt: 'Extract meeting summary, action items, and participants',
  //   schema: { ... },  // use the JSON schema above
  // },
});
```

Retrieve data:
```typescript
const entities = await client.collections.getEntities(collectionId, fileId, {
  include_thumbnails: true,
  include_chapters: true,
});

// Use with Responses API for entity-backed knowledge
const response = await client.responses.createResponse({
  model: 'nimbus-002-preview',
  input: 'Compare product prices across videos',
  knowledge_base: {
    source: 'collections',
    type: 'entity_backed_knowledge',
    collections: ['general_col_id'],
    entity_backed_knowledge_config: {
      entity_collections: [{
        name: 'Products',
        description: 'Extracted product entities',
        collection_id: 'entities_col_id',
      }],
    },
  },
});
```

## face-analysis

Runs face detection on all videos in the collection.

**Best for:** Identifying speakers, tracking people across videos.

```typescript
const collection = await client.collections.createCollection({
  name: 'Team Calls',
  collection_type: 'face-analysis',
});
```

Retrieve data:
```typescript
const faces = await client.collections.getFaceDetections(collectionId, fileId);
```

## How to Choose

```text
Do you need to search or chat over videos?
├── Yes → media-descriptions (recommended default)
└── No → Do you need structured data extraction?
    ├── Yes → entities (+ Responses API with nimbus-002-preview)
    └── No → Do you need face detection?
        ├── Yes → face-analysis
        └── No → media-descriptions (safe default)
```

## Configuration Options

All collection types support:
- `segmentation_config` — How to segment videos (`{ type: 'shot_detector' }`, `'uniform'`, `'narrative'`)
- `thumbnails_config` — Thumbnail generation settings

Type-specific configs:
- `media-descriptions`: `describe_config` with modality toggles
- `entities`: `extract_config` with prompt and schema

## Listing Data Across a Collection

```typescript
// List all entities across the collection
const allEntities = await client.collections.listEntities(collectionId, {
  limit: 50,
  order: 'added_at',
  sort: 'desc',
});

// List all media descriptions
const allDescriptions = await client.collections.listMediaDescriptions(collectionId, {
  response_format: 'json',
  modalities: ['speech'],
  limit: 50,
});

// List all rich transcripts
const allTranscripts = await client.collections.listRichTranscripts(collectionId, {
  limit: 50,
});
```
