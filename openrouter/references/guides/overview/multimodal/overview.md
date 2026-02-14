# OpenRouter Multimodal Capabilities Documentation

## Overview

OpenRouter's unified API allows developers to submit various content types beyond text, including images, PDFs, audio, and video files to compatible models. This capability enables sophisticated multimodal interactions across diverse applications.

## Supported Input Types

**Images**: Vision models can analyze, describe, and perform optical character recognition on images in multiple formats, supporting both URL and base64 submission methods.

**Image Generation**: Specialized models can create images from text descriptions, producing high-quality visual outputs based on user prompts.

**PDFs**: The platform processes documents through intelligent parsing that extracts text and handles both standard and scanned PDFs.

**Audio**: Speech-capable models can transcribe and analyze audio files in common formats.

**Video**: Video-capable models enable analysis, description, object detection, and action recognition tasks.

## Technical Implementation

All multimodal requests use the `/api/v1/chat/completions` endpoint with the `messages` parameter. Content types are specified in the message content array using:
- `image_url` for images
- `file` for PDFs
- `input_audio` for audio
- `video_url` for video

## Input Format Options

**Public Content**: URLs work efficiently for images, PDFs, and provider-specific video formats (YouTube links support varies).

**Local Files**: Base64 encoding is necessary for local files or private content, formatted as data URIs with appropriate media types.

## Model Selection

OpenRouter automatically filters available models based on request content, ensuring compatibility with required capabilities for each modality type.

## Pricing Structure

Costs vary by modality: images use per-image or token pricing; PDFs involve free extraction with paid OCR options; audio and video incur token-based charges relative to duration and resolution.
