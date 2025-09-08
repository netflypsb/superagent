# Superagent

Superagent is a powerful, subscription-based AI assistant designed for seamless research, document creation, and knowledge management. Built on a sophisticated multi-agent architecture with the latest Vercel AI SDK, Superagent acts as your personal partner for organizing and creating complex information.

-----

## 🤖 Project Overview

Superagent is an intelligent platform featuring:

  * **Secure User Authentication:** Robust user login and session management to keep your data private.
  * **Stripe Subscription:** A simple and affordable **$15/month** subscription model managed via Stripe for access to all features.
  * **Multi-Agent System:** A powerful backend where specialized AI agents collaborate on complex tasks. You interact with a primary `chatAgent`, which delegates tasks like research, document creation, saving, and retrieval to dedicated orchestrator agents.
  * **Versatile Tool Use:** Agents are equipped with tools like Google Search and Google Scholar (via SERP API) to gather real-time, accurate information from the web.
  * **Advanced AI Models:** Powered by OpenRouter, Superagent primarily uses the state-of-the-art `z-ai/glm-4.5v` model, with a reliable fallback to `google/gemini-2.0-flash`, ensuring high availability and performance.
  * **Modern, Reactive UI:** A beautiful and intuitive user interface built with Next.js, shadcn/ui, and Vercel's AI components.

-----

## 🛠️ Tech Stack

