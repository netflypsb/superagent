---
trigger: model_decision
description: vercel ai gateway intro
---
# AI Gateway

AI Gateway is available on [all plans](/docs/plans) and your use is subject to [AI Product Terms](/legal/ai-product-terms).

The [AI Gateway](https://vercel.com/ai-gateway) provides a unified API to access [hundreds of models](https://vercel.com/ai-gateway/models) through a single endpoint. It gives you the ability to set budgets, monitor usage, load-balance requests, and manage fallbacks.

The design allows it to work seamlessly with [AI SDK 5](/docs/ai-gateway/getting-started), [OpenAI SDK](/docs/ai-gateway/openai-compat), or your [preferred framework](/docs/ai-gateway/framework-integrations).

## [Key features](#key-features)

*   Unified API: helps you switch between providers and models with minimal code changes
*   High Reliability: automatically retries requests to other providers if one fails
*   Embeddings Support: generate vector embeddings for search, retrieval, and other tasks
*   Spend Monitoring: monitor your spending across different providers

AI SDKPythonOpenAI HTTP

```
import { generateText } from 'ai';
 
const { text } = generateText({
  model: 'anthropic/claude-sonnet-4',
  prompt: 'What is the capital of France?',
});
```

```
import os
from openai import OpenAI
 
client = OpenAI(
  api_key=os.getenv('AI_GATEWAY_API_KEY'),
  base_url='https://ai-gateway.vercel.sh/v1'
)
 
response = client.chat.completions.create(
  model='xai/grok-4',
  messages=[
    {
      'role': 'user',
      'content': 'Why is the sky blue?'
    }
  ]
)
```

```
curl -X POST "https://ai-gateway.vercel.sh/v1/chat/completions" \
-H "Authorization: Bearer $AI_GATEWAY_API_KEY" \
-H "Content-Type: application/json" \
-d '{
  "model": "openai/gpt-5",
  "messages": [
    {
      "role": "user",
      "content": "Why is the sky blue?"
    }
  ],
  "stream": false
}'
```

## [More resources](#more-resources)

*   [Getting started with AI Gateway](/docs/ai-gateway/getting-started)
*   [Models and providers](/docs/ai-gateway/models-and-providers)
*   [Observability](/docs/ai-gateway/observability)
*   [OpenAI compatibility](/docs/ai-gateway/openai-compat)
*   [Authentication](/docs/ai-gateway/authentication)
*   [Bring your own key](/docs/ai-gateway/byok)
*   [Community frameworks](/docs/ai-gateway/community-frameworks)
*   [App attribution](/docs/ai-gateway/app-attribution)