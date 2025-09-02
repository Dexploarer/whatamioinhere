# Authentication

To use the AI Gateway, you need to authenticate your requests. There are two authentication methods available:

1.  API Key Authentication: Create and manage API keys through the Vercel Dashboard
2.  OIDC Token Authentication: Use Vercel's automatically generated OIDC tokens

## [API key](#api-key)

API keys provide a secure way to authenticate your requests to the AI Gateway. You can create and manage multiple API keys through the Vercel Dashboard.

### [Creating an API Key](#creating-an-api-key)

1.  #### [Navigate to the AI Gateway tab](#navigate-to-the-ai-gateway-tab)
    
    From the [Vercel dashboard](https://vercel.com/dashboard), click the AI Gateway tab to access the AI Gateway settings.
    
2.  #### [Access API key management](#access-api-key-management)
    
    Click API keys on the left sidebar to view and manage your API keys.
    
3.  #### [Create a new API key](#create-a-new-api-key)
    
    Click Create key and proceed with Create key from the dialog to generate a new API key.
    
4.  #### [Save your API key](#save-your-api-key)
    
    Once you have the API key, save it to `.env.local` at the root of your project (or in your preferred environment file):
    
    ```
    AI_GATEWAY_API_KEY=your_api_key_here
    ```
    

### [Using the API key](#using-the-api-key)

When you specify a model id as a plain string, the AI SDK will automatically use the Vercel AI Gateway provider to route the request. The AI Gateway provider looks for the API key in the `AI_GATEWAY_API_KEY` environment variable by default.

```
import { generateText } from 'ai';
 
export async function GET() {
  const result = await generateText({
    model: 'xai/grok-3',
    prompt: 'Why is the sky blue?',
  });
  return Response.json(result);
}
```

## [OIDC token](#oidc-token)

The [Vercel OIDC token](/docs/oidc) is a way to authenticate your requests to the AI Gateway without needing to manage an API key. Vercel automatically generates the OIDC token that it associates with your Vercel project.

Vercel OIDC tokens are only valid for 12 hours, so you will need to refresh them periodically during local development. You can do this by running `vercel env pull` again.

### [Setting up OIDC authentication](#setting-up-oidc-authentication)

1.  #### [Link to a Vercel project](#link-to-a-vercel-project)
    
    Before you can use the OIDC token during local development, ensure that you link your application to a Vercel project:
    
    ```
    vercel link
    ```
    
2.  #### [Pull environment variables](#pull-environment-variables)
    
    Pull the environment variables from Vercel to get the OIDC token:
    
    ```
    vercel env pull
    ```
    
3.  #### [Use OIDC authentication in your code](#use-oidc-authentication-in-your-code)
    
    With OIDC authentication, you can directly use the gateway provider without needing to obtain an API key or set it in an environment variable:
    
    ```
    import { generateText } from 'ai';
     
    export async function GET() {
      const result = await generateText({
        model: 'xai/grok-3',
        prompt: 'Why is the sky blue?',
      });
      return Response.json(result);
    }
    ```
