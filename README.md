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


