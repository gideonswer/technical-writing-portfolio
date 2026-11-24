# Build Your First AI Chatbot in Python with the Gemini API

> **Type:** Tutorial · **Audience:** Beginner developers who know basic Python · **Time:** 20 minutes · **Cost:** Free

In this tutorial you'll build a command-line chatbot that remembers the conversation, has its own personality, and streams its replies word by word, like ChatGPT does.

```text
Study buddy is ready! Type 'quit' to exit.

You: What is photosynthesis?
Bot: Photosynthesis is how plants make their own food using sunlight...

You: Give me a simpler version for a 10-year-old
Bot: Sure! Plants are like little chefs...
```

Notice that the second question says "a simpler version" without repeating the topic. The bot understands because it remembers the conversation. You'll see how that works in [Step 4](#step-4-give-the-bot-a-memory).

## What you'll learn

- How to get a free Gemini API key and keep it out of your code
- How to send a message to Gemini from Python
- How to give the bot a memory and a personality
- How to stream replies and handle errors

## Before you start

You need:

- **Python 3.9 or newer.** Check by running `python --version` in a terminal.
- **A Google account**, to get a free API key.
- A code editor, such as VS Code.

## Step 1: Get a free API key

1. Go to [Google AI Studio](https://aistudio.google.com/apikey) and sign in.
2. Click **Create API key**.
3. Copy the key. It's a long string of letters and numbers.

> ⚠️ **Treat your API key like a password.** Anyone who has it can use your quota. Never paste it into your code or share it in a screenshot. The next step shows the safe way to store it.

## Step 2: Set up the project

Create a folder for the project and open a terminal in it:

```bash
mkdir study-buddy
cd study-buddy
```

Create a **virtual environment**, a private space for this project's packages, and turn it on:

```bash
python -m venv .venv

# Windows
.venv\Scripts\activate

# macOS / Linux
source .venv/bin/activate
```

You'll see `(.venv)` at the start of your terminal line. Now install the two packages you need:

```bash
pip install google-genai python-dotenv
```

| Package | What it does |
|---|---|
| `google-genai` | Google's official library for talking to Gemini |
| `python-dotenv` | Loads your API key from a file, so it never appears in your code |

Create a file named `.env` in the folder and paste your key into it:

```env
GEMINI_API_KEY=paste-your-key-here
```

If you use Git, create a `.gitignore` file containing this line, so the key is never uploaded:

```text
.env
```

> For more on this, see [How to Keep API Keys Safe with .env Files](../how-to/keep-api-keys-safe-env.md).

## Step 3: Send your first message

Create `hello.py`:

```python
from dotenv import load_dotenv
from google import genai

load_dotenv()               # puts GEMINI_API_KEY into the environment
client = genai.Client()     # finds the key automatically

response = client.models.generate_content(
    model="gemini-3.8-flash",
    contents="Say hello in Khasi, Hindi and English.",
)
print(response.text)
```

Run it:

```bash
python hello.py
```

You should see a short greeting in three languages. If you get an error instead, jump to [Troubleshooting](#troubleshooting).

**What just happened:** `genai.Client()` connected to Gemini using your key. `generate_content` sent one message and waited for the full reply. `response.text` is the reply as plain text.

> 📌 **About the model name:** `gemini-3.8-flash` is a fast, free-tier-friendly model at the time of writing. Model names change regularly. Check the current list on the [Gemini models page](https://ai.google.dev/gemini-api/docs/models) and swap in a newer one if needed.

## Step 4: Give the bot a memory

`generate_content` treats every call as a brand-new conversation. To let the bot remember, use a **chat session**. It keeps the message history and sends it to Gemini with every new message.

## Step 5: Give it a personality and stream replies

A **system instruction** is a message that tells the model how to behave for the whole conversation. It's where you set the bot's role, tone and rules.

**Streaming** means printing the reply in small pieces as Gemini writes it, instead of waiting for the whole thing. It makes the bot feel much faster.

Create `chatbot.py` with the complete program:

```python
from dotenv import load_dotenv
from google import genai
from google.genai import errors, types

load_dotenv()  # reads GEMINI_API_KEY from the .env file

MODEL = "gemini-3.8-flash"
SYSTEM_PROMPT = (
    "You are a friendly study buddy. Explain things simply, "
    "use short paragraphs, and give one example when it helps."
)

client = genai.Client()  # finds GEMINI_API_KEY automatically
chat = client.chats.create(
    model=MODEL,
    config=types.GenerateContentConfig(
        system_instruction=SYSTEM_PROMPT,
        temperature=0.7,
    ),
)

print("Study buddy is ready! Type 'quit' to exit.\n")

while True:
    user_input = input("You: ").strip()
    if user_input.lower() in {"quit", "exit"}:
        print("Bye!")
        break
    if not user_input:
        continue

    try:
        print("Bot: ", end="", flush=True)
        for chunk in chat.send_message_stream(user_input):
            print(chunk.text or "", end="", flush=True)
        print("\n")
    except errors.APIError as e:
        print(f"\n[Error {e.code}] {e.message}\n")
```

Run it:

```bash
python chatbot.py
```

Ask a question, then a follow-up that depends on it, such as *"Explain it more simply"*. The bot will know what "it" means.

### How the code works

| Code | What it does |
|---|---|
| `client.chats.create(...)` | Starts a chat session that remembers every message |
| `system_instruction=SYSTEM_PROMPT` | Sets the bot's personality for the whole chat |
| `temperature=0.7` | Controls creativity: `0` is focused and predictable, higher values are more varied |
| `chat.send_message_stream(...)` | Sends your message and returns the reply in pieces |
| `flush=True` | Prints each piece immediately instead of waiting |
| `except errors.APIError` | Catches problems such as a bad key or too many requests, so the bot doesn't crash |

## Step 6: Make it your own

Try changing `SYSTEM_PROMPT` and see how the bot's behaviour changes:

```python
SYSTEM_PROMPT = "You are a strict interview coach. Ask me one Python interview question at a time and grade my answer out of 10."
```

```python
SYSTEM_PROMPT = "You are a travel guide for Meghalaya. Recommend places, local food and the best time to visit."
```

## Troubleshooting

| Error or symptom | Likely cause | Fix |
|---|---|---|
| `Missing key inputs argument` or a message about an API key | The `.env` file wasn't found or the variable name is wrong | Make sure `.env` is in the same folder you run Python from, and the line starts with `GEMINI_API_KEY=` |
| `[Error 400] API key not valid` | The key was copied incompletely | Copy the key again from AI Studio, with no spaces or quotes |
| `[Error 404]` mentioning the model | The model name is wrong or retired | Check the [models page](https://ai.google.dev/gemini-api/docs/models) for a current name |
| `[Error 429]` | You've hit the free-tier rate limit | Wait a minute and try again |
| `ModuleNotFoundError: No module named 'google'` | Packages installed outside the virtual environment | Activate `.venv` and run `pip install google-genai python-dotenv` again |

More fixes: [Common Errors When Calling LLM APIs and How to Fix Them](../troubleshooting/llm-api-common-errors.md).

## What you built

A working AI chatbot in about 40 lines of Python that:

- keeps your API key out of the code
- remembers the conversation
- follows a personality you define
- streams replies and handles errors gracefully

## Next steps

- Let the bot answer questions about your own files: [Build a PDF Q&A App in 30 Minutes](pdf-qa-app-30-minutes.md)
- Understand why long chats eventually "forget": [Tokens and Context Windows](../concepts/tokens-and-context-windows.md)

---

<details>
<summary><strong>Writer's notes</strong>: decisions behind this piece</summary>

- **Show the result first.** The sample output at the top tells readers exactly what they'll have at the end, which keeps them going through setup.
- **Security before code.** The `.env` step comes *before* any code, so the reader never sees, or copies, a version with the key pasted in.
- **Build up in small runs.** `hello.py` proves the setup works before memory and streaming are added. If something breaks, the reader knows which step caused it.
- **Tested.** The code samples were checked against the current `google-genai` SDK (`chats.create`, `send_message_stream`, `errors.APIError`).
- **Model names will change**, so the callout tells readers where to find a current one instead of pretending the name is permanent.

</details>
