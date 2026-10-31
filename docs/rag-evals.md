# Evaluating a RAG answer

A RAG system answers from documents it retrieves. Three things meet in every run: the **question** (Q), the retrieved **context** (C) and the generated **answer** (A). Each pair can fail in two directions, which gives six checks.

<p align="center"><img src="https://benyoumen.github.io/Benyoumen/assets/rag-qca.svg" alt="Six RAG checks between question, context and answer: context relevance, answerability, answer relevance, self-containment, faithfulness and support coverage." width="100%"></p>

`X|Y` reads "check X given Y". For example, `C|Q` checks the context against the question.

## The six checks

| Check | Pair | Pass/Fail question | If it fails, look at |
|---|---|---|---|
| Context relevance | C\|Q | Do the retrieved passages help answer the question? | Retrieval: query, embeddings, filters, ranking |
| Answerability | Q\|C | Can this context answer the question at all? | Missing documents, or the agent should say it does not know |
| Faithfulness | A\|C | Does every claim in the answer stick to the context? | Generation: the prompt lets the model add facts |
| Support coverage | C\|A | Does the context back every claim the answer makes? | Retrieval returned too little, so the model filled the gap |
| Answer relevance | A\|Q | Does the answer address what the user asked? | Generation: the answer drifts or stays generic |
| Self-containment | Q\|A | Could a reader infer the question from the answer alone? | The answer depends on context the user never sees |

## Check in this order

1. **Retrieval first.** Run context relevance and answerability. When retrieval fails, every later check fails too, and the prompt is not the cause.
2. **Grounding next.** Run faithfulness and support coverage. They tell you who invented the claim: the model, or a retriever that left a gap.
3. **Usefulness last.** Run answer relevance and self-containment. An answer can be faithful and still useless.

## Making each check cheap and reliable

- **Use code where you can.** A claim with a required quote can be checked by searching the quote in the context. Missing context can be flagged by an empty retrieval result.
- **One judge per check.** Pass or Fail, a short critique first, and only the inputs that check needs: the question and context for `C|Q`, the answer and context for `A|C`. See [How to write an LLM-as-a-Judge](llm-as-a-judge.md).
- **Validate against people.** Label 50 or more runs by hand before you trust a judge's number.
- **Start with your failure modes.** Read traces first. Build only the checks your errors call for. Faithfulness and context relevance are the usual first two.

## Beyond classic RAG

The same pairs work for any agent that acts on evidence. In a CRM agent that reads a sales call, the call transcript is the context and the proposed field update is the answer. Faithfulness becomes "the quoted lines support each field value". Code checks that each quote exists, and the [example judge](../examples/evidence-supported-judge.md) checks that it supports the value.

## Source

The six checks come from Jason Liu's article [There Are Only 6 RAG Evals](https://jxnl.co/writing/2025/05/19/there-are-only-6-rag-evals/).
