# How to Measure Whether Your RAG Answers Are Accurate

> **Type:** How-to guide · **Audience:** Developers building RAG apps · **Time:** 1 hour to set up, minutes per run after that

"It looked right when I tried it" isn't a measurement. This guide shows you how to build a small, repeatable test that scores your RAG app, so you can tell whether a change, such as a new chunk size, prompt or model, made it better or worse.

You'll end up with a report like this one, from a 7-question test set run against Chat with Docs:

| Metric | Score |
|---|---|
| Retrieval hit rate | 100% |
| Citation hit rate | 100% |
| LLM-judge pass rate (score ≥ 4/5) | 100% |
| Correctly refused the unanswerable question | ✅ |

Perfect scores on a small, easy test set mean "no obvious problems yet", not "finished". [Next steps](#next-steps) explains how to make the test harder.

**Before you start,** your RAG app should return the answer, the retrieved sources (with file and page), and the source numbers it cited. If it doesn't cite pages yet, see [How to Add Page Citations to a RAG App](add-page-citations-rag.md).

The code is from [Chat with Docs](https://github.com/gideonswer/chat-with-docs/blob/main/app/evals.py) and is written in Python.

## 1. Write a test set

A **test set** (or eval set) is a list of questions where you already know the right answer and where it is in the document. Write it by hand, reading the document yourself.

```json
[
  {"question": "How many days do customers have to request a refund?",
   "expected_answer": "Within 30 days of purchase",
   "expected_filename": "acme_policy.pdf", "expected_page": 2},

  {"question": "How much does express shipping cost?",
   "expected_answer": "Rs 149, and it arrives in 2 days",
   "expected_filename": "acme_policy.pdf", "expected_page": 3},

  {"question": "Does Acme sell gift cards?",
   "expected_answer": "The documents don't say",
   "expected_filename": null, "expected_page": null}
]
```

Guidelines for a useful test set:

- **Start with 10–20 questions.** That's enough to catch big problems. Add more as you find failures.
- **Cover different pages and question types:** facts, numbers, yes/no questions and questions that need two passages.
- **Include questions the document can't answer.** Set `expected_page` to `null`. These test whether the app makes things up.
- **Write some questions in different words from the document.** If the document says "reimbursement", ask about "money back".

## 2. Measure retrieval: did search find the right page?

If search doesn't find the right passage, the model can't answer correctly, however good it is. Measure retrieval separately, so you know where a failure comes from.

For each question, check whether the expected page appears in the retrieved results, and at which position:

```python
rank = next(
    (i for i, h in enumerate(hits, start=1)
     if h.chunk.page == case.expected_page
     and h.chunk.filename.lower() == case.expected_filename.lower()),
    None,
)
retrieval_hit = rank is not None
reciprocal_rank = 1 / rank if rank else 0.0
```

| Metric | Meaning | Good target |
|---|---|---|
| **Retrieval hit rate** | Share of questions where the right page was retrieved at all | As close to 100% as possible |
| **MRR** (mean reciprocal rank) | How high the right page ranked. 1.0 means always first; 0.5 means usually second | Higher is better |

## 3. Measure citations: did the answer point to the right page?

Retrieval can succeed while the answer still cites the wrong source. Check the pages the answer **actually cited**:

```python
citation_hit = any(
    hits[r - 1].chunk.page == case.expected_page for r in cited_refs
)
cites_sources = bool(cited_refs)
```

| Metric | What it catches |
|---|---|
| **Citation hit rate** | Answers that cite a source, but the wrong one |
| **Answers with citations** | Answers written without using any source, often a sign of made-up content |

## 4. Measure the answer itself

Two complementary checks:

### Token F1: free and repeatable

Token F1 measures word overlap between the answer and the expected answer, from 0 to 1. It costs nothing and gives the same result every time, but it penalises correct answers that use different words.

```python
from collections import Counter

def token_f1(prediction: str, reference: str) -> float:
    pred, ref = normalize(prediction), normalize(reference)
    common = Counter(pred) & Counter(ref)
    same = sum(common.values())
    if same == 0:
        return 0.0
    precision, recall = same / len(pred), same / len(ref)
    return 2 * precision * recall / (precision + recall)
```

`normalize` lowercases the text and removes punctuation, citation markers and words like "the".

### LLM judge: closer to human judgement

Ask a language model to grade each answer against the expected one:

```text
You grade answers from a document Q&A system.
Compare the CANDIDATE answer to the REFERENCE answer for the QUESTION.
Score 1-5: 5 = fully correct and complete, 4 = correct with minor omissions,
3 = partially correct, 2 = mostly wrong, 1 = wrong or no answer.
Reply with JSON only: {"score": <1-5>, "reason": "<one sentence>"}
```

Count a score of 4 or 5 as a pass, and report the **pass rate**.

> ⚠️ **Don't trust the judge alone.** In my own testing, the judge gave 5/5 to an answer that left out part of the expected answer. LLM judges tend to be lenient. Read the judge's reasons for a sample of answers, and always look at the judge score alongside the other metrics.

## 5. Check refusals

For questions with no answer in the documents, the correct behaviour is to say so. If your system prompt tells the model to reply with an exact sentence when it can't find the answer, you can check this precisely:

```python
refused = answer.strip() == "I couldn't find that in the uploaded documents."
```

Every unanswerable question should be refused. Every answerable question should **not** be.

## 6. Run, record and compare

Run every question in the test set, save the results with the date, model and settings, and compare runs:

| Run | Change | Retrieval hit | Citation hit | Judge pass |
|---|---|---|---|---|
| 1 | Baseline: chunk 900, top-k 5 | 86% | 71% | 86% |
| 2 | Chunk size 500 | 100% | 86% | 86% |
| 3 | Chunk 500, top-k 8 | 100% | 100% | 100% |

*Example figures, for illustration.*

Change **one setting per run**. If you change the chunk size and the model at the same time, you won't know which one helped.

## Reading the results

| Pattern | What it means | Where to look |
|---|---|---|
| Low retrieval hit rate | Search isn't finding the right passages | Chunk size, overlap, top-k, embedding model, add keyword search |
| High retrieval, low citation hit | The right text is found, but the model uses or cites the wrong one | Prompt rules, temperature, number of sources |
| Good citations, low judge score | Right page, poor answer | Prompt wording, model choice, chunks cutting off key sentences |
| Unanswerable questions get answers | The model is guessing | Strengthen the "only use the sources" rule; check the refusal sentence |

## Next steps

- Grow the test set every time a user reports a wrong answer. Each real failure is a free test case.
- Use a larger, harder document (30+ pages) before trusting the scores.
- See the full implementation, including the scored report page: [Chat with Docs](https://github.com/gideonswer/chat-with-docs)

---

<details>
<summary><strong>Writer's notes</strong>: decisions behind this piece</summary>

- **Separates retrieval from generation.** The most useful thing an eval can tell a developer is *where* the failure is. The "Reading the results" table maps each pattern to a place to look.
- **Includes a real finding.** The lenient-judge warning comes from running this eval myself. First-hand caveats are more convincing, and more useful, than generic ones.
- **Labels illustrative numbers.** The comparison table is marked as example figures so readers don't treat it as a benchmark.
- **Each metric gets "what it catches",** not just a definition, so readers understand why it's worth measuring.

</details>
