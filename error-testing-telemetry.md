
# Error Handling

## Handling regular errors

Regular errors are thrown and can be handled using the `try/catch` block.

```ts highlight="3,8-10"
import { generateText } from 'ai';

try {
  const { text } = await generateText({
    model: 'openai/gpt-4.1',
    prompt: 'Write a vegetarian lasagna recipe for 4 people.',
  });
} catch (error) {
  // handle error
}
```

See [Error Types](/docs/reference/ai-sdk-errors) for more information on the different types of errors that may be thrown.

## Handling streaming errors (simple streams)

When errors occur during streams that do not support error chunks,
the error is thrown as a regular error.
You can handle these errors using the `try/catch` block.

```ts highlight="3,12-14"
import { generateText } from 'ai';

try {
  const { textStream } = streamText({
    model: 'openai/gpt-4.1',
    prompt: 'Write a vegetarian lasagna recipe for 4 people.',
  });

  for await (const textPart of textStream) {
    process.stdout.write(textPart);
  }
} catch (error) {
  // handle error
}
```

## Handling streaming errors (streaming with `error` support)

Full streams support error parts.
You can handle those parts similar to other parts.
It is recommended to also add a try-catch block for errors that
happen outside of the streaming.

```ts highlight="13-21"
import { generateText } from 'ai';

try {
  const { fullStream } = streamText({
    model: 'openai/gpt-4.1',
    prompt: 'Write a vegetarian lasagna recipe for 4 people.',
  });

  for await (const part of fullStream) {
    switch (part.type) {
      // ... handle other part types

      case 'error': {
        const error = part.error;
        // handle error
        break;
      }

      case 'abort': {
        // handle stream abort
        break;
      }

      case 'tool-error': {
        const error = part.error;
        // handle error
        break;
      }
    }
  }
} catch (error) {
  // handle error
}
```

## Handling stream aborts

When streams are aborted (e.g., via chat stop button), you may want to perform cleanup operations like updating stored messages in your UI. Use the `onAbort` callback to handle these cases.

The `onAbort` callback is called when a stream is aborted via `AbortSignal`, but `onFinish` is not called. This ensures you can still update your UI state appropriately.

```ts highlight="5-9"
import { streamText } from 'ai';

const { textStream } = streamText({
  model: 'openai/gpt-4.1',
  prompt: 'Write a vegetarian lasagna recipe for 4 people.',
  onAbort: ({ steps }) => {
    // Update stored messages or perform cleanup
    console.log('Stream aborted after', steps.length, 'steps');
  },
  onFinish: ({ steps, totalUsage }) => {
    // This is called on normal completion
    console.log('Stream completed normally');
  },
});

for await (const textPart of textStream) {
  process.stdout.write(textPart);
}
```

The `onAbort` callback receives:

- `steps`: An array of all completed steps before the abort

You can also handle abort events directly in the stream:

```ts highlight="10-13"
import { streamText } from 'ai';

const { fullStream } = streamText({
  model: 'openai/gpt-4.1',
  prompt: 'Write a vegetarian lasagna recipe for 4 people.',
});

for await (const chunk of fullStream) {
  switch (chunk.type) {
    case 'abort': {
      // Handle abort directly in stream
      console.log('Stream was aborted');
      break;
    }
    // ... handle other part types
  }
}
```



# Testing

Testing language models can be challenging, because they are non-deterministic
and calling them is slow and expensive.

To enable you to unit test your code that uses the AI SDK, the AI SDK Core
includes mock providers and test helpers. You can import the following helpers from `ai/test`:

