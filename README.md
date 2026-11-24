# Technical Writing Portfolio · Gideon Zion Swer

[![Docs check](https://github.com/gideonswer/technical-writing-portfolio/actions/workflows/docs-check.yml/badge.svg)](https://github.com/gideonswer/technical-writing-portfolio/actions/workflows/docs-check.yml)

I write documentation for **AI and developer tools**, and I build the tools too. Every sample here is written for a named audience, follows a [published style guide](STYLE-GUIDE.md), and is checked like code: each commit is linted and link-checked automatically.

Most of the technical samples document [Chat with Docs](https://github.com/gideonswer/chat-with-docs), a RAG application I built, so the code examples, API details and error messages come from working software, not invented examples.

## ⭐ Start here

Short on time? These three show the range:

| Sample | What it shows |
|---|---|
| [**Chat with Docs API Reference**](samples/reference/chat-with-docs-api.md) | Full REST reference written from source code: quick start, auth, every endpoint, error table with fixes |
| [**How to Measure Whether Your RAG Answers Are Accurate**](samples/how-to/measure-rag-accuracy.md) | Explaining evaluation metrics to developers, including a real finding from my own testing |
| [**Documentation Rewrite: Before & After**](samples/editing/documentation-rewrite-before-after.md) | Editing judgement: 12 numbered changes, each with the reason behind it |

## All samples

### Concept explainers

For readers who are new to the idea. No code needed.

| Title | Audience | Highlights |
|---|---|---|
| [How RAG Works: Giving AI Access to Your Documents](samples/concepts/how-rag-works.md) | Beginners | Open-book-exam analogy mapped to the real components; flow diagram |
| [What Are AI Agents? Tools, Planning and Memory Explained](samples/concepts/ai-agents-explained.md) | Beginners, non-technical teammates | One agent loop that the whole page builds on; failure modes and guard rails |
| [Embeddings Explained Without the Maths](samples/concepts/embeddings-explained.md) | Beginners | A "map of meaning" that scales from 2 to 384 dimensions |
| [Tokens and Context Windows: Why AI Forgets](samples/concepts/tokens-and-context-windows.md) | Beginners, regular AI users | Starts from a frustration every user has seen; symptom → action table |

### Tutorials

Learn by building. Every code sample was tested.

| Title | Audience | Highlights |
|---|---|---|
| [Build Your First AI Chatbot in Python with the Gemini API](samples/tutorials/gemini-chatbot-python.md) | Beginner developers | Memory, personality and streaming in ~40 lines; API key safety before any code |
| [Build a PDF Q&A App in 30 Minutes](samples/tutorials/pdf-qa-app-30-minutes.md) | Beginner developers | A complete RAG app with page citations in ~100 lines |

### How-to guides

For developers with a specific goal.

| Title | Audience | Highlights |
|---|---|---|
| [How to Add Page Citations to a RAG App](samples/how-to/add-page-citations-rag.md) | Developers | Four changes, each with a check; explains the purpose of every prompt rule |
| [How to Measure Whether Your RAG Answers Are Accurate](samples/how-to/measure-rag-accuracy.md) | Developers | Separates retrieval from generation failures; results-diagnosis table |
| [How to Keep API Keys Safe with .env Files](samples/how-to/keep-api-keys-safe-env.md) | Developers | Covers recovery after a leak, not just prevention |

### API reference

| Title | Audience | Highlights |
|---|---|---|
| [Chat with Docs API Reference](samples/reference/chat-with-docs-api.md) | Developers integrating the API | 14 endpoints, consistent structure, calls out the surprises integrators hit |

### Troubleshooting

| Title | Audience | Highlights |
|---|---|---|
| [Common Errors When Calling LLM APIs and How to Fix Them](samples/troubleshooting/llm-api-common-errors.md) | Developers | Scannable index with "retry?" column; retry-with-backoff code |

### Editing and release notes

| Title | Audience | Highlights |
|---|---|---|
| [Documentation Rewrite: Before & After](samples/editing/documentation-rewrite-before-after.md) | Hiring managers, editors | Annotated rewrite of a first-draft README |
| [Chat with Docs 1.0 Release Notes](samples/editing/chat-with-docs-release-notes.md) | App users and self-hosters | Benefits for users, details for self-hosters, honest limitations |

> Each sample ends with **Writer's notes**, a short, collapsible explanation of the audience and the decisions behind it.

## How I work

- **Audience first.** Each page states who it's for, and its depth, vocabulary and examples follow from that.
- **The right type for the job.** Concepts, tutorials, how-tos and references follow the [Diátaxis](https://diataxis.fr/) framework, so each page does one job well.
- **Consistent style.** All samples follow my [style guide](STYLE-GUIDE.md), based on the Google and Microsoft developer style guides.
- **Accurate and tested.** Code samples are run before publishing; reference docs are written from the source code.
- **Docs as code.** Markdown, Git and pull requests, with [automated checks](.github/workflows/docs-check.yml) for Markdown style and broken links on every change.

## Skills demonstrated

| Skill | Where to see it |
|---|---|
| Explaining complex AI concepts in plain language | [Concept explainers](#concept-explainers) |
| Step-by-step tutorials with working code | [Tutorials](#tutorials) |
| Task-focused developer guides | [How-to guides](#how-to-guides) |
| REST API reference documentation | [API reference](samples/reference/chat-with-docs-api.md) |
| Troubleshooting and error documentation | [Troubleshooting](samples/troubleshooting/llm-api-common-errors.md) |
| Editing and rewriting existing docs | [Before & After](samples/editing/documentation-rewrite-before-after.md) |
| Release communication | [Release notes](samples/editing/chat-with-docs-release-notes.md) |
| Diagrams as code (Mermaid) | [How RAG Works](samples/concepts/how-rag-works.md), [AI agents](samples/concepts/ai-agents-explained.md) |
| Docs tooling: Markdown, Git, GitHub Actions, markdownlint | [Workflow](.github/workflows/docs-check.yml), [lint config](.markdownlint-cli2.jsonc) |

## Repository layout

```text
samples/
  concepts/          explainers for beginners
  tutorials/         learn-by-building guides
  how-to/            task-focused guides
  reference/         API reference
  troubleshooting/   error and fix guides
  editing/           rewrite sample and release notes
STYLE-GUIDE.md       the rules every sample follows
.github/workflows/   automated lint and link checks
```

## About me

Software Developer (MCA) based in Shillong, India. I build AI applications in Python and write the documentation that makes them usable. Because I build what I document, I can read source code, run every example, and explain the "why" as well as the "how".

- 💻 GitHub: [github.com/gideonswer](https://github.com/gideonswer)
- 🧪 Featured project: [Chat with Docs](https://github.com/gideonswer/chat-with-docs), a RAG platform with page citations and built-in evaluation
