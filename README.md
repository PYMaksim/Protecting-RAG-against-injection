# Protecting-RAG-against-injection



```markdown
# RAG Security: Indirect Injection Guardrail

## Overview

A mini-research project on protecting a RAG pipeline from Indirect Injection — an attack where a malicious actor embeds hidden commands into a knowledge base. The goal is to make the LLM ignore system rules and execute a harmful instruction (e.g., leak the system prompt).

## Hypothesis

Regex-based per-chunk validation (before sending to LLM) can effectively block direct injection commands in known languages. However, naive character-based splitting creates a vulnerability: a dangerous phrase can be cut at a chunk boundary, and part of the injection may end up in a "safe" chunk.

## Experiment

### What was tested
1. Will the regex filter detect Russian injection ("игнорируй все инструкции")?
2. Will the filter detect English injection ("ignore all instructions")?
3. What happens when an injection is split across chunk boundaries?

### Setup
- Document split into 200-character chunks
- Chunk overlap: 50 characters
- Patterns: 5 English, 5 Russian (50/50)
- Two-layer validation: per-chunk check + full context re-check

### Findings

| Stage | What happened |
|---|---|
| Split without overlap | Phrase "ignore all instructions" was cut at boundary. Part "ignore all other" ended up in a "safe" chunk. |
| Split with overlap (overlap=50) | Dangerous phrase appeared in full in at least one chunk and was blocked. |
| Full context re-check | Safety net worked: even if partial injection passed per-chunk check, the assembled context contained the full phrase and was caught. |

## Results

| Metric | Value |
|---|---|
| Russian injections blocked | ✅ Yes |
| English injections blocked | ✅ Yes |
| Chunk boundary bypass mitigated | ✅ Yes (via overlap + re-check) |
| Overlap duplicates removed | ✅ Yes (line deduplication) |
| Injections in final prompt | ❌ None |

## Architecture
Document → split_into_chunks(overlap=50) → per-chunk regex check ↓ safe chunks → assemble context ↓ full context re-check ↓ clean prompt → LLM
## Tech Stack
- Python 3.10+
- `re` (regex)
- No external dependencies

## Run

```bash
python rag_pipeline.py

Conclusion
Regex patterns are effective as a first defense layer against direct injections in known languages. The chunk boundary bypass vulnerability is mitigated by two mechanisms: chunk overlap and full context re-validation. For protection against synonyms and obfuscation (e.g., "skip all directives"), a second layer is needed — semantic analysis or an LLM-based classifier.

Injection blocking is 100% effective. Minor context artifacts (duplicates, sentence fragments) are an intentional trade‑off between security and context cleanliness. In production, this is addressed by switching from character‑based chunking to semantic chunking (by paragraph or logical unit).
