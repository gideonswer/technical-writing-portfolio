# Common Errors When Calling LLM APIs and How to Fix Them

> **Type:** Troubleshooting guide · **Audience:** Developers calling Gemini, Groq, OpenAI, Claude or similar APIs · **Applies to:** any language

Most errors from AI model APIs fall into a small number of groups. Find your error code or symptom in the table, then jump to its section.

| Code or symptom | Short name | Your fault or theirs? | Retry? |
|---|---|---|---|
| [`400`](#400-bad-request) | Bad request | Yours | No, fix the request |
| [`401` / `403`](#401--403-authentication-failed) | Authentication failed | Yours | No, fix the key |
| [`404`](#404-model-not-found) | Model not found | Yours | No, fix the model name |
| [`413` or a "context length" message](#too-many-tokens) | Too many tokens | Yours | No, send less |
| [`429`](#429-rate-limit-or-quota-exceeded) | Rate limit or quota | Neither | Yes, after waiting |
| [`500` / `502` / `503`](#500--503-server-errors) | Server error | Theirs | Yes, with backoff |
| [Timeout](#timeouts) | No response in time | Usually theirs | Yes, with backoff |
| [Empty or cut-off reply](#empty-or-cut-off-replies) | No error, bad output | Varies | Sometimes |
| [Can't parse JSON from the reply](#the-reply-isnt-valid-json) | Format problem | Yours | Sometimes |

> 💡 **The golden rule:** retry `429`, `5xx` and timeouts. **Never** automatically retry `400`, `401`, `403` or `404`: the same request will fail the same way, and you'll waste quota finding out.

---

## 400 Bad request

**What it means:** the API understood the request but something in it is invalid.

### Common causes and fixes

| Cause | Fix |
|---|---|
| A parameter is out of range, such as `temperature: 5` | Check the provider's allowed range |
| The message list is malformed, such as an empty message or a wrong `role` value | Print the exact request body and compare it with the docs |
| A parameter isn't supported by this model | Remove it, or switch to a model that supports it |
| **Gemini only:** the API key is invalid | Gemini returns `400` with "API key not valid" rather than `401`. See [401 / 403](#401--403-authentication-failed). |

**Debugging tip:** the error body almost always names the problem field. Log the full error message, not just the status code.

---

## 401 / 403 Authentication failed

**What it means:** the API doesn't accept your key (`401`), or the key is valid but not allowed to do this (`403`).

### Checklist

1. **Is the key loaded at all?** Print `bool(os.getenv("YOUR_KEY_NAME"))`. If it's `False`, your `.env` file isn't being read. See [How to Keep API Keys Safe with .env Files](../how-to/keep-api-keys-safe-env.md).
2. **Is it the right key for this provider?** A Groq key won't work with Gemini.
3. **Was it copied completely?** Look for missing characters, spaces, quotes or a trailing newline.
4. **Has it been revoked or regenerated?** Check the provider's dashboard.
5. **For `403`:** is the API enabled for your project or region, and does your plan include this model?

---

## 404 Model not found

**What it means:** the model name you sent doesn't exist, or you don't have access to it.

### Causes

- A typo, for example `gemini-flash-3.8` instead of `gemini-3.8-flash`.
- The model has been **retired**. Providers regularly retire older models, so code that worked last year can start failing without any change on your side.
- You're using one provider's model name with another provider's endpoint.

**Fix:** copy the exact name from the provider's current models page. Keep the model name in configuration (such as a `.env` setting), not hard-coded, so you can change it without editing code.

---

## Too many tokens

**What it means:** the input plus the requested output is larger than the model's [context window](../concepts/tokens-and-context-windows.md). Depending on the provider, this appears as a `400` or `413` with a message mentioning tokens, context length or input size.

### Fixes

- Send less text. For documents, send only the relevant passages instead of the whole file. That's what [RAG](../concepts/how-rag-works.md) is for.
- In long chats, drop or summarise the oldest messages.
- Lower the maximum output tokens, because they count towards the limit too.
- Count tokens before sending. Most SDKs have a token-counting method.

---

## 429 Rate limit or quota exceeded

**What it means:** you've sent too many requests, or too many tokens, in a period of time. Free tiers have low limits, often per minute **and** per day.

**How to tell which limit you hit:** read the error message. It usually says whether it's requests per minute, tokens per minute or a daily quota.

| Limit | Fix |
|---|---|
| Per minute | Wait and retry with backoff (see below) |
| Per day | Wait for the reset, upgrade, or **fall back to a second provider** |

**Retry with exponential backoff:** wait a little, then retry; if it fails again, wait twice as long. If the response includes a `Retry-After` header, wait at least that long.

```python
import random
import time

RETRYABLE = {429, 500, 502, 503, 504}

def call_with_retry(make_request, max_attempts=5):
    for attempt in range(max_attempts):
        try:
            return make_request()
        except ApiError as e:   # use your SDK's error class
            if e.status_code not in RETRYABLE or attempt == max_attempts - 1:
                raise
            wait = (2 ** attempt) + random.random()   # 1s, 2s, 4s, 8s... plus jitter
            time.sleep(wait)
```

The small random "jitter" stops many clients from retrying at exactly the same moment.

> 💡 **Use a fallback model for daily limits.** For example, [Chat with Docs](https://github.com/gideonswer/chat-with-docs) uses Gemini as its main model and switches to Groq automatically when Gemini fails, so users never see the daily limit.

---

## 500 / 503 Server errors

**What it means:** something went wrong on the provider's side, or the service is overloaded (`503`). Your request may be fine.

**Fix:** retry with backoff, as above. If errors continue for more than a few minutes, check the provider's status page. Show users a friendly message such as "The AI service is busy, please try again" instead of a raw error.

---

## Timeouts

**What it means:** the response didn't arrive before your client gave up. Long outputs, large inputs and busy servers make this more likely.

### Fixes

- Set an explicit timeout (for example 60 seconds) rather than relying on the default, so requests can't hang forever.
- **Stream** long responses. The first words arrive within a second or two, and the connection stays active.
- Retry with backoff, a limited number of times.

---

## Empty or cut-off replies

There's no error, but the reply is empty or ends mid-sentence. Check the **finish reason** in the response. Most APIs include one.

| Finish reason (names vary) | Meaning | Fix |
|---|---|---|
| `length` / `max_tokens` / `MAX_TOKENS` | The output limit was reached | Raise the maximum output tokens, or ask for a shorter answer |
| `safety` / `content_filter` / `SAFETY` | The provider's safety filter blocked the output | Rephrase the request; check the provider's content policies |
| `stop` / `end_turn` / `STOP` | The model finished normally | If the text is still empty, log the full response and check your parsing code |

Always handle a missing text field safely, for example `text = response.text or ""`, so an empty reply doesn't crash your app.

---

## The reply isn't valid JSON

You asked for JSON, but `json.loads()` fails. Models sometimes wrap JSON in code fences, add a sentence before it, or leave a trailing comma.

### Fixes, best first

1. **Use the provider's structured output or JSON mode**, if it has one. It's far more reliable than asking in the prompt.
2. Show the exact format in the prompt, with an example, and set the temperature to `0`.
3. Extract the JSON part before parsing:

   ```python
   import json
   import re

   match = re.search(r"\{.*\}", reply, re.DOTALL)
   data = json.loads(match.group(0)) if match else None
   ```

4. Have a fallback for when parsing still fails, such as retrying once or using a default value. Don't let one bad reply crash the program.

---

## Before you ask for help

If none of the above fixes it, collect this information first. It answers most of the questions anyone helping you will ask:

- [ ] The provider, SDK name and SDK version (`pip show <package>`)
- [ ] The exact model name
- [ ] The full error message and status code
- [ ] A minimal request that reproduces the problem, **with the API key removed**
- [ ] Whether it fails every time or only sometimes

---

<details>
<summary><strong>Writer's notes</strong>: decisions behind this piece</summary>

- **Built for scanning.** People arrive at troubleshooting pages under pressure, with an error on screen. The first table lets them find their code, see whether to retry, and jump straight to the fix.
- **"Your fault or theirs" and "Retry?" columns** answer the two questions every developer has first.
- **Provider-neutral, with exceptions called out.** Gemini's `400` for an invalid key is a real gotcha, so it's flagged where readers will look.
- **No exact error strings** for most providers, because they change. Instead the guide teaches readers what to look for in the message.
- **Ends with a "before you ask for help" checklist**, which helps the reader and whoever supports them.

</details>
