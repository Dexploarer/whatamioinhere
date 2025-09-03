
# AI Elements

[AI Elements](https://www.npmjs.com/package/ai-elements) is a component library and custom registry built on top of [shadcn/ui](https://ui.shadcn.com/) to help you build AI-native applications faster. It provides pre-built components like conversations, messages and more.

You can install it with:

<ElementsInstaller />

Here are some basic examples of what you can achieve using components from AI Elements.

<ElementsDemo />

## Components

<ElementHeader name="Actions" path="actions" />

<Preview path="actions" />

<ElementHeader name="Branch" path="branch" />

<Preview path="branch" />

<ElementHeader name="Code Block" path="code-block" />

<Preview path="code-block" />

<ElementHeader name="Conversation" path="conversation" />

<Preview path="conversation" className="p-0" />

<ElementHeader name="Image" path="image" />

<Preview path="image" />

<ElementHeader name="Loader" path="loader" />

<Preview path="loader" />

<ElementHeader name="Message" path="message" />

<Preview path="message" />

<ElementHeader name="Prompt Input" path="prompt-input" />

<Preview path="prompt-input" />

<ElementHeader name="Reasoning" path="reasoning" />

<Preview path="reasoning" />

<ElementHeader name="Response" path="response" />

<Preview path="response" />

<ElementHeader name="Sources" path="sources" />

<Preview path="sources" />

<ElementHeader name="Suggestion" path="suggestion" />

<Preview path="suggestion" />

<ElementHeader name="Task" path="task" />

<Preview path="task" />

<ElementHeader name="Tool" path="tool" />

<Preview path="tool" />

<ElementHeader name="Web Preview" path="web-preview" />

<Preview path="web-preview" />

<ElementHeader name="Inline Citation" path="inline-citation" />

<Preview path="inline-citation" />

View the [source code](https://github.com/vercel/ai-elements) for all components on GitHub.



# Setup

Installing AI Elements is straightforward and can be done in a couple of ways. You can use the dedicated CLI command for the fastest setup, or integrate via the standard shadcn/ui CLI if you’ve already adopted shadcn’s workflow.

<ElementsInstaller />

## Prerequisites

Before installing AI Elements, make sure your environment meets the following requirements:

- [Node.js](https://nodejs.org/en/download/), version 18 or later
- A [Next.js](https://nextjs.org/) project with the [AI SDK](https://ai-sdk.dev/) installed.
- [shadcn/ui](https://ui.shadcn.com/) installed in your project. In practice, this means you have already initialized shadcn in your project (for example by running `npx shadcn@latest init` and configuring Tailwind CSS). AI Elements currently supports only the CSS Variables mode of shadcn/ui for theming.
- We also highly recommend using the [AI Gateway](https://vercel.com/docs/ai-gateway) and adding `AI_GATEWAY_API_KEY` to your `env.local` so you don't have to use an API key from every provider. AI Gateway also gives $5 in usage per month so you can experiment with models. You can obtain an API key [here](https://vercel.com/d?to=%2F%5Bteam%5D%2F%7E%2Fai%2Fapi-keys&title=Get%20your%20AI%20Gateway%20key).

## Installing Components

You can install AI Elements components using either the AI Elements CLI or the shadcn/ui CLI. Both achieve the same result: adding the selected component’s code and any needed dependencies to your project.

The CLI will download the component’s code and integrate it into your project’s directory (usually under your components folder). By default, AI Elements components are added to the `@/components/ai-elements/` directory (or whatever folder you’ve configured in your shadcn components settings).

After running the command, you should see a confirmation in your terminal that the files were added. You can then proceed to use the component in your code.




# Chatbot

The `useChat` hook makes it effortless to create a conversational user interface for your chatbot application. It enables the streaming of chat messages from your AI provider, manages the chat state, and updates the UI automatically as new messages arrive.

To summarize, the `useChat` hook provides the following features:

- **Message Streaming**: All the messages from the AI provider are streamed to the chat UI in real-time.
- **Managed States**: The hook manages the states for input, messages, status, error and more for you.
- **Seamless Integration**: Easily integrate your chat AI into any design or layout with minimal effort.

In this guide, you will learn how to use the `useChat` hook to create a chatbot application with real-time message streaming.
Check out our [chatbot with tools guide](/docs/ai-sdk-ui/chatbot-with-tool-calling) to learn how to use tools in your chatbot.
Let's start with the following example first.

## Example

```tsx filename='app/page.tsx'
'use client';

import { useChat } from '@ai-sdk/react';
import { DefaultChatTransport } from 'ai';
import { useState } from 'react';

export default function Page() {
  const { messages, sendMessage, status } = useChat({
    transport: new DefaultChatTransport({
      api: '/api/chat',
    }),
  });
  const [input, setInput] = useState('');

  return (
    <>
      {messages.map(message => (
        <div key={message.id}>
          {message.role === 'user' ? 'User: ' : 'AI: '}
          {message.parts.map((part, index) =>
            part.type === 'text' ? <span key={index}>{part.text}</span> : null,
          )}
        </div>
      ))}

      <form
        onSubmit={e => {
          e.preventDefault();
          if (input.trim()) {
            sendMessage({ text: input });
            setInput('');
          }
        }}
      >
        <input
          value={input}
          onChange={e => setInput(e.target.value)}
          disabled={status !== 'ready'}
          placeholder="Say something..."
        />
        <button type="submit" disabled={status !== 'ready'}>
          Submit
        </button>
      </form>
    </>
  );
}
```

```ts filename='app/api/chat/route.ts'
import { openai } from '@ai-sdk/openai';
import { convertToModelMessages, streamText, UIMessage } from 'ai';

// Allow streaming responses up to 30 seconds
export const maxDuration = 30;

export async function POST(req: Request) {
  const { messages }: { messages: UIMessage[] } = await req.json();

  const result = streamText({
    model: openai('gpt-4.1'),
    system: 'You are a helpful assistant.',
    messages: convertToModelMessages(messages),
  });

  return result.toUIMessageStreamResponse();
}
```

<Note>
  The UI messages have a new `parts` property that contains the message parts.
  We recommend rendering the messages using the `parts` property instead of the
  `content` property. The parts property supports different message types,
  including text, tool invocation, and tool result, and allows for more flexible
  and complex chat UIs.
</Note>

In the `Page` component, the `useChat` hook will request to your AI provider endpoint whenever the user sends a message using `sendMessage`.
The messages are then streamed back in real-time and displayed in the chat UI.

This enables a seamless chat experience where the user can see the AI response as soon as it is available,
without having to wait for the entire response to be received.

## Customized UI

`useChat` also provides ways to manage the chat message states via code, show status, and update messages without being triggered by user interactions.

### Status

The `useChat` hook returns a `status`. It has the following possible values:

- `submitted`: The message has been sent to the API and we're awaiting the start of the response stream.
- `streaming`: The response is actively streaming in from the API, receiving chunks of data.
- `ready`: The full response has been received and processed; a new user message can be submitted.
- `error`: An error occurred during the API request, preventing successful completion.

You can use `status` for e.g. the following purposes:

- To show a loading spinner while the chatbot is processing the user's message.
- To show a "Stop" button to abort the current message.
- To disable the submit button.

```tsx filename='app/page.tsx' highlight="6,22-29,36"
'use client';

import { useChat } from '@ai-sdk/react';
import { DefaultChatTransport } from 'ai';
import { useState } from 'react';

export default function Page() {
  const { messages, sendMessage, status, stop } = useChat({
    transport: new DefaultChatTransport({
      api: '/api/chat',
    }),
  });
  const [input, setInput] = useState('');

  return (
    <>
      {messages.map(message => (
        <div key={message.id}>
          {message.role === 'user' ? 'User: ' : 'AI: '}
          {message.parts.map((part, index) =>
            part.type === 'text' ? <span key={index}>{part.text}</span> : null,
          )}
        </div>
      ))}

      {(status === 'submitted' || status === 'streaming') && (
        <div>
          {status === 'submitted' && <Spinner />}
          <button type="button" onClick={() => stop()}>
            Stop
          </button>
        </div>
      )}

      <form
        onSubmit={e => {
          e.preventDefault();
          if (input.trim()) {
            sendMessage({ text: input });
            setInput('');
          }
        }}
      >
        <input
          value={input}
          onChange={e => setInput(e.target.value)}
          disabled={status !== 'ready'}
          placeholder="Say something..."
        />
        <button type="submit" disabled={status !== 'ready'}>
          Submit
        </button>
      </form>
    </>
  );
}
```

### Error State

Similarly, the `error` state reflects the error object thrown during the fetch request.
It can be used to display an error message, disable the submit button, or show a retry button:

<Note>
  We recommend showing a generic error message to the user, such as "Something
  went wrong." This is a good practice to avoid leaking information from the
  server.
</Note>

```tsx file="app/page.tsx" highlight="6,20-27,33"
'use client';

import { useChat } from '@ai-sdk/react';
import { DefaultChatTransport } from 'ai';
import { useState } from 'react';

export default function Chat() {
  const { messages, sendMessage, error, reload } = useChat({
    transport: new DefaultChatTransport({
      api: '/api/chat',
    }),
  });
  const [input, setInput] = useState('');

  return (
    <div>
      {messages.map(m => (
        <div key={m.id}>
          {m.role}:{' '}
          {m.parts.map((part, index) =>
            part.type === 'text' ? <span key={index}>{part.text}</span> : null,
          )}
        </div>
      ))}

      {error && (
        <>
          <div>An error occurred.</div>
          <button type="button" onClick={() => reload()}>
            Retry
          </button>
        </>
      )}

      <form
        onSubmit={e => {
          e.preventDefault();
          if (input.trim()) {
            sendMessage({ text: input });
            setInput('');
          }
        }}
      >
        <input
          value={input}
          onChange={e => setInput(e.target.value)}
          disabled={error != null}
        />
      </form>
    </div>
  );
}
```

Please also see the [error handling](/docs/ai-sdk-ui/error-handling) guide for more information.

### Modify messages

Sometimes, you may want to directly modify some existing messages. For example, a delete button can be added to each message to allow users to remove them from the chat history.

The `setMessages` function can help you achieve these tasks:

```tsx
const { messages, setMessages } = useChat()

const handleDelete = (id) => {
  setMessages(messages.filter(message => message.id !== id))
}

return <>
  {messages.map(message => (
    <div key={message.id}>
      {message.role === 'user' ? 'User: ' : 'AI: '}
      {message.parts.map((part, index) => (
        part.type === 'text' ? (
          <span key={index}>{part.text}</span>
        ) : null
      ))}
      <button onClick={() => handleDelete(message.id)}>Delete</button>
    </div>
  ))}
  ...
```

You can think of `messages` and `setMessages` as a pair of `state` and `setState` in React.

### Cancellation and regeneration

It's also a common use case to abort the response message while it's still streaming back from the AI provider. You can do this by calling the `stop` function returned by the `useChat` hook.

```tsx
const { stop, status } = useChat()

return <>
  <button onClick={stop} disabled={!(status === 'streaming' || status === 'submitted')}>Stop</button>
  ...
```

When the user clicks the "Stop" button, the fetch request will be aborted. This avoids consuming unnecessary resources and improves the UX of your chatbot application.

Similarly, you can also request the AI provider to reprocess the last message by calling the `regenerate` function returned by the `useChat` hook:

```tsx
const { regenerate, status } = useChat();

return (
  <>
    <button
      onClick={regenerate}
      disabled={!(status === 'ready' || status === 'error')}
    >
      Regenerate
    </button>
    ...
  </>
);
```

When the user clicks the "Regenerate" button, the AI provider will regenerate the last message and replace the current one correspondingly.

### Throttling UI Updates

<Note>This feature is currently only available for React.</Note>

By default, the `useChat` hook will trigger a render every time a new chunk is received.
You can throttle the UI updates with the `experimental_throttle` option.

```tsx filename="page.tsx" highlight="2-3"
const { messages, ... } = useChat({
  // Throttle the messages and data updates to 50ms:
  experimental_throttle: 50
})
```

## Event Callbacks

`useChat` provides optional event callbacks that you can use to handle different stages of the chatbot lifecycle:

- `onFinish`: Called when the assistant response is completed. The event includes the response message, all messages, and flags for abort, disconnect, and errors.
- `onError`: Called when an error occurs during the fetch request.
- `onData`: Called whenever a data part is received.

These callbacks can be used to trigger additional actions, such as logging, analytics, or custom UI updates.

```tsx
import { UIMessage } from 'ai';

const {
  /* ... */
} = useChat({
  onFinish: ({ message, messages, isAbort, isDisconnect, isError }) => {
    // use information to e.g. update other UI states
  },
  onError: error => {
    console.error('An error occurred:', error);
  },
  onData: data => {
    console.log('Received data part from server:', data);
  },
});
```

It's worth noting that you can abort the processing by throwing an error in the `onData` callback. This will trigger the `onError` callback and stop the message from being appended to the chat UI. This can be useful for handling unexpected responses from the AI provider.

## Request Configuration

### Custom headers, body, and credentials

By default, the `useChat` hook sends a HTTP POST request to the `/api/chat` endpoint with the message list as the request body. You can customize the request in two ways:

#### Hook-Level Configuration (Applied to all requests)

You can configure transport-level options that will be applied to all requests made by the hook:

```tsx
import { useChat } from '@ai-sdk/react';
import { DefaultChatTransport } from 'ai';

const { messages, sendMessage } = useChat({
  transport: new DefaultChatTransport({
    api: '/api/custom-chat',
    headers: {
      Authorization: 'your_token',
    },
    body: {
      user_id: '123',
    },
    credentials: 'same-origin',
  }),
});
```

#### Dynamic Hook-Level Configuration

You can also provide functions that return configuration values. This is useful for authentication tokens that need to be refreshed, or for configuration that depends on runtime conditions:

```tsx
import { useChat } from '@ai-sdk/react';
import { DefaultChatTransport } from 'ai';

const { messages, sendMessage } = useChat({
  transport: new DefaultChatTransport({
    api: '/api/custom-chat',
    headers: () => ({
      Authorization: `Bearer ${getAuthToken()}`,
      'X-User-ID': getCurrentUserId(),
    }),
    body: () => ({
      sessionId: getCurrentSessionId(),
      preferences: getUserPreferences(),
    }),
    credentials: () => 'include',
  }),
});
```

<Note>
  For component state that changes over time, use `useRef` to store the current
  value and reference `ref.current` in your configuration function, or prefer
  request-level options (see next section) for better reliability.
</Note>

#### Request-Level Configuration (Recommended)

<Note>
  **Recommended**: Use request-level options for better flexibility and control.
  Request-level options take precedence over hook-level options and allow you to
  customize each request individually.
</Note>

```tsx
// Pass options as the second parameter to sendMessage
sendMessage(
  { text: input },
  {
    headers: {
      Authorization: 'Bearer token123',
      'X-Custom-Header': 'custom-value',
    },
    body: {
      temperature: 0.7,
      max_tokens: 100,
      user_id: '123',
    },
    metadata: {
      userId: 'user123',
      sessionId: 'session456',
    },
  },
);
```

The request-level options are merged with hook-level options, with request-level options taking precedence. On your server side, you can handle the request with this additional information.

### Setting custom body fields per request

You can configure custom `body` fields on a per-request basis using the second parameter of the `sendMessage` function.
This is useful if you want to pass in additional information to your backend that is not part of the message list.

```tsx filename="app/page.tsx" highlight="20-25"
'use client';

import { useChat } from '@ai-sdk/react';
import { useState } from 'react';

export default function Chat() {
  const { messages, sendMessage } = useChat();
  const [input, setInput] = useState('');

  return (
    <div>
      {messages.map(m => (
        <div key={m.id}>
          {m.role}:{' '}
          {m.parts.map((part, index) =>
            part.type === 'text' ? <span key={index}>{part.text}</span> : null,
          )}
        </div>
      ))}

      <form
        onSubmit={event => {
          event.preventDefault();
          if (input.trim()) {
            sendMessage(
              { text: input },
              {
                body: {
                  customKey: 'customValue',
                },
              },
            );
            setInput('');
          }
        }}
      >
        <input value={input} onChange={e => setInput(e.target.value)} />
      </form>
    </div>
  );
}
```

You can retrieve these custom fields on your server side by destructuring the request body:

```ts filename="app/api/chat/route.ts" highlight="3,4"
export async function POST(req: Request) {
  // Extract additional information ("customKey") from the body of the request:
  const { messages, customKey }: { messages: UIMessage[]; customKey: string } =
    await req.json();
  //...
}
```

## Message Metadata

You can attach custom metadata to messages for tracking information like timestamps, model details, and token usage.

```ts
// Server: Send metadata about the message
return result.toUIMessageStreamResponse({
  messageMetadata: ({ part }) => {
    if (part.type === 'start') {
      return {
        createdAt: Date.now(),
        model: 'gpt-4o',
      };
    }

    if (part.type === 'finish') {
      return {
        totalTokens: part.totalUsage.totalTokens,
      };
    }
  },
});
```

```tsx
// Client: Access metadata via message.metadata
{
  messages.map(message => (
    <div key={message.id}>
      {message.role}:{' '}
      {message.metadata?.createdAt &&
        new Date(message.metadata.createdAt).toLocaleTimeString()}
      {/* Render message content */}
      {message.parts.map((part, index) =>
        part.type === 'text' ? <span key={index}>{part.text}</span> : null,
      )}
      {/* Show token count if available */}
      {message.metadata?.totalTokens && (
        <span>{message.metadata.totalTokens} tokens</span>
      )}
    </div>
  ));
}
```

For complete examples with type safety and advanced use cases, see the [Message Metadata documentation](/docs/ai-sdk-ui/message-metadata).

## Transport Configuration

You can configure custom transport behavior using the `transport` option to customize how messages are sent to your API:

```tsx filename="app/page.tsx"
import { useChat } from '@ai-sdk/react';
import { DefaultChatTransport } from 'ai';

export default function Chat() {
  const { messages, sendMessage } = useChat({
    id: 'my-chat',
    transport: new DefaultChatTransport({
      prepareSendMessagesRequest: ({ id, messages }) => {
        return {
          body: {
            id,
            message: messages[messages.length - 1],
          },
        };
      },
    }),
  });

  // ... rest of your component
}
```

The corresponding API route receives the custom request format:

```ts filename="app/api/chat/route.ts"
export async function POST(req: Request) {
  const { id, message } = await req.json();

  // Load existing messages and add the new one
  const messages = await loadMessages(id);
  messages.push(message);

  const result = streamText({
    model: openai('gpt-4.1'),
    messages: convertToModelMessages(messages),
  });

  return result.toUIMessageStreamResponse();
}
```

### Advanced: Trigger-based routing

For more complex scenarios like message regeneration, you can use trigger-based routing:

```tsx filename="app/page.tsx"
import { useChat } from '@ai-sdk/react';
import { DefaultChatTransport } from 'ai';

export default function Chat() {
  const { messages, sendMessage, regenerate } = useChat({
    id: 'my-chat',
    transport: new DefaultChatTransport({
      prepareSendMessagesRequest: ({ id, messages, trigger, messageId }) => {
        if (trigger === 'submit-user-message') {
          return {
            body: {
              trigger: 'submit-user-message',
              id,
              message: messages[messages.length - 1],
              messageId,
            },
          };
        } else if (trigger === 'regenerate-assistant-message') {
          return {
            body: {
              trigger: 'regenerate-assistant-message',
              id,
              messageId,
            },
          };
        }
        throw new Error(`Unsupported trigger: ${trigger}`);
      },
    }),
  });

  // ... rest of your component
}
```

The corresponding API route would handle different triggers:

```ts filename="app/api/chat/route.ts"
export async function POST(req: Request) {
  const { trigger, id, message, messageId } = await req.json();

  const chat = await readChat(id);
  let messages = chat.messages;

  if (trigger === 'submit-user-message') {
    // Handle new user message
    messages = [...messages, message];
  } else if (trigger === 'regenerate-assistant-message') {
    // Handle message regeneration - remove messages after messageId
    const messageIndex = messages.findIndex(m => m.id === messageId);
    if (messageIndex !== -1) {
      messages = messages.slice(0, messageIndex);
    }
  }

  const result = streamText({
    model: openai('gpt-4.1'),
    messages: convertToModelMessages(messages),
  });

  return result.toUIMessageStreamResponse();
}
```

To learn more about building custom transports, refer to the [Transport API documentation](/docs/ai-sdk-ui/transport).

## Controlling the response stream

With `streamText`, you can control how error messages and usage information are sent back to the client.

### Error Messages

By default, the error message is masked for security reasons.
The default error message is "An error occurred."
You can forward error messages or send your own error message by providing a `getErrorMessage` function:

```ts filename="app/api/chat/route.ts" highlight="13-27"
import { openai } from '@ai-sdk/openai';
import { convertToModelMessages, streamText, UIMessage } from 'ai';

export async function POST(req: Request) {
  const { messages }: { messages: UIMessage[] } = await req.json();

  const result = streamText({
    model: openai('gpt-4.1'),
    messages: convertToModelMessages(messages),
  });

  return result.toUIMessageStreamResponse({
    onError: error => {
      if (error == null) {
        return 'unknown error';
      }

      if (typeof error === 'string') {
        return error;
      }

      if (error instanceof Error) {
        return error.message;
      }

      return JSON.stringify(error);
    },
  });
}
```

### Usage Information

Track token consumption and resource usage with [message metadata](/docs/ai-sdk-ui/message-metadata):

1. Define a custom metadata type with usage fields (optional, for type safety)
2. Attach usage data using `messageMetadata` in your response
3. Display usage metrics in your UI components

Usage data is attached as metadata to messages and becomes available once the model completes its response generation.

```ts
import { openai } from '@ai-sdk/openai';
import {
  convertToModelMessages,
  streamText,
  UIMessage,
  type LanguageModelUsage,
} from 'ai';

// Create a new metadata type (optional for type-safety)
type MyMetadata = {
  totalUsage: LanguageModelUsage;
};

// Create a new custom message type with your own metadata
export type MyUIMessage = UIMessage<MyMetadata>;

export async function POST(req: Request) {
  const { messages }: { messages: MyUIMessage[] } = await req.json();

  const result = streamText({
    model: openai('gpt-4o'),
    messages: convertToModelMessages(messages),
  });

  return result.toUIMessageStreamResponse({
    originalMessages: messages,
    messageMetadata: ({ part }) => {
      // Send total usage when generation is finished
      if (part.type === 'finish') {
        return { totalUsage: part.totalUsage };
      }
    },
  });
}
```

Then, on the client, you can access the message-level metadata.

```tsx
'use client';

import { useChat } from '@ai-sdk/react';
import type { MyUIMessage } from './api/chat/route';
import { DefaultChatTransport } from 'ai';

export default function Chat() {
  // Use custom message type defined on the server (optional for type-safety)
  const { messages } = useChat<MyUIMessage>({
    transport: new DefaultChatTransport({
      api: '/api/chat',
    }),
  });

  return (
    <div className="flex flex-col w-full max-w-md py-24 mx-auto stretch">
      {messages.map(m => (
        <div key={m.id} className="whitespace-pre-wrap">
          {m.role === 'user' ? 'User: ' : 'AI: '}
          {m.parts.map(part => {
            if (part.type === 'text') {
              return part.text;
            }
          })}
          {/* Render usage via metadata */}
          {m.metadata?.totalUsage && (
            <div>Total usage: {m.metadata?.totalUsage.totalTokens} tokens</div>
          )}
        </div>
      ))}
    </div>
  );
}
```

You can also access your metadata from the `onFinish` callback of `useChat`:

```tsx
'use client';

import { useChat } from '@ai-sdk/react';
import type { MyUIMessage } from './api/chat/route';
import { DefaultChatTransport } from 'ai';

export default function Chat() {
  // Use custom message type defined on the server (optional for type-safety)
  const { messages } = useChat<MyUIMessage>({
    transport: new DefaultChatTransport({
      api: '/api/chat',
    }),
    onFinish: ({ message }) => {
      // Access message metadata via onFinish callback
      console.log(message.metadata?.totalUsage);
    },
  });
}
```

### Text Streams

`useChat` can handle plain text streams by setting the `streamProtocol` option to `text`:

```tsx filename="app/page.tsx" highlight="7"
'use client';

import { useChat } from '@ai-sdk/react';
import { TextStreamChatTransport } from 'ai';

export default function Chat() {
  const { messages } = useChat({
    transport: new TextStreamChatTransport({
      api: '/api/chat',
    }),
  });

  return <>...</>;
}
```

This configuration also works with other backend servers that stream plain text.
Check out the [stream protocol guide](/docs/ai-sdk-ui/stream-protocol) for more information.

<Note>
  When using `TextStreamChatTransport`, tool calls, usage information and finish
  reasons are not available.
</Note>

## Reasoning

Some models such as as DeepSeek `deepseek-reasoner`
and Anthropic `claude-3-7-sonnet-20250219` support reasoning tokens.
These tokens are typically sent before the message content.
You can forward them to the client with the `sendReasoning` option:

```ts filename="app/api/chat/route.ts" highlight="13"
import { deepseek } from '@ai-sdk/deepseek';
import { convertToModelMessages, streamText, UIMessage } from 'ai';

export async function POST(req: Request) {
  const { messages }: { messages: UIMessage[] } = await req.json();

  const result = streamText({
    model: deepseek('deepseek-reasoner'),
    messages: convertToModelMessages(messages),
  });

  return result.toUIMessageStreamResponse({
    sendReasoning: true,
  });
}
```

On the client side, you can access the reasoning parts of the message object.

Reasoning parts have a `text` property that contains the reasoning content.

```tsx filename="app/page.tsx"
messages.map(message => (
  <div key={message.id}>
    {message.role === 'user' ? 'User: ' : 'AI: '}
    {message.parts.map((part, index) => {
      // text parts:
      if (part.type === 'text') {
        return <div key={index}>{part.text}</div>;
      }

      // reasoning parts:
      if (part.type === 'reasoning') {
        return <pre key={index}>{part.text}</pre>;
      }
    })}
  </div>
));
```

## Sources

Some providers such as [Perplexity](/providers/ai-sdk-providers/perplexity#sources) and
[Google Generative AI](/providers/ai-sdk-providers/google-generative-ai#sources) include sources in the response.

Currently sources are limited to web pages that ground the response.
You can forward them to the client with the `sendSources` option:

```ts filename="app/api/chat/route.ts" highlight="13"
import { perplexity } from '@ai-sdk/perplexity';
import { convertToModelMessages, streamText, UIMessage } from 'ai';

export async function POST(req: Request) {
  const { messages }: { messages: UIMessage[] } = await req.json();

  const result = streamText({
    model: perplexity('sonar-pro'),
    messages: convertToModelMessages(messages),
  });

  return result.toUIMessageStreamResponse({
    sendSources: true,
  });
}
```

On the client side, you can access source parts of the message object.
There are two types of sources: `source-url` for web pages and `source-document` for documents.
Here is an example that renders both types of sources:

```tsx filename="app/page.tsx"
messages.map(message => (
  <div key={message.id}>
    {message.role === 'user' ? 'User: ' : 'AI: '}

    {/* Render URL sources */}
    {message.parts
      .filter(part => part.type === 'source-url')
      .map(part => (
        <span key={`source-${part.id}`}>
          [
          <a href={part.url} target="_blank">
            {part.title ?? new URL(part.url).hostname}
          </a>
          ]
        </span>
      ))}

    {/* Render document sources */}
    {message.parts
      .filter(part => part.type === 'source-document')
      .map(part => (
        <span key={`source-${part.id}`}>
          [<span>{part.title ?? `Document ${part.id}`}</span>]
        </span>
      ))}
  </div>
));
```

## Image Generation

Some models such as Google `gemini-2.0-flash-exp` support image generation.
When images are generated, they are exposed as files to the client.
On the client side, you can access file parts of the message object
and render them as images.

```tsx filename="app/page.tsx"
messages.map(message => (
  <div key={message.id}>
    {message.role === 'user' ? 'User: ' : 'AI: '}
    {message.parts.map((part, index) => {
      if (part.type === 'text') {
        return <div key={index}>{part.text}</div>;
      } else if (part.type === 'file' && part.mediaType.startsWith('image/')) {
        return <img key={index} src={part.url} alt="Generated image" />;
      }
    })}
  </div>
));
```

## Attachments

The `useChat` hook supports sending file attachments along with a message as well as rendering them on the client. This can be useful for building applications that involve sending images, files, or other media content to the AI provider.

There are two ways to send files with a message: using a `FileList` object from file inputs or using an array of file objects.

### FileList

By using `FileList`, you can send multiple files as attachments along with a message using the file input element. The `useChat` hook will automatically convert them into data URLs and send them to the AI provider.

<Note>
  Currently, only `image/*` and `text/*` content types get automatically
  converted into [multi-modal content
  parts](/docs/foundations/prompts#multi-modal-messages). You will need to
  handle other content types manually.
</Note>

```tsx filename="app/page.tsx"
'use client';

import { useChat } from '@ai-sdk/react';
import { useRef, useState } from 'react';

export default function Page() {
  const { messages, sendMessage, status } = useChat();

  const [input, setInput] = useState('');
  const [files, setFiles] = useState<FileList | undefined>(undefined);
  const fileInputRef = useRef<HTMLInputElement>(null);

  return (
    <div>
      <div>
        {messages.map(message => (
          <div key={message.id}>
            <div>{`${message.role}: `}</div>

            <div>
              {message.parts.map((part, index) => {
                if (part.type === 'text') {
                  return <span key={index}>{part.text}</span>;
                }

                if (
                  part.type === 'file' &&
                  part.mediaType?.startsWith('image/')
                ) {
                  return <img key={index} src={part.url} alt={part.filename} />;
                }

                return null;
              })}
            </div>
          </div>
        ))}
      </div>

      <form
        onSubmit={event => {
          event.preventDefault();
          if (input.trim()) {
            sendMessage({
              text: input,
              files,
            });
            setInput('');
            setFiles(undefined);

            if (fileInputRef.current) {
              fileInputRef.current.value = '';
            }
          }
        }}
      >
        <input
          type="file"
          onChange={event => {
            if (event.target.files) {
              setFiles(event.target.files);
            }
          }}
          multiple
          ref={fileInputRef}
        />
        <input
          value={input}
          placeholder="Send message..."
          onChange={e => setInput(e.target.value)}
          disabled={status !== 'ready'}
        />
      </form>
    </div>
  );
}
```

### File Objects

You can also send files as objects along with a message. This can be useful for sending pre-uploaded files or data URLs.

```tsx filename="app/page.tsx"
'use client';

import { useChat } from '@ai-sdk/react';
import { useState } from 'react';
import { FileUIPart } from 'ai';

export default function Page() {
  const { messages, sendMessage, status } = useChat();

  const [input, setInput] = useState('');
  const [files] = useState<FileUIPart[]>([
    {
      type: 'file',
      filename: 'earth.png',
      mediaType: 'image/png',
      url: 'https://example.com/earth.png',
    },
    {
      type: 'file',
      filename: 'moon.png',
      mediaType: 'image/png',
      url: 'data:image/png;base64,iVBORw0KGgo...',
    },
  ]);

  return (
    <div>
      <div>
        {messages.map(message => (
          <div key={message.id}>
            <div>{`${message.role}: `}</div>

            <div>
              {message.parts.map((part, index) => {
                if (part.type === 'text') {
                  return <span key={index}>{part.text}</span>;
                }

                if (
                  part.type === 'file' &&
                  part.mediaType?.startsWith('image/')
                ) {
                  return <img key={index} src={part.url} alt={part.filename} />;
                }

                return null;
              })}
            </div>
          </div>
        ))}
      </div>

      <form
        onSubmit={event => {
          event.preventDefault();
          if (input.trim()) {
            sendMessage({
              text: input,
              files,
            });
            setInput('');
          }
        }}
      >
        <input
          value={input}
          placeholder="Send message..."
          onChange={e => setInput(e.target.value)}
          disabled={status !== 'ready'}
        />
      </form>
    </div>
  );
}
```

## Type Inference for Tools

When working with tools in TypeScript, AI SDK UI provides type inference helpers to ensure type safety for your tool inputs and outputs.

### InferUITool

The `InferUITool` type helper infers the input and output types of a single tool for use in UI messages:

```tsx
import { InferUITool } from 'ai';
import { z } from 'zod';

const weatherTool = {
  description: 'Get the current weather',
  inputSchema: z.object({
    location: z.string().describe('The city and state'),
  }),
  execute: async ({ location }) => {
    return `The weather in ${location} is sunny.`;
  },
};

// Infer the types from the tool
type WeatherUITool = InferUITool<typeof weatherTool>;
// This creates a type with:
// {
//   input: { location: string };
//   output: string;
// }
```

### InferUITools

The `InferUITools` type helper infers the input and output types of a `ToolSet`:

```tsx
import { InferUITools, ToolSet } from 'ai';
import { z } from 'zod';

const tools = {
  weather: {
    description: 'Get the current weather',
    inputSchema: z.object({
      location: z.string().describe('The city and state'),
    }),
    execute: async ({ location }) => {
      return `The weather in ${location} is sunny.`;
    },
  },
  calculator: {
    description: 'Perform basic arithmetic',
    inputSchema: z.object({
      operation: z.enum(['add', 'subtract', 'multiply', 'divide']),
      a: z.number(),
      b: z.number(),
    }),
    execute: async ({ operation, a, b }) => {
      switch (operation) {
        case 'add':
          return a + b;
        case 'subtract':
          return a - b;
        case 'multiply':
          return a * b;
        case 'divide':
          return a / b;
      }
    },
  },
} satisfies ToolSet;

// Infer the types from the tool set
type MyUITools = InferUITools<typeof tools>;
// This creates a type with:
// {
//   weather: { input: { location: string }; output: string };
//   calculator: { input: { operation: 'add' | 'subtract' | 'multiply' | 'divide'; a: number; b: number }; output: number };
// }
```

### Using Inferred Types

You can use these inferred types to create a custom UIMessage type and pass it to various AI SDK UI functions:

```tsx
import { InferUITools, UIMessage, UIDataTypes } from 'ai';

type MyUITools = InferUITools<typeof tools>;
type MyUIMessage = UIMessage<never, UIDataTypes, MyUITools>;
```

Pass the custom type to `useChat` or `createUIMessageStream`:

```tsx
import { useChat } from '@ai-sdk/react';
import { createUIMessageStream } from 'ai';
import type { MyUIMessage } from './types';

// With useChat
const { messages } = useChat<MyUIMessage>();

// With createUIMessageStream
const stream = createUIMessageStream<MyUIMessage>(/* ... */);
```

This provides full type safety for tool inputs and outputs on the client and server.




# Chatbot Message Persistence

Being able to store and load chat messages is crucial for most AI chatbots.
In this guide, we'll show how to implement message persistence with `useChat` and `streamText`.

<Note>
  This guide does not cover authorization, error handling, or other real-world
  considerations. It is intended to be a simple example of how to implement
  message persistence.
</Note>

## Starting a new chat

When the user navigates to the chat page without providing a chat ID,
we need to create a new chat and redirect to the chat page with the new chat ID.

```tsx filename="app/chat/page.tsx"
import { redirect } from 'next/navigation';
import { createChat } from '@util/chat-store';

export default async function Page() {
  const id = await createChat(); // create a new chat
  redirect(`/chat/${id}`); // redirect to chat page, see below
}
```

Our example chat store implementation uses files to store the chat messages.
In a real-world application, you would use a database or a cloud storage service,
and get the chat ID from the database.
That being said, the function interfaces are designed to be easily replaced with other implementations.

```tsx filename="util/chat-store.ts"
import { generateId } from 'ai';
import { existsSync, mkdirSync } from 'fs';
import { writeFile } from 'fs/promises';
import path from 'path';

export async function createChat(): Promise<string> {
  const id = generateId(); // generate a unique chat ID
  await writeFile(getChatFile(id), '[]'); // create an empty chat file
  return id;
}

function getChatFile(id: string): string {
  const chatDir = path.join(process.cwd(), '.chats');
  if (!existsSync(chatDir)) mkdirSync(chatDir, { recursive: true });
  return path.join(chatDir, `${id}.json`);
}
```

## Loading an existing chat

When the user navigates to the chat page with a chat ID, we need to load the chat messages from storage.

The `loadChat` function in our file-based chat store is implemented as follows:

```tsx filename="util/chat-store.ts"
import { UIMessage } from 'ai';
import { readFile } from 'fs/promises';

export async function loadChat(id: string): Promise<UIMessage[]> {
  return JSON.parse(await readFile(getChatFile(id), 'utf8'));
}

// ... rest of the file
```

## Validating messages on the server

When processing messages on the server that contain tool calls, custom metadata, or data parts, you should validate them using `validateUIMessages` before sending them to the model.

### Validation with tools

When your messages include tool calls, validate them against your tool definitions:

```tsx filename="app/api/chat/route.ts" highlight="7-25,32-37"
import {
  convertToModelMessages,
  streamText,
  UIMessage,
  validateUIMessages,
  tool,
} from 'ai';
import { z } from 'zod';
import { loadChat, saveChat } from '@util/chat-store';
import { openai } from '@ai-sdk/openai';
import { dataPartsSchema, metadataSchema } from '@util/schemas';

// Define your tools
const tools = {
  weather: tool({
    description: 'Get weather information',
    parameters: z.object({
      location: z.string(),
      units: z.enum(['celsius', 'fahrenheit']),
    }),
    execute: async ({ location, units }) => {
      /* tool implementation */
    },
  }),
  // other tools
};

export async function POST(req: Request) {
  const { message, id } = await req.json();

  // Load previous messages from database
  const previousMessages = await loadChat(id);

  // Append new message to previousMessages messages
  const messages = [...previousMessages, message];

  // Validate loaded messages against
  // tools, data parts schema, and metadata schema
  const validatedMessages = await validateUIMessages({
    messages,
    tools, // Ensures tool calls in messages match current schemas
    dataPartsSchema,
    metadataSchema,
  });

  const result = streamText({
    model: openai('gpt-4o-mini'),
    messages: convertToModelMessages(validatedMessages),
    tools,
  });

  return result.toUIMessageStreamResponse({
    originalMessages: messages,
    onFinish: ({ messages }) => {
      saveChat({ chatId: id, messages });
    },
  });
}
```

### Handling validation errors

Handle validation errors gracefully when messages from the database don't match current schemas:

```tsx filename="app/api/chat/route.ts" highlight="3,10-24"
import {
  convertToModelMessages,
  streamText,
  validateUIMessages,
  TypeValidationError,
} from 'ai';
import { type MyUIMessage } from '@/types';

export async function POST(req: Request) {
  const { message, id } = await req.json();

  // Load and validate messages from database
  let validatedMessages: MyUIMessage[];

  try {
    const previousMessages = await loadMessagesFromDB(id);
    validatedMessages = await validateUIMessages({
      // append the new message to the previous messages:
      messages: [...previousMessages, message],
      tools,
      metadataSchema,
    });
  } catch (error) {
    if (error instanceof TypeValidationError) {
      // Log validation error for monitoring
      console.error('Database messages validation failed:', error);
      // Could implement message migration or filtering here
      // For now, start with empty history
      validatedMessages = [];
    } else {
      throw error;
    }
  }

  // Continue with validated messages...
}
```

## Displaying the chat

Once messages are loaded from storage, you can display them in your chat UI. Here's how to set up the page component and the chat display:

```tsx filename="app/chat/[id]/page.tsx"
import { loadChat } from '@util/chat-store';
import Chat from '@ui/chat';

export default async function Page(props: { params: Promise<{ id: string }> }) {
  const { id } = await props.params;
  const messages = await loadChat(id);
  return <Chat id={id} initialMessages={messages} />;
}
```

The chat component uses the `useChat` hook to manage the conversation:

```tsx filename="ui/chat.tsx" highlight="10-16"
'use client';

import { UIMessage, useChat } from '@ai-sdk/react';
import { DefaultChatTransport } from 'ai';
import { useState } from 'react';

export default function Chat({
  id,
  initialMessages,
}: { id?: string | undefined; initialMessages?: UIMessage[] } = {}) {
  const [input, setInput] = useState('');
  const { sendMessage, messages } = useChat({
    id, // use the provided chat ID
    messages: initialMessages, // load initial messages
    transport: new DefaultChatTransport({
      api: '/api/chat',
    }),
  });

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    if (input.trim()) {
      sendMessage({ text: input });
      setInput('');
    }
  };

  // simplified rendering code, extend as needed:
  return (
    <div>
      {messages.map(m => (
        <div key={m.id}>
          {m.role === 'user' ? 'User: ' : 'AI: '}
          {m.parts
            .map(part => (part.type === 'text' ? part.text : ''))
            .join('')}
        </div>
      ))}

      <form onSubmit={handleSubmit}>
        <input
          value={input}
          onChange={e => setInput(e.target.value)}
          placeholder="Type a message..."
        />
        <button type="submit">Send</button>
      </form>
    </div>
  );
}
```

## Storing messages

`useChat` sends the chat id and the messages to the backend.

<Note>
  The `useChat` message format is different from the `ModelMessage` format. The
  `useChat` message format is designed for frontend display, and contains
  additional fields such as `id` and `createdAt`. We recommend storing the
  messages in the `useChat` message format.

When loading messages from storage that contain tools, metadata, or custom data
parts, validate them using `validateUIMessages` before processing (see the
[validation section](#validating-messages-from-database) above).

</Note>

Storing messages is done in the `onFinish` callback of the `toUIMessageStreamResponse` function.
`onFinish` receives the complete messages including the new AI response as `UIMessage[]`.

```tsx filename="app/api/chat/route.ts" highlight="6,11-17"
import { openai } from '@ai-sdk/openai';
import { saveChat } from '@util/chat-store';
import { convertToModelMessages, streamText, UIMessage } from 'ai';

export async function POST(req: Request) {
  const { messages, chatId }: { messages: UIMessage[]; chatId: string } =
    await req.json();

  const result = streamText({
    model: openai('gpt-4o-mini'),
    messages: convertToModelMessages(messages),
  });

  return result.toUIMessageStreamResponse({
    originalMessages: messages,
    onFinish: ({ messages }) => {
      saveChat({ chatId, messages });
    },
  });
}
```

The actual storage of the messages is done in the `saveChat` function, which in
our file-based chat store is implemented as follows:

```tsx filename="util/chat-store.ts"
import { UIMessage } from 'ai';
import { writeFile } from 'fs/promises';

export async function saveChat({
  chatId,
  messages,
}: {
  chatId: string;
  messages: UIMessage[];
}): Promise<void> {
  const content = JSON.stringify(messages, null, 2);
  await writeFile(getChatFile(chatId), content);
}

// ... rest of the file
```

## Message IDs

In addition to a chat ID, each message has an ID.
You can use this message ID to e.g. manipulate individual messages.

### Client-side vs Server-side ID Generation

By default, message IDs are generated client-side:

- User message IDs are generated by the `useChat` hook on the client
- AI response message IDs are generated by `streamText` on the server

For applications without persistence, client-side ID generation works perfectly.
However, **for persistence, you need server-side generated IDs** to ensure consistency across sessions and prevent ID conflicts when messages are stored and retrieved.

### Setting Up Server-side ID Generation

When implementing persistence, you have two options for generating server-side IDs:

1. **Using `generateMessageId` in `toUIMessageStreamResponse`**
2. **Setting IDs in your start message part with `createUIMessageStream`**

#### Option 1: Using `generateMessageId` in `toUIMessageStreamResponse`

You can control the ID format by providing ID generators using [`createIdGenerator()`](/docs/reference/ai-sdk-core/create-id-generator):

```tsx filename="app/api/chat/route.ts" highlight="7-11"
import { createIdGenerator, streamText } from 'ai';

export async function POST(req: Request) {
  // ...
  const result = streamText({
    // ...
  });

  return result.toUIMessageStreamResponse({
    originalMessages: messages,
    // Generate consistent server-side IDs for persistence:
    generateMessageId: createIdGenerator({
      prefix: 'msg',
      size: 16,
    }),
    onFinish: ({ messages }) => {
      saveChat({ chatId, messages });
    },
  });
}
```

#### Option 2: Setting IDs with `createUIMessageStream`

Alternatively, you can use `createUIMessageStream` to control the message ID by writing a start message part:

```tsx filename="app/api/chat/route.ts" highlight="8-18"
import {
  generateId,
  streamText,
  createUIMessageStream,
  createUIMessageStreamResponse,
} from 'ai';

export async function POST(req: Request) {
  const { messages, chatId } = await req.json();

  const stream = createUIMessageStream({
    execute: ({ writer }) => {
      // Write start message part with custom ID
      writer.write({
        type: 'start',
        messageId: generateId(), // Generate server-side ID for persistence
      });

      const result = streamText({
        model: openai('gpt-4o-mini'),
        messages: convertToModelMessages(messages),
      });

      writer.merge(result.toUIMessageStream({ sendStart: false })); // omit start message part
    },
    originalMessages: messages,
    onFinish: ({ responseMessage }) => {
      // save your chat here
    },
  });

  return createUIMessageStreamResponse({ stream });
}
```

<Note>
  For client-side applications that don't require persistence, you can still customize client-side ID generation:

```tsx filename="ui/chat.tsx"
import { createIdGenerator } from 'ai';
import { useChat } from '@ai-sdk/react';

const { ... } = useChat({
  generateId: createIdGenerator({
    prefix: 'msgc',
    size: 16,
  }),
  // ...
});
```

</Note>

## Sending only the last message

Once you have implemented message persistence, you might want to send only the last message to the server.
This reduces the amount of data sent to the server on each request and can improve performance.

To achieve this, you can provide a `prepareSendMessagesRequest` function to the transport.
This function receives the messages and the chat ID, and returns the request body to be sent to the server.

```tsx filename="ui/chat.tsx" highlight="7-12"
import { useChat } from '@ai-sdk/react';
import { DefaultChatTransport } from 'ai';

const {
  // ...
} = useChat({
  // ...
  transport: new DefaultChatTransport({
    api: '/api/chat',
    // only send the last message to the server:
    prepareSendMessagesRequest({ messages, id }) {
      return { body: { message: messages[messages.length - 1], id } };
    },
  }),
});
```

On the server, you can then load the previous messages and append the new message to the previous messages. If your messages contain tools, metadata, or custom data parts, you should validate them:

```tsx filename="app/api/chat/route.ts" highlight="2-11,14-18"
import { convertToModelMessages, UIMessage, validateUIMessages } from 'ai';
// import your tools and schemas

export async function POST(req: Request) {
  // get the last message from the client:
  const { message, id } = await req.json();

  // load the previous messages from the server:
  const previousMessages = await loadChat(id);

  // validate messages if they contain tools, metadata, or data parts:
  const validatedMessages = await validateUIMessages({
    // append the new message to the previous messages:
    messages: [...previousMessages, message],
    tools, // if using tools
    metadataSchema, // if using custom metadata
    dataSchemas, // if using custom data parts
  });

  const result = streamText({
    // ...
    messages: convertToModelMessages(validatedMessages),
  });

  return result.toUIMessageStreamResponse({
    originalMessages: validatedMessages,
    onFinish: ({ messages }) => {
      saveChat({ chatId: id, messages });
    },
  });
}
```

## Handling client disconnects

By default, the AI SDK `streamText` function uses backpressure to the language model provider to prevent
the consumption of tokens that are not yet requested.

However, this means that when the client disconnects, e.g. by closing the browser tab or because of a network issue,
the stream from the LLM will be aborted and the conversation may end up in a broken state.

Assuming that you have a [storage solution](#storing-messages) in place, you can use the `consumeStream` method to consume the stream on the backend,
and then save the result as usual.
`consumeStream` effectively removes the backpressure,
meaning that the result is stored even when the client has already disconnected.

```tsx filename="app/api/chat/route.ts" highlight="19-21"
import { convertToModelMessages, streamText, UIMessage } from 'ai';
import { saveChat } from '@util/chat-store';

export async function POST(req: Request) {
  const { messages, chatId }: { messages: UIMessage[]; chatId: string } =
    await req.json();

  const result = streamText({
    model,
    messages: convertToModelMessages(messages),
  });

  // consume the stream to ensure it runs to completion & triggers onFinish
  // even when the client response is aborted:
  result.consumeStream(); // no await

  return result.toUIMessageStreamResponse({
    originalMessages: messages,
    onFinish: ({ messages }) => {
      saveChat({ chatId, messages });
    },
  });
}
```

When the client reloads the page after a disconnect, the chat will be restored from the storage solution.

<Note>
  In production applications, you would also track the state of the request (in
  progress, complete) in your stored messages and use it on the client to cover
  the case where the client reloads the page after a disconnection, but the
  streaming is not yet complete.
</Note>

For more robust handling of disconnects, you may want to add resumability on disconnects. Check out the [Chatbot Resume Streams](/docs/ai-sdk-ui/chatbot-resume-streams) documentation to learn more.



# Chatbot Message Persistence

Being able to store and load chat messages is crucial for most AI chatbots.
In this guide, we'll show how to implement message persistence with `useChat` and `streamText`.

<Note>
  This guide does not cover authorization, error handling, or other real-world
  considerations. It is intended to be a simple example of how to implement
  message persistence.
</Note>

## Starting a new chat

When the user navigates to the chat page without providing a chat ID,
we need to create a new chat and redirect to the chat page with the new chat ID.

```tsx filename="app/chat/page.tsx"
import { redirect } from 'next/navigation';
import { createChat } from '@util/chat-store';

export default async function Page() {
  const id = await createChat(); // create a new chat
  redirect(`/chat/${id}`); // redirect to chat page, see below
}
```

Our example chat store implementation uses files to store the chat messages.
In a real-world application, you would use a database or a cloud storage service,
and get the chat ID from the database.
That being said, the function interfaces are designed to be easily replaced with other implementations.

```tsx filename="util/chat-store.ts"
import { generateId } from 'ai';
import { existsSync, mkdirSync } from 'fs';
import { writeFile } from 'fs/promises';
import path from 'path';

export async function createChat(): Promise<string> {
  const id = generateId(); // generate a unique chat ID
  await writeFile(getChatFile(id), '[]'); // create an empty chat file
  return id;
}

function getChatFile(id: string): string {
  const chatDir = path.join(process.cwd(), '.chats');
  if (!existsSync(chatDir)) mkdirSync(chatDir, { recursive: true });
  return path.join(chatDir, `${id}.json`);
}
```

## Loading an existing chat

When the user navigates to the chat page with a chat ID, we need to load the chat messages from storage.

The `loadChat` function in our file-based chat store is implemented as follows:

```tsx filename="util/chat-store.ts"
import { UIMessage } from 'ai';
import { readFile } from 'fs/promises';

export async function loadChat(id: string): Promise<UIMessage[]> {
  return JSON.parse(await readFile(getChatFile(id), 'utf8'));
}

// ... rest of the file
```

## Validating messages on the server

When processing messages on the server that contain tool calls, custom metadata, or data parts, you should validate them using `validateUIMessages` before sending them to the model.

### Validation with tools

When your messages include tool calls, validate them against your tool definitions:

```tsx filename="app/api/chat/route.ts" highlight="7-25,32-37"
import {
  convertToModelMessages,
  streamText,
  UIMessage,
  validateUIMessages,
  tool,
} from 'ai';
import { z } from 'zod';
import { loadChat, saveChat } from '@util/chat-store';
import { openai } from '@ai-sdk/openai';
import { dataPartsSchema, metadataSchema } from '@util/schemas';

// Define your tools
const tools = {
  weather: tool({
    description: 'Get weather information',
    parameters: z.object({
      location: z.string(),
      units: z.enum(['celsius', 'fahrenheit']),
    }),
    execute: async ({ location, units }) => {
      /* tool implementation */
    },
  }),
  // other tools
};

export async function POST(req: Request) {
  const { message, id } = await req.json();

  // Load previous messages from database
  const previousMessages = await loadChat(id);

  // Append new message to previousMessages messages
  const messages = [...previousMessages, message];

  // Validate loaded messages against
  // tools, data parts schema, and metadata schema
  const validatedMessages = await validateUIMessages({
    messages,
    tools, // Ensures tool calls in messages match current schemas
    dataPartsSchema,
    metadataSchema,
  });

  const result = streamText({
    model: openai('gpt-4o-mini'),
    messages: convertToModelMessages(validatedMessages),
    tools,
  });

  return result.toUIMessageStreamResponse({
    originalMessages: messages,
    onFinish: ({ messages }) => {
      saveChat({ chatId: id, messages });
    },
  });
}
```

### Handling validation errors

Handle validation errors gracefully when messages from the database don't match current schemas:

```tsx filename="app/api/chat/route.ts" highlight="3,10-24"
import {
  convertToModelMessages,
  streamText,
  validateUIMessages,
  TypeValidationError,
} from 'ai';
import { type MyUIMessage } from '@/types';

export async function POST(req: Request) {
  const { message, id } = await req.json();

  // Load and validate messages from database
  let validatedMessages: MyUIMessage[];

  try {
    const previousMessages = await loadMessagesFromDB(id);
    validatedMessages = await validateUIMessages({
      // append the new message to the previous messages:
      messages: [...previousMessages, message],
      tools,
      metadataSchema,
    });
  } catch (error) {
    if (error instanceof TypeValidationError) {
      // Log validation error for monitoring
      console.error('Database messages validation failed:', error);
      // Could implement message migration or filtering here
      // For now, start with empty history
      validatedMessages = [];
    } else {
      throw error;
    }
  }

  // Continue with validated messages...
}
```

## Displaying the chat

Once messages are loaded from storage, you can display them in your chat UI. Here's how to set up the page component and the chat display:

```tsx filename="app/chat/[id]/page.tsx"
import { loadChat } from '@util/chat-store';
import Chat from '@ui/chat';

export default async function Page(props: { params: Promise<{ id: string }> }) {
  const { id } = await props.params;
  const messages = await loadChat(id);
  return <Chat id={id} initialMessages={messages} />;
}
```

The chat component uses the `useChat` hook to manage the conversation:

```tsx filename="ui/chat.tsx" highlight="10-16"
'use client';

import { UIMessage, useChat } from '@ai-sdk/react';
import { DefaultChatTransport } from 'ai';
import { useState } from 'react';

export default function Chat({
  id,
  initialMessages,
}: { id?: string | undefined; initialMessages?: UIMessage[] } = {}) {
  const [input, setInput] = useState('');
  const { sendMessage, messages } = useChat({
    id, // use the provided chat ID
    messages: initialMessages, // load initial messages
    transport: new DefaultChatTransport({
      api: '/api/chat',
    }),
  });

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    if (input.trim()) {
      sendMessage({ text: input });
      setInput('');
    }
  };

  // simplified rendering code, extend as needed:
  return (
    <div>
      {messages.map(m => (
        <div key={m.id}>
          {m.role === 'user' ? 'User: ' : 'AI: '}
          {m.parts
            .map(part => (part.type === 'text' ? part.text : ''))
            .join('')}
        </div>
      ))}

      <form onSubmit={handleSubmit}>
        <input
          value={input}
          onChange={e => setInput(e.target.value)}
          placeholder="Type a message..."
        />
        <button type="submit">Send</button>
      </form>
    </div>
  );
}
```

## Storing messages

`useChat` sends the chat id and the messages to the backend.

<Note>
  The `useChat` message format is different from the `ModelMessage` format. The
  `useChat` message format is designed for frontend display, and contains
  additional fields such as `id` and `createdAt`. We recommend storing the
  messages in the `useChat` message format.

When loading messages from storage that contain tools, metadata, or custom data
parts, validate them using `validateUIMessages` before processing (see the
[validation section](#validating-messages-from-database) above).

</Note>

Storing messages is done in the `onFinish` callback of the `toUIMessageStreamResponse` function.
`onFinish` receives the complete messages including the new AI response as `UIMessage[]`.

```tsx filename="app/api/chat/route.ts" highlight="6,11-17"
import { openai } from '@ai-sdk/openai';
import { saveChat } from '@util/chat-store';
import { convertToModelMessages, streamText, UIMessage } from 'ai';

export async function POST(req: Request) {
  const { messages, chatId }: { messages: UIMessage[]; chatId: string } =
    await req.json();

  const result = streamText({
    model: openai('gpt-4o-mini'),
    messages: convertToModelMessages(messages),
  });

  return result.toUIMessageStreamResponse({
    originalMessages: messages,
    onFinish: ({ messages }) => {
      saveChat({ chatId, messages });
    },
  });
}
```

The actual storage of the messages is done in the `saveChat` function, which in
our file-based chat store is implemented as follows:

```tsx filename="util/chat-store.ts"
import { UIMessage } from 'ai';
import { writeFile } from 'fs/promises';

export async function saveChat({
  chatId,
  messages,
}: {
  chatId: string;
  messages: UIMessage[];
}): Promise<void> {
  const content = JSON.stringify(messages, null, 2);
  await writeFile(getChatFile(chatId), content);
}

// ... rest of the file
```

## Message IDs

In addition to a chat ID, each message has an ID.
You can use this message ID to e.g. manipulate individual messages.

### Client-side vs Server-side ID Generation

By default, message IDs are generated client-side:

- User message IDs are generated by the `useChat` hook on the client
- AI response message IDs are generated by `streamText` on the server

For applications without persistence, client-side ID generation works perfectly.
However, **for persistence, you need server-side generated IDs** to ensure consistency across sessions and prevent ID conflicts when messages are stored and retrieved.

### Setting Up Server-side ID Generation

When implementing persistence, you have two options for generating server-side IDs:

1. **Using `generateMessageId` in `toUIMessageStreamResponse`**
2. **Setting IDs in your start message part with `createUIMessageStream`**

#### Option 1: Using `generateMessageId` in `toUIMessageStreamResponse`

You can control the ID format by providing ID generators using [`createIdGenerator()`](/docs/reference/ai-sdk-core/create-id-generator):

```tsx filename="app/api/chat/route.ts" highlight="7-11"
import { createIdGenerator, streamText } from 'ai';

export async function POST(req: Request) {
  // ...
  const result = streamText({
    // ...
  });

  return result.toUIMessageStreamResponse({
    originalMessages: messages,
    // Generate consistent server-side IDs for persistence:
    generateMessageId: createIdGenerator({
      prefix: 'msg',
      size: 16,
    }),
    onFinish: ({ messages }) => {
      saveChat({ chatId, messages });
    },
  });
}
```

#### Option 2: Setting IDs with `createUIMessageStream`

Alternatively, you can use `createUIMessageStream` to control the message ID by writing a start message part:

```tsx filename="app/api/chat/route.ts" highlight="8-18"
import {
  generateId,
  streamText,
  createUIMessageStream,
  createUIMessageStreamResponse,
} from 'ai';

export async function POST(req: Request) {
  const { messages, chatId } = await req.json();

  const stream = createUIMessageStream({
    execute: ({ writer }) => {
      // Write start message part with custom ID
      writer.write({
        type: 'start',
        messageId: generateId(), // Generate server-side ID for persistence
      });

      const result = streamText({
        model: openai('gpt-4o-mini'),
        messages: convertToModelMessages(messages),
      });

      writer.merge(result.toUIMessageStream({ sendStart: false })); // omit start message part
    },
    originalMessages: messages,
    onFinish: ({ responseMessage }) => {
      // save your chat here
    },
  });

  return createUIMessageStreamResponse({ stream });
}
```

<Note>
  For client-side applications that don't require persistence, you can still customize client-side ID generation:

```tsx filename="ui/chat.tsx"
import { createIdGenerator } from 'ai';
import { useChat } from '@ai-sdk/react';

const { ... } = useChat({
  generateId: createIdGenerator({
    prefix: 'msgc',
    size: 16,
  }),
  // ...
});
```

</Note>

## Sending only the last message

Once you have implemented message persistence, you might want to send only the last message to the server.
This reduces the amount of data sent to the server on each request and can improve performance.

To achieve this, you can provide a `prepareSendMessagesRequest` function to the transport.
This function receives the messages and the chat ID, and returns the request body to be sent to the server.

```tsx filename="ui/chat.tsx" highlight="7-12"
import { useChat } from '@ai-sdk/react';
import { DefaultChatTransport } from 'ai';

const {
  // ...
} = useChat({
  // ...
  transport: new DefaultChatTransport({
    api: '/api/chat',
    // only send the last message to the server:
    prepareSendMessagesRequest({ messages, id }) {
      return { body: { message: messages[messages.length - 1], id } };
    },
  }),
});
```

On the server, you can then load the previous messages and append the new message to the previous messages. If your messages contain tools, metadata, or custom data parts, you should validate them:

```tsx filename="app/api/chat/route.ts" highlight="2-11,14-18"
import { convertToModelMessages, UIMessage, validateUIMessages } from 'ai';
// import your tools and schemas

export async function POST(req: Request) {
  // get the last message from the client:
  const { message, id } = await req.json();

  // load the previous messages from the server:
  const previousMessages = await loadChat(id);

  // validate messages if they contain tools, metadata, or data parts:
  const validatedMessages = await validateUIMessages({
    // append the new message to the previous messages:
    messages: [...previousMessages, message],
    tools, // if using tools
    metadataSchema, // if using custom metadata
    dataSchemas, // if using custom data parts
  });

  const result = streamText({
    // ...
    messages: convertToModelMessages(validatedMessages),
  });

  return result.toUIMessageStreamResponse({
    originalMessages: validatedMessages,
    onFinish: ({ messages }) => {
      saveChat({ chatId: id, messages });
    },
  });
}
```

## Handling client disconnects

By default, the AI SDK `streamText` function uses backpressure to the language model provider to prevent
the consumption of tokens that are not yet requested.

However, this means that when the client disconnects, e.g. by closing the browser tab or because of a network issue,
the stream from the LLM will be aborted and the conversation may end up in a broken state.

Assuming that you have a [storage solution](#storing-messages) in place, you can use the `consumeStream` method to consume the stream on the backend,
and then save the result as usual.
`consumeStream` effectively removes the backpressure,
meaning that the result is stored even when the client has already disconnected.

```tsx filename="app/api/chat/route.ts" highlight="19-21"
import { convertToModelMessages, streamText, UIMessage } from 'ai';
import { saveChat } from '@util/chat-store';

export async function POST(req: Request) {
  const { messages, chatId }: { messages: UIMessage[]; chatId: string } =
    await req.json();

  const result = streamText({
    model,
    messages: convertToModelMessages(messages),
  });

  // consume the stream to ensure it runs to completion & triggers onFinish
  // even when the client response is aborted:
  result.consumeStream(); // no await

  return result.toUIMessageStreamResponse({
    originalMessages: messages,
    onFinish: ({ messages }) => {
      saveChat({ chatId, messages });
    },
  });
}
```

When the client reloads the page after a disconnect, the chat will be restored from the storage solution.

<Note>
  In production applications, you would also track the state of the request (in
  progress, complete) in your stored messages and use it on the client to cover
  the case where the client reloads the page after a disconnection, but the
  streaming is not yet complete.
</Note>

For more robust handling of disconnects, you may want to add resumability on disconnects. Check out the [Chatbot Resume Streams](/docs/ai-sdk-ui/chatbot-resume-streams) documentation to learn more.



# Chatbot Tool Usage

With [`useChat`](/docs/reference/ai-sdk-ui/use-chat) and [`streamText`](/docs/reference/ai-sdk-core/stream-text), you can use tools in your chatbot application.
The AI SDK supports three types of tools in this context:

1. Automatically executed server-side tools
2. Automatically executed client-side tools
3. Tools that require user interaction, such as confirmation dialogs

The flow is as follows:

1. The user enters a message in the chat UI.
1. The message is sent to the API route.
1. In your server side route, the language model generates tool calls during the `streamText` call.
1. All tool calls are forwarded to the client.
1. Server-side tools are executed using their `execute` method and their results are forwarded to the client.
1. Client-side tools that should be automatically executed are handled with the `onToolCall` callback.
   You must call `addToolResult` to provide the tool result.
1. Client-side tool that require user interactions can be displayed in the UI.
   The tool calls and results are available as tool invocation parts in the `parts` property of the last assistant message.
1. When the user interaction is done, `addToolResult` can be used to add the tool result to the chat.
1. The chat can be configured to automatically submit when all tool results are available using `sendAutomaticallyWhen`.
   This triggers another iteration of this flow.

The tool calls and tool executions are integrated into the assistant message as typed tool parts.
A tool part is at first a tool call, and then it becomes a tool result when the tool is executed.
The tool result contains all information about the tool call as well as the result of the tool execution.

<Note>
  Tool result submission can be configured using the `sendAutomaticallyWhen`
  option. You can use the `lastAssistantMessageIsCompleteWithToolCalls` helper
  to automatically submit when all tool results are available. This simplifies
  the client-side code while still allowing full control when needed.
</Note>

## Example

In this example, we'll use three tools:

- `getWeatherInformation`: An automatically executed server-side tool that returns the weather in a given city.
- `askForConfirmation`: A user-interaction client-side tool that asks the user for confirmation.
- `getLocation`: An automatically executed client-side tool that returns a random city.

### API route

```tsx filename='app/api/chat/route.ts'
import { openai } from '@ai-sdk/openai';
import { convertToModelMessages, streamText, UIMessage } from 'ai';
import { z } from 'zod';

// Allow streaming responses up to 30 seconds
export const maxDuration = 30;

export async function POST(req: Request) {
  const { messages }: { messages: UIMessage[] } = await req.json();

  const result = streamText({
    model: openai('gpt-4o'),
    messages: convertToModelMessages(messages),
    tools: {
      // server-side tool with execute function:
      getWeatherInformation: {
        description: 'show the weather in a given city to the user',
        inputSchema: z.object({ city: z.string() }),
        execute: async ({}: { city: string }) => {
          const weatherOptions = ['sunny', 'cloudy', 'rainy', 'snowy', 'windy'];
          return weatherOptions[
            Math.floor(Math.random() * weatherOptions.length)
          ];
        },
      },
      // client-side tool that starts user interaction:
      askForConfirmation: {
        description: 'Ask the user for confirmation.',
        inputSchema: z.object({
          message: z.string().describe('The message to ask for confirmation.'),
        }),
      },
      // client-side tool that is automatically executed on the client:
      getLocation: {
        description:
          'Get the user location. Always ask for confirmation before using this tool.',
        inputSchema: z.object({}),
      },
    },
  });

  return result.toUIMessageStreamResponse();
}
```

### Client-side page

The client-side page uses the `useChat` hook to create a chatbot application with real-time message streaming.
Tool calls are displayed in the chat UI as typed tool parts.
Please make sure to render the messages using the `parts` property of the message.

There are three things worth mentioning:

1. The [`onToolCall`](/docs/reference/ai-sdk-ui/use-chat#on-tool-call) callback is used to handle client-side tools that should be automatically executed.
   In this example, the `getLocation` tool is a client-side tool that returns a random city.
   You call `addToolResult` to provide the result (without `await` to avoid potential deadlocks).

   <Note>
     Always check `if (toolCall.dynamic)` first in your `onToolCall` handler.
     Without this check, TypeScript will throw an error like: `Type 'string' is
     not assignable to type '"toolName1" | "toolName2"'` when you try to use
     `toolCall.toolName` in `addToolResult`.
   </Note>

2. The [`sendAutomaticallyWhen`](/docs/reference/ai-sdk-ui/use-chat#send-automatically-when) option with `lastAssistantMessageIsCompleteWithToolCalls` helper automatically submits when all tool results are available.

3. The `parts` array of assistant messages contains tool parts with typed names like `tool-askForConfirmation`.
   The client-side tool `askForConfirmation` is displayed in the UI.
   It asks the user for confirmation and displays the result once the user confirms or denies the execution.
   The result is added to the chat using `addToolResult` with the `tool` parameter for type safety.

```tsx filename='app/page.tsx' highlight="2,6,10,14-20"
'use client';

import { useChat } from '@ai-sdk/react';
import {
  DefaultChatTransport,
  lastAssistantMessageIsCompleteWithToolCalls,
} from 'ai';
import { useState } from 'react';

export default function Chat() {
  const { messages, sendMessage, addToolResult } = useChat({
    transport: new DefaultChatTransport({
      api: '/api/chat',
    }),

    sendAutomaticallyWhen: lastAssistantMessageIsCompleteWithToolCalls,

    // run client-side tools that are automatically executed:
    async onToolCall({ toolCall }) {
      // Check if it's a dynamic tool first for proper type narrowing
      if (toolCall.dynamic) {
        return;
      }

      if (toolCall.toolName === 'getLocation') {
        const cities = ['New York', 'Los Angeles', 'Chicago', 'San Francisco'];

        // No await - avoids potential deadlocks
        addToolResult({
          tool: 'getLocation',
          toolCallId: toolCall.toolCallId,
          output: cities[Math.floor(Math.random() * cities.length)],
        });
      }
    },
  });
  const [input, setInput] = useState('');

  return (
    <>
      {messages?.map(message => (
        <div key={message.id}>
          <strong>{`${message.role}: `}</strong>
          {message.parts.map(part => {
            switch (part.type) {
              // render text parts as simple text:
              case 'text':
                return part.text;

              // for tool parts, use the typed tool part names:
              case 'tool-askForConfirmation': {
                const callId = part.toolCallId;

                switch (part.state) {
                  case 'input-streaming':
                    return (
                      <div key={callId}>Loading confirmation request...</div>
                    );
                  case 'input-available':
                    return (
                      <div key={callId}>
                        {part.input.message}
                        <div>
                          <button
                            onClick={() =>
                              addToolResult({
                                tool: 'askForConfirmation',
                                toolCallId: callId,
                                output: 'Yes, confirmed.',
                              })
                            }
                          >
                            Yes
                          </button>
                          <button
                            onClick={() =>
                              addToolResult({
                                tool: 'askForConfirmation',
                                toolCallId: callId,
                                output: 'No, denied',
                              })
                            }
                          >
                            No
                          </button>
                        </div>
                      </div>
                    );
                  case 'output-available':
                    return (
                      <div key={callId}>
                        Location access allowed: {part.output}
                      </div>
                    );
                  case 'output-error':
                    return <div key={callId}>Error: {part.errorText}</div>;
                }
                break;
              }

              case 'tool-getLocation': {
                const callId = part.toolCallId;

                switch (part.state) {
                  case 'input-streaming':
                    return (
                      <div key={callId}>Preparing location request...</div>
                    );
                  case 'input-available':
                    return <div key={callId}>Getting location...</div>;
                  case 'output-available':
                    return <div key={callId}>Location: {part.output}</div>;
                  case 'output-error':
                    return (
                      <div key={callId}>
                        Error getting location: {part.errorText}
                      </div>
                    );
                }
                break;
              }

              case 'tool-getWeatherInformation': {
                const callId = part.toolCallId;

                switch (part.state) {
                  // example of pre-rendering streaming tool inputs:
                  case 'input-streaming':
                    return (
                      <pre key={callId}>{JSON.stringify(part, null, 2)}</pre>
                    );
                  case 'input-available':
                    return (
                      <div key={callId}>
                        Getting weather information for {part.input.city}...
                      </div>
                    );
                  case 'output-available':
                    return (
                      <div key={callId}>
                        Weather in {part.input.city}: {part.output}
                      </div>
                    );
                  case 'output-error':
                    return (
                      <div key={callId}>
                        Error getting weather for {part.input.city}:{' '}
                        {part.errorText}
                      </div>
                    );
                }
                break;
              }
            }
          })}
          <br />
        </div>
      ))}

      <form
        onSubmit={e => {
          e.preventDefault();
          if (input.trim()) {
            sendMessage({ text: input });
            setInput('');
          }
        }}
      >
        <input value={input} onChange={e => setInput(e.target.value)} />
      </form>
    </>
  );
}
```

## Dynamic Tools

When using dynamic tools (tools with unknown types at compile time), the UI parts use a generic `dynamic-tool` type instead of specific tool types:

```tsx filename='app/page.tsx'
{
  message.parts.map((part, index) => {
    switch (part.type) {
      // Static tools with specific (`tool-${toolName}`) types
      case 'tool-getWeatherInformation':
        return <WeatherDisplay part={part} />;

      // Dynamic tools use generic `dynamic-tool` type
      case 'dynamic-tool':
        return (
          <div key={index}>
            <h4>Tool: {part.toolName}</h4>
            {part.state === 'input-streaming' && (
              <pre>{JSON.stringify(part.input, null, 2)}</pre>
            )}
            {part.state === 'output-available' && (
              <pre>{JSON.stringify(part.output, null, 2)}</pre>
            )}
            {part.state === 'output-error' && (
              <div>Error: {part.errorText}</div>
            )}
          </div>
        );
    }
  });
}
```

Dynamic tools are useful when integrating with:

- MCP (Model Context Protocol) tools without schemas
- User-defined functions loaded at runtime
- External tool providers

## Tool call streaming

Tool call streaming is **enabled by default** in AI SDK 5.0, allowing you to stream tool calls while they are being generated. This provides a better user experience by showing tool inputs as they are generated in real-time.

```tsx filename='app/api/chat/route.ts'
export async function POST(req: Request) {
  const { messages }: { messages: UIMessage[] } = await req.json();

  const result = streamText({
    model: openai('gpt-4o'),
    messages: convertToModelMessages(messages),
    // toolCallStreaming is enabled by default in v5
    // ...
  });

  return result.toUIMessageStreamResponse();
}
```

With tool call streaming enabled, partial tool calls are streamed as part of the data stream.
They are available through the `useChat` hook.
The typed tool parts of assistant messages will also contain partial tool calls.
You can use the `state` property of the tool part to render the correct UI.

```tsx filename='app/page.tsx' highlight="9,10"
export default function Chat() {
  // ...
  return (
    <>
      {messages?.map(message => (
        <div key={message.id}>
          {message.parts.map(part => {
            switch (part.type) {
              case 'tool-askForConfirmation':
              case 'tool-getLocation':
              case 'tool-getWeatherInformation':
                switch (part.state) {
                  case 'input-streaming':
                    return <pre>{JSON.stringify(part.input, null, 2)}</pre>;
                  case 'input-available':
                    return <pre>{JSON.stringify(part.input, null, 2)}</pre>;
                  case 'output-available':
                    return <pre>{JSON.stringify(part.output, null, 2)}</pre>;
                  case 'output-error':
                    return <div>Error: {part.errorText}</div>;
                }
            }
          })}
        </div>
      ))}
    </>
  );
}
```

## Step start parts

When you are using multi-step tool calls, the AI SDK will add step start parts to the assistant messages.
If you want to display boundaries between tool calls, you can use the `step-start` parts as follows:

```tsx filename='app/page.tsx'
// ...
// where you render the message parts:
message.parts.map((part, index) => {
  switch (part.type) {
    case 'step-start':
      // show step boundaries as horizontal lines:
      return index > 0 ? (
        <div key={index} className="text-gray-500">
          <hr className="my-2 border-gray-300" />
        </div>
      ) : null;
    case 'text':
    // ...
    case 'tool-askForConfirmation':
    case 'tool-getLocation':
    case 'tool-getWeatherInformation':
    // ...
  }
});
// ...
```

## Server-side Multi-Step Calls

You can also use multi-step calls on the server-side with `streamText`.
This works when all invoked tools have an `execute` function on the server side.

```tsx filename='app/api/chat/route.ts' highlight="15-21,24"
import { openai } from '@ai-sdk/openai';
import { convertToModelMessages, streamText, UIMessage, stepCountIs } from 'ai';
import { z } from 'zod';

export async function POST(req: Request) {
  const { messages }: { messages: UIMessage[] } = await req.json();

  const result = streamText({
    model: openai('gpt-4o'),
    messages: convertToModelMessages(messages),
    tools: {
      getWeatherInformation: {
        description: 'show the weather in a given city to the user',
        inputSchema: z.object({ city: z.string() }),
        // tool has execute function:
        execute: async ({}: { city: string }) => {
          const weatherOptions = ['sunny', 'cloudy', 'rainy', 'snowy', 'windy'];
          return weatherOptions[
            Math.floor(Math.random() * weatherOptions.length)
          ];
        },
      },
    },
    stopWhen: stepCountIs(5),
  });

  return result.toUIMessageStreamResponse();
}
```

## Errors

Language models can make errors when calling tools.
By default, these errors are masked for security reasons, and show up as "An error occurred" in the UI.

To surface the errors, you can use the `onError` function when calling `toUIMessageResponse`.

```tsx
export function errorHandler(error: unknown) {
  if (error == null) {
    return 'unknown error';
  }

  if (typeof error === 'string') {
    return error;
  }

  if (error instanceof Error) {
    return error.message;
  }

  return JSON.stringify(error);
}
```

```tsx
const result = streamText({
  // ...
});

return result.toUIMessageStreamResponse({
  onError: errorHandler,
});
```

In case you are using `createUIMessageResponse`, you can use the `onError` function when calling `toUIMessageResponse`:

```tsx
const response = createUIMessageResponse({
  // ...
  async execute(dataStream) {
    // ...
  },
  onError: error => `Custom error: ${error.message}`,
});
```
