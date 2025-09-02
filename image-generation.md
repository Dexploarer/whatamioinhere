# Image Generation

AI Gateway supports image generation and editing capabilities. You can generate new images from text prompts, edit existing images, and create variations with natural language instructions.

You can view all available models that support image generation by using the Image filter at the [AI Gateway Models page](https://vercel.com/ai-gateway/models).

## [Google Gemini 2.5 Flash Image](#google-gemini-2.5-flash-image)

Google's [Gemini 2.5 Flash Image model](https://developers.googleblog.com/en/introducing-gemini-2-5-flash-image/) offers state-of-the-art image generation and editing capabilities. This model supports [specifying response modalities](https://ai-sdk.dev/providers/ai-sdk-providers/google-generative-ai#image-outputs) to enable image outputs alongside text responses.

You can find details on this model in the [Model Library](https://vercel.com/ai-gateway/models/gemini-2.5-flash-image-preview).

## [Basic image generation](#basic-image-generation)

Generate images from text prompts using the `generateText` function with appropriate provider options:

TypeScript (generateText)TypeScript (streamText)TypeScript (Chatbot)

```
import 'dotenv/config';
import { generateText } from 'ai';
import fs from 'node:fs';
import path from 'node:path';
 
async function main() {
  const result = await generateText({
    model: 'google/gemini-2.5-flash-image-preview',
    providerOptions: {
      google: { responseModalities: ['TEXT', 'IMAGE'] },
    },
    prompt:
      'Render two versions of a pond tortoise sleeping on a log in a lake at sunset.',
  });
 
  if (result.text) {
    console.log(result.text);
  }
 
  // Save generated images to local filesystem
  const imageFiles = result.files.filter((f) =>
    f.mediaType?.startsWith('image/'),
  );
 
  if (imageFiles.length > 0) {
    // Create output directory if it doesn't exist
    const outputDir = 'output';
    fs.mkdirSync(outputDir, { recursive: true });
 
    const timestamp = Date.now();
 
    for (const [index, file] of imageFiles.entries()) {
      const extension = file.mediaType?.split('/')[1] || 'png';
      const filename = `image-${timestamp}-${index}.${extension}`;
      const filepath = path.join(outputDir, filename);
 
      await fs.promises.writeFile(filepath, file.uint8Array);
      console.log(`Saved image to ${filepath}`);
    }
  }
 
  console.log();
  console.log('Usage: ', JSON.stringify(result.usage, null, 2));
  console.log(
    'Provider metadata: ',
    JSON.stringify(result.providerMetadata, null, 2),
  );
}
 
main().catch(console.error);
```

```
import 'dotenv/config';
import { streamText } from 'ai';
import fs from 'node:fs';
import path from 'node:path';
 
async function main() {
  const result = streamText({
    model: 'google/gemini-2.5-flash-image-preview',
    providerOptions: {
      google: { responseModalities: ['TEXT', 'IMAGE'] },
    },
    prompt: 'Render a pond tortoise sleeping on a log in a lake at sunset.',
  });
 
  // Create output directory if it doesn't exist
  const outputDir = 'output';
  fs.mkdirSync(outputDir, { recursive: true });
  const timestamp = Date.now();
  let imageIndex = 0;
 
  for await (const delta of result.fullStream) {
    switch (delta.type) {
      case 'text-delta': {
        process.stdout.write(delta.text);
        break;
      }
 
      case 'file': {
        if (delta.file.mediaType.startsWith('image/')) {
          console.log();
 
          const extension = delta.file.mediaType?.split('/')[1] || 'png';
          const filename = `image-${timestamp}-${imageIndex}.${extension}`;
          const filepath = path.join(outputDir, filename);
 
          await fs.promises.writeFile(filepath, delta.file.uint8Array);
          console.log(`Saved image to ${filepath}`);
          imageIndex++;
        }
        break;
      }
    }
  }
  process.stdout.write('\n\n');
 
  console.log();
  console.log('Finish reason: ', result.finishReason);
  console.log('Usage: ', JSON.stringify(await result.usage, null, 2));
  console.log(
    'Provider metadata: ',
    JSON.stringify(await result.providerMetadata, null, 2),
  );
}
 
main().catch(console.error);
```

```
import { type ModelMessage, streamText } from 'ai';
import 'dotenv/config';
import * as readline from 'node:readline/promises';
import fs from 'node:fs';
import path from 'node:path';
 
const terminal = readline.createInterface({
  input: process.stdin,
  output: process.stdout,
});
 
const messages: ModelMessage[] = [];
 
async function main() {
  // Create output directory if it doesn't exist
  const outputDir = 'output';
  fs.mkdirSync(outputDir, { recursive: true });
 
  while (true) {
    messages.push({ role: 'user', content: await terminal.question('You: ') });
 
    const result = streamText({
      model: 'google/gemini-2.5-flash-image-preview',
      providerOptions: {
        google: { responseModalities: ['TEXT', 'IMAGE'] },
      },
      messages,
    });
 
    process.stdout.write('\nAssistant: ');
    const timestamp = Date.now();
    let imageIndex = 0;
 
    for await (const delta of result.fullStream) {
      switch (delta.type) {
        case 'text-delta': {
          process.stdout.write(delta.text);
          break;
        }
 
        case 'file': {
          if (delta.file.mediaType.startsWith('image/')) {
            console.log();
 
            const extension = delta.file.mediaType?.split('/')[1] || 'png';
            const filename = `chat-image-${timestamp}-${imageIndex}.${extension}`;
            const filepath = path.join(outputDir, filename);
 
            await fs.promises.writeFile(filepath, delta.file.uint8Array);
            console.log(`💾 Saved image to ${filepath}`);
            imageIndex++;
          }
          break;
        }
      }
    }
    process.stdout.write('\n\n');
 
    console.log('Usage: ', JSON.stringify(await result.usage, null, 2));
    console.log(
      'Provider metadata: ',
      JSON.stringify(await result.providerMetadata, null, 2),
    );
 
    messages.push(...(await result.response).messages);
  }
}
 
main().catch(console.error);
```

Check the [AI SDK provider documentation](https://ai-sdk.dev/providers/ai-sdk-providers) for more on provider/model-specific image generation configuration.

For OpenAI-compatible API usage with image generation, see the [OpenAI-Compatible API Image Generation section](/docs/ai-gateway/openai-compat#image-generation).

## [OpenAI-compatible API response format](#openai-compatible-api-response-format)

When using the OpenAI-compatible API (`/v1/chat/completions`) for image generation, responses follow a specific format that separates text content from generated images:

### [Response structure](#response-structure)

```
{
  "id": "chatcmpl-123",
  "object": "chat.completion",
  "created": 1677652288,
  "model": "google/gemini-2.5-flash-image-preview",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "I've generated a beautiful sunset image for you.",
        "images": [
          {
            "type": "image_url",
            "image_url": {
              "url": "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAA..."
            }
          }
        ]
      },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 15,
    "completion_tokens": 28,
    "total_tokens": 43
  }
}
```

### [Key format details](#key-format-details)

*   `content`: Contains the text description as a string
*   `images`: Array of generated images, each with:
    *   `type`: Always `"image_url"`
    *   `image_url.url`: Base64-encoded data URI of the generated image

### [Streaming responses](#streaming-responses)

For streaming requests, images are delivered in delta chunks:

```
{
  "id": "chatcmpl-123",
  "object": "chat.completion.chunk",
  "created": 1677652288,
  "model": "google/gemini-2.5-flash-image-preview",
  "choices": [
    {
      "index": 0,
      "delta": {
        "images": [
          {
            "type": "image_url",
            "image_url": {
              "url": "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAA..."
            }
          }
        ]
      },
      "finish_reason": null
    }
  ]
}
```

## [Handling generated images](#handling-generated-images)

Generated images are returned as `GeneratedFile` objects in the `result.files` array. Each contains:

*   `base64`: The file as a base 64 data string
*   `uint8Array`: The file as a `Uint8Array`
*   `mediaType`: The MIME type (e.g., `image/png`, `image/jpeg`)

## [Streaming image generation](#streaming-image-generation)

When using `streamText`, images are delivered through `fullStream` as `file` events:

```
for await (const delta of result.fullStream) {
  switch (delta.type) {
    case 'text-delta':
      // Handle text chunks
      process.stdout.write(delta.text);
      break;
 
    case 'file':
      // Handle generated files (images)
      if (delta.file.mediaType.startsWith('image/')) {
        await saveImage(delta.file);
      }
      break;
  }
}
```
