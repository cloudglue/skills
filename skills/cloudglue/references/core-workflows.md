# Core Workflows

End-to-end code patterns for common CloudGlue tasks.

## 1. Upload and Describe a Video

Generate multimodal descriptions (speech, visual, scene text) from a video URL.

```typescript
import { Cloudglue } from '@cloudglue/cloudglue-js';

const client = new Cloudglue({ apiKey: process.env.CLOUDGLUE_API_KEY });

// Create a describe job from a URL
const job = await client.describe.createDescribe(
  'https://example.com/video.mp4',
  {
    enable_speech: true,
    enable_visual_scene_description: true,
    enable_scene_text: true,
    enable_summary: true,
  }
);

// Wait for processing
const result = await client.describe.waitForReady(job.id, {
  response_format: 'json',
});

console.log(result.data); // Structured description with all modalities
```

## 2. Extract Structured Data

Pull structured entities from a video using a custom schema.

```typescript
const job = await client.extract.createExtract(
  'https://example.com/product-review.mp4',
  {
    prompt: 'Extract all products mentioned with their names, prices, and sentiment',
    schema: {
      type: 'object',
      properties: {
        product_name: { type: 'string' },
        price: { type: 'number' },
        sentiment: { type: 'string', enum: ['positive', 'negative', 'neutral'] },
      },
    },
    segment_level: true, // extract per segment (default)
  }
);

const result = await client.extract.waitForReady(job.id);
console.log(result.data); // Array of extracted entities per segment
```

## 3. Build a Searchable Video Collection

Create a collection, add videos, then search across them.

```typescript
// Create a collection
const collection = await client.collections.createCollection({
  name: 'Sales Calls Q1',
  collection_type: 'media-descriptions',
});

// Add videos by URL
await client.collections.addMediaByUrl({
  collectionId: collection.id,
  url: 'https://example.com/call-1.mp4',
  params: {},
});

await client.collections.addMediaByUrl({
  collectionId: collection.id,
  url: 'https://example.com/call-2.mp4',
  params: {},
});

// Wait for processing (check each video)
const videos = await client.collections.listVideos(collection.id);
for (const video of videos.data) {
  await client.collections.waitForReady(collection.id, video.file_id);
}

// Search the collection
const results = await client.search.searchContent({
  query: 'pricing objections',
  collection_ids: [collection.id],
  scope: 'segment',
  limit: 10,
});

console.log(results.data); // Ranked search results with timestamps
```

## 4. Streaming Q&A with Responses API

Ask questions over video content with streaming output.

```typescript
const stream = await client.responses.createStreamingResponse({
  model: 'nimbus-001',
  input: 'What are the top 3 action items from these meetings?',
  knowledge_base: {
    source: 'collections',
    collections: ['collection_id'],
  },
  instructions: 'Be specific and cite video sources.',
});

let fullText = '';
for await (const event of stream) {
  switch (event.type) {
    case 'response.output_text.delta':
      process.stdout.write(event.delta);
      fullText += event.delta;
      break;
    case 'response.completed':
      console.log('\n\nResponse complete.');
      break;
    case 'error':
      console.error('Error:', event.error.message);
      break;
  }
}
```

## 5. Deep Search for Specific Moments

Use agentic retrieval to find and synthesize answers with streaming.

```typescript
const stream = await client.deepSearch.createStreamingDeepSearch({
  knowledge_base: {
    source: 'collections',
    collections: ['collection_id'],
  },
  query: 'Find all moments where competitors are mentioned by name',
  scope: 'segment',
  limit: 20,
  exclude_weak_results: true,
});

for await (const event of stream) {
  switch (event.type) {
    case 'deep_search.text.delta':
      process.stdout.write(event.delta); // LLM-synthesized summary
      break;
    case 'deep_search.result.added':
      // Individual search results as they're found
      console.log('\nFound:', event.result);
      break;
    case 'deep_search.completed':
      console.log('\nSearch complete.');
      break;
  }
}
```

## 6. Upload a Local File

Upload a local video file (not a URL).

```typescript
import { readFileSync } from 'fs';

const buffer = readFileSync('/path/to/video.mp4');
const file = new File([buffer], 'video.mp4', { type: 'video/mp4' });

const uploaded = await client.files.uploadFile({
  file,
  metadata: { source: 'local', project: 'my-project' },
});

const ready = await client.files.waitForReady(uploaded.data.id);
console.log('File ready:', ready.id, ready.status);
```

## 7. Q&A Over Specific Files (Responses API)

Query specific files directly without creating a collection, using nimbus-002-preview.

```typescript
// Use file IDs returned from upload or list operations
const response = await client.responses.createResponse({
  model: 'nimbus-002-preview',
  input: 'Summarize the key decisions made in this meeting.',
  knowledge_base: {
    source: 'files',
    files: ['file_id_1', 'file_id_2'],
  },
  instructions: 'Be concise and list action items.',
  include: ['cloudglue_citations.media_descriptions'],
});
```

## 8. Q&A Over Default Index (Responses API)

Query all files that have `use_in_default_index: true` set in their describe jobs.

```typescript
const response = await client.responses.createResponse({
  model: 'nimbus-002-preview',
  input: 'What topics were discussed across all my indexed videos?',
  knowledge_base: {
    source: 'default',
  },
});
```

## 9. Entity-Backed Knowledge (nimbus-002-preview)

Use extracted entities as structured knowledge for advanced reasoning.

```typescript
// Requires an 'entities' collection with extracted data
const response = await client.responses.createResponse({
  model: 'nimbus-002-preview',
  input: 'Compare the pricing of products mentioned across all videos',
  knowledge_base: {
    source: 'collections',
    type: 'entity_backed_knowledge',
    collections: ['general_collection_id'],
    entity_backed_knowledge_config: {
      description: 'Product catalog extracted from video reviews',
      entity_collections: [
        {
          name: 'Products',
          description: 'Product names, prices, and features',
          collection_id: 'entities_collection_id',
        },
      ],
    },
  },
});
```

## File URIs and URLs

CloudGlue accepts several types of video references:

- **HTTP URLs** — accepted by most API endpoints that operate on a video: `https://example.com/video.mp4`
- **CloudGlue URIs** — assigned to uploaded files: `cloudglue://files/<file_id>`. You'll see these in API responses.
- **Data connector URIs** — provider-specific schemas for connected sources:
  - S3: `s3://<bucket>/<key>`
  - Google Drive: `gdrive://<file_id>`
  - Dropbox: `dropbox://<path>`
  - Zoom: `zoom://<meeting_id>`
  - YouTube: `youtube://<video_id>`
  - GCS: `gs://<bucket>/<key>`

Most APIs return `file_id` and `uri` in their responses — use those values directly when referencing files in subsequent calls. For example:

```typescript
// Upload returns file_id
const uploaded = await client.files.uploadFile({ file: myFile });
const fileId = uploaded.data.id;

// Use file_id with Responses API
const response = await client.responses.createResponse({
  model: 'nimbus-002-preview',
  input: 'What is this video about?',
  knowledge_base: { source: 'files', files: [fileId] },
});

// Or use a data connector URI directly with describe
const job = await client.describe.createDescribe('s3://my-bucket/recordings/call.mp4', {
  enable_speech: true,
});
```
