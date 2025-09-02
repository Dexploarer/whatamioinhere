# Provider Options

AI Gateway can route your AI model requests across multiple AI providers. Each provider offers different models, pricing, and performance characteristics. By default, AI Gateway will automatically choose providers for you to ensure fast and dependable responses.

With the Gateway Provider Options, you can control the routing order and fallback behavior of the models.

If you want to customize individual AI model provider settings rather than general AI Gateway behavior, please refer to the model-specific provider options in the [AI SDK documentation](https://ai-sdk.dev/docs/foundations/prompts#provider-options).

## [Basic provider ordering](#basic-provider-ordering)

You can use the `order` array to specify the sequence in which providers should be attempted. Providers are specified using their `slug` string. You can find the slugs in the [table of available providers](#available-providers).

You can also copy the provider slug using the copy button next to a provider's name on a model's detail page. In the Vercel Dashboard:

1.  Click the AI Gateway tab,
2.  Then, click the Model List sub-tab on the left
3.  Click a model entry in the list.

The bottom section of the page lists the available providers for that model. The copy button next to a provider's name will copy their slug for pasting.

### [Getting started with adding a provider option](#getting-started-with-adding-a-provider-option)

1.  #### [Install the AI SDK package](#install-the-ai-sdk-package)
    
    First, ensure you have the necessary package installed:
    
    ```
    pnpm install ai
    ```
    
2.  #### [Configure the provider order in your request](#configure-the-provider-order-in-your-request)
    
    Use the `providerOptions.gateway.order` configuration:
    
    ```
    import { streamText } from 'ai';
     
    export async function POST(request: Request) {
      const { prompt } = await request.json();
     
      const result = streamText({
        model: 'anthropic/claude-sonnet-4',
        prompt,
        providerOptions: {
          gateway: {
            order: ['bedrock', 'anthropic'], // Try Amazon Bedrock first, then Anthropic
          },
        },
      });
     
      return result.toUIMessageStreamResponse();
    }
    ```
    
    In this example:
    
    *   The gateway will first attempt to use Amazon Bedrock to serve the Claude 4 Sonnet model
    *   If Amazon Bedrock is unavailable or fails, it will fall back to Anthropic
    *   Other providers (like Vertex AI) are still available but will only be used after the specified providers
3.  #### [Test the routing behavior](#test-the-routing-behavior)
    
    You can monitor which provider you used by checking the provider metadata in the response.
    
    ```
    import { streamText } from 'ai';
     
    export async function POST(request: Request) {
      const { prompt } = await request.json();
     
      const result = streamText({
        model: 'anthropic/claude-sonnet-4',
        prompt,
        providerOptions: {
          gateway: {
            order: ['bedrock', 'anthropic'],
          },
        },
      });
     
      // Log which provider was actually used
      console.log(JSON.stringify(await result.providerMetadata, null, 2));
     
      return result.toUIMessageStreamResponse();
    }
    ```
    

## [Example provider metadata output](#example-provider-metadata-output)

```
{
  "novita": {}, // final provider-specific metadata, if any -- can be empty
  "gateway": {
    // gateway-specific metadata
    "routing": {
      "originalModelId": "zai/glm-4.5",
      "resolvedProvider": "novita",
      "resolvedProviderApiModelId": "zai-org/glm-4.5",
      "fallbacksAvailable": ["zai"],
      "planningReasoning": "System credentials planned for: novita, zai. Total execution order: novita(system) → zai(system)",
      "canonicalSlug": "zai/glm-4.5",
      "finalProvider": "novita",
      "attempts": [
        {
          "provider": "novita",
          "providerApiModelId": "zai-org/glm-4.5",
          "credentialType": "system",
          "success": true,
          "startTime": 1754638578812,
          "endTime": 1754638579575
        }
      ]
    },
    "cost": "0.0006766"
  }
}
```

The `gateway.cost` value is the amount debited from your AI Gateway Credits balance for this request. It is returned as a decimal string. For more on pricing see [Pricing](/docs/ai-gateway/pricing).

In cases where your request encounters issues with one or more providers or if your BYOK credentials fail, you'll find error detail in the `attempts` field of the provider metadata:

```
"attempts": [
  {
    "provider": "novita",
    "providerApiModelId": "zai-org/glm-4.5",
    "credentialType": "byok",
    "success": false,
    "error": "Unauthorized",
    "startTime": 1754639042520,
    "endTime": 1754639042710
  },
  {
    "provider": "novita",
    "providerApiModelId": "zai-org/glm-4.5",
    "credentialType": "system",
    "success": true,
    "startTime": 1754639042710,
    "endTime": 1754639043353
  }
]
```

## [Restrict providers with the `only` filter](#restrict-providers-with-the-only-filter)

Use the `only` array to restrict routing to a specific subset of providers. Providers are specified by their slug and are matched against the model's available providers.

```
import { streamText } from 'ai';
 
export async function POST(request: Request) {
  const { prompt } = await request.json();
 
  const result = streamText({
    model: 'anthropic/claude-sonnet-4',
    prompt,
    providerOptions: {
      gateway: {
        only: ['bedrock', 'anthropic'], // Only consider these providers.
        // This model is also available via 'vertex', but it won't be considered.
      },
    },
  });
 
  return result.toUIMessageStreamResponse();
}
```

In this example:

*   Restriction: Only `bedrock` and `anthropic` will be considered for routing and fallbacks.
*   Error on mismatch: If none of the specified providers are available for the model, the request fails with an error indicating the allowed providers.

## [Using `only` together with `order`](#using-only-together-with-order)

When both `only` and `order` are provided, the `only` filter is applied first to define the allowed set, and then `order` defines the priority within that filtered set. Practically, the end result is the same as taking your `order` list and intersecting it with the `only` list.

```
import { streamText } from 'ai';
 
export async function POST(request: Request) {
  const { prompt } = await request.json();
 
  const result = streamText({
    model: 'anthropic/claude-sonnet-4',
    prompt,
    providerOptions: {
      gateway: {
        only: ['anthropic', 'vertex'],
        order: ['vertex', 'bedrock', 'anthropic'],
      },
    },
  });
 
  return result.toUIMessageStreamResponse();
}
```

The final order will be `vertex → anthropic` (providers listed in `order` but not in `only` are ignored).

## [Combining AI Gateway provider options with provider-specific options](#combining-ai-gateway-provider-options-with-provider-specific-options)

You can combine AI Gateway provider options with provider-specific options. This allows you to control both the routing behavior and provider-specific settings in the same request:

```
import { streamText } from 'ai';
 
export async function POST(request: Request) {
  const { prompt } = await request.json();
 
  const result = streamText({
    model: 'anthropic/claude-sonnet-4',
    prompt,
    providerOptions: {
      anthropic: {
        thinkingBudget: 0.001,
      },
      gateway: {
        order: ['vertex'],
      },
    },
  });
 
  return result.toUIMessageStreamResponse();
}
```

In this example:

*   We're using an Anthropic model (e.g. Claude 4 Sonnet) but accessing it through Vertex AI
*   The Anthropic-specific options still apply to the model:
    *   `thinkingBudget` sets a cost limit of $0.001 per request for the Claude model
*   You can read more about provider-specific options in the [AI SDK documentation](https://ai-sdk.dev/docs/foundations/prompts#provider-options)

## [Reasoning](#reasoning)

For models that support reasoning (also known as "thinking"), you can use `providerOptions` to configure reasoning behavior. The example below shows how to control the computational effort and summary detail level when using OpenAI's `gpt-oss-120b` model.

For more details on reasoning support across different models and providers, see the [AI SDK providers documentation](https://ai-sdk.dev/providers/ai-sdk-providers), including [OpenAI](https://ai-sdk.dev/providers/ai-sdk-providers/openai#reasoning), [DeepSeek](https://ai-sdk.dev/providers/ai-sdk-providers/deepseek#reasoning), and [Anthropic](https://ai-sdk.dev/providers/ai-sdk-providers/anthropic#reasoning).

```
import { streamText } from 'ai';
 
export async function POST(request: Request) {
  const { prompt } = await request.json();
 
  const result = streamText({
    model: 'openai/gpt-oss-120b',
    prompt,
    providerOptions: {
      openai: {
        reasoningEffort: 'high',
        reasoningSummary: 'detailed',
      },
    },
  });
 
  return result.toUIMessageStreamResponse();
}
```

## [Available providers](#available-providers)

You can view the available models for a provider in the Model List section under the [AI Gateway](https://vercel.com/d?to=%2F%5Bteam%5D%2F%7E%2Fai&title=Go+to+AI+Gateway) tab in your Vercel dashboard or in the public [models page](https://vercel.com/ai-gateway/models).

| Slug | Name | Website |
| --- | --- | --- |
| `anthropic` | [Anthropic](https://ai-sdk.dev/providers/ai-sdk-providers/anthropic) | [anthropic.com](https://anthropic.com) |
| `azure` | [Azure](https://ai-sdk.dev/providers/ai-sdk-providers/azure) | [ai.azure.com](https://ai.azure.com/) |
| `baseten` | [Baseten](https://ai-sdk.dev/providers/openai-compatible-providers/baseten) | [baseten.co](https://www.baseten.co/)  |
| `bedrock` | [Amazon Bedrock](https://ai-sdk.dev/providers/ai-sdk-providers/amazon-bedrock) | [aws.amazon.com/bedrock](https://aws.amazon.com/bedrock) |
| `cerebras` | [Cerebras](https://ai-sdk.dev/providers/ai-sdk-providers/cerebras) | [cerebras.net](https://www.cerebras.net) |
| `cohere` | [Cohere](https://ai-sdk.dev/providers/ai-sdk-providers/cohere) | [cohere.com](https://cohere.com) |
| `deepinfra` | [DeepInfra](https://ai-sdk.dev/providers/ai-sdk-providers/deepinfra) | [deepinfra.com](https://deepinfra.com) |
| `deepseek` | [DeepSeek](https://ai-sdk.dev/providers/ai-sdk-providers/deepseek) | [deepseek.ai](https://deepseek.ai) |
| `fireworks` | [Fireworks](https://ai-sdk.dev/providers/ai-sdk-providers/fireworks) | [fireworks.ai](https://fireworks.ai) |
| `google` | [Google](https://ai-sdk.dev/providers/ai-sdk-providers/google-generative-ai) | [ai.google.dev](https://ai.google.dev/) |
| `groq` | [Groq](https://ai-sdk.dev/providers/ai-sdk-providers/groq) | [groq.com](https://groq.com) |
| `inception` | Inception | [inceptionlabs.ai](https://inceptionlabs.ai) |
| `mistral` | [Mistral](https://ai-sdk.dev/providers/ai-sdk-providers/mistral) | [mistral.ai](https://mistral.ai) |
| `moonshotai` | Moonshot AI | [moonshot.ai](https://www.moonshot.ai) |
| `morph` | Morph | [morphllm.com](https://morphllm.com) |
| `novita` | Novita | [novita.ai](https://novita.ai/) |
| `openai` | [OpenAI](https://ai-sdk.dev/providers/ai-sdk-providers/openai) | [openai.com](https://openai.com) |
| `parasail` | Parasail | [parasail.com](https://www.parasail.io) |
| `perplexity` | [Perplexity](https://ai-sdk.dev/providers/ai-sdk-providers/perplexity) | [perplexity.ai](https://www.perplexity.ai) |
| `vercel` | [Vercel](https://ai-sdk.dev/providers/ai-sdk-providers/vercel) |  |
| `vertex` | [Vertex AI](https://ai-sdk.dev/providers/ai-sdk-providers/google-vertex) | [cloud.google.com/vertex-ai](https://cloud.google.com/vertex-ai) |
| `xai` | [xAI](https://ai-sdk.dev/providers/ai-sdk-providers/xai) | [x.ai](https://x.ai) |
| `zai` | Z.ai | [z.ai](https://z.ai/model-api) |

Provider availability may vary by model. Some models may only be available through specific providers or may have different capabilities depending on the provider used.
