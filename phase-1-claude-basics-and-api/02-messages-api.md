# Lesson 2: The Messages API

## The idea in one sentence

Almost everything you do with Claude goes through one endpoint, `POST /v1/messages`. You send a list of messages and get back one new assistant message.

Tools, images, PDFs, structured output and streaming are all options on this same endpoint, not separate APIs.

## Setup

```bash
pip install anthropic
export ANTHROPIC_API_KEY="sk-ant-..."
```

```python
import anthropic

client = anthropic.Anthropic()  # reads ANTHROPIC_API_KEY from the environment
```

Never put the key in your code or commit it to git.

## The smallest possible request

```python
response = client.messages.create(
    model="claude-opus-5-5",
    max_tokens=1024,
    messages=[{"role": "user", "content": "What is a heat pump, in two sentences?"}],
)

for block in response.content:
    if block.type == "text":
        print(block.text)

print(response.stop_reason)  # e.g. "end_turn"
print(response.usage)        # input_tokens, output_tokens, ...
```

Only three fields are required: `model`, `max_tokens` and `messages`.

The response looks like this, shortened:

```json
{
  "id": "msg_01...",
  "type": "message",
  "role": "assistant",
  "model": "claude-opus-5-5",
  "content": [{ "type": "text", "text": "A heat pump moves heat..." }],
  "stop_reason": "end_turn",
  "usage": { "input_tokens": 18, "output_tokens": 54 }
}
```

`content` is a **list of blocks**, not a single string. Depending on the request it can hold `text`, `thinking` or `tool_use` blocks, so always check `block.type`.

---

## 1. Roles

There are two roles in `messages`:

| Role | Who | Notes |
|---|---|---|
| `user` | Your application or the end user | The first message must be `user` |
| `assistant` | Claude | Earlier replies you send back to keep the conversation going |

### The API is stateless

Claude doesn't remember earlier calls. To hold a conversation, **send the whole history every time**:

```python
history = []

def chat(user_text: str) -> str:
    history.append({"role": "user", "content": user_text})
    response = client.messages.create(
        model="claude-opus-5-5",
        max_tokens=4096,
        messages=history,
    )
    history.append({"role": "assistant", "content": response.content})
    return "".join(b.text for b in response.content if b.type == "text")

print(chat("My name is Imad."))
print(chat("What's my name?"))  # works only because we resent the history
```

Append the **full `response.content`**, not only its text. Thinking and tool-use blocks must go back unchanged, or later features will break.

Because the whole history is resent on every call, a long conversation costs more with each turn. That's the reason prompt caching exists, which Phase 5 covers.

### Prefilling is gone on current models

On older models you could start the assistant's reply yourself, for example ending with `{"role": "assistant", "content": "{"}` to force JSON. **Current models (Opus 4.6 and later, Sonnet 4.6 and later, Fable) reject this with a 400 error.** Use structured outputs or clear instructions instead. Haiku 4.5 still accepts it.

---

## 2. System prompts

The system prompt sets who Claude is and the rules for the whole conversation. It's a **top-level parameter**, not a message:

```python
response = client.messages.create(
    model="claude-sonnet-5-5",
    max_tokens=2048,
    system=(
        "You are a support assistant for an energy supplier. "
        "Answer only questions about bills, tariffs and meters. "
        "If you don't know, say so and offer to connect the customer to an agent."
    ),
    messages=[{"role": "user", "content": "Why is my bill higher this month?"}],
)
```

What belongs in the system prompt:

- The role and audience: who Claude is and who it's talking to
- Rules and limits: what to do and what to refuse
- Tone and output format
- Stable context such as product facts or policies. Keep it stable, because a stable prefix can be cached cheaply.

What belongs in the user message: the specific question, the document for this request and anything that changes on each call.

Newer models (Opus 5.5, Sonnet 5.5, Fable 5.1 and others) also accept `{"role": "system", ...}` entries **inside** `messages`, so you can add an instruction partway through a conversation without changing the cached system prompt. That's an advanced feature to remember for later.

---

## 3. `max_tokens`

`max_tokens` is the **hard ceiling on the output** of one response. It's required.

- When Claude reaches it, the output is **cut off mid-sentence** and `stop_reason` is `"max_tokens"`.
- Claude **doesn't see** the limit and won't shorten its answer to fit. To get shorter answers, ask for them in the prompt.
- On models with thinking, **thinking tokens count toward `max_tokens`**.
- You pay for the tokens actually produced, not for `max_tokens`. A high limit costs nothing extra, and it doesn't count against your rate limit either.
- The maximum is 128K on Opus 5.5, Sonnet 5.5 and Fable 5.1, and 64K on Haiku 4.5.

Practical values:

| Situation | `max_tokens` |
|---|---|
| Classification or a one-word answer | ~256 |
| Normal request without streaming | ~16,000 |
| Request with streaming | ~64,000 |

Don't set it too low. A truncated answer means a wasted call and a retry. For large values, use streaming: the SDK may refuse a very large `max_tokens` without it, because the HTTP request could time out.

---

## 4. `temperature`: changed on current models

`temperature` controls randomness when the model picks each token. It goes from 0.0 (more predictable) to 1.0 (more varied), and the default is 1.0. `top_p` and `top_k` are related sampling settings.

**This is an important recent change.** You'll find it in older tutorials, but current top models don't accept it:

