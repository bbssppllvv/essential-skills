# Streamdown

Streaming-optimized markdown renderer for AI applications. Drop-in replacement for `react-markdown`, purpose-built to handle incomplete and unterminated markdown that arrives token-by-token during LLM streaming.

Used in production by Mintlify, Ollama, Supabase, Trigger, Mastra, Cloudflare, ElevenLabs, Upstash, and others.

---

## Table of Contents

1. [Installation and Setup](#installation-and-setup)
2. [Basic Usage](#basic-usage)
3. [Core Props](#core-props)
4. [Plugins](#plugins)
5. [Remend Preprocessor](#remend-preprocessor)
6. [Controls Configuration](#controls-configuration)
7. [Styling and Theming](#styling-and-theming)
8. [Security](#security)
9. [Integration with AI Elements](#integration-with-ai-elements)
10. [Performance](#performance)

---

## Installation and Setup

```bash
npm i streamdown
```

Add the Tailwind CSS source directive in `globals.css` so Tailwind can detect classes used by Streamdown:

```css
@source "../node_modules/streamdown/dist/*.js";
```

---

## Basic Usage

```tsx
import { Streamdown } from 'streamdown';

function ChatMessage({ content, isStreaming }: { content: string; isStreaming: boolean }) {
  return (
    <Streamdown mode="streaming" isAnimating={isStreaming}>
      {content}
    </Streamdown>
  );
}
```

---

## Core Props

| Prop | Type | Default | Description |
|---|---|---|---|
| `children` | `string` | -- | Markdown content to render. |
| `mode` | `"streaming" \| "static"` | `"streaming"` | Operating mode. Use `"streaming"` during active generation, `"static"` for completed content. |
| `isAnimating` | `boolean` | `false` | Whether content is currently streaming. Disables copy buttons while true. |
| `parseIncompleteMarkdown` | `boolean` | `true` | Enable the remend preprocessor to auto-complete unterminated markdown. |
| `normalizeHtmlIndentation` | `boolean` | `false` | Prevent 4+ space indents from being interpreted as code blocks. |
| `animated` | `boolean \| AnimateOptions` | -- | Enable per-word text animation during streaming. |
| `caret` | `"block" \| "circle"` | -- | Display a caret indicator at the end of streaming content. |
| `className` | `string` | -- | CSS class applied to the container element. |
| `shikiTheme` | `[string, string]` | `['github-light', 'github-dark']` | Light and dark themes for syntax-highlighted code blocks. Tuple of `[lightTheme, darkTheme]`. |
| `components` | `object` | -- | Custom React component overrides for rendered markdown elements. |
| `plugins` | `PluginConfig` | -- | Configuration object for code, mermaid, math, and cjk plugins. |
| `rehypePlugins` | `Pluggable[]` | `[rehype-raw, rehype-sanitize, rehype-harden]` | Rehype plugins for HTML processing. |
| `remarkPlugins` | `Pluggable[]` | `[remark-gfm]` | Remark plugins for markdown processing. |
| `allowedTags` | `Record<string, string[]>` | -- | Custom HTML tags to allow through sanitization. |
| `linkSafety` | `LinkSafetyConfig` | `{ enabled: true }` | Link safety modal configuration. Shows a confirmation dialog before navigating to external links. |
| `controls` | `ControlsConfig` | `true` | Enable or configure controls for tables, code blocks, and mermaid diagrams. |
| `allowedElements` | `string[]` | -- | Explicit list of HTML tag names to allow in output. |
| `disallowedElements` | `string[]` | `[]` | HTML tag names to disallow in output. |
| `allowElement` | `(element, index, parent) => boolean` | -- | Custom filter function called for each element. Return `false` to remove. |
| `unwrapDisallowed` | `boolean` | `false` | When true, disallowed elements are replaced with their children instead of being removed entirely. |
| `skipHtml` | `boolean` | `false` | Ignore raw HTML in the markdown source. |
| `urlTransform` | `(url: string) => string` | `defaultUrlTransform` | Transform or validate URLs before rendering. |
| `BlockComponent` | `React.ComponentType` | -- | Custom block-level renderer component. |
| `parseMarkdownIntoBlocksFn` | `(markdown: string) => Block[]` | -- | Custom function to parse markdown into blocks. |

---

## Plugins

Streamdown plugins are tree-shakeable. Install only what you need:

```bash
npm i @streamdown/code @streamdown/mermaid @streamdown/math @streamdown/cjk
```

Each plugin package requires its own `@source` entry in your CSS for Tailwind class detection:

```css
@source "../node_modules/streamdown/dist/*.js";
@source "../node_modules/@streamdown/code/dist/*.js";
@source "../node_modules/@streamdown/mermaid/dist/*.js";
@source "../node_modules/@streamdown/math/dist/*.js";
@source "../node_modules/@streamdown/cjk/dist/*.js";
```

### Using Plugins

```tsx
import { Streamdown } from 'streamdown';
import { code } from '@streamdown/code';
import { mermaid } from '@streamdown/mermaid';
import { math } from '@streamdown/math';
import { cjk } from '@streamdown/cjk';

<Streamdown plugins={{ code, mermaid, math, cjk }}>
  {text}
</Streamdown>
```

### Plugin Descriptions

- **`@streamdown/code`** -- Syntax highlighting with Shiki. Supports light/dark themes via `shikiTheme` prop, copy buttons, and `contentVisibility` optimization.
- **`@streamdown/mermaid`** -- Renders Mermaid diagrams from fenced code blocks. Supports download, copy, fullscreen, and pan/zoom controls.
- **`@streamdown/math`** -- Renders LaTeX math expressions (inline `$...$` and block `$$...$$`) using KaTeX.
- **`@streamdown/cjk`** -- Proper handling of Chinese, Japanese, and Korean text rendering.

---

## Remend Preprocessor

The remend preprocessor auto-detects and completes unterminated markdown during streaming. This is what makes Streamdown handle partial tokens gracefully. Controlled by the `parseIncompleteMarkdown` prop (default: `true`).

### Remend Options

All options default to `true`:

| Option | Description |
|---|---|
| `links` | Complete unterminated `[text](url)` links. |
| `images` | Complete unterminated `![alt](src)` images. |
| `bold` | Complete unterminated `**bold**` markup. |
| `italic` | Complete unterminated `*italic*` markup. |
| `boldItalic` | Complete unterminated `***boldItalic***` markup. |
| `inlineCode` | Complete unterminated `` `code` `` markup. |
| `strikethrough` | Complete unterminated `~~strikethrough~~` markup. |
| `katex` | Complete unterminated `$math$` and `$$math$$` blocks. |
| `setextHeadings` | Handle incomplete setext-style headings. |
| `comparisonOperators` | Handle `<` and `>` that are comparison operators, not HTML. |
| `htmlTags` | Complete unterminated HTML tags. |

### Additional Remend Settings

| Setting | Type | Description |
|---|---|---|
| `linkMode` | `"protocol" \| "text-only"` | Controls how incomplete links are handled. |
| `handlers` | `RemendHandler[]` | Array of custom handlers for extending the preprocessor. |

---

## Controls Configuration

Controls add interactive UI (copy, download, fullscreen) to tables, code blocks, and mermaid diagrams.

```tsx
// Enable all controls (default)
<Streamdown controls={true}>
  {text}
</Streamdown>

// Disable all controls
<Streamdown controls={false}>
  {text}
</Streamdown>

// Fine-grained configuration
<Streamdown
  controls={{
    table: true,
    code: true,
    mermaid: {
      download: true,
      copy: true,
      fullscreen: true,
      panZoom: true,
    },
  }}
>
  {text}
</Streamdown>
```

---

## Styling and Theming

### Code Block Themes

Set light and dark themes for syntax highlighting via the `shikiTheme` prop:

```tsx
<Streamdown shikiTheme={['github-light', 'github-dark']}>
  {text}
</Streamdown>
```

The first value is the light theme, the second is the dark theme.

### Custom Components

Override any rendered element with custom React components:

```tsx
<Streamdown
  components={{
    h1: ({ children }) => <h1 className="text-3xl font-bold">{children}</h1>,
    a: ({ href, children }) => (
      <a href={href} className="text-blue-500 underline" target="_blank" rel="noopener noreferrer">
        {children}
      </a>
    ),
    code: ({ children, className }) => (
      <code className={className}>{children}</code>
    ),
  }}
>
  {text}
</Streamdown>
```

### Streaming Animations

```tsx
// Simple boolean toggle
<Streamdown animated={true} caret="block" isAnimating={isStreaming}>
  {text}
</Streamdown>

// With animation options
<Streamdown animated={{ speed: 50 }} caret="circle" isAnimating={isStreaming}>
  {text}
</Streamdown>
```

---

## Security

Streamdown includes security defaults out of the box:

- **rehype-sanitize** -- Sanitizes HTML output to prevent XSS.
- **rehype-harden** -- Additional hardening of rendered HTML.
- **rehype-raw** -- Processes raw HTML within markdown safely.
- **Link safety modal** -- Enabled by default (`linkSafety: { enabled: true }`). Shows a confirmation dialog before navigating to external links.

### Allowing Custom HTML Tags

```tsx
<Streamdown
  allowedTags={{
    'custom-element': ['data-value', 'class'],
    'another-tag': ['id'],
  }}
>
  {text}
</Streamdown>
```

### Filtering Elements

```tsx
// Allow only specific elements
<Streamdown allowedElements={['p', 'strong', 'em', 'a', 'code', 'pre']}>
  {text}
</Streamdown>

// Disallow specific elements
<Streamdown disallowedElements={['script', 'iframe']} unwrapDisallowed={false}>
  {text}
</Streamdown>

// Custom per-element filter
<Streamdown allowElement={(element, index, parent) => element.tagName !== 'div'}>
  {text}
</Streamdown>
```

---

## Integration with AI Elements

Streamdown powers the `MessageResponse` component in the AI Elements library. `MessageResponse` wraps Streamdown with sensible defaults for:

- GFM (GitHub Flavored Markdown) support
- Math rendering
- Security hardening

```tsx
import { MessageResponse } from '@/components/ai-elements/message';

// MessageResponse uses Streamdown internally
<MessageResponse content={message.content} isStreaming={isStreaming} />
```

When you need more control than `MessageResponse` provides, use Streamdown directly.

---

## Performance

Streamdown is optimized for real-time streaming scenarios:

- **Block-level memoization** -- Only re-renders markdown blocks that have changed, not the entire document.
- **Tree-shakeable plugins** -- Import only the plugins you use. Unused plugins are excluded from the bundle.
- **`contentVisibility` for CodeBlock** -- Uses the CSS `content-visibility` property to skip rendering of off-screen code blocks.
- **Streaming-optimized parsing** -- The remend preprocessor and block parser are designed for incremental updates.

### Throttling with AI SDK

Pair Streamdown with AI SDK's `experimental_throttle` to control update frequency:

```tsx
const { messages } = useChat({
  experimental_throttle: 50, // Update UI at most every 50ms
});

// Then render with Streamdown
<Streamdown mode="streaming" isAnimating={isGenerating}>
  {message.content}
</Streamdown>
```

This reduces React re-renders during fast token streams while maintaining a smooth visual experience.
