
# Prompt Engineering

## What is a Large Language Model (LLM)?

A Large Language Model is essentially a prediction engine that takes a sequence of words as input and aims to predict the most likely sequence to follow. It does this by assigning probabilities to potential next sequences and then selecting one. The model continues to generate sequences until it meets a specified stopping criterion.

These models learn by training on massive text corpuses, which means they will be better suited to some use cases than others. For example, a model trained on GitHub data would understand the probabilities of sequences in source code particularly well. However, it's crucial to understand that the generated sequences, while often seeming plausible, can sometimes be random and not grounded in reality. As these models become more accurate, many surprising abilities and applications emerge.

## What is a prompt?

Prompts are the starting points for LLMs. They are the inputs that trigger the model to generate text. The scope of prompt engineering involves not just crafting these prompts but also understanding related concepts such as hidden prompts, tokens, token limits, and the potential for prompt hacking, which includes phenomena like jailbreaks and leaks.

## Why is prompt engineering needed?

Prompt engineering currently plays a pivotal role in shaping the responses of LLMs. It allows us to tweak the model to respond more effectively to a broader range of queries. This includes the use of techniques like semantic search, command grammars, and the ReActive model architecture. The performance, context window, and cost of LLMs varies between models and model providers which adds further constraints to the mix. For example, the GPT-4 model is more expensive than GPT-3.5-turbo and significantly slower, but it can also be more effective at certain tasks. And so, like many things in software engineering, there is a trade-offs between cost and performance.

To assist with comparing and tweaking LLMs, we've built an AI playground that allows you to compare the performance of different models side-by-side online. When you're ready, you can even generate code with the AI SDK to quickly use your prompt and your selected model into your own applications.

## Example: Build a Slogan Generator

### Start with an instruction

Imagine you want to build a slogan generator for marketing campaigns. Creating catchy slogans isn't always straightforward!

First, you'll need a prompt that makes it clear what you want. Let's start with an instruction. Submit this prompt to generate your first completion.

<InlinePrompt initialInput="Create a slogan for a coffee shop." />

Not bad! Now, try making your instruction more specific.

<InlinePrompt initialInput="Create a slogan for an organic coffee shop." />

Introducing a single descriptive term to our prompt influences the completion. Essentially, crafting your prompt is the means by which you "instruct" or "program" the model.

### Include examples

Clear instructions are key for quality outcomes, but that might not always be enough. Let's try to enhance your instruction further.

<InlinePrompt initialInput="Create three slogans for a coffee shop with live music." />

These slogans are fine, but could be even better. It appears the model overlooked the 'live' part in our prompt. Let's change it slightly to generate more appropriate suggestions.

Often, it's beneficial to both demonstrate and tell the model your requirements. Incorporating examples in your prompt can aid in conveying patterns or subtleties. Test this prompt that carries a few examples.

<InlinePrompt
  initialInput={`Create three slogans for a business with unique features.

Business: Bookstore with cats
Slogans: "Purr-fect Pages", "Books and Whiskers", "Novels and Nuzzles"
Business: Gym with rock climbing
Slogans: "Peak Performance", "Reach New Heights", "Climb Your Way Fit"
Business: Coffee shop with live music
Slogans:`}
/>

Great! Incorporating examples of expected output for a certain input prompted the model to generate the kind of names we aimed for.

### Tweak your settings

Apart from designing prompts, you can influence completions by tweaking model settings. A crucial setting is the **temperature**.

You might have seen that the same prompt, when repeated, yielded the same or nearly the same completions. This happens when your temperature is at 0.

Attempt to re-submit the identical prompt a few times with temperature set to 1.

<InlinePrompt
  initialInput={`Create three slogans for a business with unique features.

Business: Bookstore with cats
Slogans: "Purr-fect Pages", "Books and Whiskers", "Novels and Nuzzles"
Business: Gym with rock climbing
Slogans: "Peak Performance", "Reach New Heights", "Climb Your Way Fit"
Business: Coffee shop with live music
Slogans:`}
showTemp={true}
initialTemperature={1}
/>

Notice the difference? With a temperature above 0, the same prompt delivers varied completions each time.

Keep in mind that the model forecasts the text most likely to follow the preceding text. Temperature, a value from 0 to 1, essentially governs the model's confidence level in making these predictions. A lower temperature implies lesser risks, leading to more precise and deterministic completions. A higher temperature yields a broader range of completions.

For your slogan generator, you might want a large pool of name suggestions. A moderate temperature of 0.6 should serve well.

## Recommended Resources

Prompt Engineering is evolving rapidly, with new methods and research papers surfacing every week. Here are some resources that we've found useful for learning about and experimenting with prompt engineering:

