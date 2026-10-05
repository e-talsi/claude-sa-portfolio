# Phase 1: Claude basics and the API (Weeks 1–2)

Goal: understand what Claude models are, how to call them, and what they cost, well enough to explain each choice to a customer with numbers.

> Facts in this folder (model IDs, prices, limits) were checked against the official docs on **2026-10-05**. They change often, so always re-check [platform.claude.com/docs](https://platform.claude.com/docs) before quoting them to a customer.

## Lessons

| # | Lesson | What you'll be able to do |
|---|---|---|
| 1 | [The model family](01-model-family.md) | Pick the right model for a use case and justify it |
| 2 | [The Messages API](02-messages-api.md) | Build requests with roles, system prompts, `max_tokens`, sampling, stop reasons and streaming |
| 3 | [Tokens, pricing and limits](03-tokens-pricing-limits.md) | Estimate cost, size a context window and plan around rate limits and usage tiers |

## Checklist

### Learn
- [ ] Read lesson 1, the model family
- [ ] Read lesson 2, the Messages API
- [ ] Read lesson 3, tokens, pricing and limits
- [ ] Academy course: **"Building with the Claude API"** ([anthropic.skilljar.com](https://anthropic.skilljar.com))

### Build
- [ ] A command-line chatbot with streaming and conversation history
- [ ] A cost calculator that compares the monthly cost of each model for a given workload

### Checkpoint
- [ ] Explain to a non-technical person why you'd choose Haiku over Opus for a given task, with numbers

## Official references

- [Models overview](https://platform.claude.com/docs/en/about-claude/models/overview)
- [Choosing a model](https://platform.claude.com/docs/en/about-claude/models/choosing-a-model)
- [Messages API reference](https://platform.claude.com/docs/en/api/messages)
- [Streaming](https://platform.claude.com/docs/en/build-with-claude/streaming)
- [Pricing](https://platform.claude.com/docs/en/about-claude/pricing)
- [Rate limits](https://platform.claude.com/docs/en/api/rate-limits)
- [Context windows](https://platform.claude.com/docs/en/build-with-claude/context-windows)
- [Model deprecations](https://platform.claude.com/docs/en/about-claude/model-deprecations)
