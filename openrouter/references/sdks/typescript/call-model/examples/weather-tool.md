# Weather Tool Documentation

## Overview

This guide demonstrates implementing a complete weather tool with external API integration, validation, and error handling using the OpenRouter SDK.

## Setup Requirements

Install dependencies:
```bash
pnpm add @openrouter/sdk zod
```

Configure environment variables for WeatherAPI (free tier available):
```bash
export WEATHER_API_KEY=your_api_key_here
export OPENROUTER_API_KEY=your_openrouter_key
```

## Core Implementation

The basic weather tool uses Zod schemas for input/output validation and integrates with WeatherAPI:

```typescript
import { OpenRouter, tool } from '@openrouter/sdk';
import { z } from 'zod';

const openrouter = new OpenRouter({
  apiKey: process.env.OPENROUTER_API_KEY,
});

const weatherTool = tool({
  name: 'get_weather',
  description: 'Get current weather conditions for any city worldwide',
  inputSchema: z.object({
    city: z.string().describe('City name, e.g., "San Francisco" or "London, UK"'),
    units: z
      .enum(['celsius', 'fahrenheit'])
      .default('celsius')
      .describe('Temperature units'),
  }),
  outputSchema: z.object({
    temperature: z.number(),
    feelsLike: z.number(),
    conditions: z.string(),
    humidity: z.number(),
    windSpeed: z.number(),
    windDirection: z.string(),
    location: z.object({
      name: z.string(),
      region: z.string(),
      country: z.string(),
    }),
  }),
  execute: async ({ city, units }) => {
    const apiKey = process.env.WEATHER_API_KEY;
    if (!apiKey) {
      throw new Error('WEATHER_API_KEY environment variable not set');
    }

    const response = await fetch(
      `https://api.weatherapi.com/v1/current.json?key=${apiKey}&q=${encodeURIComponent(city)}`
    );

    if (!response.ok) {
      if (response.status === 400) {
        throw new Error(`City not found: ${city}`);
      }
      throw new Error(`Weather API error: ${response.status}`);
    }

    const data = await response.json();

    return {
      temperature: units === 'celsius' ? data.current.temp_c : data.current.temp_f,
      feelsLike: units === 'celsius' ? data.current.feelslike_c : data.current.feelslike_f,
      conditions: data.current.condition.text,
      humidity: data.current.humidity,
      windSpeed: data.current.wind_kph,
      windDirection: data.current.wind_dir,
      location: {
        name: data.location.name,
        region: data.location.region,
        country: data.location.country,
      },
    };
  },
});
```

## Basic Usage

Query a single location:

```typescript
const result = openrouter.callModel({
  model: 'openai/gpt-5-nano',
  input: 'What is the weather like in Tokyo?',
  tools: [weatherTool],
});

const text = await result.getText();
console.log(text);
```

## Multiple Cities

The model automatically makes multiple tool calls to compare conditions across locations:

```typescript
const result = openrouter.callModel({
  model: 'openai/gpt-5-nano',
  input: 'Compare the weather in New York and Los Angeles',
  tools: [weatherTool],
});

const text = await result.getText();
```

## Forecast Extension

Add forecast capabilities with a separate tool:

```typescript
const forecastTool = tool({
  name: 'get_forecast',
  description: 'Get weather forecast for the next few days',
  inputSchema: z.object({
    city: z.string().describe('City name'),
    days: z.number().min(1).max(7).default(3).describe('Number of forecast days'),
    units: z.enum(['celsius', 'fahrenheit']).default('celsius'),
  }),
  outputSchema: z.object({
    location: z.string(),
    forecast: z.array(
      z.object({
        date: z.string(),
        maxTemp: z.number(),
        minTemp: z.number(),
        conditions: z.string(),
        chanceOfRain: z.number(),
      })
    ),
  }),
  execute: async ({ city, days, units }) => {
    const apiKey = process.env.WEATHER_API_KEY;
    if (!apiKey) {
      throw new Error('WEATHER_API_KEY environment variable not set');
    }

    const response = await fetch(
      `https://api.weatherapi.com/v1/forecast.json?key=${apiKey}&q=${encodeURIComponent(city)}&days=${days}`
    );

    if (!response.ok) {
      throw new Error(`Weather API error: ${response.status}`);
    }

    const data = await response.json();

    return {
      location: `${data.location.name}, ${data.location.country}`,
      forecast: data.forecast.forecastday.map((day: any) => ({
        date: day.date,
        maxTemp: units === 'celsius' ? day.day.maxtemp_c : day.day.maxtemp_f,
        minTemp: units === 'celsius' ? day.day.mintemp_c : day.day.mintemp_f,
        conditions: day.day.condition.text,
        chanceOfRain: day.day.daily_chance_of_rain,
      })),
    };
  },
});

