
# Embeddings

Embeddings are a way to represent words, phrases, or images as vectors in a high-dimensional space.
In this space, similar words are close to each other, and the distance between words can be used to measure their similarity.

## Embedding a Single Value

The AI SDK provides the [`embed`](/docs/reference/ai-sdk-core/embed) function to embed single values, which is useful for tasks such as finding similar words
or phrases or clustering text.
You can use it with embeddings models, e.g. `openai.textEmbeddingModel('text-embedding-3-large')` or `mistral.textEmbeddingModel('mistral-embed')`.

```tsx
import { embed } from 'ai';
import { openai } from '@ai-sdk/openai';

// 'embedding' is a single embedding object (number[])
const { embedding } = await embed({
  model: openai.textEmbeddingModel('text-embedding-3-small'),
  value: 'sunny day at the beach',
});
```

## Embedding Many Values

When loading data, e.g. when preparing a data store for retrieval-augmented generation (RAG),
it is often useful to embed many values at once (batch embedding).

The AI SDK provides the [`embedMany`](/docs/reference/ai-sdk-core/embed-many) function for this purpose.
Similar to `embed`, you can use it with embeddings models,
e.g. `openai.textEmbeddingModel('text-embedding-3-large')` or `mistral.textEmbeddingModel('mistral-embed')`.

```tsx
import { openai } from '@ai-sdk/openai';
import { embedMany } from 'ai';

// 'embeddings' is an array of embedding objects (number[][]).
// It is sorted in the same order as the input values.
const { embeddings } = await embedMany({
  model: openai.textEmbeddingModel('text-embedding-3-small'),
  values: [
    'sunny day at the beach',
    'rainy afternoon in the city',
    'snowy night in the mountains',
  ],
});
```

## Embedding Similarity

After embedding values, you can calculate the similarity between them using the [`cosineSimilarity`](/docs/reference/ai-sdk-core/cosine-similarity) function.
This is useful to e.g. find similar words or phrases in a dataset.
You can also rank and filter related items based on their similarity.

```ts highlight={"2,10"}
import { openai } from '@ai-sdk/openai';
import { cosineSimilarity, embedMany } from 'ai';

const { embeddings } = await embedMany({
  model: openai.textEmbeddingModel('text-embedding-3-small'),
  values: ['sunny day at the beach', 'rainy afternoon in the city'],
});

console.log(
  `cosine similarity: ${cosineSimilarity(embeddings[0], embeddings[1])}`,
);
```

## Token Usage

Many providers charge based on the number of tokens used to generate embeddings.
Both `embed` and `embedMany` provide token usage information in the `usage` property of the result object:

```ts highlight={"4,9"}
import { openai } from '@ai-sdk/openai';
import { embed } from 'ai';

const { embedding, usage } = await embed({
  model: openai.textEmbeddingModel('text-embedding-3-small'),
  value: 'sunny day at the beach',
});

console.log(usage); // { tokens: 10 }
```

## Settings

### Provider Options

Embedding model settings can be configured using `providerOptions` for provider-specific parameters:

```ts highlight={"5-9"}
import { openai } from '@ai-sdk/openai';
import { embed } from 'ai';

const { embedding } = await embed({
  model: openai.textEmbeddingModel('text-embedding-3-small'),
  value: 'sunny day at the beach',
  providerOptions: {
    openai: {
      dimensions: 512, // Reduce embedding dimensions
    },
  },
});
```

### Parallel Requests

The `embedMany` function now supports parallel processing with configurable `maxParallelCalls` to optimize performance:

```ts highlight={"4"}
import { openai } from '@ai-sdk/openai';
import { embedMany } from 'ai';

const { embeddings, usage } = await embedMany({
  maxParallelCalls: 2, // Limit parallel requests
  model: openai.textEmbeddingModel('text-embedding-3-small'),
  values: [
    'sunny day at the beach',
    'rainy afternoon in the city',
    'snowy night in the mountains',
  ],
});
```

### Retries

Both `embed` and `embedMany` accept an optional `maxRetries` parameter of type `number`
that you can use to set the maximum number of retries for the embedding process.
It defaults to `2` retries (3 attempts in total). You can set it to `0` to disable retries.

```ts highlight={"7"}
import { openai } from '@ai-sdk/openai';
import { embed } from 'ai';

const { embedding } = await embed({
  model: openai.textEmbeddingModel('text-embedding-3-small'),
  value: 'sunny day at the beach',
  maxRetries: 0, // Disable retries
});
```

### Abort Signals and Timeouts

