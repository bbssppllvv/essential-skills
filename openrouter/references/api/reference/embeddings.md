# Embeddings API Documentation

## Overview

OpenRouter provides a unified API for generating vector embeddings from text. Embeddings transform text into high-dimensional vectors where semantically similar texts are positioned closer together in vector space.

## What Embeddings Do

The service converts text into numerical representations that capture semantic meaning. Related terms like "cat" and "kitten" receive similar embeddings, while unrelated terms like "cat" and "airplane" are positioned far apart in vector space.

## Common Applications

1. **RAG Systems** - Retrieving relevant context from knowledge bases before generating answers
2. **Semantic Search** - Finding relevant documents by comparing vector similarity rather than keyword matching
3. **Recommendation Systems** - Suggesting similar items based on embedding relationships
4. **Clustering and Classification** - Grouping similar documents by analyzing embedding patterns
5. **Duplicate Detection** - Identifying similar or paraphrased content
6. **Anomaly Detection** - Spotting unusual content that deviates from typical patterns

## API Usage

Basic requests send POST calls to `/embeddings` with text input and model selection. The API accepts single strings or arrays of multiple texts for batch processing.

## Best Practices

- Select models based on your performance needs
- Batch multiple texts in single requests
- Cache results
- Use cosine similarity for comparisons
- Respect each model's context length limitations

## Key Limitations

- Does not support streaming
- Has token limits per model
- Produces deterministic outputs
- May have varying language support across different embedding models
