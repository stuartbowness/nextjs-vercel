# Layercode — Add Voice AI to Your Next.js App

A Vercel-native reference implementation for integrating voice AI into any Next.js application using Layercode.

## What This App Showcases

- **Microphone selection** via the `MicrophoneSelect` component from `@layercode/react-sdk`
- **Speaking indicators** that react to `userSpeaking` / `agentSpeaking` states
- **Streaming transcripts** rendered live with partial user-turn updates
- **Server-side message history** stored in Vercel KV (with automatic in-memory fallback)
- **Static knowledge grounding** via `lib/knowledge.ts`
- **Centralized AI logic** via `lib/ai.ts`

## What You'll Learn

This repository teaches five core patterns for adding voice AI to Next.js:

1. **Voice session startup** — How the browser initiates a connection via `/api/authorize`
2. **Serverless event handling** — How voice events flow through `/api/agent` on Vercel
3. **AI logic boundaries** — Where AI calls happen and how to extend them
4. **Knowledge grounding** — How static content grounds AI responses
5. **Extension points** — Where to add persistence, analytics, moderation, and more

## Architecture

```
Browser (microphone)
    ↓
Layercode Web SDK (@layercode/react-sdk)
    ↓
Next.js API Routes (Vercel)
    ├── /api/authorize   → Start voice session
    ├── /api/agent       → Receive voice events, generate responses
    └── lib/ai.ts        → AI logic boundary
```

## Prerequisites

- Node.js 18+ and npm
- A Layercode account with an agent configured (API key + webhook secret)
- OpenAI API key for text generation
- (Optional) Vercel KV for persistent message history in production

## Quick Start

### Option 1: Deploy to Vercel

[![Deploy with Vercel](https://vercel.com/button)](https://vercel.com/new/clone?repository-url=https://github.com/layercodedev/nextjs-vercel)

After deploying, add your environment variables in Vercel project settings.

### Option 2: Run Locally

```bash
npm install
cp .env.example .env.local
# Fill in your API keys (see Environment Variables below)
npm run dev
```

Open http://localhost:3000 and click **Connect** to start a voice session.

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `NEXT_PUBLIC_LAYERCODE_AGENT_ID` | Yes | The agent to connect to from the browser (safe to expose) |
| `LAYERCODE_API_KEY` | Yes | Used by `/api/authorize` to request websocket credentials |
| `LAYERCODE_WEBHOOK_SECRET` | Yes | Used to verify webhook calls hitting `/api/agent` |
| `OPENAI_API_KEY` | Yes | Required for the default `gpt-4o-mini` text generation |
| `GOOGLE_GENERATIVE_AI_API_KEY` | No | Only needed if your agent/tools rely on it |
| `KV_REST_API_URL`, `KV_REST_API_TOKEN` | No | Provided by Vercel KV. If omitted, falls back to in-memory storage |

> **Tip:** Keep `.env.local` out of version control; Next.js loads it automatically in dev.

## How It Works

### Authorize Route (`/api/authorize`)

Proxies the browser request to Layercode's `authorize_session` endpoint using your `LAYERCODE_API_KEY`. This keeps your API key server-side while allowing the browser to establish a voice connection.

Customize this handler to add authentication checks or modify the request payload.

### Agent Route (`/api/agent`)

Receives all voice events from Layercode via webhook:
- `session.start` — Initialize conversation state
- `message` — Handle transcribed user speech, generate AI responses
- `session.end` — Clean up, persist transcripts

See the inline comments for extension points (analytics, moderation, summarization).

### Message History Storage

When `KV_*` environment variables are present, conversations are stored in Vercel KV with a 12-hour TTL. Without them, messages are stored in-memory (resets on server restart) — good enough for local testing.

## Deploying to Vercel

1. Add all environment variables in your Vercel project settings
2. **Disable Vercel Authentication** so Layercode webhooks can reach `/api/agent`:
   - Go to Settings → Deployment Protection
   - Turn off **Vercel Authentication** and save
3. Deploy via `vercel deploy` or Git push

![disable-vercel-auth](./disable-vercel-auth.png)

Monitor webhook deliveries in the Layercode dashboard to confirm everything is wired correctly.

## Project Structure

```
app/
  api/
    authorize/route.ts  — Session authorization (keeps API key server-side)
    agent/route.ts      — Voice event handler (webhook endpoint)
  page.tsx              — Voice UI with mic selection and indicators
lib/
  ai.ts                 — Centralized AI logic boundary
  knowledge.ts          — Static knowledge base
```

## The Demo: A Layercode Support Agent

Out of the box, this app runs a voice agent that answers questions about Layercode. Try asking:

- "What is Layercode?"
- "How do I get started?"
- "What platforms do you support?"
- "How does billing work?"

The agent's knowledge comes from `lib/knowledge.ts`, which contains FAQs, product info, and support details. This demonstrates how to ground AI responses in your own content.

### Customizing the Knowledge Base

To make this your own, edit `lib/knowledge.ts`:

```ts
export function getKnowledgeBase(): KnowledgeBase {
  return {
    companyName: "Your Company",
    faqs: [
      { question: "What do you do?", answer: "..." },
      // Add your own FAQs
    ],
    productInfo: "...",
    supportInfo: "...",
  };
}
```

The knowledge base is **static by design** — no vector DB or ingestion pipeline to configure. This lets you understand the entire system in one reading session.

For production, see the comments in `lib/knowledge.ts` for guidance on connecting a CMS, database, or vector search.

## Next Steps

Once you understand the patterns here, consider:

- **Telephony** — Add phone number support via Layercode's telephony features
- **RAG** — Replace static knowledge with vector search (Pinecone, Weaviate)
- **Vercel AI SDK** — Use `useChat` hooks and streaming for richer client UX
- **Production hardening** — Add rate limiting, error tracking, and monitoring

## Resources

- [Layercode Documentation](https://docs.layercode.com)
- [Vercel AI SDK](https://sdk.vercel.ai)
- [Next.js Documentation](https://nextjs.org/docs)
