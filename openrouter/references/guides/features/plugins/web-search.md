# Web Search

Add real-time web data to AI model responses.

OpenRouter enables real-time web search capabilities through a model-agnostic plugin system. You can incorporate relevant web search results for any model on OpenRouter by activating and customizing the `web` plugin.

## Basic Usage

### Using the `:online` Shortcut

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

### Equivalent Plugin Configuration

The `:online` shortcut is equivalent to explicitly specifying the web plugin:

```json
{
  "model": "openrouter/auto",
  "plugins": [{ "id": "web" }]
}
```

## Web Search Result Format

Results follow OpenAI's Chat Completion Message schema with annotations:

```json
{
  "message": {
    "role": "assistant",
    "content": "Here's the latest news I found: ...",
    "annotations": [
      {
        "type": "url_citation",
        "url_citation": {
          "url": "https://www.example.com/web-search-result",
          "title": "Title of the web search result",
          "content": "Content of the web search result",
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
      "search_prompt": "Some relevant web results:"
    }
  ]
}
```

## Engine Options

**Native search** (default for supported providers): OpenAI, Anthropic, Perplexity, and xAI models use their built-in capabilities.

**Exa search**: Used for other models or as fallback, employing neural and keyword search combination.

### Force Native Search

```json
{
  "model": "openai/gpt-5.2",
  "plugins": [
    {
      "id": "web",
      "engine": "native"
    }
  ]
}
```

### Force Exa Search

```json
{
  "model": "openai/gpt-5.2",
  "plugins": [
    {
      "id": "web",
      "engine": "exa",
      "max_results": 3
    }
  ]
}
```

## Search Context Configuration

Specify context size for native search:

```json
{
  "model": "openai/gpt-4.1",
  "messages": [
    {
      "role": "user",
      "content": "What are the latest developments in quantum computing?"
    }
  ],
  "web_search_options": {
    "search_context_size": "high"
  }
}
```

## Pricing

- **Exa Search**: $4 per 1,000 results (approximately $0.02 per default 5-result request)
- **Native Search**: Provider-specific pricing applied directly based on search context size (low, medium, high)

Refer to individual provider documentation for native search pricing details.
