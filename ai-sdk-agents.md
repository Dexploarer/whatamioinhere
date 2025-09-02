# AI Agents on Vercel

AI agents are systems that observe their environment, make decisions, and take actions to achieve goals. You can think of an agent as a loop that interprets inputs using a large language model (LLM), selects a tool, executes it, and then updates its context before the next decision. Vercel provides the infrastructure, tools, and SDKs to build, deploy, and scale these agents.

This guide will walk you through the process of building an agent using the AI SDK. You will learn how to [call an LLM](#calling-an-llm), [define tools](#using-tools), and [create an agent](#creating-an-agent) that can perform tasks based on user input.

## [Calling an LLM](#calling-an-llm)

The first step in building an agent is to call an LLM. The AI SDK provides an API for generating text using various models, the following example uses OpenAI.

You can use the [`generateText`](https://ai-sdk.dev/docs/reference/ai-sdk-core/generate-text#generatetext) function to call an LLM and get a response. The function takes a prompt and returns the generated text.

```
import { generateText } from 'ai';
import { openai } from '@ai-sdk/openai';
 
export async function getWeather() {
  const { text } = await generateText({
    model: openai('gpt-4.1'),
    prompt: 'What is the weather like today?',
  });
 
  console.log(text);
}
```

## [Using tools](#using-tools)

Tools are functions that the model can call to perform specific tasks. An agent can use tools to extend its capabilities and perform actions based on the context of the conversation. For example, a tool could be a function that queries an external API, performs calculations, or triggers an action.

If the model decides to use a tool, it will return a structured response indicating which tool to call and the necessary arguments, which are inferred from the context:

```
{
  "tool": "weather",
  "arguments": { "location": "San Francisco" }
}
```

The AI SDK provides a way to define tools using the [`tool`](https://ai-sdk.dev/docs/reference/ai-sdk-core/tools-and-tool-calling#tool) function. This function takes a description, parameters, and an execute function that performs the action.

```
import { generateText, tool } from 'ai';
import { openai } from '@ai-sdk/openai';
import { z } from 'zod';
 
export async function getWeather() {
  const { text } = await generateText({
    model: openai('gpt-4.1'),
    prompt: 'What is the weather in San Francisco?',
    tools: {
      weather: tool({
        description: 'Get the weather in a location',
        parameters: z.object({
          location: z.string().describe('The location to get the weather for'),
        }),
        execute: async ({ location }) => ({
          location,
          temperature: 72 + Math.floor(Math.random() * 21) - 10,
        }),
      }),
      activities: tool({
        description: 'Get the activities in a location',
        parameters: z.object({
          location: z
            .string()
            .describe('The location to get the activities for'),
        }),
        execute: async ({ location }) => ({
          location,
          activities: ['hiking', 'swimming', 'sightseeing'],
        }),
      }),
    },
  });
  console.log(text);
}
```

In this example, the AI SDK:

*   Extracts the tool call from the model output.
*   Validates the arguments against the tool schema (defined in the `parameters`).
*   Executes the function, and stores both the call and its result in [`toolCalls`](https://ai-sdk.dev/docs/reference/ai-sdk-core/generate-text#tool-calls) and [`toolResults`](https://ai-sdk.dev/docs/reference/ai-sdk-core/generate-text#tool-results), which are also added to message history.

## [Creating an agent](#creating-an-agent)

At the moment, your agent only consists of a single call to the LLM and then stops. Because the AI SDK [defaults](https://ai-sdk.dev/docs/ai-sdk-core/tools-and-tool-calling#multi-step-calls-using-maxsteps) the number of steps used to 1, the model will make a decision based on the users input to decide:

*   If it should directly return a text response.
*   If it should call a tool, and if so, which one.

Either way, once the model has completed it's generation, that step is complete. In order to continue generating, whether that be to use additional tools, or solve the users query with existing tool results, you need to trigger an additional request. This can be done by configuring a loop using the [`maxSteps`](https://ai-sdk.dev/docs/ai-sdk-core/tools-and-tool-calling#multi-step-calls-using-maxsteps) parameter to specify the maximum number of steps the agent can take before stopping.

The AI SDK automatically handles the orchestration by appending each response to the conversation history, executing [tool calls](https://ai-sdk.dev/docs/ai-sdk-core/tools-and-tool-calling), and triggering additional generations until either reaching the maximum number of steps or receiving a text response.

A potential flow might then look like this:

1.  You (the developer) send a prompt to the LLM.
2.  The LLM decides to either generate text or call a tool (returning the tool name and arguments).
3.  If a tool is called, the AI SDK executes the tool and receives the result.
4.  The AI SDK appends the tool call and result to the conversation history.
5.  The AI SDK automatically triggers a new generation based on the updated conversation history.
6.  Steps 2-5 repeat until either reaching the maximum number of steps (defined by `maxSteps`) or receiving a text response without a tool call.
7.  The final text response is returned.

```
import { generateText, tool } from 'ai';
import { openai } from '@ai-sdk/openai';
import { z } from 'zod';
 
export async function getWeather() {
  const { text } = await generateText({
    model: openai('gpt-4.1'),
    prompt: 'What is the weather in San Francisco?',
    maxSteps: 2,
    tools: {
      weather: tool({
        description: 'Get the weather in a location',
        parameters: z.object({
          location: z.string().describe('The location to get the weather for'),
        }),
        execute: async ({ location }) => ({
          location,
          temperature: 72 + Math.floor(Math.random() * 21) - 10,
        }),
      }),
      activities: tool({
        description: 'Get the activities in a location',
        parameters: z.object({
          location: z
            .string()
            .describe('The location to get the activities for'),
        }),
        execute: async ({ location }) => ({
          location,
          activities: ['hiking', 'swimming', 'sightseeing'],
        }),
      }),
    },
  });
  console.log(text);
}
```

## [Deploying AI agents on Vercel](#deploying-ai-agents-on-vercel)

To deploy your agent on Vercel, create an API route with the following code. The agent will loop through the steps until it reaches a stopping condition, which is when it receives a text response.

Once deployed, your API route will be using [fluid compute](/docs/fluid-compute), which is ideal for AI agents because it:

*   Minimizes cold starts for faster response times
*   Allows tasks to run in the background after responding to users
*   Enables considerably longer function durations, perfect for agents that run for extended periods
*   Supports concurrent workloads without the typical timeouts of traditional serverless environments
*   Automatically scales with increased usage

Depending on your plan, the [default maximum duration](/docs/functions/limitations#max-duration) will be between 60 and 800 seconds. You can also set a custom maximum duration for your agent by using the [`maxDuration`](/docs/functions/configuring-functions/duration#maximum-duration-for-different-runtimes) variable in your API route. This will allow you to specify how long the agent can run before timing out.

```
import { generateText } from 'ai';
import { openai } from '@ai-sdk/openai';
import { z } from 'zod';
import { tool } from 'ai';
 
export const maxDuration = 30;
 
export const POST = async (request: Request) => {
  const { prompt }: { prompt?: string } = await request.json();
 
  if (!prompt) {
    return new Response('Prompt is required', { status: 400 });
  }
 
  const result = await generateText({
    model: openai('gpt-4.1'),
    prompt,
    maxSteps: 2,
    tools: {
      weather: tool({
        description: 'Get the weather in a location',
        parameters: z.object({
          location: z.string().describe('The location to get the weather for'),
        }),
        execute: async ({ location }) => ({
          location,
          temperature: 72 + Math.floor(Math.random() * 21) - 10,
        }),
      }),
      activities: tool({
        description: 'Get the activities in a location',
        parameters: z.object({
          location: z
            .string()
            .describe('The location to get the activities for'),
        }),
        execute: async ({ location }) => ({
          location,
          activities: ['hiking', 'swimming', 'sightseeing'],
        }),
      }),
    },
  });
 
  return Response.json({
    steps: result.steps,
    finalAnswer: result.text,
  });
};
```

And [deploy](/docs/cli/deploy) it to Vercel using the [Vercel CLI](/docs/cli):

```
vercel deploy
```

## [Testing your agent](#testing-your-agent)

You can test your deployed agent using `curl`:

```
curl -X POST https://<your-project-url.vercel.app>/api/agent \
  -H "Content-Type: application/json" \
  -d '{"prompt":"What is the weather in Tokyo?"}'
```

The response will include both the steps of the agent and the final answer:

```
{
  "steps": [
    {
      "stepType": "initial",
      "toolCalls": [
        {
          "type": "tool-call",
          "toolCallId": "call_7QGq6MRbq0L3lLwpN23OMDKm",
          "toolName": "weather",
          "args": { "location": "Tokyo" }
        }
      ],
      "toolResults": [
        {
          "type": "tool-result",
          "toolCallId": "call_7QGq6MRbq0L3lLwpN23OMDKm",
          "toolName": "weather",
          "result": {
            "location": "Tokyo",
            "temperature": 77,
            "condition": "cloudy"
          }
        }
      ]
      // ...
    },
    {
      "stepType": "tool-result",
      "text": "The weather in Tokyo is currently cloudy with a temperature of 77°F."
      // ...
    }
  ],
  "finalAnswer": "The weather in Tokyo is currently cloudy with a temperature of 77°F."
}
```

## [Observing your agent](#observing-your-agent)

Once deployed, you can observe your agent's behavior using the [Observability](https://vercel.com/d?to=%2F%5Bteam%5D%2F%7E%2Fobservability&title=Try+Observability) and [Logs](https://vercel.com/d?to=%2F%5Bteam%5D%2F%5Bproject%5D%2Flogs&title=Logs+tab) tabs in the Vercel dashboard. The Observability tab provides insights into the number of requests, response times, error rates, and more and is ideal for monitoring the performance of your agent.

If you have added logging to your agent, you can also view the logs in the Logs tab. You can use this information to debug your agent and understand its behavior.

![The Observability tab in the Vercel dashboard.](/vc-ap-vercel-docs/_next/image?url=https%3A%2F%2Fassets.vercel.com%2Fimage%2Fupload%2Fv1747401504%2Ffront%2Fdocs%2Fagents%2Fo11y-tab-light.png&w=3840&q=75)![The Observability tab in the Vercel dashboard.](/vc-ap-vercel-docs/_next/image?url=https%3A%2F%2Fassets.vercel.com%2Fimage%2Fupload%2Fv1747401504%2Ffront%2Fdocs%2Fagents%2Fo11y-tab-dark.png&w=3840&q=75)

The Observability tab in the Vercel dashboard.

## [More resources](#more-resources)

To learn more about building AI agents, see the following resources:

*   [AI SDK documentation](https://ai-sdk.dev)
*   [AI SDK examples](https://github.com/vercel/ai/tree/main/examples)
*   [AI SDK cookbook](https://ai-sdk.dev/cookbook)
