# How to Add Page Citations to a RAG App

> **Type:** How-to guide · **Audience:** Developers with a working RAG app · **Time:** 30–45 minutes

This guide shows you how to make your RAG app's answers cite the exact file and page each fact came from, like this:

```text
Sale items can be exchanged but not refunded [1]. Refunds are issued within 30 days [2].

[1] store_policy.pdf, page 2
[2] store_policy.pdf, page 2
```

Citations let users check an answer instead of trusting it, and they give you a way to measure whether the app is answering from the right place.

**This guide assumes** you already have a RAG app that splits documents into chunks, embeds them and retrieves the top matches for a question. If you don't, start with [Build a PDF Q&A App in 30 Minutes](../tutorials/pdf-qa-app-30-minutes.md).

The code samples are in Python and are taken from [Chat with Docs](https://github.com/gideonswer/chat-with-docs), but the approach works with any language, vector store or model.

## Overview

You'll make four changes:

1. [Keep the page number on every chunk](#1-keep-the-page-number-on-every-chunk)
2. [Number the sources in the prompt](#2-number-the-sources-in-the-prompt)
3. [Tell the model how to cite](#3-tell-the-model-how-to-cite)
4. [Parse and check the citations](#4-parse-and-check-the-citations)

## 1. Keep the page number on every chunk

Citations only work if every chunk knows where it came from. The page number has to be attached **when you split the document**, because the information is lost once text from several pages is joined together.

Split each page separately, and store the page number with each chunk:

```python
from dataclasses import dataclass

@dataclass
class Chunk:
    id: str
    text: str
    document_id: int
    filename: str
    page: int          # 1-based, as a human would count

def chunk_pages(pages: list[str], document_id: int, filename: str,
                size: int, overlap: int) -> list[Chunk]:
    chunks = []
    for page_no, raw in enumerate(pages, start=1):
        text = clean(raw)
        if not text:
            continue
        for piece in split_text(text, size, overlap):
            chunks.append(Chunk(id=uuid.uuid4().hex, text=piece,
                                document_id=document_id, filename=filename,
                                page=page_no))
    return chunks
```

Then save the filename and page as **metadata** in your vector store, so they come back with every search result. With ChromaDB:

```python
collection.add(
    ids=[c.id for c in chunks],
    embeddings=embeddings,
    documents=[c.text for c in chunks],
    metadatas=[{"filename": c.filename, "page": c.page,
                "document_id": c.document_id} for c in chunks],
)
```

✅ **Check:** Run a search and print the metadata of the first result. You should see a filename and a page number that match the text.

> 💡 **Use overlap.** Let consecutive chunks share 100–150 characters. A fact that straddles a chunk boundary then appears whole in at least one chunk. Because chunks are made per page, overlap never mixes text from two pages, so each chunk still has exactly one page number.

## 2. Number the sources in the prompt

Give each retrieved chunk a short number, and show the model the file and page next to it:

```python
def build_prompt(question: str, hits) -> str:
    blocks = [
        f"[{i}] (file: {h.chunk.filename}, page {h.chunk.page})\n{h.chunk.text}"
        for i, h in enumerate(hits, start=1)
    ]
    return "Sources:\n\n" + "\n\n".join(blocks) + f"\n\nQuestion: {question}\nAnswer:"
```

The prompt the model receives looks like this:

```text
Sources:

[1] (file: store_policy.pdf, page 2)
Sale items may be exchanged within 14 days but are not eligible for a refund...

[2] (file: store_policy.pdf, page 2)
Refunds are issued to the original payment method within 30 days...

Question: Can I get a refund on a sale item?
Answer:
```

Short numbers like `[1]` are easier for a model to reproduce accurately than full filenames, and easy for your code to find.

## 3. Tell the model how to cite

Put the citation rules in the **system prompt** so they apply to every question:

```python
SYSTEM_PROMPT = """You answer questions using ONLY the numbered sources provided.
Rules:
- Cite every claim with the source number in square brackets, e.g. [1] or [2][3].
- If the sources do not contain the answer, reply exactly: "I couldn't find that in the uploaded documents."
- Do not use outside knowledge. Be concise."""
```

Each rule has a job:

| Rule | Why it's there |
|---|---|
| Use only the numbered sources | Stops the model mixing in facts you can't cite |
| Cite every claim with `[n]` | Gives your code a fixed format to look for |
| Reply with an exact sentence when the answer isn't there | Lets your code detect "not found" reliably, and discourages made-up answers |
| Use a low temperature, such as `0` | Makes the model follow the format more consistently |

## 4. Parse and check the citations

Don't trust the citations blindly. Models occasionally cite a source number that doesn't exist, such as `[7]` when only 5 sources were given. Extract the numbers and keep only valid ones:

```python
import re

def parse_citations(answer: str, n_sources: int) -> list[int]:
    refs = {int(m) for m in re.findall(r"\[(\d+)\]", answer)}
    return sorted(r for r in refs if 1 <= r <= n_sources)
```

Then return the answer, the sources and the cited numbers together, so your interface can show them:

```python
cited = parse_citations(answer, len(hits))
response = {
    "answer": answer,
    "sources": [
        {"ref": i, "filename": h.chunk.filename, "page": h.chunk.page, "text": h.chunk.text}
        for i, h in enumerate(hits, start=1)
    ],
    "cited_refs": cited,
}
```

In the interface, turn each `[n]` in the answer into a link or tooltip that shows `filename, page`, with the exact passage available on click.

> ✅ **Check:** Ask a question your document answers. Every `[n]` in the reply should appear in `cited_refs`, and opening the PDF at the cited page should show the fact.

## Common problems

| Problem | Likely cause | Fix |
|---|---|---|
| Answers have no citations | The rules aren't in the system prompt, or the temperature is high | Move rules to the system prompt; set temperature to `0` |
| Page numbers are off by one | Pages counted from `0` | Use `enumerate(pages, start=1)` |
| Cited page doesn't contain the fact | The fact was in a neighbouring chunk, or the model combined sources | Add overlap; retrieve more chunks; test with an eval set |
| Citations like `[1, 2]` or `(Source 1)` | The model drifted from the format | Show the exact format in the rules (`[1]` or `[2][3]`) and adjust the regex if needed |

## Next steps

- Measure how often citations point to the right page: [How to Measure Whether Your RAG Answers Are Accurate](measure-rag-accuracy.md)
- See the full implementation: [`ingest.py`](https://github.com/gideonswer/chat-with-docs/blob/main/app/ingest.py) and [`rag.py`](https://github.com/gideonswer/chat-with-docs/blob/main/app/rag.py) in Chat with Docs

---

<details>
<summary><strong>Writer's notes</strong>: decisions behind this piece</summary>

- **How-to, not tutorial.** The reader already has a RAG app and one specific goal, so there's no setup or theory, just four changes with a check after the important ones.
- **Explains the "why" of each prompt rule** in a table. Developers copy prompts; knowing the purpose of each rule helps them adapt it instead of breaking it.
- **Real code.** Every snippet comes from a working project, which also means the guide can link to the full source.
- **Show what the model sees.** The rendered prompt example makes the abstract `build_prompt` function concrete.

</details>