- `MockEmbeddingModelV2`: A mock embedding model using the [embedding model v2 specification](https://github.com/vercel/ai/blob/v5/packages/provider/src/embedding-model/v2/embedding-model-v2.ts).
- `MockLanguageModelV2`: A mock language model using the [language model v2 specification](https://github.com/vercel/ai/blob/v5/packages/provider/src/language-model/v2/language-model-v2.ts).
- `mockId`: Provides an incrementing integer ID.
- `mockValues`: Iterates over an array of values with each call. Returns the last value when the array is exhausted.
- [`simulateReadableStream`](/docs/reference/ai-sdk-core/simulate-readable-stream): Simulates a readable stream with delays.

With mock providers and test helpers, you can control the output of the AI SDK
and test your code in a repeatable and deterministic way without actually calling
a language model provider.

## Examples

You can use the test helpers with the AI Core functions in your unit tests:

### generateText

```ts
import { generateText } from 'ai';
import { MockLanguageModelV2 } from 'ai/test';

const result = await generateText({
  model: new MockLanguageModelV2({
    doGenerate: async () => ({
      finishReason: 'stop',
      usage: { inputTokens: 10, outputTokens: 20, totalTokens: 30 },
      content: [{ type: 'text', text: `Hello, world!` }],
      warnings: [],
    }),
  }),
  prompt: 'Hello, test!',
});
```

### streamText

```ts
import { streamText, simulateReadableStream } from 'ai';
import { MockLanguageModelV2 } from 'ai/test';

const result = streamText({
  model: new MockLanguageModelV2({
    doStream: async () => ({
      stream: simulateReadableStream({
        chunks: [
          { type: 'text-start', id: 'text-1' },
          { type: 'text-delta', id: 'text-1', delta: 'Hello' },
          { type: 'text-delta', id: 'text-1', delta: ', ' },
          { type: 'text-delta', id: 'text-1', delta: 'world!' },
          { type: 'text-end', id: 'text-1' },
          {
            type: 'finish',
            finishReason: 'stop',
            logprobs: undefined,
            usage: { inputTokens: 3, outputTokens: 10, totalTokens: 13 },
          },
        ],
      }),
    }),
  }),
  prompt: 'Hello, test!',
});
```

### generateObject

```ts
import { generateObject } from 'ai';
import { MockLanguageModelV2 } from 'ai/test';
import { z } from 'zod';

const result = await generateObject({
  model: new MockLanguageModelV2({
    doGenerate: async () => ({
      finishReason: 'stop',
      usage: { inputTokens: 10, outputTokens: 20, totalTokens: 30 },
      content: [{ type: 'text', text: `{"content":"Hello, world!"}` }],
      warnings: [],
    }),
  }),
  schema: z.object({ content: z.string() }),
  prompt: 'Hello, test!',
});
```

### streamObject

```ts
import { streamObject, simulateReadableStream } from 'ai';
import { MockLanguageModelV2 } from 'ai/test';
import { z } from 'zod';

const result = streamObject({
  model: new MockLanguageModelV2({
    doStream: async () => ({
      stream: simulateReadableStream({
        chunks: [
          { type: 'text-start', id: 'text-1' },
          { type: 'text-delta', id: 'text-1', delta: '{ ' },
          { type: 'text-delta', id: 'text-1', delta: '"content": ' },
          { type: 'text-delta', id: 'text-1', delta: `"Hello, ` },
          { type: 'text-delta', id: 'text-1', delta: `world` },
          { type: 'text-delta', id: 'text-1', delta: `!"` },
          { type: 'text-delta', id: 'text-1', delta: ' }' },
          { type: 'text-end', id: 'text-1' },
          {
            type: 'finish',
            finishReason: 'stop',
            logprobs: undefined,
            usage: { inputTokens: 3, outputTokens: 10, totalTokens: 13 },
          },
        ],
      }),
    }),
  }),
  schema: z.object({ content: z.string() }),
  prompt: 'Hello, test!',
});
```

### Simulate UI Message Stream Responses

You can also simulate [UI Message Stream](/docs/ai-sdk-ui/stream-protocol#ui-message-stream) responses for testing,
debugging, or demonstration purposes.

Here is a Next example:

```ts filename="route.ts"
import { simulateReadableStream } from 'ai';

