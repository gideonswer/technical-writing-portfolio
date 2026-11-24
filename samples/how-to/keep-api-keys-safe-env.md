# How to Keep API Keys Safe with .env Files

> **Type:** How-to guide · **Audience:** Developers using AI or other paid APIs · **Time:** 10 minutes

An API key in your code is one `git push` away from being public. Automated bots scan GitHub for leaked keys constantly, and a leaked AI key can use up your quota or run up a bill within hours.

This guide shows you how to keep keys in a `.env` file that stays on your computer, load them in Python, and recover if a key does leak.

## Before you start

- A Python project folder
- Git, if you use version control (you should)

## 1. Create a .env file

In your project's top-level folder, create a file named exactly `.env`, with no name before the dot. Add one key per line:

```env
GEMINI_API_KEY=AIzaSyD-example-not-a-real-key
GROQ_API_KEY=gsk_example_not_a_real_key
```

Format rules:

- `NAME=value`, with no spaces around `=`
- No quotes needed, unless the value contains spaces
- Names in `UPPER_SNAKE_CASE`, by convention

## 2. Stop Git from tracking it

Create or open `.gitignore` in the same folder, and add:

```text
.env
```

Check that it worked:

```bash
git status
```

`.env` should **not** appear in the list. If it does, see [If .env was already committed](#if-env-was-already-committed).

> ⚠️ **Do this before your first commit.** Once a file is in your Git history, deleting it later doesn't remove it from the history.

## 3. Load the keys in Python

Install `python-dotenv`:

```bash
pip install python-dotenv
```

Load the file at the start of your program, then read keys with `os.getenv`:

```python
import os
from dotenv import load_dotenv

load_dotenv()  # reads .env and adds its values to the environment

api_key = os.getenv("GEMINI_API_KEY")
if not api_key:
    raise SystemExit("GEMINI_API_KEY is missing. Add it to your .env file.")
```

The check at the end turns a confusing error later ("401 Unauthorized") into a clear message now.

> 💡 Some libraries, such as Google's `google-genai`, read a correctly named variable automatically. With those, `load_dotenv()` is all you need.

## 4. Share a template, not your keys

Other people, and future you, need to know which keys the project expects. Commit a file called `.env.example` with the names and **empty or placeholder** values:

```env
# Copy this file to .env and fill in your own keys. Never commit .env.

# From https://aistudio.google.com/apikey
GEMINI_API_KEY=

# From https://console.groq.com/keys
GROQ_API_KEY=
```

A new contributor runs `cp .env.example .env` (or `copy .env.example .env` on Windows) and fills in their own values.

## 5. Use the platform's secret settings when you deploy

Hosting platforms don't read your local `.env` file. Add each key in the platform's settings instead. They're usually called **Environment variables** or **Secrets**. Your code doesn't change: `os.getenv("GEMINI_API_KEY")` reads from either place.

## If .env was already committed

If a key has ever been pushed to GitHub, **treat it as leaked**, even if the repository is private or you deleted the file a minute later.

1. **Revoke the key immediately** in the provider's dashboard, and create a new one.
2. **Put the new key in `.env`** only.
3. **Stop tracking the file** without deleting your local copy:

   ```bash
   git rm --cached .env
   echo ".env" >> .gitignore
   git commit -m "Stop tracking .env"
   ```

4. **Check your usage dashboard** for activity you don't recognise.

Revoking the key (step 1) is what actually protects you. Removing the file from Git stops it happening again, but the old key stays in your history, so it must be replaced.

## Checklist

- [ ] `.env` exists and holds your keys
- [ ] `.env` is listed in `.gitignore`, and `git status` doesn't show it
- [ ] Your code uses `os.getenv`; no key appears in any `.py` file
- [ ] `.env.example` is committed, with no real values
- [ ] You've never shared a screenshot that shows a key

---

<details>
<summary><strong>Writer's notes</strong>: decisions behind this piece</summary>

- **Opens with the risk, briefly.** One sentence of "why" motivates the reader without a lecture.
- **Recovery gets its own section.** Most guides stop at prevention, but people usually find this page *after* they've leaked a key. The steps are ordered by urgency: revoke first, clean up second.
- **Corrects a common misunderstanding:** deleting the file doesn't make a leaked key safe.
- **Ends with a checklist** so readers can verify their setup in seconds.

</details>