| Model | `temperature` / `top_p` / `top_k` |
|---|---|
| Opus 5.5, Opus 5, Opus 4.7–4.8, Fable 5 / 5.1, Sonnet 5 | **Removed.** Sending them returns a 400 error |
| Sonnet 5.5 | Only the default values are accepted. Anything else returns a 400 |
| Haiku 4.5, Opus 4.6, Sonnet 4.6 | Still accepted. On 4.x models, set `temperature` or `top_p`, not both |

On current models, steer the output in these ways instead:

- **`effort`**, which controls how hard the model thinks:
  ```python
  output_config={"effort": "low"}   # low | medium | high | xhigh | max
  ```
- **Clear instructions and examples** in the prompt. Phase 2 covers this.
- **Structured outputs** when you need machine-readable JSON.

For a customer, the takeaway is that consistency now comes from **good prompts and evals**, not from setting `temperature=0`. If someone asks for "temperature 0 for determinism", explain that LLM output isn't fully deterministic even at 0, and that current models are controlled through effort and prompts instead.

---

## 5. Stop reasons

Every response has a `stop_reason`. **Check it before using the content.**

| `stop_reason` | Meaning | What to do |
|---|---|---|
| `end_turn` | Claude finished normally | Use the answer |
| `max_tokens` | Hit your `max_tokens` limit, so the output is cut off | Raise `max_tokens`, or ask for shorter output |
| `stop_sequence` | Generated one of your custom `stop_sequences` | `response.stop_sequence` tells you which one |
| `tool_use` | Claude wants to call one of your tools | Run the tool and send back a `tool_result`. Phase 4 covers this |
| `pause_turn` | A long server-side tool loop paused | Send the conversation back as it is to continue |
| `refusal` | Safety systems declined the request | Don't use `content`. `response.stop_details` has the category |
| `model_context_window_exceeded` | The input plus output filled the context window | Shorten the input or summarise the history |

```python
response = client.messages.create(...)

match response.stop_reason:
    case "end_turn" | "stop_sequence":
        text = "".join(b.text for b in response.content if b.type == "text")
    case "max_tokens":
        raise RuntimeError("Output truncated: raise max_tokens or ask for less")
    case "refusal":
        print("Declined:", response.stop_details)
    case "tool_use":
        ...  # handle tools (Phase 4)
    case other:
        print("Unexpected stop reason:", other)
```

Custom stop sequences:

```python
response = client.messages.create(
    model="claude-haiku-4-5",
    max_tokens=500,
    stop_sequences=["END"],
    messages=[{"role": "user", "content": "List three tariffs, then write END."}],
)
```

---

## 6. Streaming

Without streaming, you wait until the whole answer is ready and get it all at once. **With streaming, tokens arrive as they're generated**, using Server-Sent Events.

Use streaming when:

- **People are waiting**, as in a chat UI. The first words appear in under a second instead of after 20 seconds.
- **The output is long**, or `max_tokens` is high. This avoids HTTP timeouts.

The simple way, using the SDK helper:

```python
with client.messages.stream(
    model="claude-sonnet-5-5",
    max_tokens=64000,
    messages=[{"role": "user", "content": "Explain how a smart grid balances demand."}],
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)

    final = stream.get_final_message()  # the complete message, as without streaming

print("\nStop reason:", final.stop_reason)
print("Usage:", final.usage)
```

The events underneath, which you'll see when you work with the raw stream:

```
message_start          → message metadata, input token count
content_block_start    → a new text, thinking or tool_use block begins
content_block_delta    → a piece of text (text_delta) or tool input (input_json_delta)
content_block_stop     → the block is finished
message_delta          → final stop_reason and output token count
message_stop           → done
ping                   → keep-alive, ignore it
error                  → for example overloaded_error, which can arrive mid-stream
```

Streaming doesn't change the price or the quality. It only changes **when** you receive the tokens.

---

## Putting it together: the Phase 1 chatbot

This is the starting point for the first project. Save it as `chatbot.py`:

```python
import anthropic

client = anthropic.Anthropic()
MODEL = "claude-sonnet-5-5"
SYSTEM = "You are a concise, friendly assistant for energy-sector engineers."

history = []

while True:
    user_text = input("\nYou: ").strip()
    if user_text in {"exit", "quit"}:
        break
    history.append({"role": "user", "content": user_text})

    print("Claude: ", end="", flush=True)
    with client.messages.stream(
        model=MODEL, max_tokens=8000, system=SYSTEM, messages=history
    ) as stream:
        for text in stream.text_stream:
            print(text, end="", flush=True)
        final = stream.get_final_message()

    history.append({"role": "assistant", "content": final.content})
    u = final.usage
    print(f"\n  [{final.stop_reason} | in={u.input_tokens} out={u.output_tokens}]")
```

Ideas for extending it:

- [ ] A `/model` command to switch between Haiku, Sonnet and Opus during a chat
- [ ] A running total of cost, using the prices from lesson 3
- [ ] Error handling for `anthropic.RateLimitError` and `anthropic.APIConnectionError`
- [ ] Saving the conversation to a file and loading it back

## Check yourself

- [ ] Explain why you resend the whole conversation on every call, and what that does to cost.
- [ ] Explain the difference between the `system` parameter and a `user` message.
- [ ] Say what happens when `max_tokens` is too low, and how you'd detect it.
- [ ] Explain why `temperature=0` isn't the answer on current models, and what to use instead.
- [ ] Name four stop reasons and how to handle each one.
- [ ] Explain when streaming matters and when it doesn't.