export async function POST(req: Request) {
  return new Response(
    simulateReadableStream({
      initialDelayInMs: 1000, // Delay before the first chunk
      chunkDelayInMs: 300, // Delay between chunks
      chunks: [
        `data: {"type":"start","messageId":"msg-123"}\n\n`,
        `data: {"type":"text-start","id":"text-1"}\n\n`,
        `data: {"type":"text-delta","id":"text-1","delta":"This"}\n\n`,
        `data: {"type":"text-delta","id":"text-1","delta":" is an"}\n\n`,
        `data: {"type":"text-delta","id":"text-1","delta":" example."}\n\n`,
        `data: {"type":"text-end","id":"text-1"}\n\n`,
        `data: {"type":"finish"}\n\n`,
        `data: [DONE]\n\n`,
      ],
    }).pipeThrough(new TextEncoderStream()),
    {
      status: 200,
      headers: {
        'Content-Type': 'text/event-stream',
        'Cache-Control': 'no-cache',
        Connection: 'keep-alive',
        'x-vercel-ai-ui-message-stream': 'v1',
      },
    },
  );
}
```




# Telemetry

<Note type="warning">
  AI SDK Telemetry is experimental and may change in the future.
</Note>

The AI SDK uses [OpenTelemetry](https://opentelemetry.io/) to collect telemetry data.
OpenTelemetry is an open-source observability framework designed to provide
standardized instrumentation for collecting telemetry data.

Check out the [AI SDK Observability Integrations](/providers/observability)
to see providers that offer monitoring and tracing for AI SDK applications.

## Enabling telemetry

For Next.js applications, please follow the [Next.js OpenTelemetry guide](https://nextjs.org/docs/app/building-your-application/optimizing/open-telemetry) to enable telemetry first.

You can then use the `experimental_telemetry` option to enable telemetry on specific function calls while the feature is experimental:

```ts highlight="4"
const result = await generateText({
  model: openai('gpt-4.1'),
  prompt: 'Write a short story about a cat.',
  experimental_telemetry: { isEnabled: true },
});
```

When telemetry is enabled, you can also control if you want to record the input values and the output values for the function.
By default, both are enabled. You can disable them by setting the `recordInputs` and `recordOutputs` options to `false`.

Disabling the recording of inputs and outputs can be useful for privacy, data transfer, and performance reasons.
You might for example want to disable recording inputs if they contain sensitive information.

## Telemetry Metadata

You can provide a `functionId` to identify the function that the telemetry data is for,
and `metadata` to include additional information in the telemetry data.

```ts highlight="6-10"
const result = await generateText({
  model: openai('gpt-4.1'),
  prompt: 'Write a short story about a cat.',
  experimental_telemetry: {
    isEnabled: true,
    functionId: 'my-awesome-function',
    metadata: {
      something: 'custom',
      someOtherThing: 'other-value',
    },
  },
});
```

## Custom Tracer

You may provide a `tracer` which must return an OpenTelemetry `Tracer`. This is useful in situations where
you want your traces to use a `TracerProvider` other than the one provided by the `@opentelemetry/api` singleton.

```ts highlight="7"
const tracerProvider = new NodeTracerProvider();
const result = await generateText({
  model: openai('gpt-4.1'),
  prompt: 'Write a short story about a cat.',
  experimental_telemetry: {
    isEnabled: true,
    tracer: tracerProvider.getTracer('ai'),
  },
});
```

## Collected Data

### generateText function

`generateText` records 3 types of spans:

- `ai.generateText` (span): the full length of the generateText call. It contains 1 or more `ai.generateText.doGenerate` spans.
  It contains the [basic LLM span information](#basic-llm-span-information) and the following attributes:

  - `operation.name`: `ai.generateText` and the functionId that was set through `telemetry.functionId`
  - `ai.operationId`: `"ai.generateText"`
  - `ai.prompt`: the prompt that was used when calling `generateText`
  - `ai.response.text`: the text that was generated
  - `ai.response.toolCalls`: the tool calls that were made as part of the generation (stringified JSON)
  - `ai.response.finishReason`: the reason why the generation finished
  - `ai.settings.maxOutputTokens`: the maximum number of output tokens that were set

- `ai.generateText.doGenerate` (span): a provider doGenerate call. It can contain `ai.toolCall` spans.
  It contains the [call LLM span information](#call-llm-span-information) and the following attributes:

  - `operation.name`: `ai.generateText.doGenerate` and the functionId that was set through `telemetry.functionId`
  - `ai.operationId`: `"ai.generateText.doGenerate"`
  - `ai.prompt.messages`: the messages that were passed into the provider
  - `ai.prompt.tools`: array of stringified tool definitions. The tools can be of type `function` or `provider-defined-client`.
    Function tools have a `name`, `description` (optional), and `inputSchema` (JSON schema).
    Provider-defined-client tools have a `name`, `id`, and `input` (Record).
  - `ai.prompt.toolChoice`: the stringified tool choice setting (JSON). It has a `type` property
    (`auto`, `none`, `required`, `tool`), and if the type is `tool`, a `toolName` property with the specific tool.
  - `ai.response.text`: the text that was generated
  - `ai.response.toolCalls`: the tool calls that were made as part of the generation (stringified JSON)
  - `ai.response.finishReason`: the reason why the generation finished

- `ai.toolCall` (span): a tool call that is made as part of the generateText call. See [Tool call spans](#tool-call-spans) for more details.

### streamText function

`streamText` records 3 types of spans and 2 types of events:

- `ai.streamText` (span): the full length of the streamText call. It contains a `ai.streamText.doStream` span.
  It contains the [basic LLM span information](#basic-llm-span-information) and the following attributes:

  - `operation.name`: `ai.streamText` and the functionId that was set through `telemetry.functionId`
  - `ai.operationId`: `"ai.streamText"`
  - `ai.prompt`: the prompt that was used when calling `streamText`
  - `ai.response.text`: the text that was generated
  - `ai.response.toolCalls`: the tool calls that were made as part of the generation (stringified JSON)
  - `ai.response.finishReason`: the reason why the generation finished
  - `ai.settings.maxOutputTokens`: the maximum number of output tokens that were set

- `ai.streamText.doStream` (span): a provider doStream call.
  This span contains an `ai.stream.firstChunk` event and `ai.toolCall` spans.
  It contains the [call LLM span information](#call-llm-span-information) and the following attributes:

  - `operation.name`: `ai.streamText.doStream` and the functionId that was set through `telemetry.functionId`
  - `ai.operationId`: `"ai.streamText.doStream"`
  - `ai.prompt.messages`: the messages that were passed into the provider
  - `ai.prompt.tools`: array of stringified tool definitions. The tools can be of type `function` or `provider-defined-client`.
    Function tools have a `name`, `description` (optional), and `inputSchema` (JSON schema).
    Provider-defined-client tools have a `name`, `id`, and `input` (Record).
  - `ai.prompt.toolChoice`: the stringified tool choice setting (JSON). It has a `type` property
    (`auto`, `none`, `required`, `tool`), and if the type is `tool`, a `toolName` property with the specific tool.
  - `ai.response.text`: the text that was generated
  - `ai.response.toolCalls`: the tool calls that were made as part of the generation (stringified JSON)
  - `ai.response.msToFirstChunk`: the time it took to receive the first chunk in milliseconds
  - `ai.response.msToFinish`: the time it took to receive the finish part of the LLM stream in milliseconds
  - `ai.response.avgCompletionTokensPerSecond`: the average number of completion tokens per second
  - `ai.response.finishReason`: the reason why the generation finished

- `ai.toolCall` (span): a tool call that is made as part of the generateText call. See [Tool call spans](#tool-call-spans) for more details.

- `ai.stream.firstChunk` (event): an event that is emitted when the first chunk of the stream is received.

  - `ai.response.msToFirstChunk`: the time it took to receive the first chunk

- `ai.stream.finish` (event): an event that is emitted when the finish part of the LLM stream is received.

It also records a `ai.stream.firstChunk` event when the first chunk of the stream is received.

### generateObject function

`generateObject` records 2 types of spans:

- `ai.generateObject` (span): the full length of the generateObject call. It contains 1 or more `ai.generateObject.doGenerate` spans.
  It contains the [basic LLM span information](#basic-llm-span-information) and the following attributes:

  - `operation.name`: `ai.generateObject` and the functionId that was set through `telemetry.functionId`
  - `ai.operationId`: `"ai.generateObject"`
  - `ai.prompt`: the prompt that was used when calling `generateObject`
  - `ai.schema`: Stringified JSON schema version of the schema that was passed into the `generateObject` function
  - `ai.schema.name`: the name of the schema that was passed into the `generateObject` function
  - `ai.schema.description`: the description of the schema that was passed into the `generateObject` function
  - `ai.response.object`: the object that was generated (stringified JSON)
  - `ai.settings.output`: the output type that was used, e.g. `object` or `no-schema`

- `ai.generateObject.doGenerate` (span): a provider doGenerate call.
  It contains the [call LLM span information](#call-llm-span-information) and the following attributes:

  - `operation.name`: `ai.generateObject.doGenerate` and the functionId that was set through `telemetry.functionId`
  - `ai.operationId`: `"ai.generateObject.doGenerate"`
  - `ai.prompt.messages`: the messages that were passed into the provider
  - `ai.response.object`: the object that was generated (stringified JSON)
  - `ai.response.finishReason`: the reason why the generation finished

### streamObject function

`streamObject` records 2 types of spans and 1 type of event:

- `ai.streamObject` (span): the full length of the streamObject call. It contains 1 or more `ai.streamObject.doStream` spans.
  It contains the [basic LLM span information](#basic-llm-span-information) and the following attributes:

  - `operation.name`: `ai.streamObject` and the functionId that was set through `telemetry.functionId`
  - `ai.operationId`: `"ai.streamObject"`
  - `ai.prompt`: the prompt that was used when calling `streamObject`
  - `ai.schema`: Stringified JSON schema version of the schema that was passed into the `streamObject` function
  - `ai.schema.name`: the name of the schema that was passed into the `streamObject` function
  - `ai.schema.description`: the description of the schema that was passed into the `streamObject` function
  - `ai.response.object`: the object that was generated (stringified JSON)
  - `ai.settings.output`: the output type that was used, e.g. `object` or `no-schema`

- `ai.streamObject.doStream` (span): a provider doStream call.
  This span contains an `ai.stream.firstChunk` event.
  It contains the [call LLM span information](#call-llm-span-information) and the following attributes:

  - `operation.name`: `ai.streamObject.doStream` and the functionId that was set through `telemetry.functionId`
  - `ai.operationId`: `"ai.streamObject.doStream"`
  - `ai.prompt.messages`: the messages that were passed into the provider
  - `ai.response.object`: the object that was generated (stringified JSON)
  - `ai.response.msToFirstChunk`: the time it took to receive the first chunk
  - `ai.response.finishReason`: the reason why the generation finished

- `ai.stream.firstChunk` (event): an event that is emitted when the first chunk of the stream is received.
  - `ai.response.msToFirstChunk`: the time it took to receive the first chunk

### embed function

`embed` records 2 types of spans:

- `ai.embed` (span): the full length of the embed call. It contains 1 `ai.embed.doEmbed` spans.
  It contains the [basic embedding span information](#basic-embedding-span-information) and the following attributes:

  - `operation.name`: `ai.embed` and the functionId that was set through `telemetry.functionId`
  - `ai.operationId`: `"ai.embed"`
  - `ai.value`: the value that was passed into the `embed` function
  - `ai.embedding`: a JSON-stringified embedding

- `ai.embed.doEmbed` (span): a provider doEmbed call.
  It contains the [basic embedding span information](#basic-embedding-span-information) and the following attributes:

  - `operation.name`: `ai.embed.doEmbed` and the functionId that was set through `telemetry.functionId`
  - `ai.operationId`: `"ai.embed.doEmbed"`
  - `ai.values`: the values that were passed into the provider (array)
  - `ai.embeddings`: an array of JSON-stringified embeddings

### embedMany function

`embedMany` records 2 types of spans:

- `ai.embedMany` (span): the full length of the embedMany call. It contains 1 or more `ai.embedMany.doEmbed` spans.
  It contains the [basic embedding span information](#basic-embedding-span-information) and the following attributes:

  - `operation.name`: `ai.embedMany` and the functionId that was set through `telemetry.functionId`
  - `ai.operationId`: `"ai.embedMany"`
  - `ai.values`: the values that were passed into the `embedMany` function
  - `ai.embeddings`: an array of JSON-stringified embedding

- `ai.embedMany.doEmbed` (span): a provider doEmbed call.
  It contains the [basic embedding span information](#basic-embedding-span-information) and the following attributes:

  - `operation.name`: `ai.embedMany.doEmbed` and the functionId that was set through `telemetry.functionId`
  - `ai.operationId`: `"ai.embedMany.doEmbed"`
  - `ai.values`: the values that were sent to the provider
  - `ai.embeddings`: an array of JSON-stringified embeddings for each value

## Span Details

### Basic LLM span information

Many spans that use LLMs (`ai.generateText`, `ai.generateText.doGenerate`, `ai.streamText`, `ai.streamText.doStream`,
`ai.generateObject`, `ai.generateObject.doGenerate`, `ai.streamObject`, `ai.streamObject.doStream`) contain the following attributes:

- `resource.name`: the functionId that was set through `telemetry.functionId`
- `ai.model.id`: the id of the model
- `ai.model.provider`: the provider of the model
- `ai.request.headers.*`: the request headers that were passed in through `headers`
- `ai.response.providerMetadata`: provider specific metadata returned with the generation response
- `ai.settings.maxRetries`: the maximum number of retries that were set
- `ai.telemetry.functionId`: the functionId that was set through `telemetry.functionId`
- `ai.telemetry.metadata.*`: the metadata that was passed in through `telemetry.metadata`
- `ai.usage.completionTokens`: the number of completion tokens that were used
- `ai.usage.promptTokens`: the number of prompt tokens that were used

### Call LLM span information

Spans that correspond to individual LLM calls (`ai.generateText.doGenerate`, `ai.streamText.doStream`, `ai.generateObject.doGenerate`, `ai.streamObject.doStream`) contain
[basic LLM span information](#basic-llm-span-information) and the following attributes:

- `ai.response.model`: the model that was used to generate the response. This can be different from the model that was requested if the provider supports aliases.
- `ai.response.id`: the id of the response. Uses the ID from the provider when available.
- `ai.response.timestamp`: the timestamp of the response. Uses the timestamp from the provider when available.
- [Semantic Conventions for GenAI operations](https://opentelemetry.io/docs/specs/semconv/gen-ai/gen-ai-spans/)
  - `gen_ai.system`: the provider that was used
  - `gen_ai.request.model`: the model that was requested
  - `gen_ai.request.temperature`: the temperature that was set
  - `gen_ai.request.max_tokens`: the maximum number of tokens that were set
  - `gen_ai.request.frequency_penalty`: the frequency penalty that was set
  - `gen_ai.request.presence_penalty`: the presence penalty that was set
  - `gen_ai.request.top_k`: the topK parameter value that was set
  - `gen_ai.request.top_p`: the topP parameter value that was set
  - `gen_ai.request.stop_sequences`: the stop sequences
  - `gen_ai.response.finish_reasons`: the finish reasons that were returned by the provider
  - `gen_ai.response.model`: the model that was used to generate the response. This can be different from the model that was requested if the provider supports aliases.
  - `gen_ai.response.id`: the id of the response. Uses the ID from the provider when available.
  - `gen_ai.usage.input_tokens`: the number of prompt tokens that were used
  - `gen_ai.usage.output_tokens`: the number of completion tokens that were used

### Basic embedding span information

Many spans that use embedding models (`ai.embed`, `ai.embed.doEmbed`, `ai.embedMany`, `ai.embedMany.doEmbed`) contain the following attributes:

- `ai.model.id`: the id of the model
- `ai.model.provider`: the provider of the model
- `ai.request.headers.*`: the request headers that were passed in through `headers`
- `ai.settings.maxRetries`: the maximum number of retries that were set
- `ai.telemetry.functionId`: the functionId that was set through `telemetry.functionId`
- `ai.telemetry.metadata.*`: the metadata that was passed in through `telemetry.metadata`
- `ai.usage.tokens`: the number of tokens that were used
- `resource.name`: the functionId that was set through `telemetry.functionId`

### Tool call spans

Tool call spans (`ai.toolCall`) contain the following attributes:

- `operation.name`: `"ai.toolCall"`
- `ai.operationId`: `"ai.toolCall"`
- `ai.toolCall.name`: the name of the tool
- `ai.toolCall.id`: the id of the tool call
- `ai.toolCall.args`: the input parameters of the tool call
- `ai.toolCall.result`: the output result of the tool call. Only available if the tool call is successful and the result is serializable.
