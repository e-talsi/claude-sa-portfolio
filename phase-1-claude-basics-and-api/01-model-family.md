# Lesson 1: The model family

## The idea in one sentence

Anthropic sells several Claude models with the same API. They trade **quality** against **speed** and **cost**, and choosing between them is one of the first decisions in every customer conversation.

## The current lineup (October 2026)

| | **Haiku 4.5** | **Sonnet 5.5** | **Opus 5.5** | **Fable 5.1** |
|---|---|---|---|---|
| Positioning | Fastest, near-frontier intelligence | Best balance of speed and intelligence | Long-running agentic coding and knowledge work. **The recommended default.** | Most capable. Demanding reasoning and long-horizon agents |
| API ID | `claude-haiku-4-5` | `claude-sonnet-5-5` | `claude-opus-5-5` | `claude-fable-5-1` |
| Relative latency | Fastest | Fast | Moderate | Slower |
| Input price (per 1M tokens) | $1 | $2 | $4 | $10 |
| Output price (per 1M tokens) | $5 | $10 | $20 | $50 |
| Context window | 200K tokens | 1M tokens | 1M tokens | 1M tokens |
| Max output per request | 64K tokens | 128K tokens | 128K tokens | 128K tokens |
| Thinking | Extended (manual budget) | Adaptive | Adaptive, always on | Adaptive, always on |
| Default effort | Not supported | `high` | `medium` | `high` |
| Knowledge cutoff | Feb 2025 | Jun 2026 | Jun 2026 | Jun 2026 |

All current models accept text and images, produce text, support tool use and work in many languages.

Older models such as Opus 5, Sonnet 5 and Opus 4.x are still available as **legacy** models. Every model has a retirement date: Anthropic only commits to keeping Haiku 4.5 until at least **October 15, 2026**, so check the [deprecations page](https://platform.claude.com/docs/en/about-claude/model-deprecations) before you design anything long-lived on it.

### Model names you'll hear

- **Haiku, Sonnet, Opus** are the three original tiers, from small to large.
- **Fable** is a tier above Opus, priced higher.
- **The version number** (4.5, 5.5, 5.1) marks the generation. Within a tier, a newer version is usually better, and sometimes cheaper. Opus 5.5 costs less than Opus 5 did.
- **Model IDs are pinned snapshots.** `claude-opus-5-5` will always mean the same model, so behaviour won't change under you. Upgrading is a deliberate code change.

## The three-way tradeoff

```
          Quality
            ▲
   Fable ●  │
   Opus  ●  │
  Sonnet ●  │
   Haiku ●  │
            └──────────────► Speed and low cost
```

Moving up gives you better reasoning, better handling of ambiguous or long tasks and fewer mistakes in multi-step agent work. It also costs more per token and responds more slowly.

The two cost and speed levers you can pull are:

1. **The model tier**: Haiku, Sonnet, Opus or Fable.
2. **The effort level** on a single model: `low`, `medium`, `high`, `xhigh` or `max`. Effort controls how much the model thinks before answering. Lower effort means fewer tokens, lower cost and faster answers.

The official guidance is to try the **most capable model at a lower effort** before building a mix of cheap and expensive models. A newer model at `low` effort often matches an older model at `high` effort, and running one model keeps your architecture simpler.

## How to pick a model

Four questions decide it:

1. **How hard is the task?** Classification and simple extraction are easy. Multi-step reasoning, coding and autonomous agents are hard.
2. **How fast must it answer?** A real-time chat or autocomplete needs a fast model. An overnight report doesn't.
3. **How much volume?** At millions of requests a day, a 4× price difference is the whole budget.
4. **What does a mistake cost?** Medical, legal and financial work can justify the most capable model. A draft email reply probably can't.

### A starting point by use case

| Use case | Start with | Why |
|---|---|---|
| Classifying support tickets, routing, tagging | **Haiku 4.5** | Simple task, high volume, needs to be fast |
| Pulling fields out of invoices or forms | **Haiku 4.5**, or Sonnet 5.5 for messy documents | Structured task. Upgrade only if your eval shows errors |
| Content moderation in real time | **Haiku 4.5** | Latency and volume matter most |
| Customer-facing chatbot | **Sonnet 5.5** | Good answers at chat speed |
| Questions over company documents (RAG) | **Sonnet 5.5** | Has to reason over retrieved text, still interactive |
| Coding assistant, code review | **Opus 5.5** | Coding gains a lot from intelligence |
| Agents that use tools over many steps | **Opus 5.5** | Mistakes compound over many steps |
| Hard research, complex analysis, long autonomous runs | **Fable 5.1** | Only when Opus 5.5 at higher effort still falls short in your evals |
| Sub-agents doing simple reading or summarising inside a larger agent | **Haiku 4.5** or Sonnet 5.5 | A cheap worker under a capable orchestrator |

### The method a solutions architect uses

1. **Start with a capable model** (Opus 5.5) to prove the task can be done at all.
2. **Build a small eval set** of 20–50 real examples with known good answers. Phase 3 covers this.
3. **Try cheaper options**: lower effort first, then Sonnet, then Haiku. Measure quality on the eval each time.
4. **Pick the cheapest option that meets the quality bar.**
5. **Compare cost per finished task, not per request.** A cheap model that needs retries, or more agent steps, can end up costing more.

## Worked example: Haiku or Opus?

> A utility wants to classify 50,000 customer emails a day into 8 categories. Each email is about 400 tokens. The answer is about 10 tokens.

Daily tokens: 50,000 × 400 = 20M input tokens and 50,000 × 10 = 0.5M output tokens.

| Model | Input cost | Output cost | Per day | Per month (30 days) |
|---|---|---|---|---|
| Haiku 4.5 | 20 × $1 = $20 | 0.5 × $5 = $2.50 | **$22.50** | **~$675** |
| Opus 5.5 | 20 × $4 = $80 | 0.5 × $20 = $10 | **$90** | **~$2,700** |

If Haiku scores 97% on your eval and Opus scores 98%, Opus costs four times as much for one more email in a hundred. Choose Haiku. Then send only the emails Haiku is unsure about to a bigger model, or to a person.

If the task were *"draft a legally careful reply to each complaint"*, the answer could flip, because a mistake costs much more.

These figures leave out thinking tokens, which are billed as output, and the fact that Haiku uses an older tokenizer. Lesson 3 covers both.

## How to say it to a customer

> "We'll pick the model by measuring, not guessing. We start with the most capable model to show the task works, then step down to cheaper and faster options until quality starts to drop. You pay for the smallest model that meets your bar."

## Check yourself

- [ ] Name the four current models from cheapest to most capable, with prices.
- [ ] Explain the difference between choosing a model and choosing an effort level.
- [ ] For a use case you know from work, say which model you'd start with and why.
- [ ] Explain why cost per finished task matters more than cost per token.
