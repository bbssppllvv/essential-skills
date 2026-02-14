# Audio Inputs Documentation

## Overview

OpenRouter enables developers to transmit audio files to compatible models through its API. A key requirement is that "Audio files must be **base64-encoded** - direct URLs are not supported for audio content."

## Implementation Details

The API uses the `/api/v1/chat/completions` endpoint with the `input_audio` content type. Only models with audio processing capabilities can handle these requests. Users can identify compatible models by filtering for audio input modality on OpenRouter's Models page.

## Supported Audio Formats

The platform supports multiple audio compression and encoding standards:

- WAV
- MP3
- AIFF
- AAC
- OGG Vorbis
- FLAC
- M4A
- PCM16
- PCM24

## Code Examples Provided

The documentation includes implementation examples in:
- **TypeScript SDK** - Using the official OpenRouter package
- **Python** - Raw HTTP requests with the requests library
- **TypeScript (fetch)** - Native browser/Node.js fetch API

Each example demonstrates the complete workflow: reading an audio file, encoding it to base64, constructing the appropriate request payload, and sending it to the API.

## Important Consideration

"Check your model's documentation to confirm which audio formats it supports. Not all models support all formats."
