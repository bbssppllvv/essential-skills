# OpenRouter PDF Inputs Documentation

## Overview

OpenRouter facilitates PDF processing through its `/api/v1/chat/completions` API endpoint. Users can submit PDFs as either direct URLs or base64-encoded data within message arrays using the file content type. This capability operates across all models available on the platform.

The service supports two submission approaches:
- **Public URL submission**: Directly reference accessible PDFs without local encoding
- **Base64 encoding**: Required for local or private documents

## Processing Engine Options

Three PDF processing engines are available:

1. **Mistral OCR** - Optimized for scanned documents and image-heavy PDFs ($0.003 per 1,000 pages)
2. **PDF Text** - Designed for structured text-based documents (No cost)
3. **Native** - Leverages model-native file capabilities (Charged as input tokens)

The platform defaults to native processing when available, then falls back to PDF Text parsing.

## Implementation Methods

### URL-Based Submission

```typescript
const result = await openRouter.chat.send({
  model: 'anthropic/claude-sonnet-4',
  messages: [{
    role: 'user',
    content: [{
      type: 'text',
      text: 'What are the main points?'
    }, {
      type: 'file',
      file: {
        filename: 'document.pdf',
        fileData: 'https://bitcoin.org/bitcoin.pdf'
      }
    }]
  }],
  plugins: [{
    id: 'file-parser',
    pdf: { engine: 'mistral-ocr' }
  }]
});
```

### Base64 Encoding Approach

Local files require encoding before transmission:

```python
import base64

def encode_pdf(path):
    with open(path, 'rb') as f:
        return base64.b64encode(f.read()).decode('utf-8')

data_url = f"data:application/pdf;base64,{encode_pdf('file.pdf')}"
```

## Cost Optimization

The platform enables annotation reuse to minimize parsing expenses. When initial responses include file annotations, subsequent requests can reference this parsed data rather than reprocessing:

- First request processes and returns annotations
- Follow-up requests include annotations to bypass re-parsing
- Particularly valuable for large documents or OCR-intensive operations

## File Annotation Structure

```typescript
type FileAnnotation = {
  type: 'file';
  file: {
    hash: string;
    name?: string;
    content: ContentPart[]
  }
};
```

Annotations contain a unique hash identifier, optional filename, and parsed content array with text blocks and images as base64 data URLs.
