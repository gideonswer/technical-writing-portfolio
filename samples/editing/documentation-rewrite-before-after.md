# Documentation Rewrite: Before & After

> **Type:** Editing sample · **Audience:** Hiring managers and editors · **Reading time:** 5 minutes

Writing new docs is half the job. The other half is improving docs that already exist. This sample shows a typical first-draft "Getting started" section for an AI app, my rewrite, and the reasoning behind every change.

> The "before" text was written for this exercise. It combines problems I see often in open-source READMEs, including early drafts of my own.

## Before

```text
Installation

This project is a very powerful and easy to use AI based tool which leverages state
of the art LLMs for the purpose of answering questions from PDF documents. In order
to install it, first of all you will need to clone the repo and then you should
install the requirements, it is recommended to use a virtual env. Install torch
first otherwise it may take long time. After that you need to put your keys which
you can get from google and groq, put them in the env file (GEMINI_API_KEY,
GROQ_API_KEY) also set JWT secret. Then simply run the server using uvicorn and go
to localhost. Note: the first upload will be slow. Also scanned PDFs are not
supported. If you get any error check that your keys are correct.

Usage: just upload a pdf and ask questions, its very simple!
```

## After

````markdown
## Quick start

Chat with Docs answers questions about your PDFs and cites the page each answer
came from. Setup takes about 10 minutes.

### Before you begin

You need:

- Python 3.10 or later
- A free Gemini API key from [Google AI Studio](https://aistudio.google.com/apikey)
- A free Groq API key from [Groq Console](https://console.groq.com/keys) (used as
  a backup if Gemini is unavailable)

### 1. Install

```bash
git clone https://github.com/gideonswer/chat-with-docs.git
cd chat-with-docs
python -m venv .venv
source .venv/bin/activate      # Windows: .venv\Scripts\activate

# Install the CPU-only build of PyTorch first. It's much smaller than the default.
pip install torch --index-url https://download.pytorch.org/whl/cpu
pip install -r requirements.txt
```

### 2. Add your keys

```bash
cp .env.example .env           # Windows: copy .env.example .env
```

Open `.env` and fill in:

| Setting          | Value                                   |
|------------------|-----------------------------------------|
| `GEMINI_API_KEY` | Your Gemini key                         |
| `GROQ_API_KEY`   | Your Groq key                           |
| `JWT_SECRET`     | A long random string (see the command below) |

To generate a `JWT_SECRET`:

```bash
python -c "import secrets; print(secrets.token_urlsafe(48))"
```

### 3. Run

```bash
uvicorn app.main:app --reload
```

Open http://localhost:8000, create an account, and upload a PDF.

> **The first upload takes about a minute** while the embedding model (about
> 90 MB) downloads. Later uploads are much faster.

### Limitations

- Only text-based PDFs are supported. Scanned PDFs need OCR first.

### Troubleshooting

| Symptom                        | Fix                                              |
|--------------------------------|--------------------------------------------------|
| `Both models failed` error     | Check both keys in `.env` and restart the server |
| "No text found" on upload      | The PDF is scanned. Run OCR on it first.         |
````

## What changed and why

| # | Before | After | Why |
|---|---|---|---|
| 1 | "very powerful and easy to use AI based tool which leverages state of the art LLMs" | "answers questions about your PDFs and cites the page each answer came from" | Marketing adjectives tell the reader nothing. A concrete description of what the tool does is shorter and more convincing. |
| 2 | No prerequisites | A "Before you begin" list | Readers need to know *before* starting that they need Python 3.10 and two keys, not discover it halfway through. |
| 3 | One paragraph containing six steps | Numbered sections, each with commands | Readers can follow along, keep their place, and copy each command. |
| 4 | "clone the repo", "install the requirements" | Exact commands | Vague instructions make every reader translate them into commands, and some will get it wrong. |
| 5 | "Install torch first otherwise it may take long time" | Exact command plus the reason | The tip is valuable, but useless without the actual command (the CPU-only index URL). |
| 6 | "put your keys… in the env file" | Copy the template, then a table of settings | Says which file, which settings and where to get each value. |
| 7 | "also set JWT secret" | A command that generates one | Most readers don't know what a good JWT secret looks like. Giving the command prevents weak secrets. |
| 8 | "go to localhost" | `http://localhost:8000` and what to do there | The port matters, and readers need to know the next action. |
| 9 | "Note: the first upload will be slow" | A callout saying how slow and why | "Slow" makes people think it's broken. "About a minute, because it's downloading 90 MB" makes them wait. |
| 10 | "Also scanned PDFs are not supported" | A Limitations section | Limits shouldn't be hidden in a run-on sentence. |
| 11 | "If you get any error check that your keys are correct" | A troubleshooting table | Maps specific symptoms to specific fixes. |
| 12 | "its very simple!" | Removed | Telling readers something is simple makes them feel bad when it isn't. |

## Principles applied

- **Lead with what it does, not how good it is.**
- **One step, one action**, with the exact command.
- **Say what readers need before they need it:** prerequisites first, warnings before the step they apply to.
- **Explain the reason** whenever an instruction might look optional.
- **Replace vague words** ("slow", "the env file", "localhost") with specifics.

The rewrite is longer, but it is faster to *use*: every step can be followed without guessing.

---

<details>
<summary><strong>Writer's notes</strong>: decisions behind this piece</summary>

- **Labelled the "before" honestly** as a constructed example. Rewriting someone else's real docs without permission, or presenting an invented draft as a real client's, wouldn't be fair.
- **The change table is numbered** so a reviewer can discuss a specific edit ("I'd push back on #7") in an interview or code review.
- **Acknowledged the trade-off:** the rewrite is longer. Good editing isn't always cutting; it's making the reader's job easier.

</details>
