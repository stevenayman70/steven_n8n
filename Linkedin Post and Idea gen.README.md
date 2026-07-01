# Linkedin Post and Idea gen

A fully automated content pipeline that turns a single Telegram message into a published LinkedIn post — complete with a custom AI-generated banner image.

## What it does

1. **Trigger** — Send a topic as a Telegram message.
2. **Idea generation** — An AI agent (via OpenRouter) generates a post idea and logs it to Google Sheets with a `todo` / `success` status tracker.
3. **Research** — The idea is enriched with real, current context pulled from Tavily search.
4. **Post writing** — A second AI agent synthesizes the research into a polished, ready-to-publish LinkedIn post.
5. **Image generation** — A third AI call (Google Gemini via OpenRouter) generates a custom banner image matching the post's theme — no stock photos, no Canva.
6. **Publishing** — The image is uploaded to LinkedIn's media CDN and the full post (text + image) is published automatically via the LinkedIn UGC API.
7. **Status sync** — Google Sheets is updated to `success` once the post is live, keeping a full audit trail of every idea generated and published.

## Stack

| Layer | Tool |
|---|---|
| Orchestration | n8n (self-hosted) |
| AI models | OpenRouter (LLM + Gemini image generation) |
| Trigger | Telegram Bot API |
| Data log | Google Sheets |
| Publishing | LinkedIn UGC / Assets API |
| Research | Tavily Search API |

## Why this matters

Most "AI content" workflows still require a human to copy text between tools — idea, draft, image, scheduler, publish. This pipeline collapses that entire chain into one message: multiple AI agents, structured outputs, external APIs, and a persistent data log, chained together with zero manual steps after the trigger.

## Setup notes

- Replace the placeholder API key (`YOUR_TAVILY_API_KEY`) with your own Tavily key before importing.
- Requires n8n Google Sheets, Telegram, OpenRouter, and LinkedIn OAuth2 credentials configured in your instance.
- The Google Sheet needs three columns: `Idea Title`, `Description`, `Status`.
