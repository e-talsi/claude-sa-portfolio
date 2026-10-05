# Lesson 3: Tokens, context windows, pricing and limits

## The idea in one sentence

You pay for **tokens**, you can fit only so many tokens into one request (the **context window**), and your organisation can only send so many per minute (the **rate limits**, set by your **usage tier**).

---

## 1. Tokens

A **token** is the unit a model reads and writes: a word, part of a word, a piece of punctuation or a space.

Rough rules for English on the current tokenizer:

- **1M tokens ≈ 555,000 words ≈ 2.5M characters**
- So 1 token ≈ 0.55 words, or about 2.5 characters
- Code, JSON, numbers and languages other than English usually take **more** tokens per word

### Different models count differently

Anthropic introduced a new tokenizer with Opus 4.7. Every current model **except Haiku 4.5** uses it, and it produces **about 1× to 1.35× as many tokens** for the same text as the older one.

So the same prompt can be, say, 1,000 tokens on Haiku 4.5 and 1,250 on Opus 5.5. **Measure; don't assume.**

### Counting tokens exactly

Don't use OpenAI's `tiktoken`, because it counts differently. Use the API's free token-counting endpoint:

```python
import anthropic

client = anthropic.Anthropic()

count = client.messages.count_tokens(
    model="claude-opus-5-5",
    system="You are a helpful assistant.",
    messages=[{"role": "user", "content": open("contract.txt").read()}],
)
print(count.input_tokens)
```

Every response also tells you what was actually used:

```python
response.usage.input_tokens                 # uncached input
response.usage.output_tokens                # output, including thinking
response.usage.cache_read_input_tokens      # input read from the cache
response.usage.cache_creation_input_tokens  # input written to the cache
```

### Thinking tokens are output tokens

Current models **think** before answering, and Opus 5.5 and Fable 5.1 always do. Thinking tokens are **billed as output tokens**, even when you don't see the thinking text.

That's why a "10-token answer" can cost much more than 10 output tokens. The **`effort`** setting is your main control:

```python
output_config={"effort": "low"}  # less thinking → fewer output tokens → cheaper and faster
```

---

## 2. Context windows

The **context window** is the maximum number of tokens one request can hold: system prompt, tools, the whole conversation history, documents, **plus** the output.

| Model | Context window | Max output |
|---|---|---|
| Fable 5.1 | 1M | 128K |
| Opus 5.5 | 1M | 128K |
| Sonnet 5.5 | 1M | 128K |
| Haiku 4.5 | 200K | 64K |

1M tokens is roughly **several thick books**, or a mid-sized codebase.

### What fills the window

```
┌────────────────────────── context window ──────────────────────────┐
│ tools │ system prompt │ msg 1 │ msg 2 │ … │ msg N │ ← new output →  │
└────────────────────────────────────────────────────────────────────┘
```

In a long chat or an agent loop, the **history keeps growing** with every turn. When it's full you get `stop_reason: "model_context_window_exceeded"` or an error.

### A big window doesn't mean "send everything"

- **Cost:** you pay for every input token on **every** call. A 500K-token document sent 100 times is 50M input tokens.
- **Latency:** more input means a slower first token.
- **Quality:** models do best with focused, relevant context.

So the choice is:

| Approach | When |
|---|---|
| Put the whole document in context, with prompt caching | A few documents, reused often |
| Retrieve only the relevant chunks (RAG) | Large or growing knowledge bases. Phase 5 covers this |
| Summarise or compact old history | Long conversations and agent runs |

Check a model's limits from code with `client.models.retrieve("claude-opus-5-5")`. It returns `max_input_tokens` and `max_tokens`.

---

## 3. Pricing

### Base prices per million tokens (MTok)

| Model | Input | Output | Cache read |
|---|---|---|---|
| Fable 5.1 | $10 | $50 | $0.25 (2.5% of input) |
| Opus 5.5 | $4 | $20 | $0.20 (5% of input) |
| Sonnet 5.5 | $2 | $10 | $0.20 (10% of input) |
| Haiku 4.5 | $1 | $5 | $0.10 (10% of input) |

