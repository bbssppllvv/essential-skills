# Web Search Documentation

## Overview

OpenRouter enables real-time web search integration across any model through the web plugin. You can activate this feature by appending `:online` to your model slug or explicitly configuring the `web` plugin.

## Basic Usage

The simplest approach is using the `:online` suffix:

```json
{
  "model": "openai/gpt-5.2:online"
}
```

This works with free model variants too:

```json
{
  "model": "openai/gpt-oss-20b:free:online"
}
```

The `:online` shortcut is equivalent to explicitly specifying the web plugin with `"routers/auto"` as the model.

## Search Result Formatting

Web search results follow OpenAI's Chat Completion Message standard and include URL citations with annotations:

```json
{
  "message": {
    "role": "assistant",
    "content": "Here's the latest news...",
    "annotations": [
      {
        "type": "url_citation",
        "url_citation": {
          "url": "https://example.com",
          "title": "Result Title",
          "content": "Result content",
          "start_index": 100,
          "end_index": 200
        }
      }
    ]
  }
}
```

## Plugin Customization

Customize search behavior through plugin parameters:

```json
{
  "model": "openai/gpt-5.2:online",
  "plugins": [
    {
      "id": "web",
      "engine": "exa",
      "max_results": 1,
      "search_prompt": "Relevant web results:"
    }
  ]
}
```

## Engine Options

**Native search** (default for supported providers): OpenAI, Anthropic, Perplexity, and xAI models use their built-in capabilities. **Exa search**: Used for other models or as fallback, employing neural and keyword search combination.

Force specific engines:

```json
{
  "model": "openai/gpt-5.2",
  "plugins": [{"id": "web", "engine": "native"}]
}
```

## Pricing Structure

**Exa search**: $4 per 1000 results (approximately $0.02 per default 5-result request)

**Native search**: Provider pricing based on search context size (low, medium, high)

Specify context size:

```json
{
  "model": "openai/gpt-4.1",
  "web_search_options": {"search_context_size": "high"}
}
```

Refer to individual provider documentation for native search pricing details.