### Frontend

  * **Framework:** [Next.js](https://nextjs.org/) (React)
  * **UI Components:** [shadcn/ui](https://ui.shadcn.com/)
  * **AI UI:** [Vercel AI SDK UI](https://www.google.com/search?q=https://sdk.vercel.ai/docs/ui-sdk) & [Vercel AI Elements](https://www.google.com/search?q=https://sdk.vercel.ai/docs/elements) for building the chat interface.
  * **Styling:** [Tailwind CSS](https://tailwindcss.com/)

### Backend

  * **Framework:** [Next.js](https://nextjs.org/) (API Routes)
  * **AI SDK:** [Vercel AI SDK](https://sdk.vercel.ai/) for core AI logic, tool use, and streaming.
  * **AI Gateway:** [Vercel AI Gateway](https://vercel.com/ai-gateway) to manage model access, fallbacks, and API keys.
  * **Search Tools:** [SERP API](https://serpapi.com/) for Google Search and Google Scholar integration.
  * **Payment:** [Stripe SDK](https://stripe.com/docs/api) for managing monthly subscriptions.

### Database

  * **Database:** [Neon](https://neon.tech/) (Serverless Postgres) for relational data.
  * **Blob Storage:** [Vercel Blob](https://vercel.com/storage/blob) for storing exported files (e.g., PDFs, images).

-----

## 🗄️ Database Schema & Strategy

To support versioning, rich content types, and efficient retrieval, Superagent uses a relational model. This structure separates a document's core identity from its versioned content, making the system scalable and organized.

### Schema ERD

```
+-------------+      +---------------------+      +------------------------+
|    users    |      |      documents      |      |   document_versions    |
+-------------+      +---------------------+      +------------------------+
| id (PK)     |----<| user_id (FK)        |----<| document_id (FK)     |
| email       |      | id (PK)             |      | id (PK)                |
| ...         |      | type                |      | version_number         |
+-------------+      | title               |      | content (jsonb)        |
                     | created_at          |      | description (text)     |
                     | latest_version      |      | created_at             |
                     +---------------------+      +------------------------+
                                                        |
                                                        |
                                                        V
                                              +------------------------+
                                              | document_version_tags  |
                                              +------------------------+
                                              | version_id (FK)        |
                                              | tag_id (FK)            |
                                              +------------------------+
                                                        ^
                                                        |
                                              +------------------------+
                                              |          tags          |
                                              +------------------------+
                                              | id (PK)                |
                                              | name (unique)          |
                                              +------------------------+
```

### Table Definitions

#### 1\. `documents`

This table acts as the parent container for each unique document, holding metadata that doesn't change between versions.

| Column | Type | Description |
| :--- | :--- | :--- |
| `id` | `uuid` | Primary key. A unique identifier for the document entity. |
| `user_id` | `uuid` | Foreign key to `users.id`. |
| `type` | `varchar` | The document type. Can be `text`, `sheet`, `code`, `slides`, `website`, or `design`. |
| `title` | `varchar` | The main title of the document, given by the user or an agent. |
| `created_at` | `timestamptz` | Timestamp when the document was first created. |
| `latest_version`| `integer` | The number of the current, most recent version. |

#### 2\. `document_versions`

This table stores the specific content and metadata for each version of a document. Every time a user saves a document, a new row is created here.

| Column | Type | Description |
| :--- | :--- | :--- |
| `id` | `uuid` | Primary key for the version. |
| `document_id` | `uuid` | Foreign key to `documents.id`. |
| `version_number`| `integer` | The sequential version number (e.g., 1, 2, 3). |
| `content` | `jsonb` | The actual content of the document, stored in a flexible JSON format. |
| `description` | `text` | An AI-generated summary of this version's content for search and retrieval. |
| `created_at` | `timestamptz` | The exact date and time this specific version was created. |

#### 3\. `tags` & `document_version_tags` (Many-to-Many)

This structure allows for flexible tagging and efficient querying.

  * **`tags` table:** Holds unique tag names (`id`, `name`).
  * **`document_version_tags` table:** A join table linking `document_versions` to `tags` (`version_id`, `tag_id`).

### Content Storage Strategy

Using the `jsonb` data type for the `content` field allows us to store different structures for each document type efficiently.

  * **`text`**: Stores rich text content in a structured JSON format (e.g., from an editor like TipTap).
  * **`sheet`**: Stores spreadsheet data as an array of JSON objects, where each object is a row.
  * **`code`**: Stores the code, language, and other metadata needed for a code editor.
  * **`slides`**: Stores a presentation as an array of slide objects, each defining its layout and content.
  * **`website`**: Stores HTML, CSS, and JavaScript as separate keys within the JSON object.
  * **`design`**: Stores the serialized state of a design canvas (e.g., from Fabric.js), including all objects, layers, and properties.

-----

## 🚀 Development Plan

This project is broken down into four manageable phases.

### Phase 1: Project Setup & Core Infrastructure

  * **Goal:** Establish the foundation of the application.
  * **Tasks:**
    1.  Initialize a new Next.js project with TypeScript and Tailwind CSS.
    2.  Set up the Neon database and define the schema using a migration tool (e.g., Drizzle ORM).
    3.  Implement secure user authentication (e.g., NextAuth.js, Clerk).
    4.  Integrate the Stripe SDK, create the `$15/month` product, and build the subscription management UI.
    5.  Set up Vercel AI Gateway and configure environment variables for OpenRouter.

### Phase 2: The `chatAgent` & Basic Chat UI

  * **Goal:** Build a working chat interface that connects to a basic AI agent.
  * **Tasks:**
    1.  Create the main `chatAgent` using the Vercel AI SDK. This agent will handle user interaction and delegate to other agents.
    2.  Build the chat UI using Vercel AI Elements and `shadcn/ui` components.
    3.  Implement streaming responses from the `chatAgent` to the UI for a real-time feel.

### Phase 3: Orchestrator Agents & Workflows

  * **Goal:** Implement the specialized agents and their tools to enable Superagent's core functionality.
  * **Tasks:**
    1.  **Build the `researchOrchestrator`:**
          * Create `googleSearchAgent` and `googleScholarAgent` tools using the SERP API.
          * Create a `webScraperAgent` tool to extract information from URLs.
    2.  **Build the `documentCreatorOrchestrator`:**
          * Create `textSummarizerAgent`, `reportGeneratorAgent`, and other content-generation tools.
    3.  **Build the `documentSavingOrchestrator` & `metadataAgent`:**
          * Implement the workflow to save new document versions to the database.
          * Create the specialized `metadataAgent` tool, which analyzes content to generate a `description` and `tags`.
          * Integrate the `metadataAgent` into the saving workflow to automatically index new content.
    4.  **Build the `documentRetrievalOrchestrator`:**
          * Create a `documentSearchAgent` tool that queries the database based on titles, descriptions, and tags.

### Phase 4: Integration & Deployment

  * **Goal:** Connect all the pieces, conduct thorough testing, and deploy the application.
  * **Tasks:**
    1.  Integrate all orchestrator agents as callable tools for the main `chatAgent`.
    2.  Perform end-to-end testing of all user flows: signup, subscription, chat, research, document creation, saving, and retrieval.
    3.  Deploy the application to Vercel, ensuring all environment variables and services are correctly configured for production.

---

Suggested project structure:

superagent/ (root directory)
├── app/ (Next.js app directory)
│   ├── api/
│   │   ├── chat/
│   │   │   └── route.ts
│   │   ├── auth/
│   │   │   └── route.ts
│   │   ├── agents/
│   │   │   ├── research/
│   │   │   │   └── route.ts
│   │   │   ├── document-creator/
│   │   │   │   └── route.ts
│   │   │   └── tools/
│   │   │       ├── google-search/
│   │   │       │   └── route.ts
│   │   │       └── google-scholar/
│   │   │           └── route.ts
│   │   ├── documents/
│   │   │   └── route.ts
│   │   └── subscriptions/
│   │       └── route.ts
│   ├── lib/
│   │   ├── agents/
│   │   │   ├── chat-agent.ts
│   │   │   ├── research-orchestrator.ts
│   │   │   ├── document-creator-orchestrator.ts
│   │   │   ├── document-saving-orchestrator.ts
│   │   │   ├── document-retrieval-orchestrator.ts
│   │   │   └── tools/
│   │   │       ├── google-search-agent.ts
│   │   │       ├── google-scholar-agent.ts
│   │   │       ├── web-scraper-agent.ts
│   │   │       ├── text-summarizer-agent.ts
│   │   │       └── metadata-agent.ts
│   │   ├── tools/
│   │   │   ├── serp-api.ts
│   │   │   ├── web-scraper.ts
│   │   │   ├── document-processor.ts
│   │   │   └── metadata-extractor.ts
│   │   ├── db.ts (database connection)
│   │   ├── auth.ts (authentication logic)
│   │   └── stripe.ts (Stripe integration)
│   ├── components/
│   │   ├── ui/
│   │   │   └── (shadcn/ui components)
│   │   ├── chat/
│   │   │   ├── ChatInterface.tsx
│   │   │   ├── MessageBubble.tsx
│   │   │   ├── ChatInput.tsx
│   │   │   └── (other chat components)
│   │   ├── documents/
│   │   │   ├── DocumentList.tsx
│   │   │   ├── DocumentEditor.tsx
│   │   │   └── (other document components)
│   │   ├── sidebar/
│   │   │   ├── Sidebar.tsx
│   │   │   ├── Navigation.tsx
│   │   │   └── (other sidebar components)
│   │   └── (other shared components)
│   ├── hooks/
│   │   └── (custom React hooks)
│   ├── styles/
│   │   └── globals.css
│   ├── utils/
│   │   └── (utility functions)
│   ├── layout.tsx
│   └── page.tsx
├── database/
│   ├── schema.ts (database schema)
│   ├── migrations/ (migration files)
│   └── seed.ts (seed data)
├── documents/
│   ├── superbot/ (sample/reference project)
│   ├── tools_documentation/
│   │   ├── Google Scholarly Articles API.txt
│   │   ├── OpenRouter.internet.use.txt
│   │   ├── OpenRouter.llm.txt
│   │   ├── Serp google search API.txt
│   │   ├── exa.txt
│   │   └── stripe.agentic.txt
│   └── vercel_documentation/
│       ├── ai_sdk_AGENTS.md
│       ├── ai_sdk_core_embeddings.md
│       ├── ai_sdk_core_generate_stream_text.md
│       ├── ai_sdk_core_image.md
│       ├── ai_sdk_core_intro.md
│       ├── ai_sdk_core_middleware.md
│       ├── ai_sdk_core_provider.md
│       ├── ai_sdk_core_settings.md
│       ├── ai_sdk_core_structured_data.md
│       ├── ai_sdk_core_tool_calling.md
│       ├── ai_sdk_core_transcription.md
│       ├── ai_sdk_prompt.md
│       ├── ai_sdk_speech.md
│       ├── ai_sdk_streaming.md
│       ├── ai_sdk_tool_browserbase.md
│       ├── ai_sdk_tool_browserless.md
│       ├── ai_sdk_tool_browsgpt.md
│       ├── ai_sdk_tool_codeexecution.md
│       ├── ai_sdk_tool_jigsawstack.md
│       ├── ai_sdk_tool_stripe.md
│       ├── ai_sdk_tool_tavily.md
│       ├── ai_sdk_tools_basic.md
│       ├── error_handling.md
│       ├── gateway_authentication.md
│       ├── gateway_image_gen.md
│       ├── gateway_intro.md
│       ├── gateway_providers.md
│       ├── gateway_start.md
│       ├── nextjs_app_router.md
│       ├── nextjs_pages_router.md
│       ├── prompt_engineering.md
│       ├── ui_sdk_basics.md
│       ├── ui_sdk_chatbot.md
│       ├── ui_sdk_chatbot_tooluse.md
│       ├── ui_sdk_completion.md
│       ├── ui_sdk_customstream.md
│       ├── ui_sdk_errorHandling.md
│       ├── ui_sdk_genui.md
│       ├── ui_sdk_message_metadata.md
│       ├── ui_sdk_message_persistence.md
│       ├── ui_sdk_object_generation.md
│       ├── ui_sdk_stream_protocol.md
│       ├── ui_sdk_transport.md
│       └── ui_sdk_uistreams.md
├── public/ (static assets)
│   └── (images, icons, etc.)
├── .env (environment variables)
├── .env.local (local environment variables)
├── .env.production (production environment variables)
├── .gitignore
├── README.md
├── package.json
├── pnpm-lock.yaml
├── tsconfig.json
├── next.config.mjs
├── tailwind.config.ts
├── postcss.config.mjs
└── components.json (shadcn/ui config)