**Output costs 5× as much as input** on every model. Long answers and heavy thinking drive the bill.

### Discounts and extras

| Feature | Effect on price | Use it when |
|---|---|---|
| **Prompt caching**: reading from cache | 2.5–10% of the input price | The same long prefix (system prompt, documents, tools) is reused across calls |
| **Prompt caching**: writing to cache | More than the base input price | Paid once, then every reuse is cheap |
| **Batch API** | **50% off** everything | The work can wait up to 24 hours: overnight reports, bulk classification, backfills |
| **Fast mode** (Opus only, research preview) | About 2× the price for up to 2.5× faster output | Latency is critical and the budget allows |
| Very long context | May carry a premium | Check the pricing page for the current rules |

Cache-write multipliers, long-context rules and server-tool fees change. Read the [pricing page](https://platform.claude.com/docs/en/about-claude/pricing) for exact figures.

**Amazon Bedrock and Google Vertex AI set their own prices.** Microsoft Foundry charges the same as Anthropic's API.

### The cost formula

```
cost per request = (input_tokens  × input_price
                  + output_tokens × output_price
                  + cache_read    × cache_read_price
                  + cache_write   × cache_write_price) / 1,000,000
```

### Worked example: a customer-support assistant

> 10,000 conversations a day, each ~2,000 input tokens and ~500 output tokens, including thinking.

| Model | Per request | Per day | Per month (30 days) |
|---|---|---|---|
| Haiku 4.5 | $0.0045 | $45 | **$1,350** |
| Sonnet 5.5 | $0.009 | $90 | **$2,700** |
| Opus 5.5 | $0.018 | $180 | **$5,400** |
| Fable 5.1 | $0.045 | $450 | **$13,500** |

Working for Sonnet 5.5: (2,000 × $2 + 500 × $10) ÷ 1,000,000 = $0.004 + $0.005 = **$0.009**.

Now add **prompt caching**. Say 1,500 of the 2,000 input tokens are a fixed system prompt and policy text, read from the cache:

```
Sonnet 5.5 with caching:
  uncached input: 500   × $2.00 / 1M = $0.0010
  cache read:     1,500 × $0.20 / 1M = $0.0003
  output:         500   × $10   / 1M = $0.0050
  total                              = $0.0063 per request  (−30%)
```

Output now dominates, so the next savings come from **shorter answers** and **lower effort**.

### The cost-reduction ladder

Try these in order. The first ones are free and don't affect quality.

1. **Prompt caching** for anything repeated
2. **Batch API** for anything that isn't urgent
3. **Trim the input**: no unnecessary history or documents
4. **Shorter output**: ask for concise answers or structured fields
5. **Lower effort** on simple routes
6. **A smaller model** for routes where your eval shows quality holds

---

## 4. Rate limits

Rate limits cap how much your **organisation** can send per minute, **per model**. They're measured three ways:

| Limit | Means |
|---|---|
| **RPM** | Requests per minute |
| **ITPM** | Input tokens per minute |
| **OTPM** | Output tokens per minute |

How they behave:

- **Token bucket.** Capacity refills continuously rather than resetting at the top of each minute. A short burst can still trip the limit: 60 RPM may be enforced as 1 request per second.
- **Cached input doesn't count toward ITPM** on current models. Only uncached input and cache writes count. With an 80% cache hit rate, a 2M ITPM limit lets you process about **10M input tokens a minute**. Caching therefore saves money **and** raises your effective throughput.
- **`max_tokens` doesn't count toward OTPM.** Only tokens actually generated do, so a high `max_tokens` doesn't hurt.
- **Each model has its own limits.** Opus 5.5 and Sonnet 5.5 traffic don't compete with each other.
- **Acceleration limits:** a sudden spike in traffic can return 429s even under your limits. Ramp up gradually.

### When you hit a limit

You get **HTTP 429** with a `rate_limit_error` and a **`retry-after`** header giving the number of seconds to wait. Every response also carries headers that show your remaining headroom:

```
anthropic-ratelimit-requests-remaining
anthropic-ratelimit-input-tokens-remaining
anthropic-ratelimit-output-tokens-remaining
...-reset   (when the bucket is full again, RFC 3339)
```

The Python SDK **retries 429 and 5xx errors automatically**, twice by default, with exponential backoff. You can tune it:

```python
client = anthropic.Anthropic(max_retries=5)

try:
    response = client.messages.create(...)
except anthropic.RateLimitError as e:
    print("Still rate-limited after retries:", e)
```

Also handle `529 overloaded_error`. It means Anthropic is temporarily overloaded, not that you went over your limits. Retry with backoff, or fall back to another model.

---

## 5. Usage tiers and spend limits

Organisations are placed in a **tier automatically**, based on usage history and account standing. They move up over time.

| Tier | Monthly spend cap | Example limit: Opus 5.5 / Sonnet 5.5 / Haiku 4.5 |
|---|---|---|
| **Evaluation** | Below Start | Lower limits for new organisations while account history builds up |
| **Start** | $500 | 1,000 RPM · 2M ITPM · 400K OTPM |
| **Build** | $1,000 | 5,000 RPM · 5M ITPM · 1M OTPM |
| **Scale** | $200,000 | 10,000 RPM · 10M ITPM · 2M OTPM |
| **Custom** | None (negotiated) | Arranged with Anthropic sales |

Fable 5.1 has lower limits in each tier. At Start it's 1,000 RPM, 500K ITPM and 100K OTPM.

Things to know:

- **Hitting the tier's spend cap** pauses API use until the 1st of next month. You get a 429 with `error_code: "enforced_spend_limit_reached"` and **no** `retry-after`, so retrying won't help. Request a higher limit instead.
- **You can set your own lower spend limit** under Console → Settings → Billing. Reaching it returns HTTP 400.
- **Workspaces** let you split one organisation by team or environment, each with its own lower limits. That stops a runaway dev job from starving production.
- **To go higher**, use *Request rate limit increase* on the Console's Rate limits page, or talk to Anthropic sales for Custom.
- **The Batch API has separate limits**, shared across models. For example, the Start tier allows 200K batch requests waiting in the queue.
- **Bedrock and Vertex have their own quotas**, managed in AWS or GCP, not Anthropic's tiers.

### A question SAs get asked: "Can it handle our volume?"

> 2,000 requests per minute at peak, 3,000 input tokens each, of which 2,000 are cached, and 400 output tokens each, on Sonnet 5.5.

- RPM: 2,000. That fits **Build** (5,000), not Start (1,000).
- ITPM: 2,000 × 1,000 uncached = 2M. That's right at the Start limit, and comfortable at Build (5M).
- OTPM: 2,000 × 400 = 800K. That's over Start (400K) and fine at Build (1M).

The answer is **Build tier, with caching enabled**. Without caching, ITPM would be 6M, which would need Scale. Caching is what makes the plan work.

---

## Exercise: the cost calculator

This is the second Phase 1 project. Build `cost_calculator.py` that:

- [ ] Takes requests per day, average input tokens, average output tokens and the cached share of input
- [ ] Prints the daily and monthly cost for Haiku 4.5, Sonnet 5.5, Opus 5.5 and Fable 5.1
- [ ] Shows the cost with and without caching, and with the Batch API
- [ ] Checks the peak RPM, ITPM and OTPM against the Start, Build and Scale limits and recommends a tier
- [ ] Bonus: reads the real `usage` from one live API call to calibrate the estimates

## Check yourself

- [ ] Roughly how many words are in 1M tokens, and why can the same text cost more tokens on one model than another?
- [ ] Why does output cost matter more than input cost for most chat applications?
- [ ] Name two ways prompt caching helps, not just one.
- [ ] What's the difference between a 429 from a rate limit and a 429 from the spend cap?
- [ ] A customer needs 3,000 RPM on Opus 5.5. Which tier do they need?
