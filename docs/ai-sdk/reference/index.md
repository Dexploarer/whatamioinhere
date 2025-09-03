
# AI SDK RSC

<Note type="warning">
  AI SDK RSC is currently experimental. We recommend using [AI SDK
  UI](/docs/ai-sdk-ui/overview) for production. For guidance on migrating from
  RSC to UI, see our [migration guide](/docs/ai-sdk-rsc/migrating-to-ui).
</Note>

<Note>
  The `@ai-sdk/rsc` package is compatible with frameworks that support React
  Server Components.
</Note>

[React Server Components](https://nextjs.org/docs/app/building-your-application/rendering/server-components) (RSC) allow you to write UI that can be rendered on the server and streamed to the client. RSCs enable [ Server Actions ](https://nextjs.org/docs/app/building-your-application/data-fetching/server-actions-and-mutations#with-client-components), a new way to call server-side code directly from the client just like any other function with end-to-end type-safety. This combination opens the door to a new way of building AI applications, allowing the large language model (LLM) to generate and stream UI directly from the server to the client.

## AI SDK RSC Functions

AI SDK RSC has various functions designed to help you build AI-native applications with React Server Components. These functions:

1. Provide abstractions for building Generative UI applications.
   - [`streamUI`](/docs/reference/ai-sdk-rsc/stream-ui): calls a model and allows it to respond with React Server Components.
   - [`useUIState`](/docs/reference/ai-sdk-rsc/use-ui-state): returns the current UI state and a function to update the UI State (like React's `useState`). UI State is the visual representation of the AI state.
   - [`useAIState`](/docs/reference/ai-sdk-rsc/use-ai-state): returns the current AI state and a function to update the AI State (like React's `useState`). The AI state is intended to contain context and information shared with the AI model, such as system messages, function responses, and other relevant data.
   - [`useActions`](/docs/reference/ai-sdk-rsc/use-actions): provides access to your Server Actions from the client. This is particularly useful for building interfaces that require user interactions with the server.
   - [`createAI`](/docs/reference/ai-sdk-rsc/create-ai): creates a client-server context provider that can be used to wrap parts of your application tree to easily manage both UI and AI states of your application.
2. Make it simple to work with streamable values between the server and client.
   - [`createStreamableValue`](/docs/reference/ai-sdk-rsc/create-streamable-value): creates a stream that sends values from the server to the client. The value can be any serializable data.
   - [`readStreamableValue`](/docs/reference/ai-sdk-rsc/read-streamable-value): reads a streamable value from the client that was originally created using `createStreamableValue`.
   - [`createStreamableUI`](/docs/reference/ai-sdk-rsc/create-streamable-ui): creates a stream that sends UI from the server to the client.
   - [`useStreamableValue`](/docs/reference/ai-sdk-rsc/use-streamable-value): accepts a streamable value created using `createStreamableValue` and returns the current value, error, and pending state.

## Templates

Check out the following templates to see AI SDK RSC in action.

<Templates type="generative-ui" />

## API Reference

Please check out the [AI SDK RSC API Reference](/docs/reference/ai-sdk-rsc) for more details on each function.



import { UIPreviewCard, Card } from '@/components/home/card';
import { EventPlanning } from '@/components/home/event-planning';
import { Searching } from '@/components/home/searching';
import { Weather } from '@/components/home/weather';

# Streaming React Components

<Note type="warning">
  AI SDK RSC is currently experimental. We recommend using [AI SDK
  UI](/docs/ai-sdk-ui/overview) for production. For guidance on migrating from
  RSC to UI, see our [migration guide](/docs/ai-sdk-rsc/migrating-to-ui).
</Note>

The RSC API allows you to stream React components from the server to the client with the [`streamUI`](/docs/reference/ai-sdk-rsc/stream-ui) function. This is useful when you want to go beyond raw text and stream components to the client in real-time.

Similar to [ AI SDK Core ](/docs/ai-sdk-core/overview) APIs (like [ `streamText` ](/docs/reference/ai-sdk-core/stream-text) and [ `streamObject` ](/docs/reference/ai-sdk-core/stream-object)), `streamUI` provides a single function to call a model and allow it to respond with React Server Components.
It supports the same model interfaces as AI SDK Core APIs.

### Concepts

To give the model the ability to respond to a user's prompt with a React component, you can leverage [tools](/docs/ai-sdk-core/tools-and-tool-calling).

<Note>
  Remember, tools are like programs you can give to the model, and the model can
  decide as and when to use based on the context of the conversation.
</Note>

With the `streamUI` function, **you provide tools that return React components**. With the ability to stream components, the model is akin to a dynamic router that is able to understand the user's intention and display relevant UI.

At a high level, the `streamUI` works like other AI SDK Core functions: you can provide the model with a prompt or some conversation history and, optionally, some tools. If the model decides, based on the context of the conversation, to call a tool, it will generate a tool call. The `streamUI` function will then run the respective tool, returning a React component. If the model doesn't have a relevant tool to use, it will return a text generation, which will be passed to the `text` function, for you to handle (render and return as a React component).

<Note>Remember, the `streamUI` function must return a React component. </Note>

```tsx
const result = await streamUI({
  model: openai('gpt-4o'),
  prompt: 'Get the weather for San Francisco',
  text: ({ content }) => <div>{content}</div>,
  tools: {},
});
```

This example calls the `streamUI` function using OpenAI's `gpt-4o` model, passes a prompt, specifies how the model's plain text response (`content`) should be rendered, and then provides an empty object for tools. Even though this example does not define any tools, it will stream the model's response as a `div` rather than plain text.

### Adding A Tool

Using tools with `streamUI` is similar to how you use tools with `generateText` and `streamText`.
A tool is an object that has:

- `description`: a string telling the model what the tool does and when to use it
- `inputSchema`: a Zod schema describing what the tool needs in order to run
- `generate`: an asynchronous function that will be run if the model calls the tool. This must return a React component

Let's expand the previous example to add a tool.

```tsx highlight="6-14"
const result = await streamUI({
  model: openai('gpt-4o'),
  prompt: 'Get the weather for San Francisco',
  text: ({ content }) => <div>{content}</div>,
  tools: {
    getWeather: {
      description: 'Get the weather for a location',
      inputSchema: z.object({ location: z.string() }),
      generate: async function* ({ location }) {
        yield <LoadingComponent />;
        const weather = await getWeather(location);
        return <WeatherComponent weather={weather} location={location} />;
      },
    },
  },
});
```

This tool would be run if the user asks for the weather for their location. If the user hasn't specified a location, the model will ask for it before calling the tool. When the model calls the tool, the generate function will initially return a loading component. This component will show until the awaited call to `getWeather` is resolved, at which point, the model will stream the `<WeatherComponent />` to the user.

<Note>
  Note: This example uses a [ generator function
  ](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function*)
  (`function*`), which allows you to pause its execution and return a value,
  then resume from where it left off on the next call. This is useful for
  handling data streams, as you can fetch and return data from an asynchronous
  source like an API, then resume the function to fetch the next chunk when
  needed. By yielding values one at a time, generator functions enable efficient
  processing of streaming data without blocking the main thread.
</Note>

## Using `streamUI` with Next.js

Let's see how you can use the example above in a Next.js application.

To use `streamUI` in a Next.js application, you will need two things:

1. A Server Action (where you will call `streamUI`)
2. A page to call the Server Action and render the resulting components

### Step 1: Create a Server Action

<Note>
  Server Actions are server-side functions that you can call directly from the
  frontend. For more info, see [the
  documentation](https://nextjs.org/docs/app/building-your-application/data-fetching/server-actions-and-mutations#with-client-components).
</Note>

Create a Server Action at `app/actions.tsx` and add the following code:

```tsx filename="app/actions.tsx"
'use server';

import { streamUI } from '@ai-sdk/rsc';
import { openai } from '@ai-sdk/openai';
import { z } from 'zod';

const LoadingComponent = () => (
  <div className="animate-pulse p-4">getting weather...</div>
);

const getWeather = async (location: string) => {
  await new Promise(resolve => setTimeout(resolve, 2000));
  return '82°F️ ☀️';
};

interface WeatherProps {
  location: string;
  weather: string;
}

const WeatherComponent = (props: WeatherProps) => (
  <div className="border border-neutral-200 p-4 rounded-lg max-w-fit">
    The weather in {props.location} is {props.weather}
  </div>
);

export async function streamComponent() {
  const result = await streamUI({
    model: openai('gpt-4o'),
    prompt: 'Get the weather for San Francisco',
    text: ({ content }) => <div>{content}</div>,
    tools: {
      getWeather: {
        description: 'Get the weather for a location',
        inputSchema: z.object({
          location: z.string(),
        }),
        generate: async function* ({ location }) {
          yield <LoadingComponent />;
          const weather = await getWeather(location);
          return <WeatherComponent weather={weather} location={location} />;
        },
      },
    },
  });

  return result.value;
}
```

The `getWeather` tool should look familiar as it is identical to the example in the previous section. In order for this tool to work:

1. First define a `LoadingComponent`, which renders a pulsing `div` that will show some loading text.
2. Next, define a `getWeather` function that will timeout for 2 seconds (to simulate fetching the weather externally) before returning the "weather" for a `location`. Note: you could run any asynchronous TypeScript code here.
3. Finally, define a `WeatherComponent` which takes in `location` and `weather` as props, which are then rendered within a `div`.

Your Server Action is an asynchronous function called `streamComponent` that takes no inputs, and returns a `ReactNode`. Within the action, you call the `streamUI` function, specifying the model (`gpt-4o`), the prompt, the component that should be rendered if the model chooses to return text, and finally, your `getWeather` tool. Last but not least, you return the resulting component generated by the model with `result.value`.

To call this Server Action and display the resulting React Component, you will need a page.

### Step 2: Create a Page

Create or update your root page (`app/page.tsx`) with the following code:

```tsx filename="app/page.tsx"
'use client';

import { useState } from 'react';
import { Button } from '@/components/ui/button';
import { streamComponent } from './actions';

export default function Page() {
  const [component, setComponent] = useState<React.ReactNode>();

  return (
    <div>
      <form
        onSubmit={async e => {
          e.preventDefault();
          setComponent(await streamComponent());
        }}
      >
        <Button>Stream Component</Button>
      </form>
      <div>{component}</div>
    </div>
  );
}
```

This page is first marked as a client component with the `"use client";` directive given it will be using hooks and interactivity. On the page, you render a form. When that form is submitted, you call the `streamComponent` action created in the previous step (just like any other function). The `streamComponent` action returns a `ReactNode` that you can then render on the page using React state (`setComponent`).

## Going beyond a single prompt

You can now allow the model to respond to your prompt with a React component. However, this example is limited to a static prompt that is set within your Server Action. You could make this example interactive by turning it into a chatbot.

Learn how to stream React components with the Next.js App Router using `streamUI` with this [example](/examples/next-app/interface/route-components).


# Managing Generative UI State

<Note type="warning">
  AI SDK RSC is currently experimental. We recommend using [AI SDK
  UI](/docs/ai-sdk-ui/overview) for production. For guidance on migrating from
  RSC to UI, see our [migration guide](/docs/ai-sdk-rsc/migrating-to-ui).
</Note>

State is an essential part of any application. State is particularly important in AI applications as it is passed to large language models (LLMs) on each request to ensure they have the necessary context to produce a great generation. Traditional chatbots are text-based and have a structure that mirrors that of any chat application.

For example, in a chatbot, state is an array of `messages` where each `message` has:

- `id`: a unique identifier
- `role`: who sent the message (user/assistant/system/tool)
- `content`: the content of the message

This state can be rendered in the UI and sent to the model without any modifications.

With Generative UI, the model can now return a React component, rather than a plain text message. The client can render that component without issue, but that state can't be sent back to the model because React components aren't serialisable. So, what can you do?

**The solution is to split the state in two, where one (AI State) becomes a proxy for the other (UI State)**.

One way to understand this concept is through a Lego analogy. Imagine a 10,000 piece Lego model that, once built, cannot be easily transported because it is fragile. By taking the model apart, it can be easily transported, and then rebuilt following the steps outlined in the instructions pamphlet. In this way, the instructions pamphlet is a proxy to the physical structure. Similarly, AI State provides a serialisable (JSON) representation of your UI that can be passed back and forth to the model.

## What is AI and UI State?

The RSC API simplifies how you manage AI State and UI State, providing a robust way to keep them in sync between your database, server and client.

### AI State

AI State refers to the state of your application in a serialisable format that will be used on the server and can be shared with the language model.

For a chat app, the AI State is the conversation history (messages) between the user and the assistant. Components generated by the model would be represented in a JSON format as a tool alongside any necessary props. AI State can also be used to store other values and meta information such as `createdAt` for each message and `chatId` for each conversation. The LLM reads this history so it can generate the next message. This state serves as the source of truth for the current application state.

<Note>
  **Note**: AI state can be accessed/modified from both the server and the
  client.
</Note>

### UI State

UI State refers to the state of your application that is rendered on the client. It is a fully client-side state (similar to `useState`) that can store anything from Javascript values to React elements. UI state is a list of actual UI elements that are rendered on the client.

<Note>**Note**: UI State can only be accessed client-side.</Note>

## Using AI / UI State

### Creating the AI Context

AI SDK RSC simplifies managing AI and UI state across your application by providing several hooks. These hooks are powered by [ React context ](https://react.dev/reference/react/hooks#context-hooks) under the hood.

Notably, this means you do not have to pass the message history to the server explicitly for each request. You also can access and update your application state in any child component of the context provider. As you begin building [multistep generative interfaces](/docs/ai-sdk-rsc/multistep-interfaces), this will be particularly helpful.

To use `@ai-sdk/rsc` to manage AI and UI State in your application, you can create a React context using [`createAI`](/docs/reference/ai-sdk-rsc/create-ai):

```tsx filename='app/actions.tsx'
// Define the AI state and UI state types
export type ServerMessage = {
  role: 'user' | 'assistant';
  content: string;
};

export type ClientMessage = {
  id: string;
  role: 'user' | 'assistant';
  display: ReactNode;
};

export const sendMessage = async (input: string): Promise<ClientMessage> => {
  "use server"
  ...
}
```

```tsx filename='app/ai.ts'
import { createAI } from '@ai-sdk/rsc';
import { ClientMessage, ServerMessage, sendMessage } from './actions';

export type AIState = ServerMessage[];
export type UIState = ClientMessage[];

// Create the AI provider with the initial states and allowed actions
export const AI = createAI<AIState, UIState>({
  initialAIState: [],
  initialUIState: [],
  actions: {
    sendMessage,
  },
});
```

<Note>You must pass Server Actions to the `actions` object.</Note>

In this example, you define types for AI State and UI State, respectively.

Next, wrap your application with your newly created context. With that, you can get and set AI and UI State across your entire application.

```tsx filename='app/layout.tsx'
import { type ReactNode } from 'react';
import { AI } from './ai';

export default function RootLayout({
  children,
}: Readonly<{ children: ReactNode }>) {
  return (
    <AI>
      <html lang="en">
        <body>{children}</body>
      </html>
    </AI>
  );
}
```

## Reading UI State in Client

The UI state can be accessed in Client Components using the [`useUIState`](/docs/reference/ai-sdk-rsc/use-ui-state) hook provided by the RSC API. The hook returns the current UI state and a function to update the UI state like React's `useState`.

```tsx filename='app/page.tsx'
'use client';

import { useUIState } from '@ai-sdk/rsc';

export default function Page() {
  const [messages, setMessages] = useUIState();

  return (
    <ul>
      {messages.map(message => (
        <li key={message.id}>{message.display}</li>
      ))}
    </ul>
  );
}
```

## Reading AI State in Client

The AI state can be accessed in Client Components using the [`useAIState`](/docs/reference/ai-sdk-rsc/use-ai-state) hook provided by the RSC API. The hook returns the current AI state.

```tsx filename='app/page.tsx'
'use client';

import { useAIState } from '@ai-sdk/rsc';

export default function Page() {
  const [messages, setMessages] = useAIState();

  return (
    <ul>
      {messages.map(message => (
        <li key={message.id}>{message.content}</li>
      ))}
    </ul>
  );
}
```

## Reading AI State on Server

The AI State can be accessed within any Server Action provided to the `createAI` context using the [`getAIState`](/docs/reference/ai-sdk-rsc/get-ai-state) function. It returns the current AI state as a read-only value:

```tsx filename='app/actions.ts'
import { getAIState } from '@ai-sdk/rsc';

export async function sendMessage(message: string) {
  'use server';

  const history = getAIState();

  const response = await generateText({
    model: openai('gpt-3.5-turbo'),
    messages: [...history, { role: 'user', content: message }],
  });

  return response;
}
```

<Note>
  Remember, you can only access state within actions that have been passed to
  the `createAI` context within the `actions` key.
</Note>

## Updating AI State on Server

The AI State can also be updated from within your Server Action with the [`getMutableAIState`](/docs/reference/ai-sdk-rsc/get-mutable-ai-state) function. This function is similar to `getAIState`, but it returns the state with methods to read and update it:

```tsx filename='app/actions.ts'
import { getMutableAIState } from '@ai-sdk/rsc';

export async function sendMessage(message: string) {
  'use server';

  const history = getMutableAIState();

  // Update the AI state with the new user message.
  history.update([...history.get(), { role: 'user', content: message }]);

  const response = await generateText({
    model: openai('gpt-3.5-turbo'),
    messages: history.get(),
  });

  // Update the AI state again with the response from the model.
  history.done([...history.get(), { role: 'assistant', content: response }]);

  return response;
}
```

<Note>
  It is important to update the AI State with new responses using `.update()`
  and `.done()` to keep the conversation history in sync.
</Note>

## Calling Server Actions from the Client

To call the `sendMessage` action from the client, you can use the [`useActions`](/docs/reference/ai-sdk-rsc/use-actions) hook. The hook returns all the available Actions that were provided to `createAI`:

```tsx filename='app/page.tsx'
'use client';

import { useActions, useUIState } from '@ai-sdk/rsc';
import { AI } from './ai';

export default function Page() {
  const { sendMessage } = useActions<typeof AI>();
  const [messages, setMessages] = useUIState();

  const handleSubmit = async event => {
    event.preventDefault();

    setMessages([
      ...messages,
      { id: Date.now(), role: 'user', display: event.target.message.value },
    ]);

    const response = await sendMessage(event.target.message.value);

    setMessages([
      ...messages,
      { id: Date.now(), role: 'assistant', display: response },
    ]);
  };

  return (
    <>
      <ul>
        {messages.map(message => (
          <li key={message.id}>{message.display}</li>
        ))}
      </ul>
      <form onSubmit={handleSubmit}>
        <input type="text" name="message" />
        <button type="submit">Send</button>
      </form>
    </>
  );
}
```

When the user submits a message, the `sendMessage` action is called with the message content. The response from the action is then added to the UI state, updating the displayed messages.

<Note>
  Important! Don't forget to update the UI State after you call your Server
  Action otherwise the streamed component will not show in the UI.
</Note>

To learn more, check out this [example](/examples/next-app/state-management/ai-ui-states) on managing AI and UI state using `@ai-sdk/rsc`.

---

Next, you will learn how you can save and restore state with `@ai-sdk/rsc`.


# Saving and Restoring States

<Note type="warning">
  AI SDK RSC is currently experimental. We recommend using [AI SDK
  UI](/docs/ai-sdk-ui/overview) for production. For guidance on migrating from
  RSC to UI, see our [migration guide](/docs/ai-sdk-rsc/migrating-to-ui).
</Note>

AI SDK RSC provides convenient methods for saving and restoring AI and UI state. This is useful for saving the state of your application after every model generation, and restoring it when the user revisits the generations.

## AI State

### Saving AI state

The AI state can be saved using the [`onSetAIState`](/docs/reference/ai-sdk-rsc/create-ai#on-set-ai-state) callback, which gets called whenever the AI state is updated. In the following example, you save the chat history to a database whenever the generation is marked as done.

```tsx filename='app/ai.ts'
export const AI = createAI<ServerMessage[], ClientMessage[]>({
  actions: {
    continueConversation,
  },
  onSetAIState: async ({ state, done }) => {
    'use server';

    if (done) {
      saveChatToDB(state);
    }
  },
});
```

### Restoring AI state

The AI state can be restored using the [`initialAIState`](/docs/reference/ai-sdk-rsc/create-ai#initial-ai-state) prop passed to the context provider created by the [`createAI`](/docs/reference/ai-sdk-rsc/create-ai) function. In the following example, you restore the chat history from a database when the component is mounted.

```tsx file='app/layout.tsx'
import { ReactNode } from 'react';
import { AI } from './ai';

export default async function RootLayout({
  children,
}: Readonly<{ children: ReactNode }>) {
  const chat = await loadChatFromDB();

  return (
    <html lang="en">
      <body>
        <AI initialAIState={chat}>{children}</AI>
      </body>
    </html>
  );
}
```

## UI State

### Saving UI state

The UI state cannot be saved directly, since the contents aren't yet serializable. Instead, you can use the AI state as proxy to store details about the UI state and use it to restore the UI state when needed.

### Restoring UI state

The UI state can be restored using the AI state as a proxy. In the following example, you restore the chat history from the AI state when the component is mounted. You use the [`onGetUIState`](/docs/reference/ai-sdk-rsc/create-ai#on-get-ui-state) callback to listen for SSR events and restore the UI state.

```tsx filename='app/ai.ts'
export const AI = createAI<ServerMessage[], ClientMessage[]>({
  actions: {
    continueConversation,
  },
  onGetUIState: async () => {
    'use server';

    const historyFromDB: ServerMessage[] = await loadChatFromDB();
    const historyFromApp: ServerMessage[] = getAIState();

    // If the history from the database is different from the
    // history in the app, they're not in sync so return the UIState
    // based on the history from the database

    if (historyFromDB.length !== historyFromApp.length) {
      return historyFromDB.map(({ role, content }) => ({
        id: generateId(),
        role,
        display:
          role === 'function' ? (
            <Component {...JSON.parse(content)} />
          ) : (
            content
          ),
      }));
    }
  },
});
```

To learn more, check out this [example](/examples/next-app/state-management/save-and-restore-states) that persists and restores states in your Next.js application.

---

Next, you will learn how you can use `@ai-sdk/rsc` functions like `useActions` and `useUIState` to create interactive, multistep interfaces.



# Designing Multistep Interfaces

<Note type="warning">
  AI SDK RSC is currently experimental. We recommend using [AI SDK
  UI](/docs/ai-sdk-ui/overview) for production. For guidance on migrating from
  RSC to UI, see our [migration guide](/docs/ai-sdk-rsc/migrating-to-ui).
</Note>

Multistep interfaces refer to user interfaces that require multiple independent steps to be executed in order to complete a specific task.

For example, if you wanted to build a Generative UI chatbot capable of booking flights, it could have three steps:

- Search all flights
- Pick flight
- Check availability

To build this kind of application you will leverage two concepts, **tool composition** and **application context**.

**Tool composition** is the process of combining multiple [tools](/docs/ai-sdk-core/tools-and-tool-calling) to create a new tool. This is a powerful concept that allows you to break down complex tasks into smaller, more manageable steps. In the example above, _"search all flights"_, _"pick flight"_, and _"check availability"_ come together to create a holistic _"book flight"_ tool.

**Application context** refers to the state of the application at any given point in time. This includes the user's input, the output of the language model, and any other relevant information. In the example above, the flight selected in _"pick flight"_ would be used as context necessary to complete the _"check availability"_ task.

## Overview

In order to build a multistep interface with `@ai-sdk/rsc`, you will need a few things:

- A Server Action that calls and returns the result from the `streamUI` function
- Tool(s) (sub-tasks necessary to complete your overall task)
- React component(s) that should be rendered when the tool is called
- A page to render your chatbot

The general flow that you will follow is:

- User sends a message (calls your Server Action with `useActions`, passing the message as an input)
- Message is appended to the AI State and then passed to the model alongside a number of tools
- Model can decide to call a tool, which will render the `<SomeTool />` component
- Within that component, you can add interactivity by using `useActions` to call the model with your Server Action and `useUIState` to append the model's response (`<SomeOtherTool />`) to the UI State
- And so on...

## Implementation

The turn-by-turn implementation is the simplest form of multistep interfaces. In this implementation, the user and the model take turns during the conversation. For every user input, the model generates a response, and the conversation continues in this turn-by-turn fashion.

In the following example, you specify two tools (`searchFlights` and `lookupFlight`) that the model can use to search for flights and lookup details for a specific flight.

```tsx filename="app/actions.tsx"
import { streamUI } from '@ai-sdk/rsc';
import { openai } from '@ai-sdk/openai';
import { z } from 'zod';

const searchFlights = async (
  source: string,
  destination: string,
  date: string,
) => {
  return [
    {
      id: '1',
      flightNumber: 'AA123',
    },
    {
      id: '2',
      flightNumber: 'AA456',
    },
  ];
};

const lookupFlight = async (flightNumber: string) => {
  return {
    flightNumber: flightNumber,
    departureTime: '10:00 AM',
    arrivalTime: '12:00 PM',
  };
};

export async function submitUserMessage(input: string) {
  'use server';

  const ui = await streamUI({
    model: openai('gpt-4o'),
    system: 'you are a flight booking assistant',
    prompt: input,
    text: async ({ content }) => <div>{content}</div>,
    tools: {
      searchFlights: {
        description: 'search for flights',
        inputSchema: z.object({
          source: z.string().describe('The origin of the flight'),
          destination: z.string().describe('The destination of the flight'),
          date: z.string().describe('The date of the flight'),
        }),
        generate: async function* ({ source, destination, date }) {
          yield `Searching for flights from ${source} to ${destination} on ${date}...`;
          const results = await searchFlights(source, destination, date);

          return (
            <div>
              {results.map(result => (
                <div key={result.id}>
                  <div>{result.flightNumber}</div>
                </div>
              ))}
            </div>
          );
        },
      },
      lookupFlight: {
        description: 'lookup details for a flight',
        parameters: z.object({
          flightNumber: z.string().describe('The flight number'),
        }),
        generate: async function* ({ flightNumber }) {
          yield `Looking up details for flight ${flightNumber}...`;
          const details = await lookupFlight(flightNumber);

          return (
            <div>
              <div>Flight Number: {details.flightNumber}</div>
              <div>Departure Time: {details.departureTime}</div>
              <div>Arrival Time: {details.arrivalTime}</div>
            </div>
          );
        },
      },
    },
  });

  return ui.value;
}
```

Next, create an AI context that will hold the UI State and AI State.

```ts filename='app/ai.ts'
import { createAI } from '@ai-sdk/rsc';
import { submitUserMessage } from './actions';

export const AI = createAI<any[], React.ReactNode[]>({
  initialUIState: [],
  initialAIState: [],
  actions: {
    submitUserMessage,
  },
});
```

Next, wrap your application with your newly created context.

```tsx filename='app/layout.tsx'
import { type ReactNode } from 'react';
import { AI } from './ai';

export default function RootLayout({
  children,
}: Readonly<{ children: ReactNode }>) {
  return (
    <AI>
      <html lang="en">
        <body>{children}</body>
      </html>
    </AI>
  );
}
```

To call your Server Action, update your root page with the following:

```tsx filename="app/page.tsx"
'use client';

import { useState } from 'react';
import { AI } from './ai';
import { useActions, useUIState } from '@ai-sdk/rsc';

export default function Page() {
  const [input, setInput] = useState<string>('');
  const [conversation, setConversation] = useUIState<typeof AI>();
  const { submitUserMessage } = useActions();

  const handleSubmit = async (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    setInput('');
    setConversation(currentConversation => [
      ...currentConversation,
      <div>{input}</div>,
    ]);
    const message = await submitUserMessage(input);
    setConversation(currentConversation => [...currentConversation, message]);
  };

  return (
    <div>
      <div>
        {conversation.map((message, i) => (
          <div key={i}>{message}</div>
        ))}
      </div>
      <div>
        <form onSubmit={handleSubmit}>
          <input
            type="text"
            value={input}
            onChange={e => setInput(e.target.value)}
          />
          <button>Send Message</button>
        </form>
      </div>
    </div>
  );
}
```

This page pulls in the current UI State using the `useUIState` hook, which is then mapped over and rendered in the UI. To access the Server Action, you use the `useActions` hook which will return all actions that were passed to the `actions` key of the `createAI` function in your `actions.tsx` file. Finally, you call the `submitUserMessage` function like any other TypeScript function. This function returns a React component (`message`) that is then rendered in the UI by updating the UI State with `setConversation`.

In this example, to call the next tool, the user must respond with plain text. **Given you are streaming a React component, you can add a button to trigger the next step in the conversation**.

To add user interaction, you will have to convert the component into a client component and use the `useAction` hook to trigger the next step in the conversation.

```tsx filename="components/flights.tsx"
'use client';

import { useActions, useUIState } from '@ai-sdk/rsc';
import { ReactNode } from 'react';

interface FlightsProps {
  flights: { id: string; flightNumber: string }[];
}

export const Flights = ({ flights }: FlightsProps) => {
  const { submitUserMessage } = useActions();
  const [_, setMessages] = useUIState();

  return (
    <div>
      {flights.map(result => (
        <div key={result.id}>
          <div
            onClick={async () => {
              const display = await submitUserMessage(
                `lookupFlight ${result.flightNumber}`,
              );

              setMessages((messages: ReactNode[]) => [...messages, display]);
            }}
          >
            {result.flightNumber}
          </div>
        </div>
      ))}
    </div>
  );
};
```

Now, update your `searchFlights` tool to render the new `<Flights />` component.

```tsx filename="actions.tsx"
...
searchFlights: {
  description: 'search for flights',
  parameters: z.object({
    source: z.string().describe('The origin of the flight'),
    destination: z.string().describe('The destination of the flight'),
    date: z.string().describe('The date of the flight'),
  }),
  generate: async function* ({ source, destination, date }) {
    yield `Searching for flights from ${source} to ${destination} on ${date}...`;
    const results = await searchFlights(source, destination, date);
    return (<Flights flights={results} />);
  },
}
...
```

In the above example, the `Flights` component is used to display the search results. When the user clicks on a flight number, the `lookupFlight` tool is called with the flight number as a parameter. The `submitUserMessage` action is then called to trigger the next step in the conversation.

Learn more about tool calling in Next.js App Router by checking out examples [here](/examples/next-app/tools).


import { UIPreviewCard, Card } from '@/components/home/card';
import { EventPlanning } from '@/components/home/event-planning';
import { Searching } from '@/components/home/searching';
import { Weather } from '@/components/home/weather';

# Streaming Values

<Note type="warning">
  AI SDK RSC is currently experimental. We recommend using [AI SDK
  UI](/docs/ai-sdk-ui/overview) for production. For guidance on migrating from
  RSC to UI, see our [migration guide](/docs/ai-sdk-rsc/migrating-to-ui).
</Note>

The RSC API provides several utility functions to allow you to stream values from the server to the client. This is useful when you need more granular control over what you are streaming and how you are streaming it.

<Note>
  These utilities can also be paired with [AI SDK Core](/docs/ai-sdk-core)
  functions like [`streamText`](/docs/reference/ai-sdk-core/stream-text) and
  [`streamObject`](/docs/reference/ai-sdk-core/stream-object) to easily stream
  LLM generations from the server to the client.
</Note>

There are two functions provided by the RSC API that allow you to create streamable values:

- [`createStreamableValue`](/docs/reference/ai-sdk-rsc/create-streamable-value) - creates a streamable (serializable) value, with full control over how you create, update, and close the stream.
- [`createStreamableUI`](/docs/reference/ai-sdk-rsc/create-streamable-ui) - creates a streamable React component, with full control over how you create, update, and close the stream.

## `createStreamableValue`

The RSC API allows you to stream serializable Javascript values from the server to the client using [`createStreamableValue`](/docs/reference/ai-sdk-rsc/create-streamable-value), such as strings, numbers, objects, and arrays.

This is useful when you want to stream:

- Text generations from the language model in real-time.
- Buffer values of image and audio generations from multi-modal models.
- Progress updates from multi-step agent runs.

## Creating a Streamable Value

You can import `createStreamableValue` from `@ai-sdk/rsc` and use it to create a streamable value.

```tsx file='app/actions.ts'
'use server';

import { createStreamableValue } from '@ai-sdk/rsc';

export const runThread = async () => {
  const streamableStatus = createStreamableValue('thread.init');

  setTimeout(() => {
    streamableStatus.update('thread.run.create');
    streamableStatus.update('thread.run.update');
    streamableStatus.update('thread.run.end');
    streamableStatus.done('thread.end');
  }, 1000);

  return {
    status: streamableStatus.value,
  };
};
```

## Reading a Streamable Value

You can read streamable values on the client using `readStreamableValue`. It returns an async iterator that yields the value of the streamable as it is updated:

```tsx file='app/page.tsx'
import { readStreamableValue } from '@ai-sdk/rsc';
import { runThread } from '@/actions';

export default function Page() {
  return (
    <button
      onClick={async () => {
        const { status } = await runThread();

        for await (const value of readStreamableValue(status)) {
          console.log(value);
        }
      }}
    >
      Ask
    </button>
  );
}
```

Learn how to stream a text generation (with `streamText`) using the Next.js App Router and `createStreamableValue` in this [example](/examples/next-app/basics/streaming-text-generation).

## `createStreamableUI`

`createStreamableUI` creates a stream that holds a React component. Unlike AI SDK Core APIs, this function does not call a large language model. Instead, it provides a primitive that can be used to have granular control over streaming a React component.

## Using `createStreamableUI`

Let's look at how you can use the `createStreamableUI` function with a Server Action.

```tsx filename='app/actions.tsx'
'use server';

import { createStreamableUI } from '@ai-sdk/rsc';

export async function getWeather() {
  const weatherUI = createStreamableUI();

  weatherUI.update(<div style={{ color: 'gray' }}>Loading...</div>);

  setTimeout(() => {
    weatherUI.done(<div>It&apos;s a sunny day!</div>);
  }, 1000);

  return weatherUI.value;
}
```

First, you create a streamable UI with an empty state and then update it with a loading message. After 1 second, you mark the stream as done passing in the actual weather information as its final value. The `.value` property contains the actual UI that can be sent to the client.

## Reading a Streamable UI

On the client side, you can call the `getWeather` Server Action and render the returned UI like any other React component.

```tsx filename='app/page.tsx'
'use client';

import { useState } from 'react';
import { readStreamableValue } from '@ai-sdk/rsc';
import { getWeather } from '@/actions';

export default function Page() {
  const [weather, setWeather] = useState<React.ReactNode | null>(null);

  return (
    <div>
      <button
        onClick={async () => {
          const weatherUI = await getWeather();
          setWeather(weatherUI);
        }}
      >
        What&apos;s the weather?
      </button>

      {weather}
    </div>
  );
}
```

When the button is clicked, the `getWeather` function is called, and the returned UI is set to the `weather` state and rendered on the page. Users will see the loading message first and then the actual weather information after 1 second.

Learn more about handling multiple streams in a single request in the [Multiple Streamables](/docs/advanced/multiple-streamables) guide.

Learn more about handling state for more complex use cases with [ AI/UI State ](/docs/ai-sdk-rsc/generative-ui-state).




# Handling Loading State

<Note type="warning">
  AI SDK RSC is currently experimental. We recommend using [AI SDK
  UI](/docs/ai-sdk-ui/overview) for production. For guidance on migrating from
  RSC to UI, see our [migration guide](/docs/ai-sdk-rsc/migrating-to-ui).
</Note>

Given that responses from language models can often take a while to complete, it's crucial to be able to show loading state to users. This provides visual feedback that the system is working on their request and helps maintain a positive user experience.

There are three approaches you can take to handle loading state with the AI SDK RSC:

- Managing loading state similar to how you would in a traditional Next.js application. This involves setting a loading state variable in the client and updating it when the response is received.
- Streaming loading state from the server to the client. This approach allows you to track loading state on a more granular level and provide more detailed feedback to the user.
- Streaming loading component from the server to the client. This approach allows you to stream a React Server Component to the client while awaiting the model's response.

## Handling Loading State on the Client

### Client

Let's create a simple Next.js page that will call the `generateResponse` function when the form is submitted. The function will take in the user's prompt (`input`) and then generate a response (`response`). To handle the loading state, use the `loading` state variable. When the form is submitted, set `loading` to `true`, and when the response is received, set it back to `false`. While the response is being streamed, the input field will be disabled.

```tsx filename='app/page.tsx'
'use client';

import { useState } from 'react';
import { generateResponse } from './actions';
import { readStreamableValue } from '@ai-sdk/rsc';

// Force the page to be dynamic and allow streaming responses up to 30 seconds
export const maxDuration = 30;

export default function Home() {
  const [input, setInput] = useState<string>('');
  const [generation, setGeneration] = useState<string>('');
  const [loading, setLoading] = useState<boolean>(false);

  return (
    <div>
      <div>{generation}</div>
      <form
        onSubmit={async e => {
          e.preventDefault();
          setLoading(true);
          const response = await generateResponse(input);

          let textContent = '';

          for await (const delta of readStreamableValue(response)) {
            textContent = `${textContent}${delta}`;
            setGeneration(textContent);
          }
          setInput('');
          setLoading(false);
        }}
      >
        <input
          type="text"
          value={input}
          disabled={loading}
          className="disabled:opacity-50"
          onChange={event => {
            setInput(event.target.value);
          }}
        />
        <button>Send Message</button>
      </form>
    </div>
  );
}
```

### Server

Now let's implement the `generateResponse` function. Use the `streamText` function to generate a response to the input.

```typescript filename='app/actions.ts'
'use server';

import { streamText } from 'ai';
import { openai } from '@ai-sdk/openai';
import { createStreamableValue } from '@ai-sdk/rsc';

export async function generateResponse(prompt: string) {
  const stream = createStreamableValue();

  (async () => {
    const { textStream } = streamText({
      model: openai('gpt-4o'),
      prompt,
    });

    for await (const text of textStream) {
      stream.update(text);
    }

    stream.done();
  })();

  return stream.value;
}
```

## Streaming Loading State from the Server

If you are looking to track loading state on a more granular level, you can create a new streamable value to store a custom variable and then read this on the frontend. Let's update the example to create a new streamable value for tracking loading state:

### Server

```typescript filename='app/actions.ts' highlight='9,22,25'
'use server';

import { streamText } from 'ai';
import { openai } from '@ai-sdk/openai';
import { createStreamableValue } from '@ai-sdk/rsc';

export async function generateResponse(prompt: string) {
  const stream = createStreamableValue();
  const loadingState = createStreamableValue({ loading: true });

  (async () => {
    const { textStream } = streamText({
      model: openai('gpt-4o'),
      prompt,
    });

    for await (const text of textStream) {
      stream.update(text);
    }

    stream.done();
    loadingState.done({ loading: false });
  })();

  return { response: stream.value, loadingState: loadingState.value };
}
```

### Client

```tsx filename='app/page.tsx' highlight="22,30-34"
'use client';

import { useState } from 'react';
import { generateResponse } from './actions';
import { readStreamableValue } from '@ai-sdk/rsc';

// Force the page to be dynamic and allow streaming responses up to 30 seconds
export const maxDuration = 30;

export default function Home() {
  const [input, setInput] = useState<string>('');
  const [generation, setGeneration] = useState<string>('');
  const [loading, setLoading] = useState<boolean>(false);

  return (
    <div>
      <div>{generation}</div>
      <form
        onSubmit={async e => {
          e.preventDefault();
          setLoading(true);
          const { response, loadingState } = await generateResponse(input);

          let textContent = '';

          for await (const responseDelta of readStreamableValue(response)) {
            textContent = `${textContent}${responseDelta}`;
            setGeneration(textContent);
          }
          for await (const loadingDelta of readStreamableValue(loadingState)) {
            if (loadingDelta) {
              setLoading(loadingDelta.loading);
            }
          }
          setInput('');
          setLoading(false);
        }}
      >
        <input
          type="text"
          value={input}
          disabled={loading}
          className="disabled:opacity-50"
          onChange={event => {
            setInput(event.target.value);
          }}
        />
        <button>Send Message</button>
      </form>
    </div>
  );
}
```

This allows you to provide more detailed feedback about the generation process to your users.

## Streaming Loading Components with `streamUI`

If you are using the [ `streamUI` ](/docs/reference/ai-sdk-rsc/stream-ui) function, you can stream the loading state to the client in the form of a React component. `streamUI` supports the usage of [ JavaScript generator functions ](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function*), which allow you to yield some value (in this case a React component) while some other blocking work completes.

## Server

```ts
'use server';

import { openai } from '@ai-sdk/openai';
import { streamUI } from '@ai-sdk/rsc';

export async function generateResponse(prompt: string) {
  const result = await streamUI({
    model: openai('gpt-4o'),
    prompt,
    text: async function* ({ content }) {
      yield <div>loading...</div>;
      return <div>{content}</div>;
    },
  });

  return result.value;
}
```

<Note>
  Remember to update the file from `.ts` to `.tsx` because you are defining a
  React component in the `streamUI` function.
</Note>

## Client

```tsx
'use client';

import { useState } from 'react';
import { generateResponse } from './actions';
import { readStreamableValue } from '@ai-sdk/rsc';

// Force the page to be dynamic and allow streaming responses up to 30 seconds
export const maxDuration = 30;

export default function Home() {
  const [input, setInput] = useState<string>('');
  const [generation, setGeneration] = useState<React.ReactNode>();

  return (
    <div>
      <div>{generation}</div>
      <form
        onSubmit={async e => {
          e.preventDefault();
          const result = await generateResponse(input);
          setGeneration(result);
          setInput('');
        }}
      >
        <input
          type="text"
          value={input}
          onChange={event => {
            setInput(event.target.value);
          }}
        />
        <button>Send Message</button>
      </form>
    </div>
  );
}
```

# Error Handling

<Note type="warning">
  AI SDK RSC is currently experimental. We recommend using [AI SDK
  UI](/docs/ai-sdk-ui/overview) for production. For guidance on migrating from
  RSC to UI, see our [migration guide](/docs/ai-sdk-rsc/migrating-to-ui).
</Note>

Two categories of errors can occur when working with the RSC API: errors while streaming user interfaces and errors while streaming other values.

## Handling UI Errors

To handle errors while generating UI, the [`streamableUI`](/docs/reference/ai-sdk-rsc/create-streamable-ui) object exposes an `error()` method.

```tsx filename='app/actions.tsx'
'use server';

import { createStreamableUI } from '@ai-sdk/rsc';

export async function getStreamedUI() {
  const ui = createStreamableUI();

  (async () => {
    ui.update(<div>loading</div>);
    const data = await fetchData();
    ui.done(<div>{data}</div>);
  })().catch(e => {
    ui.error(<div>Error: {e.message}</div>);
  });

  return ui.value;
}
```

With this method, you can catch any error with the stream, and return relevant UI. On the client, you can also use a [React Error Boundary](https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary) to wrap the streamed component and catch any additional errors.

```tsx filename='app/page.tsx'
import { getStreamedUI } from '@/actions';
import { useState } from 'react';
import { ErrorBoundary } from './ErrorBoundary';

export default function Page() {
  const [streamedUI, setStreamedUI] = useState(null);

  return (
    <div>
      <button
        onClick={async () => {
          const newUI = await getStreamedUI();
          setStreamedUI(newUI);
        }}
      >
        What does the new UI look like?
      </button>
      <ErrorBoundary>{streamedUI}</ErrorBoundary>
    </div>
  );
}
```

## Handling Other Errors

To handle other errors while streaming, you can return an error object that the receiver can use to determine why the failure occurred.

```tsx filename='app/actions.tsx'
'use server';

import { createStreamableValue } from '@ai-sdk/rsc';
import { fetchData, emptyData } from '../utils/data';

export const getStreamedData = async () => {
  const streamableData = createStreamableValue<string>(emptyData);

  try {
    (() => {
      const data1 = await fetchData();
      streamableData.update(data1);

      const data2 = await fetchData();
      streamableData.update(data2);

      const data3 = await fetchData();
      streamableData.done(data3);
    })();

    return { data: streamableData.value };
  } catch (e) {
    return { error: e.message };
  }
};
```




# Authentication

<Note type="warning">
  AI SDK RSC is currently experimental. We recommend using [AI SDK
  UI](/docs/ai-sdk-ui/overview) for production. For guidance on migrating from
  RSC to UI, see our [migration guide](/docs/ai-sdk-rsc/migrating-to-ui).
</Note>

The RSC API makes extensive use of [`Server Actions`](https://nextjs.org/docs/app/building-your-application/data-fetching/server-actions-and-mutations) to power streaming values and UI from the server.

Server Actions are exposed as public, unprotected endpoints. As a result, you should treat Server Actions as you would public-facing API endpoints and ensure that the user is authorized to perform the action before returning any data.

```tsx filename="app/actions.tsx"
'use server';

import { cookies } from 'next/headers';
import { createStremableUI } from '@ai-sdk/rsc';
import { validateToken } from '../utils/auth';

export const getWeather = async () => {
  const token = cookies().get('token');

  if (!token || !validateToken(token)) {
    return {
      error: 'This action requires authentication',
    };
  }
  const streamableDisplay = createStreamableUI(null);

  streamableDisplay.update(<Skeleton />);
  streamableDisplay.done(<Weather />);

  return {
    display: streamableDisplay.value,
  };
};
```




# Migrating from RSC to UI

This guide helps you migrate from AI SDK RSC to AI SDK UI.

## Background

The AI SDK has two packages that help you build the frontend for your applications – [AI SDK UI](/docs/ai-sdk-ui) and [AI SDK RSC](/docs/ai-sdk-rsc).

We introduced support for using [React Server Components](https://react.dev/reference/rsc/server-components) (RSC) within the AI SDK to simplify building generative user interfaces for frameworks that support RSC.

However, given we're pushing the boundaries of this technology, AI SDK RSC currently faces significant limitations that make it unsuitable for stable production use.

- It is not possible to abort a stream using server actions. This will be improved in future releases of React and Next.js [(1122)](https://github.com/vercel/ai/issues/1122).
- When using `createStreamableUI` and `streamUI`, components remount on `.done()`, causing them to flicker [(2939)](https://github.com/vercel/ai/issues/2939).
- Many suspense boundaries can lead to crashes [(2843)](https://github.com/vercel/ai/issues/2843).
- Using `createStreamableUI` can lead to quadratic data transfer. You can avoid this using createStreamableValue instead, and rendering the component client-side.
- Closed RSC streams cause update issues [(3007)](https://github.com/vercel/ai/issues/3007).

Due to these limitations, AI SDK RSC is marked as experimental, and we do not recommend using it for stable production environments.

As a result, we strongly recommend migrating to AI SDK UI, which has undergone extensive development to provide a more stable and production grade experience.

In building [v0](https://v0.dev), we have invested considerable time exploring how to create the best chat experience on the web. AI SDK UI ships with many of these best practices and commonly used patterns like [language model middleware](/docs/ai-sdk-core/middleware), [multi-step tool calls](/docs/ai-sdk-core/tools-and-tool-calling#multi-step-calls), [attachments](/docs/ai-sdk-ui/chatbot#attachments-experimental), [telemetry](/docs/ai-sdk-core/telemetry), [provider registry](/docs/ai-sdk-core/provider-management#provider-registry), and many more. These features have been considerately designed into a neat abstraction that you can use to reliably integrate AI into your applications.

## Streaming Chat Completions

### Basic Setup

The `streamUI` function executes as part of a server action as illustrated below.

#### Before: Handle generation and rendering in a single server action

```tsx filename="@/app/actions.tsx"
import { openai } from '@ai-sdk/openai';
import { getMutableAIState, streamUI } from '@ai-sdk/rsc';

export async function sendMessage(message: string) {
  'use server';

  const messages = getMutableAIState('messages');

  messages.update([...messages.get(), { role: 'user', content: message }]);

  const { value: stream } = await streamUI({
    model: openai('gpt-4o'),
    system: 'you are a friendly assistant!',
    messages: messages.get(),
    text: async function* ({ content, done }) {
      // process text
    },
    tools: {
      // tool definitions
    },
  });

  return stream;
}
```

#### Before: Call server action and update UI state

The chat interface calls the server action. The response is then saved using the `useUIState` hook.

```tsx filename="@/app/page.tsx"
'use client';

import { useState, ReactNode } from 'react';
import { useActions, useUIState } from '@ai-sdk/rsc';

export default function Page() {
  const { sendMessage } = useActions();
  const [input, setInput] = useState('');
  const [messages, setMessages] = useUIState();

  return (
    <div>
      {messages.map(message => message)}

      <form
        onSubmit={async () => {
          const response: ReactNode = await sendMessage(input);
          setMessages(msgs => [...msgs, response]);
        }}
      >
        <input type="text" />
        <button type="submit">Submit</button>
      </form>
    </div>
  );
}
```

The `streamUI` function combines generating text and rendering the user interface. To migrate to AI SDK UI, you need to **separate these concerns** – streaming generations with `streamText` and rendering the UI with `useChat`.

#### After: Replace server action with route handler

The `streamText` function executes as part of a route handler and streams the response to the client. The `useChat` hook on the client decodes this stream and renders the response within the chat interface.

```ts filename="@/app/api/chat/route.ts"
import { streamText } from 'ai';
import { openai } from '@ai-sdk/openai';

export async function POST(request) {
  const { messages } = await request.json();

  const result = streamText({
    model: openai('gpt-4o'),
    system: 'you are a friendly assistant!',
    messages,
    tools: {
      // tool definitions
    },
  });

  return result.toUIMessageStreamResponse();
}
```

#### After: Update client to use chat hook

```tsx filename="@/app/page.tsx"
'use client';

import { useChat } from '@ai-sdk/react';

export default function Page() {
  const { messages, input, setInput, handleSubmit } = useChat();

  return (
    <div>
      {messages.map(message => (
        <div key={message.id}>
          <div>{message.role}</div>
          <div>{message.content}</div>
        </div>
      ))}

      <form onSubmit={handleSubmit}>
        <input
          type="text"
          value={input}
          onChange={event => {
            setInput(event.target.value);
          }}
        />
        <button type="submit">Send</button>
      </form>
    </div>
  );
}
```

### Parallel Tool Calls

In AI SDK RSC, `streamUI` does not support parallel tool calls. You will have to use a combination of `streamText`, `createStreamableUI` and `createStreamableValue`.

With AI SDK UI, `useChat` comes with built-in support for parallel tool calls. You can define multiple tools in the `streamText` and have them called them in parallel. The `useChat` hook will then handle the parallel tool calls for you automatically.

### Multi-Step Tool Calls

In AI SDK RSC, `streamUI` does not support multi-step tool calls. You will have to use a combination of `streamText`, `createStreamableUI` and `createStreamableValue`.

With AI SDK UI, `useChat` comes with built-in support for multi-step tool calls. You can set `maxSteps` in the `streamText` function to define the number of steps the language model can make in a single call. The `useChat` hook will then handle the multi-step tool calls for you automatically.

### Generative User Interfaces

The `streamUI` function uses `tools` as a way to execute functions based on user input and renders React components based on the function output to go beyond text in the chat interface.

#### Before: Render components within the server action and stream to client

```tsx filename="@/app/actions.tsx"
import { z } from 'zod';
import { streamUI } from '@ai-sdk/rsc';
import { openai } from '@ai-sdk/openai';
import { getWeather } from '@/utils/queries';
import { Weather } from '@/components/weather';

const { value: stream } = await streamUI({
  model: openai('gpt-4o'),
  system: 'you are a friendly assistant!',
  messages,
  text: async function* ({ content, done }) {
    // process text
  },
  tools: {
    displayWeather: {
      description: 'Display the weather for a location',
      inputSchema: z.object({
        latitude: z.number(),
        longitude: z.number(),
      }),
      generate: async function* ({ latitude, longitude }) {
        yield <div>Loading weather...</div>;

        const { value, unit } = await getWeather({ latitude, longitude });

        return <Weather value={value} unit={unit} />;
      },
    },
  },
});
```

As mentioned earlier, `streamUI` generates text and renders the React component in a single server action call.

#### After: Replace with route handler and stream props data to client

The `streamText` function streams the props data as response to the client, while `useChat` decode the stream as `toolInvocations` and renders the chat interface.

```ts filename="@/app/api/chat/route.ts"
import { z } from 'zod';
import { openai } from '@ai-sdk/openai';
import { getWeather } from '@/utils/queries';
import { streamText } from 'ai';

export async function POST(request) {
  const { messages } = await request.json();

  const result = streamText({
    model: openai('gpt-4o'),
    system: 'you are a friendly assistant!',
    messages,
    tools: {
      displayWeather: {
        description: 'Display the weather for a location',
        parameters: z.object({
          latitude: z.number(),
          longitude: z.number(),
        }),
        execute: async function ({ latitude, longitude }) {
          const props = await getWeather({ latitude, longitude });
          return props;
        },
      },
    },
  });

  return result.toUIMessageStreamResponse();
}
```

#### After: Update client to use chat hook and render components using tool invocations

```tsx filename="@/app/page.tsx"
'use client';

import { useChat } from '@ai-sdk/react';
import { Weather } from '@/components/weather';

export default function Page() {
  const { messages, input, setInput, handleSubmit } = useChat();

  return (
    <div>
      {messages.map(message => (
        <div key={message.id}>
          <div>{message.role}</div>
          <div>{message.content}</div>

          <div>
            {message.toolInvocations.map(toolInvocation => {
              const { toolName, toolCallId, state } = toolInvocation;

              if (state === 'result') {
                const { result } = toolInvocation;

                return (
                  <div key={toolCallId}>
                    {toolName === 'displayWeather' ? (
                      <Weather weatherAtLocation={result} />
                    ) : null}
                  </div>
                );
              } else {
                return (
                  <div key={toolCallId}>
                    {toolName === 'displayWeather' ? (
                      <div>Loading weather...</div>
                    ) : null}
                  </div>
                );
              }
            })}
          </div>
        </div>
      ))}

      <form onSubmit={handleSubmit}>
        <input
          type="text"
          value={input}
          onChange={event => {
            setInput(event.target.value);
          }}
        />
        <button type="submit">Send</button>
      </form>
    </div>
  );
}
```

### Handling Client Interactions

With AI SDK RSC, components streamed to the client can trigger subsequent generations by calling the relevant server action using the `useActions` hooks. This is possible as long as the component is a descendant of the `<AI/>` context provider.

#### Before: Use actions hook to send messages

```tsx filename="@/app/components/list-flights.tsx"
'use client';

import { useActions, useUIState } from '@ai-sdk/rsc';

export function ListFlights({ flights }) {
  const { sendMessage } = useActions();
  const [_, setMessages] = useUIState();

  return (
    <div>
      {flights.map(flight => (
        <div
          key={flight.id}
          onClick={async () => {
            const response = await sendMessage(
              `I would like to choose flight ${flight.id}!`,
            );

            setMessages(msgs => [...msgs, response]);
          }}
        >
          {flight.name}
        </div>
      ))}
    </div>
  );
}
```

#### After: Use another chat hook with same ID from the component

After switching to AI SDK UI, these messages are synced by initializing the `useChat` hook in the component with the same `id` as the parent component.

```tsx filename="@/app/components/list-flights.tsx"
'use client';

import { useChat } from '@ai-sdk/react';

export function ListFlights({ chatId, flights }) {
  const { append } = useChat({
    id: chatId,
    body: { id: chatId },
    maxSteps: 5,
  });

  return (
    <div>
      {flights.map(flight => (
        <div
          key={flight.id}
          onClick={async () => {
            await append({
              role: 'user',
              content: `I would like to choose flight ${flight.id}!`,
            });
          }}
        >
          {flight.name}
        </div>
      ))}
    </div>
  );
}
```

### Loading Indicators

In AI SDK RSC, you can use the `initial` parameter of `streamUI` to define the component to display while the generation is in progress.

#### Before: Use `loading` to show loading indicator

```tsx filename="@/app/actions.tsx"
import { openai } from '@ai-sdk/openai';
import { streamUI } from '@ai-sdk/rsc';

const { value: stream } = await streamUI({
  model: openai('gpt-4o'),
  system: 'you are a friendly assistant!',
  messages,
  initial: <div>Loading...</div>,
  text: async function* ({ content, done }) {
    // process text
  },
  tools: {
    // tool definitions
  },
});

return stream;
```

With AI SDK UI, you can use the tool invocation state to show a loading indicator while the tool is executing.

#### After: Use tool invocation state to show loading indicator

```tsx filename="@/app/components/message.tsx"
'use client';

export function Message({ role, content, toolInvocations }) {
  return (
    <div>
      <div>{role}</div>
      <div>{content}</div>

      {toolInvocations && (
        <div>
          {toolInvocations.map(toolInvocation => {
            const { toolName, toolCallId, state } = toolInvocation;

            if (state === 'result') {
              const { result } = toolInvocation;

              return (
                <div key={toolCallId}>
                  {toolName === 'getWeather' ? (
                    <Weather weatherAtLocation={result} />
                  ) : null}
                </div>
              );
            } else {
              return (
                <div key={toolCallId}>
                  {toolName === 'getWeather' ? (
                    <Weather isLoading={true} />
                  ) : (
                    <div>Loading...</div>
                  )}
                </div>
              );
            }
          })}
        </div>
      )}
    </div>
  );
}
```

### Saving Chats

Before implementing `streamUI` as a server action, you should create an `<AI/>` provider and wrap your application at the root layout to sync the AI and UI states. During initialization, you typically use the `onSetAIState` callback function to track updates to the AI state and save it to the database when `done(...)` is called.

#### Before: Save chats using callback function of context provider

```ts filename="@/app/actions.ts"
import { createAI } from '@ai-sdk/rsc';
import { saveChat } from '@/utils/queries';

export const AI = createAI({
  initialAIState: {},
  initialUIState: {},
  actions: {
    // server actions
  },
  onSetAIState: async ({ state, done }) => {
    'use server';

    if (done) {
      await saveChat(state);
    }
  },
});
```

#### After: Save chats using callback function of `streamText`

With AI SDK UI, you will save chats using the `onFinish` callback function of `streamText` in your route handler.

```ts filename="@/app/api/chat/route.ts"
import { openai } from '@ai-sdk/openai';
import { saveChat } from '@/utils/queries';
import { streamText, convertToModelMessages } from 'ai';

export async function POST(request) {
  const { id, messages } = await request.json();

  const coreMessages = convertToModelMessages(messages);

  const result = streamText({
    model: openai('gpt-4o'),
    system: 'you are a friendly assistant!',
    messages: coreMessages,
    onFinish: async ({ response }) => {
      try {
        await saveChat({
          id,
          messages: [...coreMessages, ...response.messages],
        });
      } catch (error) {
        console.error('Failed to save chat');
      }
    },
  });

  return result.toUIMessageStreamResponse();
}
```

### Restoring Chats

When using AI SDK RSC, the `useUIState` hook contains the UI state of the chat. When restoring a previously saved chat, the UI state needs to be loaded with messages.

Similar to how you typically save chats in AI SDK RSC, you should use the `onGetUIState` callback function to retrieve the chat from the database, convert it into UI state, and return it to be accessible through `useUIState`.

#### Before: Load chat from database using callback function of context provider

```ts filename="@/app/actions.ts"
import { createAI } from '@ai-sdk/rsc';
import { loadChatFromDB, convertToUIState } from '@/utils/queries';

export const AI = createAI({
  actions: {
    // server actions
  },
  onGetUIState: async () => {
    'use server';

    const chat = await loadChatFromDB();
    const uiState = convertToUIState(chat);

    return uiState;
  },
});
```

AI SDK UI uses the `messages` field of `useChat` to store messages. To load messages when `useChat` is mounted, you should use `initialMessages`.

As messages are typically loaded from the database, we can use a server actions inside a Page component to fetch an older chat from the database during static generation and pass the messages as props to the `<Chat/>` component.

#### After: Load chat from database during static generation of page

```tsx filename="@/app/chat/[id]/page.tsx"
import { Chat } from '@/app/components/chat';
import { getChatById } from '@/utils/queries';

// link to example implementation: https://github.com/vercel/ai-chatbot/blob/00b125378c998d19ef60b73fe576df0fe5a0e9d4/lib/utils.ts#L87-L127
import { convertToUIMessages } from '@/utils/functions';

export default async function Page({ params }: { params: any }) {
  const { id } = params;
  const chatFromDb = await getChatById({ id });

  const chat: Chat = {
    ...chatFromDb,
    messages: convertToUIMessages(chatFromDb.messages),
  };

  return <Chat key={id} id={chat.id} initialMessages={chat.messages} />;
}
```

#### After: Pass chat messages as props and load into chat hook

```tsx filename="@/app/components/chat.tsx"
'use client';

import { Message } from 'ai';
import { useChat } from '@ai-sdk/react';

export function Chat({
  id,
  initialMessages,
}: {
  id;
  initialMessages: Array<Message>;
}) {
  const { messages } = useChat({
    id,
    initialMessages,
  });

  return (
    <div>
      {messages.map(message => (
        <div key={message.id}>
          <div>{message.role}</div>
          <div>{message.content}</div>
        </div>
      ))}
    </div>
  );
}
```

## Streaming Object Generation

The `createStreamableValue` function streams any serializable data from the server to the client. As a result, this function allows you to stream object generations from the server to the client when paired with `streamObject`.

#### Before: Use streamable value to stream object generations

```ts filename="@/app/actions.ts"
import { streamObject } from 'ai';
import { openai } from '@ai-sdk/openai';
import { createStreamableValue } from '@ai-sdk/rsc';
import { notificationsSchema } from '@/utils/schemas';

export async function generateSampleNotifications() {
  'use server';

  const stream = createStreamableValue();

  (async () => {
    const { partialObjectStream } = streamObject({
      model: openai('gpt-4o'),
      system: 'generate sample ios messages for testing',
      prompt: 'messages from a family group chat during diwali, max 4',
      schema: notificationsSchema,
    });

    for await (const partialObject of partialObjectStream) {
      stream.update(partialObject);
    }
  })();

  stream.done();

  return { partialNotificationsStream: stream.value };
}
```

#### Before: Read streamable value and update object

```tsx filename="@/app/page.tsx"
'use client';

import { useState } from 'react';
import { readStreamableValue } from '@ai-sdk/rsc';
import { generateSampleNotifications } from '@/app/actions';

export default function Page() {
  const [notifications, setNotifications] = useState(null);

  return (
    <div>
      <button
        onClick={async () => {
          const { partialNotificationsStream } =
            await generateSampleNotifications();

          for await (const partialNotifications of readStreamableValue(
            partialNotificationsStream,
          )) {
            if (partialNotifications) {
              setNotifications(partialNotifications.notifications);
            }
          }
        }}
      >
        Generate
      </button>
    </div>
  );
}
```

To migrate to AI SDK UI, you should use the `useObject` hook and implement `streamObject` within your route handler.

#### After: Replace with route handler and stream text response

```ts filename="@/app/api/object/route.ts"
import { streamObject } from 'ai';
import { openai } from '@ai-sdk/openai';
import { notificationSchema } from '@/utils/schemas';

export async function POST(req: Request) {
  const context = await req.json();

  const result = streamObject({
    model: openai('gpt-4.1'),
    schema: notificationSchema,
    prompt:
      `Generate 3 notifications for a messages app in this context:` + context,
  });

  return result.toTextStreamResponse();
}
```

#### After: Use object hook to decode stream and update object

```tsx filename="@/app/page.tsx"
'use client';

import { useObject } from '@ai-sdk/react';
import { notificationSchema } from '@/utils/schemas';

export default function Page() {
  const { object, submit } = useObject({
    api: '/api/object',
    schema: notificationSchema,
  });

  return (
    <div>
      <button onClick={() => submit('Messages during finals week.')}>
        Generate notifications
      </button>

      {object?.notifications?.map((notification, index) => (
        <div key={index}>
          <p>{notification?.name}</p>
          <p>{notification?.message}</p>
        </div>
      ))}
    </div>
  );
}
```
