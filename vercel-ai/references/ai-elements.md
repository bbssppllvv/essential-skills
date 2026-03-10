# AI Elements Reference

48 production-ready React components for AI-native interfaces, built on shadcn/ui.

**Install**: `npx ai-elements@latest add <component>`
**Docs**: https://elements.ai-sdk.dev/

## Table of Contents
1. [Setup](#setup)
2. [Chat Components](#chat-components)
3. [AI Insight Components](#ai-insight-components)
4. [Code & Dev Components](#code--dev-components)
5. [Voice Components](#voice-components)
6. [Streamdown Markdown Engine](#streamdown-markdown-engine)
7. [Complete Chat Example](#complete-chat-example)
8. [Generative UI Patterns](#generative-ui-patterns)

---

## Setup

**Prerequisites**: React/Next.js project, Tailwind CSS in CSS Variables mode, AI SDK installed.

```bash
# Install components individually
npx ai-elements@latest add message conversation prompt-input reasoning tool code-block

# Or install all via shadcn registry
npx shadcn@latest add https://elements.ai-sdk.dev/api/registry/all.json
```

Components are installed as source code to `@/components/ai-elements/` — you own and can modify them.

---

## Chat Components

### Conversation

Main chat container with auto-scroll and empty state.

```tsx
import {
  Conversation, ConversationContent, ConversationEmptyState, ConversationScrollButton
} from '@/components/ai-elements/conversation';

<Conversation>
  <ConversationContent>
    {messages.length === 0 ? (
      <ConversationEmptyState
        icon={<MessageSquare className="size-12" />}
        title="Start a conversation"
        description="Type a message below"
      />
    ) : (
      messages.map(msg => /* render messages */)
    )}
  </ConversationContent>
  <ConversationScrollButton />
</Conversation>
```

Sub-components: `ConversationContent`, `ConversationEmptyState`, `ConversationScrollButton`, `ConversationDownload`

### Message

Comprehensive message display with role styling.

```tsx
import {
  Message, MessageContent, MessageResponse, MessageActions, MessageBranch,
  MessageBranchContent, MessageBranchNext, MessageBranchPrevious, MessageBranchPage
} from '@/components/ai-elements/message';

<Message from={message.role}>
  <MessageContent>
    <MessageResponse>{part.text}</MessageResponse>
  </MessageContent>
  <MessageActions>
    <CopyButton /><RegenerateButton />
  </MessageActions>
</Message>

// Message branching (response versions)
<MessageBranch defaultBranch={0}>
  <MessageBranchPrevious />
  <MessageBranchPage /> {/* "1 of 3" */}
  <MessageBranchNext />
  <MessageBranchContent>{/* version content */}</MessageBranchContent>
</MessageBranch>
```

**MessageResponse** is optimized for streaming — it handles incremental markdown updates via Streamdown without re-parsing the entire content on each chunk.

### PromptInput

User input with attachments, model selection, and submit.

```tsx
import {
  PromptInput, PromptInputTextarea, PromptInputSubmit,
  PromptInputAttachments, PromptInputActionMenu,
  PromptInputSelect, PromptInputSelectTrigger,
  PromptInputSelectValue, PromptInputSelectContent, PromptInputSelectItem
} from '@/components/ai-elements/prompt-input';

<PromptInput onSubmit={(value) => sendMessage({ text: value })}>
  <PromptInputAttachments />
  <PromptInputTextarea value={input} onChange={(e) => setInput(e.target.value)} placeholder="Message..." />
  <PromptInputActionMenu />
  <PromptInputSelect>
    <PromptInputSelectTrigger><PromptInputSelectValue /></PromptInputSelectTrigger>
    <PromptInputSelectContent>
      <PromptInputSelectItem value="gpt-4o">GPT-4o</PromptInputSelectItem>
      <PromptInputSelectItem value="claude-sonnet">Claude Sonnet</PromptInputSelectItem>
    </PromptInputSelectContent>
  </PromptInputSelect>
  <PromptInputSubmit />
</PromptInput>
```

#### PromptInput Advanced Sub-components

Full list of sub-components for building rich prompt inputs:

**Layout**: `PromptInputHeader`, `PromptInputBody`, `PromptInputFooter`, `PromptInputTools`

**Command palette**:
```tsx
import {
  PromptInputCommand, PromptInputCommandInput, PromptInputCommandList,
  PromptInputCommandEmpty, PromptInputCommandGroup, PromptInputCommandItem
} from '@/components/ai-elements/prompt-input';

<PromptInputCommand>
  <PromptInputCommandInput placeholder="Search commands..." />
  <PromptInputCommandList>
    <PromptInputCommandEmpty>No results</PromptInputCommandEmpty>
    <PromptInputCommandGroup heading="Actions">
      <PromptInputCommandItem onSelect={() => {}}>Search web</PromptInputCommandItem>
    </PromptInputCommandGroup>
  </PromptInputCommandList>
</PromptInputCommand>
```

**Tabs**:
```tsx
<PromptInputTabsList>
  <PromptInputTab value="chat"><PromptInputTabLabel>Chat</PromptInputTabLabel></PromptInputTab>
  <PromptInputTab value="code"><PromptInputTabLabel>Code</PromptInputTabLabel></PromptInputTab>
</PromptInputTabsList>
```

**Hooks**: `usePromptInputAttachments()`, `usePromptInputController()`, `usePromptInputReferencedSources()`

### Attachments

Unified display for files, images, videos, audio, and source documents.

---

## AI Insight Components

### Reasoning

Collapsible panel for AI thinking/reasoning content. Auto-opens during streaming, auto-closes when done.

```tsx
import { Reasoning, ReasoningTrigger, ReasoningContent } from '@/components/ai-elements/reasoning';

<Reasoning>
  <ReasoningTrigger />
  <ReasoningContent>{part.text}</ReasoningContent>
</Reasoning>
```

### Chain of Thought

Visual step-by-step reasoning with search results and images.

```tsx
import {
  ChainOfThought, ChainOfThoughtHeader, ChainOfThoughtContent,
  ChainOfThoughtStep, ChainOfThoughtSearchResults, ChainOfThoughtSearchResult,
  ChainOfThoughtImage
} from '@/components/ai-elements/chain-of-thought';

<ChainOfThought>
  <ChainOfThoughtHeader>Thinking...</ChainOfThoughtHeader>
  <ChainOfThoughtContent>
    <ChainOfThoughtStep label="Searching" description="Looking for info" status="complete" />
    <ChainOfThoughtSearchResults>
      <ChainOfThoughtSearchResult title="Result 1" url="https://..." />
    </ChainOfThoughtSearchResults>
    <ChainOfThoughtStep label="Analyzing" status="in-progress" />
  </ChainOfThoughtContent>
</ChainOfThought>
```

### Tool

Collapsible tool invocation display. Handles the full lifecycle: parameter streaming, execution, completion, error.

```tsx
import { Tool } from '@/components/ai-elements/tool';

// In message parts rendering (v6 — tool parts use dynamic type names):
if (part.type.startsWith('tool-')) {
  return <Tool part={part} />;
}
```

### Confirmation (Tool Approval)

Alert-based tool execution approval workflow with accept/reject states.

```tsx
import {
  Confirmation, ConfirmationRequest, ConfirmationAccepted,
  ConfirmationRejected, ConfirmationActions, ConfirmationAction
} from '@/components/ai-elements/confirmation';

<Confirmation>
  <ConfirmationRequest>
    Tool wants to delete a file. Allow?
    <ConfirmationActions>
      <ConfirmationAction onClick={() => addToolApprovalResponse({ id: part.approval.id, approved: true })}>
        Accept
      </ConfirmationAction>
      <ConfirmationAction onClick={() => addToolApprovalResponse({ id: part.approval.id, approved: false })}>
        Reject
      </ConfirmationAction>
    </ConfirmationActions>
  </ConfirmationRequest>
  <ConfirmationAccepted>Approved</ConfirmationAccepted>
  <ConfirmationRejected>Rejected</ConfirmationRejected>
</Confirmation>
```

### Context (Token Usage & Cost)

Hover card showing context window, token breakdown, and cost estimation via tokenlens.

```tsx
import {
  Context, ContextTrigger, ContextContent, ContextContentHeader,
  ContextContentBody, ContextContentFooter,
  ContextInputUsage, ContextOutputUsage, ContextReasoningUsage, ContextCacheUsage
} from '@/components/ai-elements/context';

<Context modelId="gpt-4o">
  <ContextTrigger />
  <ContextContent>
    <ContextContentHeader>Usage</ContextContentHeader>
    <ContextContentBody>
      <ContextInputUsage /><ContextOutputUsage />
      <ContextReasoningUsage /><ContextCacheUsage />
    </ContextContentBody>
    <ContextContentFooter />
  </ContextContent>
</Context>
```

### Other Insight Components

- **Sources** — Collapsible "Used X sources" with citation links
- **InlineCitation** — Hoverable citation pill with source details. Sub-components: `InlineCitationCard`, `InlineCitationCardTrigger`, `InlineCitationCardBody`, `InlineCitationCarousel`, `InlineCitationSource`, `InlineCitationQuote`
- **Image** — AI-generated image display (accepts `Experimental_GeneratedImage`)
- **Task** — Collapsible task list with status indicators
- **Checkpoint** — Mark and restore conversation states
- **Plan** — Step-by-step plans display
- **Queue** — Queued items display

---

## Code & Dev Components

### CodeBlock

Syntax highlighting with Shiki, line numbers, copy button, auto light/dark theme.

```tsx
import {
  CodeBlock, CodeBlockHeader, CodeBlockTitle, CodeBlockActions,
  CodeBlockCopyButton, CodeBlockFilename
} from '@/components/ai-elements/code-block';

<CodeBlock language="typescript">
  <CodeBlockHeader>
    <CodeBlockFilename>app.ts</CodeBlockFilename>
    <CodeBlockActions><CodeBlockCopyButton /></CodeBlockActions>
  </CodeBlockHeader>
  {`const greeting = "Hello";`}
</CodeBlock>
```

### Other Dev Components

- **Sandbox** — Collapsible code + execution output with tabs
- **Terminal** — Console output with ANSI color support, streaming, auto-scroll
- **FileTree** — Hierarchical file/folder structure with expand/collapse
- **StackTrace** — Formatted error traces with clickable paths
- **Snippet** — Lightweight inline code
- **PackageInfo** — Package version change badges
- **Commit** — Git commit information
- **WebPreview** — Web content preview with navigation
- **JSXPreview** — Dynamic JSX renderer supporting streaming with auto tag completion

---

## Voice Components

### SpeechInput

Voice-to-text using Web Speech API (Chrome/Edge), falls back to MediaRecorder (Firefox/Safari).

```tsx
import { SpeechInput } from '@/components/ai-elements/speech-input';

<SpeechInput
  onTranscriptionChange={(text) => setInput(text)}
  onAudioRecorded={async (audioBlob) => {
    // Fallback: send to Whisper API for Firefox/Safari
    const text = await transcribeWithWhisper(audioBlob);
    return text;
  }}
/>
```

### VoiceSelector

Searchable voice selection with metadata (gender, accent, age).

### AudioPlayer

Audio playback for AI-generated speech.

### MicSelector

Microphone selection dropdown for voice input.

### Persona

Animated AI avatar with Rive animations. Supports 6 visual variants and 5 states: idle, listening, thinking, speaking, asleep.

```tsx
import { Persona } from '@/components/ai-elements/persona';

<Persona variant="default" state={isStreaming ? 'speaking' : 'idle'} />
```

### Transcription

Real-time transcription display for voice input.

---

## Agent & Collaboration Components

### Agent

Agent identity and status display for multi-agent UIs.

### Artifact

Rendered output display (code, documents, previews) with edit/copy actions.

### Suggestion

Clickable suggestion chips for guiding user interaction.

### OpenInChat

Button to open content in the main chat interface.

### Canvas & Workflow Components

Built on @xyflow/react for building node-based UIs (agent graphs, flow editors).

```tsx
import { Canvas } from '@/components/ai-elements/canvas';
import { Node } from '@/components/ai-elements/node';
import { Edge, AnimatedEdge, TemporaryEdge } from '@/components/ai-elements/edge';
import { Connection } from '@/components/ai-elements/connection';
import { Controls } from '@/components/ai-elements/controls';
import { Panel } from '@/components/ai-elements/panel';
import { Toolbar } from '@/components/ai-elements/toolbar';

<Canvas>
  <Node id="1" data={{ title: 'Research', description: 'Gather info' }} />
  <Node id="2" data={{ title: 'Analyze', description: 'Process data' }} />
  <AnimatedEdge source="1" target="2" />
  <Controls />
</Canvas>
```

Components: `Canvas` (AI-optimized React Flow wrapper), `Node` (card-based), `Edge` (two types: Temporary dashed, Animated with indicator), `Connection` (bezier curves), `Controls`, `Panel`, `Toolbar`.

---

## Additional Components

### EnvironmentVariables

Display and manage environment variable configuration.

### SchemaDisplay

Visual JSON schema display with type annotations.

### TestResults

Test execution results with pass/fail indicators.

---

## Streamdown Markdown Engine

AI Elements uses [Streamdown](https://streamdown.ai/) for streaming-optimized markdown rendering. It powers the `MessageResponse` component.

```tsx
import { Streamdown } from 'streamdown';
import { code } from '@streamdown/code';
import { mermaid } from '@streamdown/mermaid';
import { math } from '@streamdown/math';

<Streamdown
  animated
  plugins={{ code, mermaid, math }}
  isAnimating={status === 'streaming'}
>
  {part.text}
</Streamdown>
```

> For the complete Streamdown API reference including all props, plugins, remend preprocessor, security configuration, and performance tips, see `references/streamdown.md`.

---

## Complete Chat Example

```tsx
'use client';
import { useChat } from '@ai-sdk/react';
import { Conversation, ConversationContent, ConversationEmptyState, ConversationScrollButton } from '@/components/ai-elements/conversation';
import { Message, MessageContent, MessageResponse, MessageActions } from '@/components/ai-elements/message';
import { PromptInput, PromptInputTextarea, PromptInputSubmit, PromptInputAttachments } from '@/components/ai-elements/prompt-input';
import { Reasoning, ReasoningTrigger, ReasoningContent } from '@/components/ai-elements/reasoning';
import { Tool } from '@/components/ai-elements/tool';
import { Confirmation, ConfirmationRequest, ConfirmationAccepted, ConfirmationRejected, ConfirmationActions, ConfirmationAction } from '@/components/ai-elements/confirmation';
import { MessageSquare } from 'lucide-react';

export default function Chatbot() {
  const { messages, sendMessage, status, input, setInput, addToolApprovalResponse } = useChat();

  return (
    <div className="flex flex-col h-screen max-w-3xl mx-auto">
      <Conversation>
        <ConversationContent>
          {messages.length === 0 ? (
            <ConversationEmptyState icon={<MessageSquare className="size-12" />} title="AI Assistant" />
          ) : (
            messages.map((message) => (
              <Message from={message.role} key={message.id}>
                <MessageContent>
                  {message.parts.map((part, i) => {
                    const key = `${message.id}-${i}`;
                    // v6: tool parts use dynamic types like 'tool-getWeather'
                    if (part.type === 'text') {
                      return <MessageResponse key={key}>{part.text}</MessageResponse>;
                    }
                    if (part.type === 'reasoning') {
                      return <Reasoning key={key}><ReasoningTrigger /><ReasoningContent>{part.text}</ReasoningContent></Reasoning>;
                    }
                    if (part.type.startsWith('tool-')) {
                      if (part.state === 'approval-requested') {
                        return (
                          <Confirmation key={key}>
                            <ConfirmationRequest>
                              Allow tool execution?
                              <ConfirmationActions>
                                <ConfirmationAction onClick={() => addToolApprovalResponse({ id: part.approval.id, approved: true })}>Accept</ConfirmationAction>
                                <ConfirmationAction onClick={() => addToolApprovalResponse({ id: part.approval.id, approved: false })}>Reject</ConfirmationAction>
                              </ConfirmationActions>
                            </ConfirmationRequest>
                            <ConfirmationAccepted>Approved</ConfirmationAccepted>
                            <ConfirmationRejected>Rejected</ConfirmationRejected>
                          </Confirmation>
                        );
                      }
                      return <Tool key={key} part={part} />;
                    }
                    return null;
                  })}
                </MessageContent>
              </Message>
            ))
          )}
        </ConversationContent>
        <ConversationScrollButton />
      </Conversation>
      <PromptInput onSubmit={(v) => sendMessage({ text: v })}>
        <PromptInputAttachments />
        <PromptInputTextarea value={input} onChange={(e) => setInput(e.target.value)} placeholder="Message..." />
        <PromptInputSubmit />
      </PromptInput>
    </div>
  );
}
```

---

## Generative UI Patterns

Instead of the generic `<Tool>` component, render custom UIs per tool:

```tsx
// v6: tool parts have dynamic type names like 'tool-getWeather'
if (part.type.startsWith('tool-')) {
  if (part.type === 'tool-getWeather' && part.state === 'output-available') {
    return <WeatherCard data={part.output} />;
  }
  if (part.type === 'tool-generateChart' && part.state === 'output-available') {
    return <Chart data={part.output} />;
  }
  return <Tool part={part} />; // Default: loading state or generic display
}
```

Built-in example apps at:
- https://elements.ai-sdk.dev/examples/chatbot
- https://elements.ai-sdk.dev/examples/ide
- https://elements.ai-sdk.dev/examples/v0
