---
title: Xcode
description: Using OpenRouter with Apple Intelligence in Xcode
---

# Xcode Integration

Apple Intelligence in Xcode 26 provides built-in AI assistance for coding. By integrating OpenRouter, you can access hundreds of AI models directly in your Xcode development environment, going far beyond the default ChatGPT integration.

This enables access to models from Anthropic, Google, Meta, OpenAI, and many other providers without leaving your development environment.

## Prerequisites

The Apple Intelligence feature in Xcode is currently in Beta and requires:

- **macOS Tahoe 26.0 Beta** or later
- **Xcode 26 beta 4** or later

## Setup Instructions

### Step 1: Access Intelligence Settings

Navigate to **Settings > Intelligence > Add a Model Provider** in macOS system preferences.

### Step 2: Configure OpenRouter Provider

Enter the following details in the "Add a Model Provider" dialog:

| Field | Value |
|---|---|
| **URL** | `https://openrouter.ai/api` |
| **API Key Header** | `Authorization` |
| **API Key** | `Bearer YOUR_API_KEY_HERE` |
| **Description** | `OpenRouter` (or your preferred name) |

> **Note:** Do not include `/v1` at the end of the URL. Use `https://openrouter.ai/api` exactly. This differs from typical direct API calls.

Your API key should start with `sk-or-v1-`. Enter it in the format `Bearer YOUR_API_KEY_HERE`, substituting your actual OpenRouter key.

### Step 3: Browse and Bookmark Models

After configuration, you can view all available models from OpenRouter. Bookmarked models will appear at the top of the list, making them easily accessible from within the pane whenever you need them.

Available providers include:

- Anthropic Claude
- Google Gemini
- Meta Llama
- OpenAI GPT
- And hundreds of additional options

### Step 4: Start Using AI in Xcode

Return to the chat interface and begin chatting with your selected models in Xcode.

## Apple Intelligence Features

Supported capabilities include:

- **Code Completion**: Receive intelligent code suggestions as you type
- **Code Explanation**: Ask questions about code to understand how it works
- **Refactoring Assistance**: Get guidance on improving code structure
- **Documentation Generation**: Generate comments and documentation for your code

## Additional Resources

- [Writing Code with Intelligence in Xcode](https://developer.apple.com/documentation/Xcode/writing-code-with-intelligence-in-xcode) - Apple's official guide
- [OpenRouter Quick Start](https://openrouter.ai/docs/quickstart) - Getting started with OpenRouter
- [Browse OpenRouter Models](https://openrouter.ai/models) - Available models catalog