const result = openrouter.callModel({
  model: 'openai/gpt-5-nano',
  input: 'What is the weather in Paris today and for the next 3 days?',
  tools: [weatherTool, forecastTool],
});
```

## Advanced Error Handling

Implement retry logic with graceful failure handling:

```typescript
const weatherToolWithRetry = tool({
  name: 'get_weather',
  description: 'Get current weather with retry logic',
  inputSchema: z.object({
    city: z.string(),
    units: z.enum(['celsius', 'fahrenheit']).default('celsius'),
  }),
  outputSchema: z.object({
    temperature: z.number(),
    conditions: z.string(),
    error: z.string().optional(),
  }),
  execute: async ({ city, units }) => {
    const maxRetries = 3;
    let lastError: Error | null = null;

    for (let attempt = 1; attempt <= maxRetries; attempt++) {
      try {
        const response = await fetch(
          `https://api.weatherapi.com/v1/current.json?key=${process.env.WEATHER_API_KEY}&q=${encodeURIComponent(city)}`
        );

        if (response.status === 429) {
          await new Promise((resolve) => setTimeout(resolve, 1000 * attempt));
          continue;
        }

        if (!response.ok) {
          throw new Error(`API error: ${response.status}`);
        }

        const data = await response.json();
        return {
          temperature: units === 'celsius' ? data.current.temp_c : data.current.temp_f,
          conditions: data.current.condition.text,
        };
      } catch (error) {
        lastError = error as Error;
      }
    }

    return {
      temperature: 0,
      conditions: 'Unknown',
      error: `Failed after ${maxRetries} attempts: ${lastError?.message}`,
    };
  },
});
```

## Testing Example

Unit tests using Bun's testing framework:

```typescript
import { describe, it, expect, mock } from 'bun:test';

describe('weatherTool', () => {
  it('returns weather data for valid city', async () => {
    global.fetch = mock(() =>
      Promise.resolve({
        ok: true,
        json: () =>
          Promise.resolve({
            current: {
              temp_c: 22,
              temp_f: 72,
              feelslike_c: 24,
              feelslike_f: 75,
              condition: { text: 'Sunny' },
              humidity: 45,
              wind_kph: 10,
              wind_dir: 'NW',
            },
            location: {
              name: 'London',
              region: 'City of London',
              country: 'UK',
            },
          }),
      })
    );

    const result = await weatherTool.function.execute(
      { city: 'London', units: 'celsius' },
      { numberOfTurns: 1 }
    );

    expect(result.temperature).toBe(22);
    expect(result.conditions).toBe('Sunny');
    expect(result.location.name).toBe('London');
  });

  it('handles city not found', async () => {
    global.fetch = mock(() =>
      Promise.resolve({
        ok: false,
        status: 400,
      })
    );

    await expect(
      weatherTool.function.execute(
        { city: 'InvalidCity123', units: 'celsius' },
        { numberOfTurns: 1 }
      )
    ).rejects.toThrow('City not found');
  });
});
```

## Related Resources

- Tools Guide - Fundamental tool creation concepts
- API Reference - Complete type definitions and specifications
