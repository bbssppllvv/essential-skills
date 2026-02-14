# Soniox Async (File-Based) REST API Reference

Base URL: `https://api.soniox.com`

All requests require header: `Authorization: Bearer <SONIOX_API_KEY>`

---

## Table of Contents

1. [Overview](#overview)
2. [Async Transcription Flow](#async-transcription-flow)
3. [Audio Sources and Formats](#audio-sources-and-formats)
4. [Transcription Endpoints](#transcription-endpoints)
5. [Translation](#translation)
6. [Webhooks](#webhooks)
7. [File Management Endpoints](#file-management-endpoints)
8. [Models Endpoint](#models-endpoint)
9. [Temporary API Keys](#temporary-api-keys)
10. [Limits and Quotas](#limits-and-quotas)
11. [Error Handling](#error-handling)
12. [Error Response Schema](#error-response-schema)

---

## Overview

Soniox async transcription processes pre-recorded audio files without a live connection. Submit audio via a public URL or an uploaded file ID, then poll for results or receive a webhook callback.

Model for async: `stt-async-v4` (latest), `stt-async-v3`, or preview aliases.

---

## Async Transcription Flow

1. **(Optional)** Upload a local file via `POST /v1/files` to get a `file_id`.
2. Create a transcription job via `POST /v1/transcriptions` with `audio_url` or `file_id`.
3. Poll `GET /v1/transcriptions/{id}` until `status` is `completed` or `error` (or use a webhook).
4. Retrieve results via `GET /v1/transcriptions/{id}/transcript`.
5. Clean up: `DELETE /v1/transcriptions/{id}` and `DELETE /v1/files/{file_id}`.

---

## Audio Sources and Formats

### Source Options (mutually exclusive)

| Parameter   | Description                                      |
|-------------|--------------------------------------------------|
| `audio_url` | Public HTTP(S) URL to the audio file             |
| `file_id`   | ID returned from the Files API upload            |

### Supported Formats (auto-detected)

`aac`, `aiff`, `amr`, `asf`, `flac`, `mp3`, `ogg`, `wav`, `webm`, `m4a`, `mp4`

Maximum file size: 1 GB (1,073,741,824 bytes). Maximum duration: 300 minutes.

---

## Transcription Endpoints

### Create Transcription

`POST /v1/transcriptions`

**Request** (`application/json`):

```json
{
  "model": "stt-async-v4",
  "audio_url": "https://example.com/audio.mp3",
  "language_hints": ["en"],
  "enable_language_identification": true,
  "enable_speaker_diarization": true,
  "client_reference_id": "optional-tracking-id",
  "context": {
    "general": [{"key": "domain", "value": "Healthcare"}],
    "text": "Background context paragraph...",
    "terms": ["Celebrex", "Zyrtec"],
    "translation_terms": [{"source": "Mr. Smith", "target": "Sr. Smith"}]
  },
  "translation": {
    "type": "one_way",
    "target_language": "es"
  },
  "webhook_url": "https://example.com/webhook",
  "webhook_auth_header_name": "Authorization",
  "webhook_auth_header_value": "Bearer <secret>"
}
```

Key fields:
- `model` (required): `stt-async-v4` or `stt-async-v3`
- `audio_url` or `file_id` (one required): audio source
- `language_hints`: array of ISO 639-1 codes to bias recognition
- `enable_speaker_diarization`: adds `speaker` field to tokens
- `enable_language_identification`: adds `language` field to tokens
- `context`: structured context for domain, terms, translation terms
- `translation`: see [Translation](#translation) section
- `client_reference_id`: optional string (max 256 chars)
- `webhook_url`, `webhook_auth_header_name`, `webhook_auth_header_value`: see [Webhooks](#webhooks)

**Response** (`201`): Returns transcription object with `id` and `status`.

### Get Transcription Status

`GET /v1/transcriptions/{transcription_id}`

**Response** (`200`):

```json
{
  "id": "548d023b-2b3d-4dc2-a3ef-cca26d05fd9a",
  "status": "completed",
  "error_message": null
}
```

Status values: `pending`, `processing`, `completed`, `error`.

### Get Transcript

`GET /v1/transcriptions/{transcription_id}/transcript`

**Response** (`200`):

```json
{
  "tokens": [
    {
      "text": "Hello",
      "start_ms": 600,
      "end_ms": 760,
      "confidence": 0.97,
      "is_final": true,
      "speaker": "1",
      "language": "en",
      "translation_status": null
    }
  ],
  "text": "Hello world..."
}
```

Token fields:
- `text`: token text
- `start_ms` / `end_ms`: timestamps in milliseconds (absent for translation tokens)
- `confidence`: 0.0-1.0
- `is_final`: always `true` for async results
- `speaker`: speaker label (if diarization enabled)
- `language`: detected language code (if language ID enabled)
- `translation_status`: `null` for original, `"translation"` for translated tokens

### List Transcriptions

`GET /v1/transcriptions?cursor={cursor}&limit={limit}`

**Response** (`200`):

```json
{
  "transcriptions": [
    {"id": "...", "status": "completed", ...}
  ],
  "next_page_cursor": "cursor_or_null"
}
```

### Delete Transcription

`DELETE /v1/transcriptions/{transcription_id}`

**Response**: `204` No Content.

---

## Translation

Add a `translation` object to the create-transcription request. Both modes supported for async.

### One-Way Translation

Translates all detected languages into a single target language.

```json
{
  "translation": {
    "type": "one_way",
    "target_language": "es"
  }
}
```

### Two-Way Translation

Translates between two specified languages (A to B, B to A).

```json
{
  "translation": {
    "type": "two_way",
    "language_a": "en",
    "language_b": "es"
  }
}
```

Translation tokens in the result have `translation_status: "translation"` and lack `start_ms`/`end_ms`. Use `context.translation_terms` for term-level control.

---

## Webhooks

### Configuration

Pass these fields when creating a transcription:

| Field                        | Type   | Description                              |
|------------------------------|--------|------------------------------------------|
| `webhook_url`                | string | Publicly accessible URL for POST callback |
| `webhook_auth_header_name`   | string | Custom HTTP header name (e.g. `Authorization`) |
| `webhook_auth_header_value`  | string | Header value (e.g. `Bearer <secret>`)    |

### Webhook POST Payload

Soniox sends a POST request to your URL when the job completes or fails:

```json
{
  "id": "548d023b-2b3d-4dc2-a3ef-cca26d05fd9a",
  "status": "completed"
}
```

`status` is either `completed` or `error`.

### Metadata via Query Parameters

Encode custom metadata in the webhook URL:

```
https://example.com/webhook?customer_id=1234&order_id=5678
```

### Retry Behavior

- Soniox retries automatically on delivery failure.
- After all retries fail, delivery is permanently failed.
- Always log `transcription_id` as fallback; poll manually if webhook fails.

---

## File Management Endpoints

### Upload File

`POST /v1/files`

**Content-Type**: `multipart/form-data`

| Field                 | Type   | Required | Description                        |
|-----------------------|--------|----------|------------------------------------|
| `file`                | binary | yes      | Audio file to upload               |
| `client_reference_id` | string | no       | Optional tracking ID (max 256 chars) |

**Response** (`201`):

```json
{
  "id": "84c32fc6-4fb5-4e7a-b656-b5ec70493753",
  "filename": "example.mp3",
  "size": 123456,
  "created_at": "2024-11-26T00:00:00Z",
  "client_reference_id": "some_internal_id"
}
```

Max file size: 1,073,741,824 bytes (1 GB).

### Get File

`GET /v1/files/{file_id}`

**Response** (`200`): Same schema as upload response.

### List Files

`GET /v1/files?cursor={cursor}&limit={limit}`

**Response** (`200`):

```json
{
  "files": [
    {"id": "...", "filename": "example.mp3", "size": 123456, "created_at": "..."}
  ],
  "next_page_cursor": "cursor_or_null"
}
```

Paginate by passing `next_page_cursor` as `cursor` in subsequent requests. `null` means no more pages.

### Delete File

`DELETE /v1/files/{file_id}`

**Response**: `204` No Content.

Files are NOT auto-deleted. You must delete them manually after transcription.

---

## Models Endpoint

### List Models

`GET /v1/models`

**Response** (`200`):

```json
{
  "models": [
    {
      "id": "stt-async-v4",
      "name": "Speech-to-Text Async v4",
      "transcription_mode": "async",
      "context_version": 2,
      "aliased_model_id": null,
      "languages": [{"code": "en", "name": "English"}, ...],
      "one_way_translation": "all_languages",
      "two_way_translation": "all_languages",
      "supports_language_hints_strict": true,
      "supports_max_endpoint_delay": false,
      "translation_targets": [],
      "two_way_translation_pairs": []
    }
  ]
}
```

Key async models:

| Model ID              | Description                  | Alias For       |
|-----------------------|------------------------------|-----------------|
| `stt-async-v4`       | Latest async (recommended)   | -               |
| `stt-async-v3`       | Previous stable async        | -               |
| `stt-async-preview`  | Preview alias                | `stt-async-v3`  |

Filter by `transcription_mode: "async"` to find async-capable models.

---

## Temporary API Keys

`POST /v1/auth/temporary-api-key`

Creates short-lived keys for client-side WebSocket connections.

**Request** (`application/json`):

```json
{
  "usage_type": "transcribe_websocket",
  "expires_in_seconds": 60,
  "client_reference_id": "optional"
}
```

| Field                 | Type    | Required | Constraints         |
|-----------------------|---------|----------|---------------------|
| `usage_type`          | string  | yes      | `transcribe_websocket` |
| `expires_in_seconds`  | integer | yes      | 1-3600              |
| `client_reference_id` | string  | no       | max 256 chars       |

**Response** (`201`):

```json
{
  "api_key": "temp:WYJ67RBEFUWQXXPKYPD2UGXKWB",
  "expires_at": "2025-02-22T22:47:37.150Z"
}
```

Note: Temporary keys are for WebSocket real-time use, not async REST endpoints.

---

## Limits and Quotas

### File Limits

| Limit              | Default     | Notes                         |
|--------------------|-------------|-------------------------------|
| Total file storage | 10 GB       | Across all uploaded files     |
| Uploaded files     | 1,000       | Max files stored at once      |
| File duration      | 300 minutes | Cannot be increased           |
| File size          | 1 GB        | Per individual file           |

### Transcription Limits

| Limit                  | Default | Notes                                   |
|------------------------|---------|------------------------------------------|
| Pending transcriptions | 100     | Created but not yet processing           |
| Total transcriptions   | 2,000   | Includes pending + completed + failed    |
| File duration          | 300 min | Cannot be increased                      |

Files are NOT auto-deleted. Delete completed/failed transcriptions and files to stay within quotas.

Limit increases (except duration) can be requested in the [Soniox Console](https://console.soniox.com).

---

## Error Handling

### File Upload Errors

- File duration exceeds 300 minutes.
- Storage or file count quota exceeded.
- File size exceeds 1 GB.
- Recovery: delete old files, request higher limits.

### Transcription Request Errors

- Exceeds 100 pending transcriptions.
- Total transcriptions exceeds 2,000.
- Recovery: wait for pending jobs, delete completed/failed jobs.

### Webhook Delivery Failures

- Server unavailable, timeout, or invalid response.
- Soniox retries automatically; if all fail, delivery is permanently failed.
- Recovery: poll `GET /v1/transcriptions/{id}` manually using the saved transcription ID.

---

## Error Response Schema

All error responses follow this structure:

```json
{
  "status_code": 400,
  "error_type": "invalid_request",
  "message": "Description of the error",
  "validation_errors": [
    {
      "error_type": "...",
      "location": "...",
      "message": "..."
    }
  ],
  "request_id": "req_abc123"
}
```

### HTTP Status Codes

| Code | Meaning               | Common `error_type` Values                        |
|------|-----------------------|---------------------------------------------------|
| 400  | Bad request           | `invalid_request`, `invalid_cursor`               |
| 401  | Authentication error  | Invalid or missing API key                        |
| 402  | Payment required      | Balance or budget exhausted                       |
| 404  | Not found             | `file_not_found`, transcription not found          |
| 429  | Rate limit exceeded   | Concurrent request or rate limit exceeded         |
| 500  | Internal server error | Server-side failure, safe to retry                |
