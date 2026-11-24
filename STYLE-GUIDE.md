# Style Guide

The rules every sample in this portfolio follows. It's based on the [Google developer documentation style guide](https://developers.google.com/style) and the [Microsoft Writing Style Guide](https://learn.microsoft.com/style-guide/welcome/), adapted for AI and developer-tool docs.

## Voice and tone

- **Talk to the reader as "you".** Use "we" only for the project team.
- **Be direct and friendly, not chatty.** No jokes that need explaining, no exclamation marks in instructions.
- **Don't say something is easy, simple or just.** If it's easy, the reader will notice. If it isn't, those words make them feel at fault.
- **No marketing language** in technical docs: avoid *powerful*, *seamless*, *cutting-edge*, *leverage*.

## Sentences and words

- **One idea per sentence.** Aim for under 25 words.
- **Active voice:** "The server returns a token", not "A token is returned".
- **Present tense:** "The app shows", not "The app will show".
- **Plain words:** *use* not *utilise*; *so that* not *in order to*; *start* not *initiate*.
- **Define a term the first time you use it**, or link to a page that does.
- **Use contractions** (*don't*, *it's*) for a natural tone.

## Structure

Every page states its **type, audience and time** at the top, then follows the pattern for its type:

| Type | Purpose | Structure |
|---|---|---|
| **Concept explainer** | Help the reader understand | Problem → analogy → how it works → limits → summary |
| **Tutorial** | Help a beginner learn by building | Result first → prerequisites → numbered steps with checks → what you built |
| **How-to guide** | Help someone with a specific goal | Goal → assumptions → numbered steps → common problems |
| **Reference** | Help someone look up a fact | Consistent fields for every entry; examples; errors |
| **Troubleshooting** | Help someone fix a problem fast | Symptom index → cause → fix |

These four main types follow the [Diátaxis](https://diataxis.fr/) framework.

## Procedures

- **Number steps** when order matters; use bullets when it doesn't.
- **One action per step.** Put the expected result after the step when it helps the reader confirm they're on track.
- **Give the exact command**, in a code block, ready to copy.
- **Warn before the step**, not after it.
- **Give both commands** when Windows and macOS/Linux differ.

## Formatting

- **Sentence case** for headings: "How it works", not "How It Works".
- **Bold** for UI labels the reader clicks: click **Create API key**.
- `Code font` for file names, commands, settings, field names and values.
- Code blocks always have a **language tag** (`python`, `bash`, `json`, `text`).
- **Tables** for comparisons and anything with two or more attributes per item.
- **Callouts** sparingly: ⚠️ for risk of harm or loss, 💡 for helpful tips, 📌 for notes that may go out of date.

## Code samples

- **Must run.** Test code before publishing; say what was tested in the writer's notes.
- **Never include real secrets.** Use placeholders like `paste-your-key-here` and load keys from environment variables.
- **Keep them minimal:** only what the reader needs for this task.
- **Comment the why**, not the what.

## AI-specific conventions

- **Model names change.** When naming a specific model, add a note pointing to the provider's current model list.
- **Don't overstate reliability.** Say what a technique reduces (for example, made-up answers), not that it eliminates the problem.
- **Mark illustrative numbers** as examples so readers don't treat them as benchmarks.

## Links

- Use descriptive link text: "see [How RAG Works](samples/concepts/how-rag-works.md)", never "click here".
- Use relative links between pages in this repository.
- Every page ends with **Next steps** linking to related pages.
