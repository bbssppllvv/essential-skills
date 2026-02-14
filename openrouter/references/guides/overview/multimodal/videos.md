# OpenRouter Video Inputs Documentation

## Overview

OpenRouter enables sending video files to compatible models through the API using two primary methods: direct URLs for publicly accessible content and base64-encoded data URLs for local or private videos.

## Supported Input Methods

**Direct URLs**: Recommended for publicly hosted videos as they avoid local encoding overhead.

**Base64 Data URLs**: Required for local files or content requiring privacy protection.

**Important limitation**: "Video inputs are currently only supported via the API. Video uploads are not available in the OpenRouter chatroom interface."

## Implementation Examples

### URL-Based Video Input

The API accepts video content via the `/api/v1/chat/completions` endpoint with a `video_url` content type. For Google Gemini on AI Studio, only YouTube links are supported.

Basic request structure includes:
- Message role and content array
- Text prompt component
- Video URL component specifying the video location

### Base64 Encoding Approach

Local videos require encoding as data URLs in the format `data:video/[format];base64,[encoded-content]`. Implementation involves reading the file, encoding to base64, and including in the content array.

## Technical Specifications

**Supported formats**: MP4, MPEG, MOV, WebM

**Content type**: `video_url` with nested `url` parameter

## Provider Considerations

Google Gemini's video support varies by deployment:
- **AI Studio**: YouTube links only
- **Vertex AI**: Base64 data URLs required (no URL support)

## Performance Optimization

Recommendations for managing large files:
- Compress videos when feasible
- Trim to relevant segments only
- Consider 720p resolution over 4K for most tasks
- Evaluate frame rate requirements

## Common Applications

Video analysis capabilities include summarization, object recognition, scene understanding, sports analysis, surveillance monitoring, and educational content assessment.

## Troubleshooting Checklist

Verify model supports video input via `input_modalities`. Confirm provider compatibility with video URLs. Test with base64 encoding if URL methods fail. Validate video format and file integrity.
