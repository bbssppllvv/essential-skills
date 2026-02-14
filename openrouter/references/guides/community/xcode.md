# Xcode Integration with OpenRouter Apple Intelligence

## Overview

OpenRouter enables developers to integrate hundreds of AI models directly into Xcode 26's Apple Intelligence features. This setup provides access to models from Anthropic, Google, Meta, OpenAI, and other providers without leaving the development environment.

## Requirements

Apple Intelligence in Xcode is currently in beta and requires:
- macOS Tahoe 26.0 Beta or later
- Xcode 26 beta 4 or later

## Configuration Steps

### Step 1: Access Intelligence Settings
Navigate to **Settings > Intelligence > Add a Model Provider** in macOS system preferences.

### Step 2: Configure OpenRouter Provider
Enter these details in the "Add a Model Provider" dialog:

| Field | Value |
|-------|-------|
| URL | `https://openrouter.ai/api` |
| API Key Header | `Authorization` |
| API Key | `Bearer YOUR_API_KEY_HERE` |
| Description | `OpenRouter` |

**Note:** Omit `/v1` from the endpoint URL, which differs from typical direct API calls.

### Step 3: Browse and Bookmark Models
After configuration, access the model list through OpenRouter. The platform offers hundreds of options; bookmarking favorites places them at the top for quick access.

### Step 4: Begin Development
Open the chat interface and start using your selected models within Xcode.

## Available Capabilities

The integration supports:
- Intelligent code completion suggestions
- Code explanation and analysis
- Refactoring assistance
- Automated documentation generation

## Additional Resources

- [Apple's Intelligence Guide](https://developer.apple.com/documentation/Xcode/writing-code-with-intelligence-in-xcode)
- [OpenRouter Quick Start](https://openrouter.ai/docs/quickstart)
- [Model Browser](https://openrouter.ai/models)