- [The Vercel AI Playground](/playground)
- [Brex Prompt Engineering](https://github.com/brexhq/prompt-engineering)
- [Prompt Engineering Guide by Dair AI](https://www.promptingguide.ai/)



# Stopping Streams

Cancelling ongoing streams is often needed.
For example, users might want to stop a stream when they realize that the response is not what they want.

The different parts of the AI SDK support cancelling streams in different ways.

## AI SDK Core

The AI SDK functions have an `abortSignal` argument that you can use to cancel a stream.
You would use this if you want to cancel a stream from the server side to the LLM API, e.g. by
forwarding the `abortSignal` from the request.

```tsx highlight="10,11,12-16"
import { openai } from '@ai-sdk/openai';
import { streamText } from 'ai';

export async function POST(req: Request) {
  const { prompt } = await req.json();

  const result = streamText({
    model: openai('gpt-4.1'),
    prompt,
    // forward the abort signal:
    abortSignal: req.signal,
    onAbort: ({ steps }) => {
      // Handle cleanup when stream is aborted
      console.log('Stream aborted after', steps.length, 'steps');
      // Persist partial results to database
    },
  });

  return result.toTextStreamResponse();
}
```

## AI SDK UI

The hooks, e.g. `useChat` or `useCompletion`, provide a `stop` helper function that can be used to cancel a stream.
This will cancel the stream from the client side to the server.

<Note type="warning">
  Stream abort functionality is not compatible with stream resumption. If you're
  using `resume: true` in `useChat`, the abort functionality will break the
  resumption mechanism. Choose either abort or resume functionality, but not
  both.
</Note>

```tsx file="app/page.tsx" highlight="9,18-20"
'use client';

import { useCompletion } from '@ai-sdk/react';

export default function Chat() {
  const { input, completion, stop, status, handleSubmit, handleInputChange } =
    useCompletion();

  return (
    <div>
      {(status === 'submitted' || status === 'streaming') && (
        <button type="button" onClick={() => stop()}>
          Stop
        </button>
      )}
      {completion}
      <form onSubmit={handleSubmit}>
        <input value={input} onChange={handleInputChange} />
      </form>
    </div>
  );
}
```

## Handling stream abort cleanup

When streams are aborted, you may need to perform cleanup operations such as persisting partial results or cleaning up resources. The `onAbort` callback provides a way to handle these scenarios on the server side.

Unlike `onFinish`, which is called when a stream completes normally, `onAbort` is specifically called when a stream is aborted via `AbortSignal`. This distinction allows you to handle normal completion and aborted streams differently.

<Note>
  For UI message streams (`toUIMessageStreamResponse`), the `onFinish` callback
  also receives an `isAborted` parameter that indicates whether the stream was
  aborted. This allows you to handle both completion and abort scenarios in a
  single callback.
</Note>

```tsx highlight="8-12"
import { streamText } from 'ai';

const result = streamText({
  model: openai('gpt-4.1'),
  prompt: 'Write a long story...',
  abortSignal: controller.signal,
  onAbort: ({ steps }) => {
    // Called when stream is aborted - persist partial results
    await savePartialResults(steps);
    await logAbortEvent(steps.length);
  },
  onFinish: ({ steps, totalUsage }) => {
    // Called when stream completes normally
    await saveFinalResults(steps, totalUsage);
  },
});
```

The `onAbort` callback receives:

- `steps`: Array of all completed steps before the abort occurred

This is particularly useful for:

- Persisting partial conversation history to database
- Saving partial progress for later continuation
- Cleaning up server-side resources or connections
- Logging abort events for analytics

You can also handle abort events directly in the stream using the `abort` stream part:

```tsx highlight="8-12"
for await (const part of result.fullStream) {
  switch (part.type) {
    case 'text-delta':
      // Handle text delta content
      break;
    case 'abort':
      // Handle abort event directly in stream
      console.log('Stream was aborted');
      break;
    // ... other cases
  }
}
```

## UI Message Streams

When using `toUIMessageStreamResponse`, you need to handle stream abortion slightly differently. The `onFinish` callback receives an `isAborted` parameter, and you should pass the `consumeStream` function to ensure proper abort handling:

```tsx highlight="5,19,20-24,26"
import { openai } from '@ai-sdk/openai';
import {
  consumeStream,
  convertToModelMessages,
  streamText,
  UIMessage,
} from 'ai';

export async function POST(req: Request) {
  const { messages }: { messages: UIMessage[] } = await req.json();

  const result = streamText({
    model: openai('gpt-4o'),
    messages: convertToModelMessages(messages),
    abortSignal: req.signal,
  });

  return result.toUIMessageStreamResponse({
    onFinish: async ({ isAborted }) => {
      if (isAborted) {
        console.log('Stream was aborted');
        // Handle abort-specific cleanup
      } else {
        console.log('Stream completed normally');
        // Handle normal completion
      }
    },
    consumeSseStream: consumeStream,
  });
}
```

The `consumeStream` function is necessary for proper abort handling in UI message streams. It ensures that the stream is properly consumed even when aborted, preventing potential memory leaks or hanging connections.

## AI SDK RSC

<Note type="warning">
  The AI SDK RSC does not currently support stopping streams.
</Note>




# Stream Back-pressure and Cancellation

This page focuses on understanding back-pressure and cancellation when working with streams. You do not need to know this information to use the AI SDK, but for those interested, it offers a deeper dive on why and how the SDK optimally streams responses.

In the following sections, we'll explore back-pressure and cancellation in the context of a simple example program. We'll discuss the issues that can arise from an eager approach and demonstrate how a lazy approach can resolve them.

## Back-pressure and Cancellation with Streams

Let's begin by setting up a simple example program:

```jsx
// A generator that will yield positive integers
async function* integers() {
  let i = 1;
  while (true) {
    console.log(`yielding ${i}`);
    yield i++;

    await sleep(100);
  }
}
function sleep(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}

// Wraps a generator into a ReadableStream
function createStream(iterator) {
  return new ReadableStream({
    async start(controller) {
      for await (const v of iterator) {
        controller.enqueue(v);
      }
      controller.close();
    },
  });
}

// Collect data from stream
async function run() {
  // Set up a stream of integers
  const stream = createStream(integers());

  // Read values from our stream
  const reader = stream.getReader();
  for (let i = 0; i < 10_000; i++) {
    // we know our stream is infinite, so there's no need to check `done`.
    const { value } = await reader.read();
    console.log(`read ${value}`);

    await sleep(1_000);
  }
}
run();
```

In this example, we create an async-generator that yields positive integers, a `ReadableStream` that wraps our integer generator, and a reader which will read values out of our stream. Notice, too, that our integer generator logs out `"yielding ${i}"`, and our reader logs out `"read ${value}"`. Both take an arbitrary amount of time to process data, represented with a 100ms sleep in our generator, and a 1sec sleep in our reader.

## Back-pressure

If you were to run this program, you'd notice something funny. We'll see roughly 10 "yield" logs for every "read" log. This might seem obvious, the generator can push values 10x faster than the reader can pull them out. But it represents a problem, our `stream` has to maintain an ever expanding queue of items that have been pushed in but not pulled out.

The problem stems from the way we wrap our generator into a stream. Notice the use of `for await (…)` inside our `start` handler. This is an **eager** for-loop, and it is constantly running to get the next value from our generator to be enqueued in our stream. This means our stream does not respect back-pressure, the signal from the consumer to the producer that more values aren't needed _yet_. We've essentially spawned a thread that will perpetually push more data into the stream, one that runs as fast as possible to push new data immediately. Worse, there's no way to signal to this thread to stop running when we don't need additional data.

To fix this, `ReadableStream` allows a `pull` handler. `pull` is called every time the consumer attempts to read more data from our stream (if there's no data already queued internally). But it's not enough to just move the `for await(…)` into `pull`, we also need to convert from an eager enqueuing to a **lazy** one. By making these 2 changes, we'll be able to react to the consumer. If they need more data, we can easily produce it, and if they don't, then we don't need to spend any time doing unnecessary work.

```jsx
function createStream(iterator) {
  return new ReadableStream({
    async pull(controller) {
      const { value, done } = await iterator.next();

      if (done) {
        controller.close();
      } else {
        controller.enqueue(value);
      }
    },
  });
}
```

Our `createStream` is a little more verbose now, but the new code is important. First, we need to manually call our `iterator.next()` method. This returns a `Promise` for an object with the type signature `{ done: boolean, value: T }`. If `done` is `true`, then we know that our iterator won't yield any more values and we must `close` the stream (this allows the consumer to know that the stream is also finished producing values). Else, we need to `enqueue` our newly produced value.

When we run this program, we see that our "yield" and "read" logs are now paired. We're no longer yielding 10x integers for every read! And, our stream now only needs to maintain 1 item in its internal buffer. We've essentially given control to the consumer, so that it's responsible for producing new values as it needs it. Neato!

## Cancellation

Let's go back to our initial eager example, with 1 small edit. Now instead of reading 10,000 integers, we're only going to read 3:

```jsx
// A generator that will yield positive integers
async function* integers() {
  let i = 1;
  while (true) {
    console.log(`yielding ${i}`);
    yield i++;

    await sleep(100);
  }
}
function sleep(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}

// Wraps a generator into a ReadableStream
function createStream(iterator) {
  return new ReadableStream({
    async start(controller) {
      for await (const v of iterator) {
        controller.enqueue(v);
      }
      controller.close();
    },
  });
}
// Collect data from stream
async function run() {
  // Set up a stream that of integers
  const stream = createStream(integers());

  // Read values from our stream
  const reader = stream.getReader();
  // We're only reading 3 items this time:
  for (let i = 0; i < 3; i++) {
    // we know our stream is infinite, so there's no need to check `done`.
    const { value } = await reader.read();
    console.log(`read ${value}`);

    await sleep(1000);
  }
}
run();
```

We're back to yielding 10x the number of values read. But notice now, after we've read 3 values, we're continuing to yield new values. We know that our reader will never read another value, but our stream doesn't! The eager `for await (…)` will continue forever, loudly enqueuing new values into our stream's buffer and increasing our memory usage until it consumes all available program memory.

The fix to this is exactly the same: use `pull` and manual iteration. By producing values _**lazily**_, we tie the lifetime of our integer generator to the lifetime of the reader. Once the reads stop, the yields will stop too:

```jsx
// Wraps a generator into a ReadableStream
function createStream(iterator) {
  return new ReadableStream({
    async pull(controller) {
      const { value, done } = await iterator.next();

      if (done) {
        controller.close();
      } else {
        controller.enqueue(value);
      }
    },
  });
}
```

Since the solution is the same as implementing back-pressure, it shows that they're just 2 facets of the same problem: Pushing values into a stream should be done **lazily**, and doing it eagerly results in expected problems.

## Tying Stream Laziness to AI Responses

Now let's imagine you're integrating AIBot service into your product. Users will be able to prompt "count from 1 to infinity", the browser will fetch your AI API endpoint, and your servers connect to AIBot to get a response. But "infinity" is, well, infinite. The response will never end!

After a few seconds, the user gets bored and navigates away. Or maybe you're doing local development and a hot-module reload refreshes your page. The browser will have ended its connection to the API endpoint, but will your server end its connection with AIBot?

If you used the eager `for await (...)` approach, then the connection is still running and your server is asking for more and more data from AIBot. Our server spawned a "thread" and there's no signal when we can end the eager pulls. Eventually, the server is going to run out of memory (remember, there's no active fetch connection to read the buffering responses and free them).

{/* When we started writing the streaming code for the AI SDK, we confirm aborting a fetch will end a streamed response from Next.js */}

With the lazy approach, this is taken care of for you. Because the stream will only request new data from AIBot when the consumer requests it, navigating away from the page naturally frees all resources. The fetch connection aborts and the server can clean up the response. The `ReadableStream` tied to that response can now be garbage collected. When that happens, the connection it holds to AIBot can then be freed.



# Caching Responses

Depending on the type of application you're building, you may want to cache the responses you receive from your AI provider, at least temporarily.

## Using Language Model Middleware (Recommended)

The recommended approach to caching responses is using [language model middleware](/docs/ai-sdk-core/middleware)
and the [`simulateReadableStream`](/docs/reference/ai-sdk-core/simulate-readable-stream) function.

Language model middleware is a way to enhance the behavior of language models by intercepting and modifying the calls to the language model.
Let's see how you can use language model middleware to cache responses.

```ts filename="ai/middleware.ts"
import { Redis } from '@upstash/redis';
import {
  type LanguageModelV2,
  type LanguageModelV2Middleware,
  type LanguageModelV2StreamPart,
  simulateReadableStream,
} from 'ai';

const redis = new Redis({
  url: process.env.KV_URL,
  token: process.env.KV_TOKEN,
});

export const cacheMiddleware: LanguageModelV2Middleware = {
  wrapGenerate: async ({ doGenerate, params }) => {
    const cacheKey = JSON.stringify(params);

    const cached = (await redis.get(cacheKey)) as Awaited<
      ReturnType<LanguageModelV2['doGenerate']>
    > | null;

    if (cached !== null) {
      return {
        ...cached,
        response: {
          ...cached.response,
          timestamp: cached?.response?.timestamp
            ? new Date(cached?.response?.timestamp)
            : undefined,
        },
      };
    }

    const result = await doGenerate();

    redis.set(cacheKey, result);

    return result;
  },
  wrapStream: async ({ doStream, params }) => {
    const cacheKey = JSON.stringify(params);

    // Check if the result is in the cache
    const cached = await redis.get(cacheKey);

    // If cached, return a simulated ReadableStream that yields the cached result
    if (cached !== null) {
      // Format the timestamps in the cached response
      const formattedChunks = (cached as LanguageModelV2StreamPart[]).map(p => {
        if (p.type === 'response-metadata' && p.timestamp) {
          return { ...p, timestamp: new Date(p.timestamp) };
        } else return p;
      });
      return {
        stream: simulateReadableStream({
          initialDelayInMs: 0,
          chunkDelayInMs: 10,
          chunks: formattedChunks,
        }),
      };
    }

    // If not cached, proceed with streaming
    const { stream, ...rest } = await doStream();

    const fullResponse: LanguageModelV2StreamPart[] = [];

    const transformStream = new TransformStream<
      LanguageModelV2StreamPart,
      LanguageModelV2StreamPart
    >({
      transform(chunk, controller) {
        fullResponse.push(chunk);
        controller.enqueue(chunk);
      },
      flush() {
        // Store the full response in the cache after streaming is complete
        redis.set(cacheKey, fullResponse);
      },
    });

    return {
      stream: stream.pipeThrough(transformStream),
      ...rest,
    };
  },
};
```

<Note>
  This example uses `@upstash/redis` to store and retrieve the assistant's
  responses but you can use any KV storage provider you would like.
</Note>

`LanguageModelMiddleware` has two methods: `wrapGenerate` and `wrapStream`. `wrapGenerate` is called when using [`generateText`](/docs/reference/ai-sdk-core/generate-text) and [`generateObject`](/docs/reference/ai-sdk-core/generate-object), while `wrapStream` is called when using [`streamText`](/docs/reference/ai-sdk-core/stream-text) and [`streamObject`](/docs/reference/ai-sdk-core/stream-object).

For `wrapGenerate`, you can cache the response directly. Instead, for `wrapStream`, you cache an array of the stream parts, which can then be used with [`simulateReadableStream`](/docs/ai-sdk-core/testing#simulate-data-stream-protocol-responses) function to create a simulated `ReadableStream` that returns the cached response. In this way, the cached response is returned chunk-by-chunk as if it were being generated by the model. You can control the initial delay and delay between chunks by adjusting the `initialDelayInMs` and `chunkDelayInMs` parameters of `simulateReadableStream`.

You can see a full example of caching with Redis in a Next.js application in our [Caching Middleware Recipe](/cookbook/next/caching-middleware).

## Using Lifecycle Callbacks

Alternatively, each AI SDK Core function has special lifecycle callbacks you can use. The one of interest is likely `onFinish`, which is called when the generation is complete. This is where you can cache the full response.

Here's an example of how you can implement caching using Vercel KV and Next.js to cache the OpenAI response for 1 hour:

This example uses [Upstash Redis](https://upstash.com/docs/redis/overall/getstarted) and Next.js to cache the response for 1 hour.

```tsx filename="app/api/chat/route.ts"
import { openai } from '@ai-sdk/openai';
import { formatDataStreamPart, streamText, UIMessage } from 'ai';
import { Redis } from '@upstash/redis';

// Allow streaming responses up to 30 seconds
export const maxDuration = 30;

const redis = new Redis({
  url: process.env.KV_URL,
  token: process.env.KV_TOKEN,
});

export async function POST(req: Request) {
  const { messages }: { messages: UIMessage[] } = await req.json();

  // come up with a key based on the request:
  const key = JSON.stringify(messages);

  // Check if we have a cached response
  const cached = await redis.get(key);
  if (cached != null) {
    return new Response(formatDataStreamPart('text', cached), {
      status: 200,
      headers: { 'Content-Type': 'text/plain' },
    });
  }

  // Call the language model:
  const result = streamText({
    model: openai('gpt-4o'),
    messages: convertToModelMessages(messages),
    async onFinish({ text }) {
      // Cache the response text:
      await redis.set(key, text);
      await redis.expire(key, 60 * 60);
    },
  });

  // Respond with the stream
  return result.toUIMessageStreamResponse();
}
```

# Multiple Streams

## Multiple Streamable UIs

The AI SDK RSC APIs allow you to compose and return any number of streamable UIs, along with other data, in a single request. This can be useful when you want to decouple the UI into smaller components and stream them separately.

```tsx file='app/actions.tsx'
'use server';

import { createStreamableUI } from '@ai-sdk/rsc';

export async function getWeather() {
  const weatherUI = createStreamableUI();
  const forecastUI = createStreamableUI();

  weatherUI.update(<div>Loading weather...</div>);
  forecastUI.update(<div>Loading forecast...</div>);

  getWeatherData().then(weatherData => {
    weatherUI.done(<div>{weatherData}</div>);
  });

  getForecastData().then(forecastData => {
    forecastUI.done(<div>{forecastData}</div>);
  });

  // Return both streamable UIs and other data fields.
  return {
    requestedAt: Date.now(),
    weather: weatherUI.value,
    forecast: forecastUI.value,
  };
}
```

The client side code is similar to the previous example, but the [tool call](/docs/ai-sdk-core/tools-and-tool-calling) will return the new data structure with the weather and forecast UIs. Depending on the speed of getting weather and forecast data, these two components might be updated independently.

## Nested Streamable UIs

You can stream UI components within other UI components. This allows you to create complex UIs that are built up from smaller, reusable components. In the example below, we pass a `historyChart` streamable as a prop to a `StockCard` component. The StockCard can render the `historyChart` streamable, and it will automatically update as the server responds with new data.

```tsx file='app/actions.tsx'
async function getStockHistoryChart({ symbol: string }) {
  'use server';

  const ui = createStreamableUI(<Spinner />);

  // We need to wrap this in an async IIFE to avoid blocking.
  (async () => {
    const price = await getStockPrice({ symbol });

    // Show a spinner as the history chart for now.
    const historyChart = createStreamableUI(<Spinner />);
    ui.done(<StockCard historyChart={historyChart.value} price={price} />);

    // Getting the history data and then update that part of the UI.
    const historyData = await fetch('https://my-stock-data-api.com');
    historyChart.done(<HistoryChart data={historyData} />);
  })();

  return ui;
}
```

# Rate Limiting

Rate limiting helps you protect your APIs from abuse. It involves setting a
maximum threshold on the number of requests a client can make within a
specified timeframe. This simple technique acts as a gatekeeper,
preventing excessive usage that can degrade service performance and incur
unnecessary costs.

## Rate Limiting with Vercel KV and Upstash Ratelimit

In this example, you will protect an API endpoint using [Vercel KV](https://vercel.com/storage/kv)
and [Upstash Ratelimit](https://github.com/upstash/ratelimit).

```tsx filename='app/api/generate/route.ts'
import kv from '@vercel/kv';
import { openai } from '@ai-sdk/openai';
import { streamText } from 'ai';
import { Ratelimit } from '@upstash/ratelimit';
import { NextRequest } from 'next/server';

// Allow streaming responses up to 30 seconds
export const maxDuration = 30;

// Create Rate limit
const ratelimit = new Ratelimit({
  redis: kv,
  limiter: Ratelimit.fixedWindow(5, '30s'),
});

export async function POST(req: NextRequest) {
  // call ratelimit with request ip
  const ip = req.ip ?? 'ip';
  const { success, remaining } = await ratelimit.limit(ip);

  // block the request if unsuccessfull
  if (!success) {
    return new Response('Ratelimited!', { status: 429 });
  }

  const { messages } = await req.json();

  const result = streamText({
    model: openai('gpt-3.5-turbo'),
    messages,
  });

  return result.toUIMessageStreamResponse();
}
```

## Simplify API Protection

With Vercel KV and Upstash Ratelimit, it is possible to protect your APIs
from such attacks with ease. To learn more about how Ratelimit works and
how it can be configured to your needs, see [Ratelimit Documentation](https://upstash.com/docs/oss/sdks/ts/ratelimit/overview).




# Rendering User Interfaces with Language Models

Language models generate text, so at first it may seem like you would only need to render text in your application.

```tsx highlight="16" filename="app/actions.tsx"
const text = generateText({
  model: openai('gpt-3.5-turbo'),
  system: 'You are a friendly assistant',
  prompt: 'What is the weather in SF?',
  tools: {
    getWeather: {
      description: 'Get the weather for a location',
      parameters: z.object({
        city: z.string().describe('The city to get the weather for'),
        unit: z
          .enum(['C', 'F'])
          .describe('The unit to display the temperature in'),
      }),
      execute: async ({ city, unit }) => {
        const weather = getWeather({ city, unit });
        return `It is currently ${weather.value}°${unit} and ${weather.description} in ${city}!`;
      },
    },
  },
});
```

Above, the language model is passed a [tool](/docs/ai-sdk-core/tools-and-tool-calling) called `getWeather` that returns the weather information as text. However, instead of returning text, if you return a JSON object that represents the weather information, you can use it to render a React component instead.

```tsx highlight="18-23" filename="app/action.ts"
const text = generateText({
  model: openai('gpt-3.5-turbo'),
  system: 'You are a friendly assistant',
  prompt: 'What is the weather in SF?',
  tools: {
    getWeather: {
      description: 'Get the weather for a location',
      parameters: z.object({
        city: z.string().describe('The city to get the weather for'),
        unit: z
          .enum(['C', 'F'])
          .describe('The unit to display the temperature in'),
      }),
      execute: async ({ city, unit }) => {
        const weather = getWeather({ city, unit });
        const { temperature, unit, description, forecast } = weather;

        return {
          temperature,
          unit,
          description,
          forecast,
        };
      },
    },
  },
});
```

Now you can use the object returned by the `getWeather` function to conditionally render a React component `<WeatherCard/>` that displays the weather information by passing the object as props.

```tsx filename="app/page.tsx"
return (
  <div>
    {messages.map(message => {
      if (message.role === 'function') {
        const { name, content } = message
        const { temperature, unit, description, forecast } = content;

        return (
          <WeatherCard
            weather={{
              temperature: 47,
              unit: 'F',
              description: 'sunny'
              forecast,
            }}
          />
        )
      }
    })}
  </div>
)
```

Here's a little preview of what that might look like.

<div className="not-prose flex flex-col2">
  <CardPlayer
    type="weather"
    title="Weather"
    description="An example of an assistant that renders the weather information in a streamed component."
  />
</div>

Rendering interfaces as part of language model generations elevates the user experience of your application, allowing people to interact with language models beyond text.

They also make it easier for you to interpret [sequential tool calls](/docs/ai-sdk-rsc/multistep-interfaces) that take place in multiple steps and help identify and debug where the model reasoned incorrectly.

## Rendering Multiple User Interfaces

To recap, an application has to go through the following steps to render user interfaces as part of model generations:

1. The user prompts the language model.
2. The language model generates a response that includes a tool call.
3. The tool call returns a JSON object that represents the user interface.
4. The response is sent to the client.
5. The client receives the response and checks if the latest message was a tool call.
6. If it was a tool call, the client renders the user interface based on the JSON object returned by the tool call.

Most applications have multiple tools that are called by the language model, and each tool can return a different user interface.

For example, a tool that searches for courses can return a list of courses, while a tool that searches for people can return a list of people. As this list grows, the complexity of your application will grow as well and it can become increasingly difficult to manage these user interfaces.

```tsx filename='app/page.tsx'
{
  message.role === 'tool' ? (
    message.name === 'api-search-course' ? (
      <Courses courses={message.content} />
    ) : message.name === 'api-search-profile' ? (
      <People people={message.content} />
    ) : message.name === 'api-meetings' ? (
      <Meetings meetings={message.content} />
    ) : message.name === 'api-search-building' ? (
      <Buildings buildings={message.content} />
    ) : message.name === 'api-events' ? (
      <Events events={message.content} />
    ) : message.name === 'api-meals' ? (
      <Meals meals={message.content} />
    ) : null
  ) : (
    <div>{message.content}</div>
  );
}
```

## Rendering User Interfaces on the Server

The **AI SDK RSC (`@ai-sdk/rsc`)** takes advantage of RSCs to solve the problem of managing all your React components on the client side, allowing you to render React components on the server and stream them to the client.

Rather than conditionally rendering user interfaces on the client based on the data returned by the language model, you can directly stream them from the server during a model generation.

```tsx highlight="3,22-31,38" filename="app/action.ts"
import { createStreamableUI } from '@ai-sdk/rsc'

const uiStream = createStreamableUI();

const text = generateText({
  model: openai('gpt-3.5-turbo'),
  system: 'you are a friendly assistant'
  prompt: 'what is the weather in SF?'
  tools: {
    getWeather: {
      description: 'Get the weather for a location',
      parameters: z.object({
        city: z.string().describe('The city to get the weather for'),
        unit: z
          .enum(['C', 'F'])
          .describe('The unit to display the temperature in')
      }),
      execute: async ({ city, unit }) => {
        const weather = getWeather({ city, unit })
        const { temperature, unit, description, forecast } = weather

        uiStream.done(
          <WeatherCard
            weather={{
              temperature: 47,
              unit: 'F',
              description: 'sunny'
              forecast,
            }}
          />
        )
      }
    }
  }
})

return {
  display: uiStream.value
}
```

The [`createStreamableUI`](/docs/reference/ai-sdk-rsc/create-streamable-ui) function belongs to the `@ai-sdk/rsc` module and creates a stream that can send React components to the client.

On the server, you render the `<WeatherCard/>` component with the props passed to it, and then stream it to the client. On the client side, you only need to render the UI that is streamed from the server.

```tsx filename="app/page.tsx" highlight="4"
return (
  <div>
    {messages.map(message => (
      <div>{message.display}</div>
    ))}
  </div>
);
```

Now the steps involved are simplified:

1. The user prompts the language model.
2. The language model generates a response that includes a tool call.
3. The tool call renders a React component along with relevant props that represent the user interface.
4. The response is streamed to the client and rendered directly.

> **Note:** You can also render text on the server and stream it to the client using React Server Components. This way, all operations from language model generation to UI rendering can be done on the server, while the client only needs to render the UI that is streamed from the server.

Check out this [example](/examples/next-app/interface/stream-component-updates) for a full illustration of how to stream component updates with React Server Components in Next.js App Router.




# Generative User Interfaces

Since language models can render user interfaces as part of their generations, the resulting model generations are referred to as generative user interfaces.

In this section we will learn more about generative user interfaces and their impact on the way AI applications are built.

## Deterministic Routes and Probabilistic Routing

Generative user interfaces are not deterministic in nature because they depend on the model's generation output. Since these generations are probabilistic in nature, it is possible for every user query to result in a different user interface.

Users expect their experience using your application to be predictable, so non-deterministic user interfaces can sound like a bad idea at first. However, language models can be set up to limit their generations to a particular set of outputs using their ability to call functions.

When language models are provided with a set of function definitions and instructed to execute any of them based on user query, they do either one of the following things:

- Execute a function that is most relevant to the user query.
- Not execute any function if the user query is out of bounds of the set of functions available to them.

```tsx filename='app/actions.ts'
const sendMessage = (prompt: string) =>
  generateText({
    model: 'gpt-3.5-turbo',
    system: 'you are a friendly weather assistant!',
    prompt,
    tools: {
      getWeather: {
        description: 'Get the weather in a location',
        parameters: z.object({
          location: z.string().describe('The location to get the weather for'),
        }),
        execute: async ({ location }: { location: string }) => ({
          location,
          temperature: 72 + Math.floor(Math.random() * 21) - 10,
        }),
      },
    },
  });

sendMessage('What is the weather in San Francisco?'); // getWeather is called
sendMessage('What is the weather in New York?'); // getWeather is called
sendMessage('What events are happening in London?'); // No function is called
```

This way, it is possible to ensure that the generations result in deterministic outputs, while the choice a model makes still remains to be probabilistic.

This emergent ability exhibited by a language model to choose whether a function needs to be executed or not based on a user query is believed to be models emulating "reasoning".

As a result, the combination of language models being able to reason which function to execute as well as render user interfaces at the same time gives you the ability to build applications where language models can be used as a router.

## Language Models as Routers

Historically, developers had to write routing logic that connected different parts of an application to be navigable by a user and complete a specific task.

In web applications today, most of the routing logic takes place in the form of routes:

- `/login` would navigate you to a page with a login form.
- `/user/john` would navigate you to a page with profile details about John.
- `/api/events?limit=5` would display the five most recent events from an events database.

While routes help you build web applications that connect different parts of an application into a seamless user experience, it can also be a burden to manage them as the complexity of applications grow.

Next.js has helped reduce complexity in developing with routes by introducing:

- File-based routing system
- Dynamic routing
- API routes
- Middleware
- App router, and so on...

With language models becoming better at reasoning, we believe that there is a future where developers only write core application specific components while models take care of routing them based on the user's state in an application.

With generative user interfaces, the language model decides which user interface to render based on the user's state in the application, giving users the flexibility to interact with your application in a conversational manner instead of navigating through a series of predefined routes.

### Routing by parameters

For routes like:

- `/profile/[username]`
- `/search?q=[query]`
- `/media/[id]`

that have segments dependent on dynamic data, the language model can generate the correct parameters and render the user interface.

For example, when you're in a search application, you can ask the language model to search for artworks from different artists. The language model will call the search function with the artist's name as a parameter and render the search results.

<div className="not-prose">
  <CardPlayer
    type="media-search"
    title="Media Search"
    description="Let your users see more than words can say by rendering components directly within your search experience."
  />
</div>

### Routing by sequence

For actions that require a sequence of steps to be completed by navigating through different routes, the language model can generate the correct sequence of routes to complete in order to fulfill the user's request.

For example, when you're in a calendar application, you can ask the language model to schedule a happy hour evening with your friends. The language model will then understand your request and will perform the right sequence of [tool calls](/docs/ai-sdk-core/tools-and-tool-calling) to:

1. Lookup your calendar
2. Lookup your friends' calendars
3. Determine the best time for everyone
4. Search for nearby happy hour spots
5. Create an event and send out invites to your friends

<div className="not-prose">
  <CardPlayer
    type="event-planning"
    title="Planning an Event"
    description="The model calls functions and generates interfaces based on user intent, acting like a router."
  />
</div>

Just by defining functions to lookup contacts, pull events from a calendar, and search for nearby locations, the model is able to sequentially navigate the routes for you.

To learn more, check out these [examples](/examples/next-app/interface) using the `streamUI` function to stream generative user interfaces to the client based on the response from the language model.



# Multistep Interfaces

Multistep interfaces refer to user interfaces that require multiple independent steps to be executed in order to complete a specific task.

In order to understand multistep interfaces, it is important to understand two concepts:

- Tool composition
- Application context

**Tool composition** is the process of combining multiple [tools](/docs/ai-sdk-core/tools-and-tool-calling) to create a new tool. This is a powerful concept that allows you to break down complex tasks into smaller, more manageable steps.

**Application context** refers to the state of the application at any given point in time. This includes the user's input, the output of the language model, and any other relevant information.

When designing multistep interfaces, you need to consider how the tools in your application can be composed together to form a coherent user experience as well as how the application context changes as the user progresses through the interface.

## Application Context

The application context can be thought of as the conversation history between the user and the language model. The richer the context, the more information the model has to generate relevant responses.

In the context of multistep interfaces, the application context becomes even more important. This is because **the user's input in one step may affect the output of the model in the next step**.

For example, consider a meal logging application that helps users track their daily food intake. The language model is provided with the following tools:

- `log_meal` takes in parameters like the name of the food, the quantity, and the time of consumption to log a meal.
- `delete_meal` takes in the name of the meal to be deleted.

When the user logs a meal, the model generates a response confirming the meal has been logged.

```txt highlight="2"
User: Log a chicken shawarma for lunch.
Tool: log_meal("chicken shawarma", "250g", "12:00 PM")
Model: Chicken shawarma has been logged for lunch.
```

Now when the user decides to delete the meal, the model should be able to reference the previous step to identify the meal to be deleted.

```txt highlight="7"
User: Log a chicken shawarma for lunch.
Tool: log_meal("chicken shawarma", "250g", "12:00 PM")
Model: Chicken shawarma has been logged for lunch.
...
...
User: I skipped lunch today, can you update my log?
Tool: delete_meal("chicken shawarma")
Model: Chicken shawarma has been deleted from your log.
```

In this example, managing the application context is important for the model to generate the correct response. The model needs to have information about the previous actions in order for it to use generate the parameters for the `delete_meal` tool.

## Tool Composition

Tool composition is the process of combining multiple tools to create a new tool. This involves defining the inputs and outputs of each tool, as well as how they interact with each other.

The design of how these tools can be composed together to form a multistep interface is crucial to both the user experience of your application and the model's ability to generate the correct output.

For example, consider a flight booking assistant that can help users book flights. The assistant can be designed to have the following tools:

- `searchFlights`: Searches for flights based on the user's query.
- `lookupFlight`: Looks up details of a specific flight based on the flight number.
- `bookFlight`: Books a flight based on the user's selection.

The `searchFlights` tool is called when the user wants to lookup flights for a specific route. This would typically mean the tool should be able to take in parameters like the origin and destination of the flight.

The `lookupFlight` tool is called when the user wants to get more details about a specific flight. This would typically mean the tool should be able to take in parameters like the flight number and return information about seat availability.

The `bookFlight` tool is called when the user decides to book a flight. In order to identify the flight to book, the tool should be able to take in parameters like the flight number, trip date, and passenger details.

So the conversation between the user and the model could look like this:

```txt highlight="8"
User: I want to book a flight from New York to London.
Tool: searchFlights("New York", "London")
Model: Here are the available flights from New York to London.
User: I want to book flight number BA123 on 12th December for myself and my wife.
Tool: lookupFlight("BA123") -> "4 seats available"
Model: Sure, there are seats available! Can you provide the names of the passengers?
User: John Doe and Jane Doe.
Tool: bookFlight("BA123", "12th December", ["John Doe", "Jane Doe"])
Model: Your flight has been booked!
```

In the last tool call, the `bookFlight` tool does not include passenger details as it is not available in the application context. As a result, it requests the user to provide the passenger details before proceeding with the booking.

Looking up passenger information could've been another tool that the model could've called before calling the `bookFlight` tool assuming that the user is logged into the application. This way, the model does not have to ask the user for the passenger details and can proceed with the booking.

```txt highlight="5,6"
User: I want to book a flight from New York to London.
Tool: searchFlights("New York", "London")
Model: Here are the available flights from New York to London.
User: I want to book flight number BA123 on 12th December for myself an my wife.
Tool: lookupContacts() -> ["John Doe", "Jane Doe"]
Tool: bookFlight("BA123", "12th December", ["John Doe", "Jane Doe"])
Model: Your flight has been booked!
```

The `lookupContacts` tool is called before the `bookFlight` tool to ensure that the passenger details are available in the application context when booking the flight. This way, the model can reduce the number of steps required from the user and use its ability to call tools that populate its context and use that information to complete the booking process.

Now, let's introduce another tool called `lookupBooking` that can be used to show booking details by taking in the name of the passenger as parameter. This tool can be composed with the existing tools to provide a more complete user experience.

```txt highlight="2-4"
User: What's the status of my wife's upcoming flight?
Tool: lookupContacts() -> ["John Doe", "Jane Doe"]
Tool: lookupBooking("Jane Doe") -> "BA123 confirmed"
Tool: lookupFlight("BA123") -> "Flight BA123 is scheduled to depart on 12th December."
Model: Your wife's flight BA123 is confirmed and scheduled to depart on 12th December.
```

In this example, the `lookupBooking` tool is used to provide the user with the status of their wife's upcoming flight. By composing this tool with the existing tools, the model is able to generate a response that includes the booking status and the departure date of the flight without requiring the user to provide additional information.

As a result, the more tools you design that can be composed together, the more complex and powerful your application can become.



# Sequential Generations

When working with the AI SDK, you may want to create sequences of generations (often referred to as "chains" or "pipes"), where the output of one becomes the input for the next. This can be useful for creating more complex AI-powered workflows or for breaking down larger tasks into smaller, more manageable steps.

## Example

In a sequential chain, the output of one generation is directly used as input for the next generation. This allows you to create a series of dependent generations, where each step builds upon the previous one.

Here's an example of how you can implement sequential actions:

```typescript
import { openai } from '@ai-sdk/openai';
import { generateText } from 'ai';

async function sequentialActions() {
  // Generate blog post ideas
  const ideasGeneration = await generateText({
    model: openai('gpt-4o'),
    prompt: 'Generate 10 ideas for a blog post about making spaghetti.',
  });

  console.log('Generated Ideas:\n', ideasGeneration);

  // Pick the best idea
  const bestIdeaGeneration = await generateText({
    model: openai('gpt-4o'),
    prompt: `Here are some blog post ideas about making spaghetti:
${ideasGeneration}

Pick the best idea from the list above and explain why it's the best.`,
  });

  console.log('\nBest Idea:\n', bestIdeaGeneration);

  // Generate an outline
  const outlineGeneration = await generateText({
    model: openai('gpt-4o'),
    prompt: `We've chosen the following blog post idea about making spaghetti:
${bestIdeaGeneration}

Create a detailed outline for a blog post based on this idea.`,
  });

  console.log('\nBlog Post Outline:\n', outlineGeneration);
}

sequentialActions().catch(console.error);
```

In this example, we first generate ideas for a blog post, then pick the best idea, and finally create an outline based on that idea. Each step uses the output from the previous step as input for the next generation.



# Vercel Deployment Guide

In this guide, you will deploy an AI application to [Vercel](https://vercel.com) using [Next.js](https://nextjs.org) (App Router).

Vercel is a platform for developers that provides the tools, workflows, and infrastructure you need to build and deploy your web apps faster, without the need for additional configuration.

Vercel allows for automatic deployments on every branch push and merges onto the production branch of your GitHub, GitLab, and Bitbucket projects. It is a great option for deploying your AI application.

## Before You Begin

To follow along with this guide, you will need:

- a Vercel account
- an account with a Git provider (this tutorial will use [Github](https://github.com))
- an OpenAI API key

This guide will teach you how to deploy the application you built in the Next.js (App Router) quickstart tutorial to Vercel. If you haven’t completed the quickstart guide, you can start with [this repo](https://github.com/vercel-labs/ai-sdk-deployment-guide).

## Commit Changes

Vercel offers a powerful git-centered workflow that automatically deploys your application to production every time you push to your repository’s main branch.

Before committing your local changes, make sure that you have a `.gitignore`. Within your `.gitignore`, ensure that you are excluding your environment variables (`.env`) and your node modules (`node_modules`).

If you have any local changes, you can commit them by running the following commands:

```bash
git add .
git commit -m "init"
```

## Create Git Repo

You can create a GitHub repository from within your terminal, or on [github.com](https://github.com/). For this tutorial, you will use the GitHub CLI ([more info here](https://cli.github.com/)).

To create your GitHub repository:

1. Navigate to [github.com](http://github.com/)
2. In the top right corner, click the "plus" icon and select "New repository"
3. Pick a name for your repository (this can be anything)
4. Click "Create repository"

Once you have created your repository, GitHub will redirect you to your new repository.

1. Scroll down the page and copy the commands under the title "...or push an existing repository from the command line"
2. Go back to the terminal, paste and then run the commands

Note: if you run into the error "error: remote origin already exists.", this is because your local repository is still linked to the repository you cloned. To "unlink", you can run the following command:

```bash
rm -rf .git
git init
git add .
git commit -m "init"
```

Rerun the code snippet from the previous step.

## Import Project in Vercel

On the [New Project](https://vercel.com/new) page, under the **Import Git Repository** section, select the Git provider that you would like to import your project from. Follow the prompts to sign in to your GitHub account.

Once you have signed in, you should see your newly created repository from the previous step in the "Import Git Repository" section. Click the "Import" button next to that project.

### Add Environment Variables

Your application stores uses environment secrets to store your OpenAI API key using a `.env.local` file locally in development. To add this API key to your production deployment, expand the "Environment Variables" section and paste in your `.env.local` file. Vercel will automatically parse your variables and enter them in the appropriate `key:value` format.

### Deploy

Press the **Deploy** button. Vercel will create the Project and deploy it based on the chosen configurations.

### Enjoy the confetti!

To view your deployment, select the Project in the dashboard and then select the **Domain**. This page is now visible to anyone who has the URL.

## Considerations

When deploying an AI application, there are infrastructure-related considerations to be aware of.

### Function Duration

In most cases, you will call the large language model (LLM) on the server. By default, Vercel serverless functions have a maximum duration of 10 seconds on the Hobby Tier. Depending on your prompt, it can take an LLM more than this limit to complete a response. If the response is not resolved within this limit, the server will throw an error.

You can specify the maximum duration of your Vercel function using [route segment config](https://nextjs.org/docs/app/api-reference/file-conventions/route-segment-config). To update your maximum duration, add the following route segment config to the top of your route handler or the page which is calling your server action.

```ts
export const maxDuration = 30;
```

You can increase the max duration to 60 seconds on the Hobby Tier. For other tiers, [see the documentation](https://vercel.com/docs/functions/runtimes#max-duration) for limits.

## Security Considerations

Given the high cost of calling an LLM, it's important to have measures in place that can protect your application from abuse.

### Rate Limit

Rate limiting is a method used to regulate network traffic by defining a maximum number of requests that a client can send to a server within a given time frame.

Follow [this guide](https://vercel.com/guides/securing-ai-app-rate-limiting) to add rate limiting to your application.

### Firewall

A firewall helps protect your applications and websites from DDoS attacks and unauthorized access.

[Vercel Firewall](https://vercel.com/docs/security/vercel-firewall) is a set of tools and infrastructure, created specifically with security in mind. It automatically mitigates DDoS attacks and Enterprise teams can get further customization for their site, including dedicated support and custom rules for IP blocking.

## Troubleshooting

- Streaming not working when [proxied](/docs/troubleshooting/streaming-not-working-when-proxied)
- Experiencing [Timeouts](/docs/troubleshooting/timeout-on-vercel)

