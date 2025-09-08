---
trigger: model_decision
description: how to use jigsawstack API to enable tools, function calls, and other AI-assisted tasks
---
Overview
JigsawStack seamlessly integrates with Vercel AI SDK, enabling you to extend your AI applications with powerful tools for web scraping, computer vision, audio processing, sentiment analysis, and more. This integration allows you to leverage JigsawStack’s comprehensive API suite within Vercel’s AI framework.
​
Prerequisites
Before you begin, ensure you have:
A JigsawStack account with an active subscription
Your JigsawStack API key
Node.js (version 18.10.0 or higher)
A project with Vercel AI SDK configured
An AI provider (OpenAI, Anthropic, etc.) set up with Vercel AI SDK
​
Installation
Install the required packages:
npm

Copy
npm install jigsawstack ai @ai-sdk/openai
yarn

Copy
yarn add jigsawstack ai @ai-sdk/openai
​
Access JigsawStack APIs through Vercel’s AI SDK
You can pass JigsawStack into Vercel AI. This allows you to extend the functionality of Vercel AI by bringing any of your JigsawStack tools into your apps.
To use JigsawStack with Vercel AI, simply instantiate the JigsawStack SDK by specifying Vercel AI as your provider:

Copy
import { JigsawStackToolSet } from 'jigsawstack';

const toolset = new JigsawStackToolSet({
  apiKey,
});

const allTools = await toolset.getTools();
This configures JigsawStack to return tools that can be used into your AI functions. From there, simply pass your tools into generateText or streamText. You can use any LLM supported by Vercel.
​
Complete Example
Here is a simple example using all the tools available with JigsawStack

Copy
import { JigsawStackToolSet } from 'jigsawstack';
import { openai } from "@ai-sdk/openai";
import { generateText } from "ai";

const toolset = new JigsawStackToolSet({
  apiKey,
});

const allTools = await toolset.getTools();
const result = await generateText({
  model: openai("gpt-4o-mini"),
  tools: allTools,
  prompt: "I absolutely love this new AI technology! It's revolutionary and amazing!",
});

console.log(`text: "${text}"`);
console.log("toolResults");
console.log(JSON.stringify(toolResults));

​
Single and Specific Tools
To get single / specific tools available with JigsawStack

Copy
const toolset = new JigsawStackToolSet({
  apiKey,
});

const specificTools = await toolset.getTools({
  tools: ['web_search', 'vocr', 'object_detection']
});

const sentiment = await toolset.getTools({
  tools: ['sentiment']
});

​
Available Tools
JigsawStack provides access to various tool categories:
General Tools:
sentiment
summary
embedding
prediction
text_to_sql
Translate Tools:
translate_text
translate_image
Web Tools:
web_search
ai_scrape
html_to_any
search_suggestions
Vision Tools:
vocr
object_detection
Audio Tools:
speech_to_text
Validation Tools:
nsfw_detection
profanity_check
spell_check
spam_check