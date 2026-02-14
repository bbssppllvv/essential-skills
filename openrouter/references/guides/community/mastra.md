# Mastra Integration with OpenRouter

## Overview

The documentation provides a comprehensive guide for integrating OpenRouter with the Mastra framework to access multiple AI models through a unified interface.

## Setup Process

**Project Initialization**: Users begin by creating a new Mastra project using `npx create-mastra@latest`, selecting Agents as the component and OpenAI as the default provider (which will be replaced).

**Environment Configuration**: The `.env.development` file requires an OpenRouter API key to be added: `OPENROUTER_API_KEY=sk-or-your-api-key-here`. The guide recommends removing the OpenAI package and installing `@openrouter/ai-sdk-provider` instead.

**Agent Setup**: Agents are configured in `src/mastra/agents/agent.ts` by importing the OpenRouter provider and creating agents with specific models like Claude 3 Opus.

## Running the Application

After configuration, developers run `npm run dev` to start the development server. This provides:
- REST API at `http://localhost:4111/api/agents/assistant/generate`
- Interactive playground at `http://localhost:4111`

The guide includes a curl example for testing the API endpoint directly.

## Key Features Demonstrated

**Basic Integration**: Simple agent creation using OpenRouter's provider with message generation capabilities.

**Advanced Configuration**: Support for additional options like reasoning parameters and model-specific settings through `extraBody` parameters.

**Multiple Models**: Examples showing how to create separate agents for different providers (Claude and GPT-4) within the same application.

**Provider-Specific Options**: Implementation of provider-specific settings such as Anthropic's cache control features within requests.