Both `embed` and `embedMany` accept an optional `abortSignal` parameter of
type [`AbortSignal`](https://developer.mozilla.org/en-US/docs/Web/API/AbortSignal)
that you can use to abort the embedding process or set a timeout.

```ts highlight={"7"}
import { openai } from '@ai-sdk/openai';
import { embed } from 'ai';

const { embedding } = await embed({
  model: openai.textEmbeddingModel('text-embedding-3-small'),
  value: 'sunny day at the beach',
  abortSignal: AbortSignal.timeout(1000), // Abort after 1 second
});
```

### Custom Headers

Both `embed` and `embedMany` accept an optional `headers` parameter of type `Record<string, string>`
that you can use to add custom headers to the embedding request.

```ts highlight={"7"}
import { openai } from '@ai-sdk/openai';
import { embed } from 'ai';

const { embedding } = await embed({
  model: openai.textEmbeddingModel('text-embedding-3-small'),
  value: 'sunny day at the beach',
  headers: { 'X-Custom-Header': 'custom-value' },
});
```

## Response Information

Both `embed` and `embedMany` return response information that includes the raw provider response:

```ts highlight={"4,9"}
import { openai } from '@ai-sdk/openai';
import { embed } from 'ai';

const { embedding, response } = await embed({
  model: openai.textEmbeddingModel('text-embedding-3-small'),
  value: 'sunny day at the beach',
});

console.log(response); // Raw provider response
```

## Embedding Providers & Models

Several providers offer embedding models:

| Provider                                                                                  | Model                           | Embedding Dimensions |
| ----------------------------------------------------------------------------------------- | ------------------------------- | -------------------- |
| [OpenAI](/providers/ai-sdk-providers/openai#embedding-models)                             | `text-embedding-3-large`        | 3072                 |
| [OpenAI](/providers/ai-sdk-providers/openai#embedding-models)                             | `text-embedding-3-small`        | 1536                 |
| [OpenAI](/providers/ai-sdk-providers/openai#embedding-models)                             | `text-embedding-ada-002`        | 1536                 |
| [Google Generative AI](/providers/ai-sdk-providers/google-generative-ai#embedding-models) | `gemini-embedding-001`          | 3072                 |
| [Google Generative AI](/providers/ai-sdk-providers/google-generative-ai#embedding-models) | `text-embedding-004`            | 768                  |
| [Mistral](/providers/ai-sdk-providers/mistral#embedding-models)                           | `mistral-embed`                 | 1024                 |
| [Cohere](/providers/ai-sdk-providers/cohere#embedding-models)                             | `embed-english-v3.0`            | 1024                 |
| [Cohere](/providers/ai-sdk-providers/cohere#embedding-models)                             | `embed-multilingual-v3.0`       | 1024                 |
| [Cohere](/providers/ai-sdk-providers/cohere#embedding-models)                             | `embed-english-light-v3.0`      | 384                  |
| [Cohere](/providers/ai-sdk-providers/cohere#embedding-models)                             | `embed-multilingual-light-v3.0` | 384                  |
| [Cohere](/providers/ai-sdk-providers/cohere#embedding-models)                             | `embed-english-v2.0`            | 4096                 |
| [Cohere](/providers/ai-sdk-providers/cohere#embedding-models)                             | `embed-english-light-v2.0`      | 1024                 |
| [Cohere](/providers/ai-sdk-providers/cohere#embedding-models)                             | `embed-multilingual-v2.0`       | 768                  |
| [Amazon Bedrock](/providers/ai-sdk-providers/amazon-bedrock#embedding-models)             | `amazon.titan-embed-text-v1`    | 1536                 |
| [Amazon Bedrock](/providers/ai-sdk-providers/amazon-bedrock#embedding-models)             | `amazon.titan-embed-text-v2:0`  | 1024                 |



# Image Generation

<Note type="warning">Image generation is an experimental feature.</Note>

The AI SDK provides the [`generateImage`](/docs/reference/ai-sdk-core/generate-image)
function to generate images based on a given prompt using an image model.

```tsx
import { experimental_generateImage as generateImage } from 'ai';
import { openai } from '@ai-sdk/openai';

const { image } = await generateImage({
  model: openai.image('dall-e-3'),
  prompt: 'Santa Claus driving a Cadillac',
});
```

You can access the image data using the `base64` or `uint8Array` properties:

```tsx
const base64 = image.base64; // base64 image data
const uint8Array = image.uint8Array; // Uint8Array image data
```

## Settings

### Size and Aspect Ratio

Depending on the model, you can either specify the size or the aspect ratio.

##### Size

The size is specified as a string in the format `{width}x{height}`.
Models only support a few sizes, and the supported sizes are different for each model and provider.

```tsx highlight={"7"}
import { experimental_generateImage as generateImage } from 'ai';
import { openai } from '@ai-sdk/openai';

const { image } = await generateImage({
  model: openai.image('dall-e-3'),
  prompt: 'Santa Claus driving a Cadillac',
  size: '1024x1024',
});
```

##### Aspect Ratio

The aspect ratio is specified as a string in the format `{width}:{height}`.
Models only support a few aspect ratios, and the supported aspect ratios are different for each model and provider.

```tsx highlight={"7"}
import { experimental_generateImage as generateImage } from 'ai';
import { vertex } from '@ai-sdk/google-vertex';

const { image } = await generateImage({
  model: vertex.image('imagen-3.0-generate-002'),
  prompt: 'Santa Claus driving a Cadillac',
  aspectRatio: '16:9',
});
```

### Generating Multiple Images

`generateImage` also supports generating multiple images at once:

```tsx highlight={"7"}
import { experimental_generateImage as generateImage } from 'ai';
import { openai } from '@ai-sdk/openai';

const { images } = await generateImage({
  model: openai.image('dall-e-2'),
  prompt: 'Santa Claus driving a Cadillac',
  n: 4, // number of images to generate
});
```

<Note>
  `generateImage` will automatically call the model as often as needed (in
  parallel) to generate the requested number of images.
</Note>

Each image model has an internal limit on how many images it can generate in a single API call. The AI SDK manages this automatically by batching requests appropriately when you request multiple images using the `n` parameter. By default, the SDK uses provider-documented limits (for example, DALL-E 3 can only generate 1 image per call, while DALL-E 2 supports up to 10).

If needed, you can override this behavior using the `maxImagesPerCall` setting when generating your image. This is particularly useful when working with new or custom models where the default batch size might not be optimal:

```tsx
const { images } = await generateImage({
  model: openai.image('dall-e-2'),
  prompt: 'Santa Claus driving a Cadillac',
  maxImagesPerCall: 5, // Override the default batch size
  n: 10, // Will make 2 calls of 5 images each
});
```

### Providing a Seed

You can provide a seed to the `generateImage` function to control the output of the image generation process.
If supported by the model, the same seed will always produce the same image.

```tsx highlight={"7"}
import { experimental_generateImage as generateImage } from 'ai';
import { openai } from '@ai-sdk/openai';

const { image } = await generateImage({
  model: openai.image('dall-e-3'),
  prompt: 'Santa Claus driving a Cadillac',
  seed: 1234567890,
});
```

### Provider-specific Settings

Image models often have provider- or even model-specific settings.
You can pass such settings to the `generateImage` function
using the `providerOptions` parameter. The options for the provider
(`openai` in the example below) become request body properties.

```tsx highlight={"9"}
import { experimental_generateImage as generateImage } from 'ai';
import { openai } from '@ai-sdk/openai';

const { image } = await generateImage({
  model: openai.image('dall-e-3'),
  prompt: 'Santa Claus driving a Cadillac',
  size: '1024x1024',
  providerOptions: {
    openai: { style: 'vivid', quality: 'hd' },
  },
});
```

### Abort Signals and Timeouts

`generateImage` accepts an optional `abortSignal` parameter of
type [`AbortSignal`](https://developer.mozilla.org/en-US/docs/Web/API/AbortSignal)
that you can use to abort the image generation process or set a timeout.

```ts highlight={"7"}
import { openai } from '@ai-sdk/openai';
import { experimental_generateImage as generateImage } from 'ai';

const { image } = await generateImage({
  model: openai.image('dall-e-3'),
  prompt: 'Santa Claus driving a Cadillac',
  abortSignal: AbortSignal.timeout(1000), // Abort after 1 second
});
```

### Custom Headers

`generateImage` accepts an optional `headers` parameter of type `Record<string, string>`
that you can use to add custom headers to the image generation request.

```ts highlight={"7"}
import { openai } from '@ai-sdk/openai';
import { experimental_generateImage as generateImage } from 'ai';

const { image } = await generateImage({
  model: openai.image('dall-e-3'),
  prompt: 'Santa Claus driving a Cadillac',
  headers: { 'X-Custom-Header': 'custom-value' },
});
```

### Warnings

If the model returns warnings, e.g. for unsupported parameters, they will be available in the `warnings` property of the response.

```tsx
const { image, warnings } = await generateImage({
  model: openai.image('dall-e-3'),
  prompt: 'Santa Claus driving a Cadillac',
});
```

### Additional provider-specific meta data

Some providers expose additional meta data for the result overall or per image.

```tsx
const prompt = 'Santa Claus driving a Cadillac';

const { image, providerMetadata } = await generateImage({
  model: openai.image('dall-e-3'),
  prompt,
});

const revisedPrompt = providerMetadata.openai.images[0]?.revisedPrompt;

console.log({
  prompt,
  revisedPrompt,
});
```

The outer key of the returned `providerMetadata` is the provider name. The inner values are the metadata. An `images` key is always present in the metadata and is an array with the same length as the top level `images` key.

### Error Handling

When `generateImage` cannot generate a valid image, it throws a [`AI_NoImageGeneratedError`](/docs/reference/ai-sdk-errors/ai-no-image-generated-error).

This error occurs when the AI provider fails to generate an image. It can arise due to the following reasons:

- The model failed to generate a response
- The model generated a response that could not be parsed

The error preserves the following information to help you log the issue:

- `responses`: Metadata about the image model responses, including timestamp, model, and headers.
- `cause`: The cause of the error. You can use this for more detailed error handling

```ts
import { generateImage, NoImageGeneratedError } from 'ai';

try {
  await generateImage({ model, prompt });
} catch (error) {
  if (NoImageGeneratedError.isInstance(error)) {
    console.log('NoImageGeneratedError');
    console.log('Cause:', error.cause);
    console.log('Responses:', error.responses);
  }
}
```

## Generating Images with Language Models

Some language models such as Google `gemini-2.0-flash-exp` support multi-modal outputs including images.
With such models, you can access the generated images using the `files` property of the response.

```ts
import { google } from '@ai-sdk/google';
import { generateText } from 'ai';

const result = await generateText({
  model: google('gemini-2.0-flash-exp'),
  providerOptions: {
    google: { responseModalities: ['TEXT', 'IMAGE'] },
  },
  prompt: 'Generate an image of a comic cat',
});

for (const file of result.files) {
  if (file.mediaType.startsWith('image/')) {
    // The file object provides multiple data formats:
    // Access images as base64 string, Uint8Array binary data, or check type
    // - file.base64: string (data URL format)
    // - file.uint8Array: Uint8Array (binary data)
    // - file.mediaType: string (e.g. "image/png")
  }
}
```

## Image Models

| Provider                                                                  | Model                                                        | Support sizes (`width x height`) or aspect ratios (`width : height`)                                                                                                |
| ------------------------------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [xAI Grok](/providers/ai-sdk-providers/xai#image-models)                  | `grok-2-image`                                               | 1024x768 (default)                                                                                                                                                  |
| [OpenAI](/providers/ai-sdk-providers/openai#image-models)                 | `gpt-image-1`                                                | 1024x1024, 1536x1024, 1024x1536                                                                                                                                     |
| [OpenAI](/providers/ai-sdk-providers/openai#image-models)                 | `dall-e-3`                                                   | 1024x1024, 1792x1024, 1024x1792                                                                                                                                     |
| [OpenAI](/providers/ai-sdk-providers/openai#image-models)                 | `dall-e-2`                                                   | 256x256, 512x512, 1024x1024                                                                                                                                         |
| [Amazon Bedrock](/providers/ai-sdk-providers/amazon-bedrock#image-models) | `amazon.nova-canvas-v1:0`                                    | 320-4096 (multiples of 16), 1:4 to 4:1, max 4.2M pixels                                                                                                             |
| [Fal](/providers/ai-sdk-providers/fal#image-models)                       | `fal-ai/flux/dev`                                            | 1:1, 3:4, 4:3, 9:16, 16:9, 9:21, 21:9                                                                                                                               |
| [Fal](/providers/ai-sdk-providers/fal#image-models)                       | `fal-ai/flux-lora`                                           | 1:1, 3:4, 4:3, 9:16, 16:9, 9:21, 21:9                                                                                                                               |
| [Fal](/providers/ai-sdk-providers/fal#image-models)                       | `fal-ai/fast-sdxl`                                           | 1:1, 3:4, 4:3, 9:16, 16:9, 9:21, 21:9                                                                                                                               |
| [Fal](/providers/ai-sdk-providers/fal#image-models)                       | `fal-ai/flux-pro/v1.1-ultra`                                 | 1:1, 3:4, 4:3, 9:16, 16:9, 9:21, 21:9                                                                                                                               |
| [Fal](/providers/ai-sdk-providers/fal#image-models)                       | `fal-ai/ideogram/v2`                                         | 1:1, 3:4, 4:3, 9:16, 16:9, 9:21, 21:9                                                                                                                               |
| [Fal](/providers/ai-sdk-providers/fal#image-models)                       | `fal-ai/recraft-v3`                                          | 1:1, 3:4, 4:3, 9:16, 16:9, 9:21, 21:9                                                                                                                               |
| [Fal](/providers/ai-sdk-providers/fal#image-models)                       | `fal-ai/stable-diffusion-3.5-large`                          | 1:1, 3:4, 4:3, 9:16, 16:9, 9:21, 21:9                                                                                                                               |
| [Fal](/providers/ai-sdk-providers/fal#image-models)                       | `fal-ai/hyper-sdxl`                                          | 1:1, 3:4, 4:3, 9:16, 16:9, 9:21, 21:9                                                                                                                               |
| [DeepInfra](/providers/ai-sdk-providers/deepinfra#image-models)           | `stabilityai/sd3.5`                                          | 1:1, 16:9, 1:9, 3:2, 2:3, 4:5, 5:4, 9:16, 9:21                                                                                                                      |
| [DeepInfra](/providers/ai-sdk-providers/deepinfra#image-models)           | `black-forest-labs/FLUX-1.1-pro`                             | 256-1440 (multiples of 32)                                                                                                                                          |
| [DeepInfra](/providers/ai-sdk-providers/deepinfra#image-models)           | `black-forest-labs/FLUX-1-schnell`                           | 256-1440 (multiples of 32)                                                                                                                                          |
| [DeepInfra](/providers/ai-sdk-providers/deepinfra#image-models)           | `black-forest-labs/FLUX-1-dev`                               | 256-1440 (multiples of 32)                                                                                                                                          |
| [DeepInfra](/providers/ai-sdk-providers/deepinfra#image-models)           | `black-forest-labs/FLUX-pro`                                 | 256-1440 (multiples of 32)                                                                                                                                          |
| [DeepInfra](/providers/ai-sdk-providers/deepinfra#image-models)           | `stabilityai/sd3.5-medium`                                   | 1:1, 16:9, 1:9, 3:2, 2:3, 4:5, 5:4, 9:16, 9:21                                                                                                                      |
| [DeepInfra](/providers/ai-sdk-providers/deepinfra#image-models)           | `stabilityai/sdxl-turbo`                                     | 1:1, 16:9, 1:9, 3:2, 2:3, 4:5, 5:4, 9:16, 9:21                                                                                                                      |
| [Replicate](/providers/ai-sdk-providers/replicate)                        | `black-forest-labs/flux-schnell`                             | 1:1, 2:3, 3:2, 4:5, 5:4, 16:9, 9:16, 9:21, 21:9                                                                                                                     |
| [Replicate](/providers/ai-sdk-providers/replicate)                        | `recraft-ai/recraft-v3`                                      | 1024x1024, 1365x1024, 1024x1365, 1536x1024, 1024x1536, 1820x1024, 1024x1820, 1024x2048, 2048x1024, 1434x1024, 1024x1434, 1024x1280, 1280x1024, 1024x1707, 1707x1024 |
| [Google](/providers/ai-sdk-providers/google#image-models)                 | `imagen-3.0-generate-002`                                    | 1:1, 3:4, 4:3, 9:16, 16:9                                                                                                                                           |
| [Google Vertex](/providers/ai-sdk-providers/google-vertex#image-models)   | `imagen-3.0-generate-002`                                    | 1:1, 3:4, 4:3, 9:16, 16:9                                                                                                                                           |
| [Google Vertex](/providers/ai-sdk-providers/google-vertex#image-models)   | `imagen-3.0-fast-generate-001`                               | 1:1, 3:4, 4:3, 9:16, 16:9                                                                                                                                           |
| [Fireworks](/providers/ai-sdk-providers/fireworks#image-models)           | `accounts/fireworks/models/flux-1-dev-fp8`                   | 1:1, 2:3, 3:2, 4:5, 5:4, 16:9, 9:16, 9:21, 21:9                                                                                                                     |
| [Fireworks](/providers/ai-sdk-providers/fireworks#image-models)           | `accounts/fireworks/models/flux-1-schnell-fp8`               | 1:1, 2:3, 3:2, 4:5, 5:4, 16:9, 9:16, 9:21, 21:9                                                                                                                     |
| [Fireworks](/providers/ai-sdk-providers/fireworks#image-models)           | `accounts/fireworks/models/playground-v2-5-1024px-aesthetic` | 640x1536, 768x1344, 832x1216, 896x1152, 1024x1024, 1152x896, 1216x832, 1344x768, 1536x640                                                                           |
| [Fireworks](/providers/ai-sdk-providers/fireworks#image-models)           | `accounts/fireworks/models/japanese-stable-diffusion-xl`     | 640x1536, 768x1344, 832x1216, 896x1152, 1024x1024, 1152x896, 1216x832, 1344x768, 1536x640                                                                           |
| [Fireworks](/providers/ai-sdk-providers/fireworks#image-models)           | `accounts/fireworks/models/playground-v2-1024px-aesthetic`   | 640x1536, 768x1344, 832x1216, 896x1152, 1024x1024, 1152x896, 1216x832, 1344x768, 1536x640                                                                           |
| [Fireworks](/providers/ai-sdk-providers/fireworks#image-models)           | `accounts/fireworks/models/SSD-1B`                           | 640x1536, 768x1344, 832x1216, 896x1152, 1024x1024, 1152x896, 1216x832, 1344x768, 1536x640                                                                           |
| [Fireworks](/providers/ai-sdk-providers/fireworks#image-models)           | `accounts/fireworks/models/stable-diffusion-xl-1024-v1-0`    | 640x1536, 768x1344, 832x1216, 896x1152, 1024x1024, 1152x896, 1216x832, 1344x768, 1536x640                                                                           |
| [Luma](/providers/ai-sdk-providers/luma#image-models)                     | `photon-1`                                                   | 1:1, 3:4, 4:3, 9:16, 16:9, 9:21, 21:9                                                                                                                               |
| [Luma](/providers/ai-sdk-providers/luma#image-models)                     | `photon-flash-1`                                             | 1:1, 3:4, 4:3, 9:16, 16:9, 9:21, 21:9                                                                                                                               |
| [Together.ai](/providers/ai-sdk-providers/togetherai#image-models)        | `stabilityai/stable-diffusion-xl-base-1.0`                   | 512x512, 768x768, 1024x1024                                                                                                                                         |
| [Together.ai](/providers/ai-sdk-providers/togetherai#image-models)        | `black-forest-labs/FLUX.1-dev`                               | 512x512, 768x768, 1024x1024                                                                                                                                         |
| [Together.ai](/providers/ai-sdk-providers/togetherai#image-models)        | `black-forest-labs/FLUX.1-dev-lora`                          | 512x512, 768x768, 1024x1024                                                                                                                                         |
| [Together.ai](/providers/ai-sdk-providers/togetherai#image-models)        | `black-forest-labs/FLUX.1-schnell`                           | 512x512, 768x768, 1024x1024                                                                                                                                         |
| [Together.ai](/providers/ai-sdk-providers/togetherai#image-models)        | `black-forest-labs/FLUX.1-canny`                             | 512x512, 768x768, 1024x1024                                                                                                                                         |
| [Together.ai](/providers/ai-sdk-providers/togetherai#image-models)        | `black-forest-labs/FLUX.1-depth`                             | 512x512, 768x768, 1024x1024                                                                                                                                         |
| [Together.ai](/providers/ai-sdk-providers/togetherai#image-models)        | `black-forest-labs/FLUX.1-redux`                             | 512x512, 768x768, 1024x1024                                                                                                                                         |
| [Together.ai](/providers/ai-sdk-providers/togetherai#image-models)        | `black-forest-labs/FLUX.1.1-pro`                             | 512x512, 768x768, 1024x1024                                                                                                                                         |
| [Together.ai](/providers/ai-sdk-providers/togetherai#image-models)        | `black-forest-labs/FLUX.1-pro`                               | 512x512, 768x768, 1024x1024                                                                                                                                         |
| [Together.ai](/providers/ai-sdk-providers/togetherai#image-models)        | `black-forest-labs/FLUX.1-schnell-Free`                      | 512x512, 768x768, 1024x1024                                                                                                                                         |

Above are a small subset of the image models supported by the AI SDK providers. For more, see the respective provider documentation.

# Transcription

<Note type="warning">Transcription is an experimental feature.</Note>

The AI SDK provides the [`transcribe`](/docs/reference/ai-sdk-core/transcribe)
function to transcribe audio using a transcription model.

```ts
import { experimental_transcribe as transcribe } from 'ai';
import { openai } from '@ai-sdk/openai';
import { readFile } from 'fs/promises';

const transcript = await transcribe({
  model: openai.transcription('whisper-1'),
  audio: await readFile('audio.mp3'),
});
```

The `audio` property can be a `Uint8Array`, `ArrayBuffer`, `Buffer`, `string` (base64 encoded audio data), or a `URL`.

To access the generated transcript:

```ts
const text = transcript.text; // transcript text e.g. "Hello, world!"
const segments = transcript.segments; // array of segments with start and end times, if available
const language = transcript.language; // language of the transcript e.g. "en", if available
const durationInSeconds = transcript.durationInSeconds; // duration of the transcript in seconds, if available
```

## Settings

### Provider-Specific settings

Transcription models often have provider or model-specific settings which you can set using the `providerOptions` parameter.

```ts highlight="8-12"
import { experimental_transcribe as transcribe } from 'ai';
import { openai } from '@ai-sdk/openai';
import { readFile } from 'fs/promises';

const transcript = await transcribe({
  model: openai.transcription('whisper-1'),
  audio: await readFile('audio.mp3'),
  providerOptions: {
    openai: {
      timestampGranularities: ['word'],
    },
  },
});
```

### Abort Signals and Timeouts

`transcribe` accepts an optional `abortSignal` parameter of
type [`AbortSignal`](https://developer.mozilla.org/en-US/docs/Web/API/AbortSignal)
that you can use to abort the transcription process or set a timeout.

```ts highlight="8"
import { openai } from '@ai-sdk/openai';
import { experimental_transcribe as transcribe } from 'ai';
import { readFile } from 'fs/promises';

const transcript = await transcribe({
  model: openai.transcription('whisper-1'),
  audio: await readFile('audio.mp3'),
  abortSignal: AbortSignal.timeout(1000), // Abort after 1 second
});
```

### Custom Headers

`transcribe` accepts an optional `headers` parameter of type `Record<string, string>`
that you can use to add custom headers to the transcription request.

```ts highlight="8"
import { openai } from '@ai-sdk/openai';
import { experimental_transcribe as transcribe } from 'ai';
import { readFile } from 'fs/promises';

const transcript = await transcribe({
  model: openai.transcription('whisper-1'),
  audio: await readFile('audio.mp3'),
  headers: { 'X-Custom-Header': 'custom-value' },
});
```

### Warnings

Warnings (e.g. unsupported parameters) are available on the `warnings` property.

```ts
import { openai } from '@ai-sdk/openai';
import { experimental_transcribe as transcribe } from 'ai';
import { readFile } from 'fs/promises';

const transcript = await transcribe({
  model: openai.transcription('whisper-1'),
  audio: await readFile('audio.mp3'),
});

const warnings = transcript.warnings;
```

### Error Handling

When `transcribe` cannot generate a valid transcript, it throws a [`AI_NoTranscriptGeneratedError`](/docs/reference/ai-sdk-errors/ai-no-transcript-generated-error).

This error can arise for any the following reasons:

- The model failed to generate a response
- The model generated a response that could not be parsed

The error preserves the following information to help you log the issue:

- `responses`: Metadata about the transcription model responses, including timestamp, model, and headers.
- `cause`: The cause of the error. You can use this for more detailed error handling.

```ts
import {
  experimental_transcribe as transcribe,
  NoTranscriptGeneratedError,
} from 'ai';
import { openai } from '@ai-sdk/openai';
import { readFile } from 'fs/promises';

try {
  await transcribe({
    model: openai.transcription('whisper-1'),
    audio: await readFile('audio.mp3'),
  });
} catch (error) {
  if (NoTranscriptGeneratedError.isInstance(error)) {
    console.log('NoTranscriptGeneratedError');
    console.log('Cause:', error.cause);
    console.log('Responses:', error.responses);
  }
}
```

## Transcription Models

| Provider                                                                  | Model                        |
| ------------------------------------------------------------------------- | ---------------------------- |
| [OpenAI](/providers/ai-sdk-providers/openai#transcription-models)         | `whisper-1`                  |
| [OpenAI](/providers/ai-sdk-providers/openai#transcription-models)         | `gpt-4o-transcribe`          |
| [OpenAI](/providers/ai-sdk-providers/openai#transcription-models)         | `gpt-4o-mini-transcribe`     |
| [ElevenLabs](/providers/ai-sdk-providers/elevenlabs#transcription-models) | `scribe_v1`                  |
| [ElevenLabs](/providers/ai-sdk-providers/elevenlabs#transcription-models) | `scribe_v1_experimental`     |
| [Groq](/providers/ai-sdk-providers/groq#transcription-models)             | `whisper-large-v3-turbo`     |
| [Groq](/providers/ai-sdk-providers/groq#transcription-models)             | `distil-whisper-large-v3-en` |
| [Groq](/providers/ai-sdk-providers/groq#transcription-models)             | `whisper-large-v3`           |
| [Azure OpenAI](/providers/ai-sdk-providers/azure#transcription-models)    | `whisper-1`                  |
| [Azure OpenAI](/providers/ai-sdk-providers/azure#transcription-models)    | `gpt-4o-transcribe`          |
| [Azure OpenAI](/providers/ai-sdk-providers/azure#transcription-models)    | `gpt-4o-mini-transcribe`     |
| [Rev.ai](/providers/ai-sdk-providers/revai#transcription-models)          | `machine`                    |
| [Rev.ai](/providers/ai-sdk-providers/revai#transcription-models)          | `low_cost`                   |
| [Rev.ai](/providers/ai-sdk-providers/revai#transcription-models)          | `fusion`                     |
| [Deepgram](/providers/ai-sdk-providers/deepgram#transcription-models)     | `base` (+ variants)          |
| [Deepgram](/providers/ai-sdk-providers/deepgram#transcription-models)     | `enhanced` (+ variants)      |
| [Deepgram](/providers/ai-sdk-providers/deepgram#transcription-models)     | `nova` (+ variants)          |
| [Deepgram](/providers/ai-sdk-providers/deepgram#transcription-models)     | `nova-2` (+ variants)        |
| [Deepgram](/providers/ai-sdk-providers/deepgram#transcription-models)     | `nova-3` (+ variants)        |
| [Gladia](/providers/ai-sdk-providers/gladia#transcription-models)         | `default`                    |
| [AssemblyAI](/providers/ai-sdk-providers/assemblyai#transcription-models) | `best`                       |
| [AssemblyAI](/providers/ai-sdk-providers/assemblyai#transcription-models) | `nano`                       |
| [Fal](/providers/ai-sdk-providers/fal#transcription-models)               | `whisper`                    |
| [Fal](/providers/ai-sdk-providers/fal#transcription-models)               | `wizper`                     |

Above are a small subset of the transcription models supported by the AI SDK providers. For more, see the respective provider documentation.

# Speech

<Note type="warning">Speech is an experimental feature.</Note>

The AI SDK provides the [`generateSpeech`](/docs/reference/ai-sdk-core/generate-speech)
function to generate speech from text using a speech model.

```ts
import { experimental_generateSpeech as generateSpeech } from 'ai';
import { openai } from '@ai-sdk/openai';

const audio = await generateSpeech({
  model: openai.speech('tts-1'),
  text: 'Hello, world!',
  voice: 'alloy',
});
```

### Language Setting

You can specify the language for speech generation (provider support varies):

```ts
import { experimental_generateSpeech as generateSpeech } from 'ai';
import { lmnt } from '@ai-sdk/lmnt';

const audio = await generateSpeech({
  model: lmnt.speech('aurora'),
  text: 'Hola, mundo!',
  language: 'es', // Spanish
});
```

To access the generated audio:

```ts
const audio = audio.audioData; // audio data e.g. Uint8Array
```

## Settings

### Provider-Specific settings

You can set model-specific settings with the `providerOptions` parameter.

```ts highlight="7-11"
import { experimental_generateSpeech as generateSpeech } from 'ai';
import { openai } from '@ai-sdk/openai';

const audio = await generateSpeech({
  model: openai.speech('tts-1'),
  text: 'Hello, world!',
  providerOptions: {
    openai: {
      // ...
    },
  },
});
```

### Abort Signals and Timeouts

`generateSpeech` accepts an optional `abortSignal` parameter of
type [`AbortSignal`](https://developer.mozilla.org/en-US/docs/Web/API/AbortSignal)
that you can use to abort the speech generation process or set a timeout.

```ts highlight="7"
import { openai } from '@ai-sdk/openai';
import { experimental_generateSpeech as generateSpeech } from 'ai';

const audio = await generateSpeech({
  model: openai.speech('tts-1'),
  text: 'Hello, world!',
  abortSignal: AbortSignal.timeout(1000), // Abort after 1 second
});
```

### Custom Headers

`generateSpeech` accepts an optional `headers` parameter of type `Record<string, string>`
that you can use to add custom headers to the speech generation request.

```ts highlight="7"
import { openai } from '@ai-sdk/openai';
import { experimental_generateSpeech as generateSpeech } from 'ai';

const audio = await generateSpeech({
  model: openai.speech('tts-1'),
  text: 'Hello, world!',
  headers: { 'X-Custom-Header': 'custom-value' },
});
```

### Warnings

Warnings (e.g. unsupported parameters) are available on the `warnings` property.

```ts
import { openai } from '@ai-sdk/openai';
import { experimental_generateSpeech as generateSpeech } from 'ai';

const audio = await generateSpeech({
  model: openai.speech('tts-1'),
  text: 'Hello, world!',
});

const warnings = audio.warnings;
```

### Error Handling

When `generateSpeech` cannot generate a valid audio, it throws a [`AI_NoAudioGeneratedError`](/docs/reference/ai-sdk-errors/ai-no-audio-generated-error).

This error can arise for any the following reasons:

- The model failed to generate a response
- The model generated a response that could not be parsed

The error preserves the following information to help you log the issue:

- `responses`: Metadata about the speech model responses, including timestamp, model, and headers.
- `cause`: The cause of the error. You can use this for more detailed error handling.

```ts
import {
  experimental_generateSpeech as generateSpeech,
  AI_NoAudioGeneratedError,
} from 'ai';
import { openai } from '@ai-sdk/openai';

try {
  await generateSpeech({
    model: openai.speech('tts-1'),
    text: 'Hello, world!',
  });
} catch (error) {
  if (AI_NoAudioGeneratedError.isInstance(error)) {
    console.log('AI_NoAudioGeneratedError');
    console.log('Cause:', error.cause);
    console.log('Responses:', error.responses);
  }
}
```

## Speech Models

| Provider                                                           | Model                    |
| ------------------------------------------------------------------ | ------------------------ |
| [OpenAI](/providers/ai-sdk-providers/openai#speech-models)         | `tts-1`                  |
| [OpenAI](/providers/ai-sdk-providers/openai#speech-models)         | `tts-1-hd`               |
| [OpenAI](/providers/ai-sdk-providers/openai#speech-models)         | `gpt-4o-mini-tts`        |
| [ElevenLabs](/providers/ai-sdk-providers/elevenlabs#speech-models) | `eleven_v3`              |
| [ElevenLabs](/providers/ai-sdk-providers/elevenlabs#speech-models) | `eleven_multilingual_v2` |
| [ElevenLabs](/providers/ai-sdk-providers/elevenlabs#speech-models) | `eleven_flash_v2_5`      |
| [ElevenLabs](/providers/ai-sdk-providers/elevenlabs#speech-models) | `eleven_flash_v2`        |
| [ElevenLabs](/providers/ai-sdk-providers/elevenlabs#speech-models) | `eleven_turbo_v2_5`      |
| [ElevenLabs](/providers/ai-sdk-providers/elevenlabs#speech-models) | `eleven_turbo_v2`        |
| [LMNT](/providers/ai-sdk-providers/lmnt#speech-models)             | `aurora`                 |
| [LMNT](/providers/ai-sdk-providers/lmnt#speech-models)             | `blizzard`               |
| [Hume](/providers/ai-sdk-providers/hume#speech-models)             | `default`                |

Above are a small subset of the speech models supported by the AI SDK providers. For more, see the respective provider documentation.

# Language Model Middleware

Language model middleware is a way to enhance the behavior of language models
by intercepting and modifying the calls to the language model.

It can be used to add features like guardrails, RAG, caching, and logging
in a language model agnostic way. Such middleware can be developed and
distributed independently from the language models that they are applied to.

## Using Language Model Middleware

You can use language model middleware with the `wrapLanguageModel` function.
It takes a language model and a language model middleware and returns a new
language model that incorporates the middleware.

```ts
import { wrapLanguageModel } from 'ai';

const wrappedLanguageModel = wrapLanguageModel({
  model: yourModel,
  middleware: yourLanguageModelMiddleware,
});
```

The wrapped language model can be used just like any other language model, e.g. in `streamText`:

```ts highlight="2"
const result = streamText({
  model: wrappedLanguageModel,
  prompt: 'What cities are in the United States?',
});
```

## Multiple middlewares

You can provide multiple middlewares to the `wrapLanguageModel` function.
The middlewares will be applied in the order they are provided.

```ts
const wrappedLanguageModel = wrapLanguageModel({
  model: yourModel,
  middleware: [firstMiddleware, secondMiddleware],
});

// applied as: firstMiddleware(secondMiddleware(yourModel))
```

## Built-in Middleware

The AI SDK comes with several built-in middlewares that you can use to configure language models:

- `extractReasoningMiddleware`: Extracts reasoning information from the generated text and exposes it as a `reasoning` property on the result.
- `simulateStreamingMiddleware`: Simulates streaming behavior with responses from non-streaming language models.
- `defaultSettingsMiddleware`: Applies default settings to a language model.

### Extract Reasoning

Some providers and models expose reasoning information in the generated text using special tags,
e.g. &lt;think&gt; and &lt;/think&gt;.

The `extractReasoningMiddleware` function can be used to extract this reasoning information and expose it as a `reasoning` property on the result.

```ts
import { wrapLanguageModel, extractReasoningMiddleware } from 'ai';

const model = wrapLanguageModel({
  model: yourModel,
  middleware: extractReasoningMiddleware({ tagName: 'think' }),
});
```

You can then use that enhanced model in functions like `generateText` and `streamText`.

The `extractReasoningMiddleware` function also includes a `startWithReasoning` option.
When set to `true`, the reasoning tag will be prepended to the generated text.
This is useful for models that do not include the reasoning tag at the beginning of the response.
For more details, see the [DeepSeek R1 guide](/docs/guides/r1#deepseek-r1-middleware).

### Simulate Streaming

The `simulateStreamingMiddleware` function can be used to simulate streaming behavior with responses from non-streaming language models.
This is useful when you want to maintain a consistent streaming interface even when using models that only provide complete responses.

```ts
import { wrapLanguageModel, simulateStreamingMiddleware } from 'ai';

const model = wrapLanguageModel({
  model: yourModel,
  middleware: simulateStreamingMiddleware(),
});
```

### Default Settings

The `defaultSettingsMiddleware` function can be used to apply default settings to a language model.

```ts
import { wrapLanguageModel, defaultSettingsMiddleware } from 'ai';

const model = wrapLanguageModel({
  model: yourModel,
  middleware: defaultSettingsMiddleware({
    settings: {
      temperature: 0.5,
      maxOutputTokens: 800,
      providerOptions: { openai: { store: false } },
    },
  }),
});
```

## Community Middleware

The AI SDK provides a Language Model Middleware specification. Community members can develop middleware that adheres to this specification, making it compatible with the AI SDK ecosystem.

Here are some community middlewares that you can explore:

### Custom tool call parser

The [Custom tool call parser](https://github.com/minpeter/ai-sdk-tool-call-middleware) middleware extends tool call capabilities to models that don't natively support the OpenAI-style `tools` parameter. This includes many self-hosted and third-party models that lack native function calling features.

<Note>
  Using this middleware on models that support native function calls may result
  in unintended performance degradation, so check whether your model supports
  native function calls before deciding to use it.
</Note>

This middleware enables function calling capabilities by converting function schemas into prompt instructions and parsing the model's responses into structured function calls. It works by transforming the JSON function definitions into natural language instructions the model can understand, then analyzing the generated text to extract function call attempts. This approach allows developers to use the same function calling API across different model providers, even with models that don't natively support the OpenAI-style function calling format, providing a consistent function calling experience regardless of the underlying model implementation.

The `@ai-sdk-tool/parser` package offers three middleware variants:

- `createToolMiddleware`: A flexible function for creating custom tool call middleware tailored to specific models
- `hermesToolMiddleware`: Ready-to-use middleware for Hermes & Qwen format function calls
- `gemmaToolMiddleware`: Pre-configured middleware for Gemma 3 model series function call format

Here's how you can enable function calls with Gemma models that don't support them natively:

```ts
import { wrapLanguageModel } from 'ai';
import { gemmaToolMiddleware } from '@ai-sdk-tool/parser';

const model = wrapLanguageModel({
  model: openrouter('google/gemma-3-27b-it'),
  middleware: gemmaToolMiddleware,
});
```

Find more examples at this [link](https://github.com/minpeter/ai-sdk-tool-call-middleware/tree/main/examples/core/src).

## Implementing Language Model Middleware

<Note>
  Implementing language model middleware is advanced functionality and requires
  a solid understanding of the [language model
  specification](https://github.com/vercel/ai/blob/v5/packages/provider/src/language-model/v2/language-model-v2.ts).
</Note>

You can implement any of the following three function to modify the behavior of the language model:

1. `transformParams`: Transforms the parameters before they are passed to the language model, for both `doGenerate` and `doStream`.
2. `wrapGenerate`: Wraps the `doGenerate` method of the [language model](https://github.com/vercel/ai/blob/v5/packages/provider/src/language-model/v2/language-model-v2.ts).
   You can modify the parameters, call the language model, and modify the result.
3. `wrapStream`: Wraps the `doStream` method of the [language model](https://github.com/vercel/ai/blob/v5/packages/provider/src/language-model/v2/language-model-v2.ts).
   You can modify the parameters, call the language model, and modify the result.

Here are some examples of how to implement language model middleware:

## Examples

<Note>
  These examples are not meant to be used in production. They are just to show
  how you can use middleware to enhance the behavior of language models.
</Note>

### Logging

This example shows how to log the parameters and generated text of a language model call.

```ts
import type {
  LanguageModelV2Middleware,
  LanguageModelV2StreamPart,
} from '@ai-sdk/provider';

export const yourLogMiddleware: LanguageModelV2Middleware = {
  wrapGenerate: async ({ doGenerate, params }) => {
    console.log('doGenerate called');
    console.log(`params: ${JSON.stringify(params, null, 2)}`);

    const result = await doGenerate();

    console.log('doGenerate finished');
    console.log(`generated text: ${result.text}`);

    return result;
  },

  wrapStream: async ({ doStream, params }) => {
    console.log('doStream called');
    console.log(`params: ${JSON.stringify(params, null, 2)}`);

    const { stream, ...rest } = await doStream();

    let generatedText = '';
    const textBlocks = new Map<string, string>();

    const transformStream = new TransformStream<
      LanguageModelV2StreamPart,
      LanguageModelV2StreamPart
    >({
      transform(chunk, controller) {
        switch (chunk.type) {
          case 'text-start': {
            textBlocks.set(chunk.id, '');
            break;
          }
          case 'text-delta': {
            const existing = textBlocks.get(chunk.id) || '';
            textBlocks.set(chunk.id, existing + chunk.delta);
            generatedText += chunk.delta;
            break;
          }
          case 'text-end': {
            console.log(
              `Text block ${chunk.id} completed:`,
              textBlocks.get(chunk.id),
            );
            break;
          }
        }

        controller.enqueue(chunk);
      },

      flush() {
        console.log('doStream finished');
        console.log(`generated text: ${generatedText}`);
      },
    });

    return {
      stream: stream.pipeThrough(transformStream),
      ...rest,
    };
  },
};
```

### Caching

This example shows how to build a simple cache for the generated text of a language model call.

```ts
import type { LanguageModelV2Middleware } from '@ai-sdk/provider';

const cache = new Map<string, any>();

export const yourCacheMiddleware: LanguageModelV2Middleware = {
  wrapGenerate: async ({ doGenerate, params }) => {
    const cacheKey = JSON.stringify(params);

    if (cache.has(cacheKey)) {
      return cache.get(cacheKey);
    }

    const result = await doGenerate();

    cache.set(cacheKey, result);

    return result;
  },

  // here you would implement the caching logic for streaming
};
```

### Retrieval Augmented Generation (RAG)

This example shows how to use RAG as middleware.

<Note>
  Helper functions like `getLastUserMessageText` and `findSources` are not part
  of the AI SDK. They are just used in this example to illustrate the concept of
  RAG.
</Note>

```ts
import type { LanguageModelV2Middleware } from '@ai-sdk/provider';

export const yourRagMiddleware: LanguageModelV2Middleware = {
  transformParams: async ({ params }) => {
    const lastUserMessageText = getLastUserMessageText({
      prompt: params.prompt,
    });

    if (lastUserMessageText == null) {
      return params; // do not use RAG (send unmodified parameters)
    }

    const instruction =
      'Use the following information to answer the question:\n' +
      findSources({ text: lastUserMessageText })
        .map(chunk => JSON.stringify(chunk))
        .join('\n');

    return addToLastUserMessage({ params, text: instruction });
  },
};
```

### Guardrails

Guard rails are a way to ensure that the generated text of a language model call
is safe and appropriate. This example shows how to use guardrails as middleware.

```ts
import type { LanguageModelV2Middleware } from '@ai-sdk/provider';

export const yourGuardrailMiddleware: LanguageModelV2Middleware = {
  wrapGenerate: async ({ doGenerate }) => {
    const { text, ...rest } = await doGenerate();

    // filtering approach, e.g. for PII or other sensitive information:
    const cleanedText = text?.replace(/badword/g, '<REDACTED>');

    return { text: cleanedText, ...rest };
  },

  // here you would implement the guardrail logic for streaming
  // Note: streaming guardrails are difficult to implement, because
  // you do not know the full content of the stream until it's finished.
};
```

## Configuring Per Request Custom Metadata

To send and access custom metadata in Middleware, you can use `providerOptions`. This is useful when building logging middleware where you want to pass additional context like user IDs, timestamps, or other contextual data that can help with tracking and debugging.

```ts
import { openai } from '@ai-sdk/openai';
import { generateText, wrapLanguageModel } from 'ai';
import type { LanguageModelV2Middleware } from '@ai-sdk/provider';

export const yourLogMiddleware: LanguageModelV2Middleware = {
  wrapGenerate: async ({ doGenerate, params }) => {
    console.log('METADATA', params?.providerMetadata?.yourLogMiddleware);
    const result = await doGenerate();
    return result;
  },
};

const { text } = await generateText({
  model: wrapLanguageModel({
    model: openai('gpt-4o'),
    middleware: yourLogMiddleware,
  }),
  prompt: 'Invent a new holiday and describe its traditions.',
  providerOptions: {
    yourLogMiddleware: {
      hello: 'world',
    },
  },
});

console.log(text);
```

# Provider & Model Management

When you work with multiple providers and models, it is often desirable to manage them in a central place
and access the models through simple string ids.

The AI SDK offers [custom providers](/docs/reference/ai-sdk-core/custom-provider) and
a [provider registry](/docs/reference/ai-sdk-core/provider-registry) for this purpose:

- With **custom providers**, you can pre-configure model settings, provide model name aliases,
  and limit the available models.
- The **provider registry** lets you mix multiple providers and access them through simple string ids.

You can mix and match custom providers, the provider registry, and [middleware](/docs/ai-sdk-core/middleware) in your application.

## Custom Providers

You can create a [custom provider](/docs/reference/ai-sdk-core/custom-provider) using `customProvider`.

### Example: custom model settings

You might want to override the default model settings for a provider or provide model name aliases
with pre-configured settings.

```ts
import { openai as originalOpenAI } from '@ai-sdk/openai';
import {
  customProvider,
  defaultSettingsMiddleware,
  wrapLanguageModel,
} from 'ai';

// custom provider with different provider options:
export const openai = customProvider({
  languageModels: {
    // replacement model with custom provider options:
    'gpt-4o': wrapLanguageModel({
      model: originalOpenAI('gpt-4o'),
      middleware: defaultSettingsMiddleware({
        settings: {
          providerOptions: {
            openai: {
              reasoningEffort: 'high',
            },
          },
        },
      }),
    }),
    // alias model with custom provider options:
    'gpt-4o-mini-high-reasoning': wrapLanguageModel({
      model: originalOpenAI('gpt-4o-mini'),
      middleware: defaultSettingsMiddleware({
        settings: {
          providerOptions: {
            openai: {
              reasoningEffort: 'high',
            },
          },
        },
      }),
    }),
  },
  fallbackProvider: originalOpenAI,
});
```

### Example: model name alias

You can also provide model name aliases, so you can update the model version in one place in the future:

```ts
import { anthropic as originalAnthropic } from '@ai-sdk/anthropic';
import { customProvider } from 'ai';

// custom provider with alias names:
export const anthropic = customProvider({
  languageModels: {
    opus: originalAnthropic('claude-3-opus-20240229'),
    sonnet: originalAnthropic('claude-3-5-sonnet-20240620'),
    haiku: originalAnthropic('claude-3-haiku-20240307'),
  },
  fallbackProvider: originalAnthropic,
});
```

### Example: limit available models

You can limit the available models in the system, even if you have multiple providers.

```ts
import { anthropic } from '@ai-sdk/anthropic';
import { openai } from '@ai-sdk/openai';
import {
  customProvider,
  defaultSettingsMiddleware,
  wrapLanguageModel,
} from 'ai';

export const myProvider = customProvider({
  languageModels: {
    'text-medium': anthropic('claude-3-5-sonnet-20240620'),
    'text-small': openai('gpt-4o-mini'),
    'reasoning-medium': wrapLanguageModel({
      model: openai('gpt-4o'),
      middleware: defaultSettingsMiddleware({
        settings: {
          providerOptions: {
            openai: {
              reasoningEffort: 'high',
            },
          },
        },
      }),
    }),
    'reasoning-fast': wrapLanguageModel({
      model: openai('gpt-4o-mini'),
      middleware: defaultSettingsMiddleware({
        settings: {
          providerOptions: {
            openai: {
              reasoningEffort: 'high',
            },
          },
        },
      }),
    }),
  },
  embeddingModels: {
    embedding: openai.textEmbeddingModel('text-embedding-3-small'),
  },
  // no fallback provider
});
```

## Provider Registry

You can create a [provider registry](/docs/reference/ai-sdk-core/provider-registry) with multiple providers and models using `createProviderRegistry`.

### Setup

```ts filename={"registry.ts"}
import { anthropic } from '@ai-sdk/anthropic';
import { createOpenAI } from '@ai-sdk/openai';
import { createProviderRegistry } from 'ai';

export const registry = createProviderRegistry({
  // register provider with prefix and default setup:
  anthropic,

  // register provider with prefix and custom setup:
  openai: createOpenAI({
    apiKey: process.env.OPENAI_API_KEY,
  }),
});
```

### Setup with Custom Separator

By default, the registry uses `:` as the separator between provider and model IDs. You can customize this separator:

```ts filename={"registry.ts"}
import { createProviderRegistry } from 'ai';
import { anthropic } from '@ai-sdk/anthropic';
import { openai } from '@ai-sdk/openai';

export const customSeparatorRegistry = createProviderRegistry(
  {
    anthropic,
    openai,
  },
  { separator: ' > ' },
);
```

### Example: Use language models

You can access language models by using the `languageModel` method on the registry.
The provider id will become the prefix of the model id: `providerId:modelId`.

```ts highlight={"5"}
import { generateText } from 'ai';
import { registry } from './registry';

const { text } = await generateText({
  model: registry.languageModel('openai:gpt-4.1'), // default separator
  // or with custom separator:
  // model: customSeparatorRegistry.languageModel('openai > gpt-4.1'),
  prompt: 'Invent a new holiday and describe its traditions.',
});
```

### Example: Use text embedding models

You can access text embedding models by using the `textEmbeddingModel` method on the registry.
The provider id will become the prefix of the model id: `providerId:modelId`.

```ts highlight={"5"}
import { embed } from 'ai';
import { registry } from './registry';

const { embedding } = await embed({
  model: registry.textEmbeddingModel('openai:text-embedding-3-small'),
  value: 'sunny day at the beach',
});
```

### Example: Use image models

You can access image models by using the `imageModel` method on the registry.
The provider id will become the prefix of the model id: `providerId:modelId`.

```ts highlight={"5"}
import { generateImage } from 'ai';
import { registry } from './registry';

const { image } = await generateImage({
  model: registry.imageModel('openai:dall-e-3'),
  prompt: 'A beautiful sunset over a calm ocean',
});
```

## Combining Custom Providers, Provider Registry, and Middleware

The central idea of provider management is to set up a file that contains all the providers and models you want to use.
You may want to pre-configure model settings, provide model name aliases, limit the available models, and more.

Here is an example that implements the following concepts:

- pass through a full provider with a namespace prefix (here: `xai > *`)
- setup an OpenAI-compatible provider with custom api key and base URL (here: `custom > *`)
- setup model name aliases (here: `anthropic > fast`, `anthropic > writing`, `anthropic > reasoning`)
- pre-configure model settings (here: `anthropic > reasoning`)
- validate the provider-specific options (here: `AnthropicProviderOptions`)
- use a fallback provider (here: `anthropic > *`)
- limit a provider to certain models without a fallback (here: `groq > gemma2-9b-it`, `groq > qwen-qwq-32b`)
- define a custom separator for the provider registry (here: `>`)

```ts
import { anthropic, AnthropicProviderOptions } from '@ai-sdk/anthropic';
import { createOpenAICompatible } from '@ai-sdk/openai-compatible';
import { xai } from '@ai-sdk/xai';
import { groq } from '@ai-sdk/groq';
import {
  createProviderRegistry,
  customProvider,
  defaultSettingsMiddleware,
  wrapLanguageModel,
} from 'ai';

export const registry = createProviderRegistry(
  {
    // pass through a full provider with a namespace prefix
    xai,

    // access an OpenAI-compatible provider with custom setup
    custom: createOpenAICompatible({
      name: 'provider-name',
      apiKey: process.env.CUSTOM_API_KEY,
      baseURL: 'https://api.custom.com/v1',
    }),

    // setup model name aliases
    anthropic: customProvider({
      languageModels: {
        fast: anthropic('claude-3-haiku-20240307'),

        // simple model
        writing: anthropic('claude-3-7-sonnet-20250219'),

        // extended reasoning model configuration:
        reasoning: wrapLanguageModel({
          model: anthropic('claude-3-7-sonnet-20250219'),
          middleware: defaultSettingsMiddleware({
            settings: {
              maxOutputTokens: 100000, // example default setting
              providerOptions: {
                anthropic: {
                  thinking: {
                    type: 'enabled',
                    budgetTokens: 32000,
                  },
                } satisfies AnthropicProviderOptions,
              },
            },
          }),
        }),
      },
      fallbackProvider: anthropic,
    }),

    // limit a provider to certain models without a fallback
    groq: customProvider({
      languageModels: {
        'gemma2-9b-it': groq('gemma2-9b-it'),
        'qwen-qwq-32b': groq('qwen-qwq-32b'),
      },
    }),
  },
  { separator: ' > ' },
);

// usage:
const model = registry.languageModel('anthropic > reasoning');
```

## Global Provider Configuration

The AI SDK 5 includes a global provider feature that allows you to specify a model using just a plain model ID string:

```ts
import { streamText } from 'ai';

const result = await streamText({
  model: 'openai/gpt-4o', // Uses the global provider (defaults to AI Gateway)
  prompt: 'Invent a new holiday and describe its traditions.',
});
```

By default, the global provider is set to the Vercel AI Gateway.

### Customizing the Global Provider

You can set your own preferred global provider:

```ts filename="setup.ts"
import { openai } from '@ai-sdk/openai';

// Initialize once during startup:
globalThis.AI_SDK_DEFAULT_PROVIDER = openai;
```

```ts filename="app.ts"
import { streamText } from 'ai';

const result = await streamText({
  model: 'gpt-4o', // Uses OpenAI provider without prefix
  prompt: 'Invent a new holiday and describe its traditions.',
});
```

This simplifies provider usage and makes it easier to switch between providers without changing your model references throughout your codebase.
