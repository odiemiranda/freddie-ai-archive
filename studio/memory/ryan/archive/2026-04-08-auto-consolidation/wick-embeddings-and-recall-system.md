---
name: Wick's Embeddings and Recall System
description: Describes Wick's updated embedding infrastructure using local ONNX models and the new `recall` tool for query classification and full-text search.
type: knowledge
agent: wick
tags: [embeddings, onnx, transformers, recall, search, query-classification, information-retrieval, brain]
---

## Knowledge
Wick has undergone a significant upgrade to its embedding infrastructure (OT-0015). The `@huggingface/transformers@4.0.0` library has been installed, and the previous `embeddings.ts` module has been replaced with a local ONNX model (`all-MiniLM-L6-v2`), reducing embedding dimensions from 768 to 384. A migration guard in `brain/index.ts` ensures a smooth transition for `vec0` tables. The `generateEmbedding` function has been removed from `brain.ts`, with all call sites migrated to use the `embedText` function.

A new tool, `src/tools/recall.ts`, has been created, providing functionalities for `classifyQuery`, `checkFts`, and `recall`, accessible via a CLI entry point. This system enhances Wick's ability to process and retrieve information based on semantic understanding.

## Why
This upgrade improves efficiency and performance of embedding generation by using a local, smaller ONNX model, reducing computational overhead. The new `recall` tool, combined with the enhanced embedding system, significantly boosts Wick's ability to understand and respond to queries, enabling more sophisticated information retrieval and knowledge application.

## How to apply
Leverage the `recall` tool for advanced query processing, including classifying queries, checking full-text search (FTS) results, and retrieving relevant information based on semantic embeddings. The underlying embedding system automatically handles the generation of 384-dimensional embeddings for various text inputs, supporting more efficient and nuanced data analysis within Wick's operations.

## Consolidated from
- `raw-log-20260401-233116-note.md`
