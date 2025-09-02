# Observability

The AI Gateway logs observability metrics related to your requests, which you can use to monitor and debug.

You can view these [metrics](#metrics):

*   [The Observability tab in your Vercel dashboard](#observability-tab)
*   [The AI Gateway tab in your Vercel dashboard](#ai-gateway-tab)

## [Observability tab](#observability-tab)

You can access these metrics from the Observability tab of your Vercel dashboard by clicking AI Gateway on the left side of the Observability Overview page

![](/vc-ap-vercel-docs/_next/image?url=https%3A%2F%2Fassets.vercel.com%2Fimage%2Fupload%2Fv1750212341%2Fdocs-assets%2Fstatic%2Fdocs%2Fai-gateway%2Flrhvrkgfsmh7lkqbzbgi.png&w=3840&q=75)![](/vc-ap-vercel-docs/_next/image?url=https%3A%2F%2Fassets.vercel.com%2Fimage%2Fupload%2Fv1750212341%2Fdocs-assets%2Fstatic%2Fdocs%2Fai-gateway%2Fh3eofbxma8gjfjfmiaac.png&w=3840&q=75)

### [Team scope](#team-scope)

When you access the AI Gateway section of the Observability tab under the [team scope](/docs/dashboard-features#scope-selector), you can view the metrics for all requests made to the AI Gateway across all projects in your team. This is useful for monitoring the overall usage and performance of the AI Gateway.

![](/vc-ap-vercel-docs/_next/image?url=https%3A%2F%2Fassets.vercel.com%2Fimage%2Fupload%2Fv1750218123%2Fdocs-assets%2Fstatic%2Fdocs%2Fai-gateway%2Frrectrxazvow2qvkcusn.png&w=3840&q=75)![](/vc-ap-vercel-docs/_next/image?url=https%3A%2F%2Fassets.vercel.com%2Fimage%2Fupload%2Fv1750218123%2Fdocs-assets%2Fstatic%2Fdocs%2Fai-gateway%2Feefy948y9bt3byjccsdx.png&w=3840&q=75)

### [Project scope](#project-scope)

When you access the AI Gateway section of the Observability tab for a specific project, you can view metrics for all requests to the AI Gateway for that project.

![](/vc-ap-vercel-docs/_next/image?url=https%3A%2F%2Fassets.vercel.com%2Fimage%2Fupload%2Fv1750218426%2Fdocs-assets%2Fstatic%2Fdocs%2Fai-gateway%2Fjfvdu3ac3bgyg4cobrs7.png&w=3840&q=75)![](/vc-ap-vercel-docs/_next/image?url=https%3A%2F%2Fassets.vercel.com%2Fimage%2Fupload%2Fv1750218426%2Fdocs-assets%2Fstatic%2Fdocs%2Fai-gateway%2Fibp9anutc5p7ussyotir.png&w=3840&q=75)

## [AI Gateway tab](#ai-gateway-tab)

You can also access these metrics by clicking the AI Gateway tab of your Vercel dashboard under the team scope. You can see a recent overview of the requests made to the AI Gateway in the Activity section.

![](/vc-ap-vercel-docs/_next/image?url=https%3A%2F%2Fassets.vercel.com%2Fimage%2Fupload%2Fv1750478794%2Fdocs-assets%2Fstatic%2Fdocs%2Fai-gateway%2Fpgvk5xxep9zwsvl1ygm3.png&w=3840&q=75)![](/vc-ap-vercel-docs/_next/image?url=https%3A%2F%2Fassets.vercel.com%2Fimage%2Fupload%2Fv1750221829%2Fdocs-assets%2Fstatic%2Fdocs%2Fai-gateway%2Fsntkvarxttyl5trdmlhb.png&w=3840&q=75)

## [Metrics](#metrics)

### [Requests by Model](#requests-by-model)

The Requests by Model chart shows the number of requests made to each model over time. This can help you identify which models are being used most frequently and whether there are any spikes in usage.

### [Time to First Token (TTFT)](#time-to-first-token-ttft)

The Time to First Token chart shows the average time it takes for the AI Gateway to return the first token of a response. This can help you understand the latency of your requests and identify any performance issues.

### [Input/output Token Counts](#input/output-token-counts)

The Input/output Token Counts chart shows the number of input and output tokens for each request. This can help you understand the size of the requests being made and the responses being returned.

### [Spend](#spend)

The Spend chart shows the total amount spent on AI Gateway requests over time. This can help you monitor your spending and identify any unexpected costs.
