# Common Errors & Troubleshooting

## CloudglueError

All API errors throw `CloudglueError` with these properties:

```typescript
import { CloudglueError } from '@cloudglue/cloudglue-js';

try {
  await client.files.getFile('bad-id');
} catch (error) {
  if (error instanceof CloudglueError) {
    error.message;       // Human-readable error description
    error.statusCode;    // HTTP status code (401, 404, 429, etc.)
    error.data;          // Request body that caused the error
    error.headers;       // Response headers
    error.responseData;  // Full response body from the API
  }
}
```

## Authentication Errors

### "API key is required"
**Cause:** No API key provided and `CLOUDGLUE_API_KEY` env var is not set.
**Fix:** Either pass `apiKey` to the constructor or set the environment variable:
```typescript
const client = new Cloudglue({ apiKey: 'cg-...' });
// OR
export CLOUDGLUE_API_KEY=cg-...
```

### 401 Unauthorized
**Cause:** API key is invalid, expired, or malformed.
**Fix:** CloudGlue API keys start with `cg-`. Verify your key in the CloudGlue dashboard.

## Rate Limiting

### 429 Too Many Requests
**Cause:** Exceeded rate limits for your plan tier.
**Fix:** Back off and retry with exponential backoff. Check rate limits at `https://docs.cloudglue.dev/getting-started/rate-limits`.

## Resource Errors

### 404 Not Found
**Cause:** The resource ID doesn't exist or belongs to a different account.
**Fix:** Verify the ID is correct. Resource IDs (file_id, collection_id, job_id) are UUIDs.

### "File processing failed"
**Cause:** `waitForReady()` detected a file reached `failed` status.
**Fix:** Check the file details via `client.files.getFile(fileId)` for error information. Common causes:
- Unsupported video format
- Corrupted file
- File too large

## Polling Timeouts

### "Timeout waiting for file ... to process after N attempts"
**Cause:** `waitForReady()` exhausted all polling attempts before the job completed.
**Fix:** Increase polling parameters:
```typescript
await client.files.waitForReady(fileId, {
  pollingInterval: 10000,  // 10s between polls (default: 5s)
  maxAttempts: 120,        // 120 attempts = 20 min (default: 36 = 3 min)
});
```
Large videos can take several minutes to process.

## Collection Errors

### Videos not processing in collection
**Cause:** The source file hasn't finished processing yet.
**Fix:** Ensure the file is `completed` before adding to a collection:
```typescript
await client.files.waitForReady(fileId);
await client.collections.addMedia(collectionId, fileId, {});
await client.collections.waitForReady(collectionId, fileId);
```

### Chat/search returns no results
**Cause:** Collection has no `completed` videos, or the collection type doesn't support the query API.
**Fix:**
1. Check video status: `client.collections.listVideos(collectionId, { status: 'completed' })`
2. Verify collection type supports your query API (see collection-types.md)

## Streaming Errors

### "Response body is empty — streaming not supported in this environment"
**Cause:** The runtime doesn't support `ReadableStream` (used for SSE parsing).
**Fix:** Use Node.js 18+ or a modern browser. The SDK uses `fetch()` and `ReadableStream` for streaming.

## File Upload Errors

### Upload fails silently or with FormData errors
**Cause:** The `file` parameter must be a `globalThis.File` object.
**Fix:** In Node.js, create a File from a Buffer or use the `File` constructor:
```typescript
import { readFileSync } from 'fs';
const buffer = readFileSync('video.mp4');
const file = new File([buffer], 'video.mp4', { type: 'video/mp4' });
await client.files.uploadFile({ file });
```

## Network Errors

### 408 Timeout / ECONNABORTED
**Cause:** Request timed out at the HTTP transport level.
**Fix:** Increase the client timeout:
```typescript
const client = new Cloudglue({
  apiKey: process.env.CLOUDGLUE_API_KEY,
  timeout: 60000,  // 60 seconds
});
```

## File Limits & Unsupported Formats

CloudGlue has the following file requirements:
- **Max file size:** 2 GB (250 MB for local uploads via API; use a data connector like S3 for larger files)
- **Max duration:** 2 hours (min 2 seconds)
- **Supported video formats:** MP4, QuickTime, AVI, WebM
- **Supported audio formats:** MP3, M4A/AAC, WAV, FLAC, OGG, WebM

### File too large or too long

Split videos longer than 2 hours into parts:
```bash
ffmpeg -i long_video.mp4 -map 0:v:0 -map 0:a:0? -dn -c copy -f segment \
       -segment_time 01:30:00 -reset_timestamps 1 \
       part_%03d.mp4
```

### Optimize video for upload

Transcode to an optimal format (H.264, ≤1920px, AAC audio):
```bash
ffmpeg -i input_video.ext \
       -c:v libx264 \
       -vf "scale=w=min(iw\,1920):h=-2" \
       -c:a aac -b:a 192k -ac 2 \
       -movflags +faststart \
       output.mp4
```

For full details, see [File Limits & Requirements](https://docs.cloudglue.dev/core-concepts/files#file-limits-%26-requirements).

## YouTube Limitations

YouTube URLs only support speech/summary processing due to copyright restrictions. Visual modalities (`enable_visual_scene_description`, `enable_scene_text`, `enable_audio_description`) are not available for YouTube content. To take advantage of fully multimodal understanding, download and upload the video directly instead.

## Deprecation Warnings

- `client.transcribe` → Use `client.describe` instead
- `client.collections.addVideo()` → Use `client.collections.addMedia()`
- `client.collections.addVideoByUrl()` → Use `client.collections.addMediaByUrl()`
- `rich-transcripts` collection type → Use `media-descriptions` instead
