# Free Models Router Documentation

## Overview

OpenRouter's Free Models Router (`openrouter/free`) provides "zero-cost AI inference" by automatically selecting from available free models that match your request's requirements, such as vision support or tool calling capabilities.

## Key Features

The router intelligently filters models based on needed features and randomly selects from the qualified pool. This approach suits experimentation and learning scenarios where "you want zero-cost inference without worrying about which specific model to use."

## Implementation

Users set their model parameter to `openrouter/free` via the API. The service supports multiple integration methods including TypeScript SDK, fetch requests, Python, and cURL. Response objects include a `model` field identifying which specific model handled the request.

## How Selection Works

The routing process follows five steps: analyzing request requirements, filtering compatible free models, randomly selecting from candidates, forwarding the request, and returning response metadata.

## Available Options

The router draws from OpenRouter's free model pool, including DeepSeek R1, Meta Llama variants, and Alibaba's Qwen family. "Free model availability changes frequently," so users should check the models page for current options.

## Cost

The service carries "no charge" for router usage or requests directed to free models.

## Practical Applications

Ideal uses include prototyping applications, educational exploration, low-volume personal projects, and testing AI capabilities before adopting paid alternatives.

## Constraints

Limitations include variable rate limits, fluctuating model availability, potentially higher latency during peak times, and inability to specify particular models within random selection.

## Alternative Approach

Users preferring specific free models can append `:free` to model names (e.g., `meta-llama/llama-3.2-3b-instruct:free`) or browse the dedicated free models listing.
