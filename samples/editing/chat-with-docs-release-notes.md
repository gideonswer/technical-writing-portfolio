# Chat with Docs 1.0 Release Notes

> **Type:** Release notes · **Audience:** People using or self-hosting the app · **Reading time:** 3 minutes

**Released:** November 2026

Chat with Docs 1.0 is the first public release. Upload PDFs, ask questions about them, and get answers that point to the exact page they came from. You can also measure how accurate those answers are.

## Highlights

### Every answer shows its sources

Answers include numbered citations like **[1]**. Click one to see the file, page number and the exact passage the answer used, so you can check any claim in seconds.

If the answer isn't in your documents, Chat with Docs says *"I couldn't find that in the uploaded documents"* instead of guessing.

### Private workspaces

Create an account and organise documents into separate workspaces, for example one for HR policies and one for product manuals. Each question is answered only from the workspace you ask in, and other users can never see your documents.

### Find out how accurate your answers are

The new **Evaluation** page lets you run a set of test questions with known answers and get a scored report:

- Did search find the right page?
- Did the answer cite the right page?
- Was the answer correct? (graded by the AI model, 1–5)
- How fast was it?

Results are saved, so you can change a setting and compare runs.

### Keeps working when the AI service doesn't

Chat with Docs uses Google Gemini by default. If Gemini is unavailable or you reach its daily free limit, it switches automatically to a backup model from Groq. Each answer shows which model wrote it.

## For self-hosters

- **Runs on free tiers.** Search runs on your own machine at no cost. Gemini and Groq both have free plans.
- **Change the AI model in one line.** Set `LLM_PROVIDER` and `LLM_MODEL` in `.env` to use Gemini, Groq, Claude or a local model through Ollama. No code changes needed.
- **Tune search without code.** `CHUNK_SIZE`, `CHUNK_OVERLAP` and `TOP_K` are `.env` settings. Change one, then re-run your evaluation to see whether it helped.
- **Tested.** 22 automated tests run offline, with no API keys needed.
- **API included.** Everything in the interface is available through a REST API. See the [API reference](../reference/chat-with-docs-api.md).

## Known limitations

- **Scanned PDFs aren't supported.** Files made of page images have no text to read. Run OCR on them before uploading.
- **Large uploads hold the request open.** A long PDF is processed before the upload request finishes, which can take a while. Background processing is planned.
- **The first upload is slow.** The search model (about 90 MB) downloads the first time it's used.
- **PDF only.** Word, text and web pages aren't supported yet.

## Upgrade notes

This is the first release, so there's nothing to migrate. Before exposing the app to other people, **set a long random `JWT_SECRET`** in `.env`. The server logs a warning at startup if you haven't.

## What's next

We're exploring:

- Combined keyword and meaning-based search, for better results on names and codes
- Answers that appear word by word as they're written
- Docker images for easier deployment
- OCR support for scanned PDFs

Have feedback or found a bug? [Open an issue on GitHub](https://github.com/gideonswer/chat-with-docs/issues).

---

<details>
<summary><strong>Writer's notes</strong>: decisions behind this piece</summary>

- **Benefits, not implementation.** The README says "JWT auth with bcrypt"; release notes for users say "other users can never see your documents". Technical details are kept to the self-hoster section, where that audience wants them.
- **Two audiences, two sections.** End users read Highlights; people running the server read "For self-hosters" and "Upgrade notes".
- **Limitations are stated plainly**, each with a workaround where one exists. Users who find a limit themselves trust the product less than users who were told up front.
- **"What's next" uses "exploring", not dates**, so the notes don't make promises the project can't keep.
- **Accurate to the code:** every feature listed exists in the 1.0 source.

</details>
