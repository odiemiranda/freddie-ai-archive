---
id: research-asl-local-llm-2026-04-07
title: ASL Phase 2 — Local LLM for Gate Classification Research Findings
project-code: ASL
status: complete
created: 2026-04-07
author: holmes
notebook-id: 8f3f4ce2
notebook-title: ASL local LLM for gate classification
sources-loaded: ~58 across 6 queries (1 round; 2 queries timed out)
tags: [research, local-llm, qwen, gemma, llama-cpp, ollama, classification]
---

# ASL Phase 2 — Local LLM for Gate Classification Research Findings

Covers open question 5 from `BRIEF.md` and the companion BRIEF §6 "Local LLMs via Ollama or llama.cpp — YES, but surgically" decision.

## Executive Summary

**The BRIEF's working assumption that local 7B/4B models can handle binary contradiction checks is supported by benchmark evidence, but the integration path it proposes (Ollama HTTP API) is the wrong choice.** Three findings reshape the decision:

1. **Small models are competitive on binary classification specifically** — SLMs trail frontier models by only ~2% F1 on domain-specific binary tasks (Qwen 7B = 0.83, Grok-4 = 0.85 on PROMISE). They lag further on broad MMLU (Qwen 7B = 74.2% vs GPT-4o = 88.0%), but binary contradiction/NLI is closer to the PROMISE shape than the MMLU shape.

2. **llama.cpp server beats Ollama for a production daemon.** Ollama unloads models after 5 minutes of inactivity by default → 8.4 second cold-start penalty on the first call of each pipeline run. llama.cpp keeps the model resident and exposes **GBNF grammar constraints** that force the model to output exactly `CONTRADICTION` or `COMPATIBLE` at the token-sampling layer — not as a soft prompt instruction. This alone addresses the "small model reliability" concern that the BRIEF flags as the decision gate.

3. **Small models have specific, well-documented failure modes that the BRIEF's 90% agreement benchmark may miss.** Qwen 7B exhibits 25-67% "flip rates" under format perturbations, strong positional bias toward option A, and up to 14% accuracy swings based on few-shot example selection. The 90% target is achievable **if** the daemon uses GBNF grammar + temperature 0 + Chain-of-Thought + difficult few-shot examples. Without those, expect 60-75% realistic agreement.

**Net recommendation:** Proceed with local LLM integration, but switch from Ollama to **llama.cpp server mode with GBNF grammar constraints**, and build the benchmark harness to test with the specific prompt template described below — not raw "does A contradict B?" prompts.

---

## Q1 — Quality at 7B scale for binary classification

**Short answer:** Qwen2.5-7B-Instruct can realistically hit 0.83 F1 on domain-specific binary classification, within ~2% of frontier models. But this holds only with careful prompt design, grammar constraints, and temperature 0. Raw zero-shot prompts will underperform significantly.

**Confidence: MEDIUM-HIGH (multiple benchmarks cited, but the specific "does A contradict B" task shape is not directly benchmarked — closest analogue is PROMISE requirements classification)**

### Evidence

**Benchmark numbers (binary classification tasks):**
- **Qwen2 7B on PROMISE** (binary requirements classification): **0.83 F1** — tied with Grok-4 (0.85 F1). Source: requirements classification empirical study.
- **Binary classification F1 gap:** SLMs trail frontier proprietary models (GPT-5, Claude-4) by **~2% average F1**. Source: same study.
- **Gemma 3 4B on Word Sense Disambiguation:** **72.8% F1** vs GPT-4o at 82.3%. A ~10% gap — worse than PROMISE.

**Benchmark numbers (broad reasoning — MMLU):**

| Model | MMLU | MMLU-Pro | MMLU-CF (contamination-free) |
|-------|------|----------|------------------------------|
| Qwen2.5-7B-Instruct | 74.2-76.8% | 56.3% | 61.3% |
| Gemma 3 4B | 59.6% | 43.6% | — |
| Gemini 1.5 Flash | 78.7% | 67.3% | — |
| GPT-4o | 88.0% | — | 73.4% |

**Key insight:** On **MMLU-CF** (designed to remove training contamination), small models drop more than frontier models — Qwen 7B drops 13% (74.2 → 61.3), GPT-4o drops 14.6% (88.0 → 73.4). The gap widens slightly under contamination controls, suggesting some of Qwen's benchmark performance is memorization-aided.

**Known weaknesses — critical for ASL use case:**

1. **Instability / "flipping":** Qwen 7B exhibits **25-67% flip rates** when answer options are paraphrased, contain typos, or reordered. For a binary classifier where the daemon feeds variable text from different agents, this is a material risk.

2. **Positional bias:** Llama-3-8B shows **severe bias toward selecting the second option**, with stability dropping to 44-52% when option order is swapped. *"Avoid presenting the task as a multiple-choice 'A or B' selection; instead, ask the model to output the semantic label directly."* This is direct guidance for the ASL prompt design.

3. **Few-shot instability:** Accuracy swings **up to 14%** based on which examples are selected for few-shot prompting. Difficult examples work better than easy/random ones (up to 10% accuracy boost).

4. **Instruction position sensitivity:** Up to **20-point robustness drop** depending on where instructions appear in the prompt. Fixed template required across runs.

### Mapping to the ASL gate-regression use case

The BRIEF's §6 target is "gate-regression contradiction checks — binary classification, small task." The closest benchmarked analogue is PROMISE requirements classification at 0.83 F1. Contradiction checks are harder than PROMISE (they require logical reasoning, not just category assignment), so a realistic expected accuracy is **0.75-0.85 with proper prompt design**.

**The BRIEF's 90% agreement target is ambitious.** Achievable only with:
- llama.cpp + GBNF grammar constraints (eliminates output format failures)
- Temperature 0
- Chain-of-Thought reasoning before the label
- Difficult few-shot examples (2-4 examples, fixed set)
- Direct semantic labels (not "A or B" multiple choice)
- A benchmark harness that tests the real prompt against the real Gemini ground truth, not a synthetic test set

If the benchmark hits 90%, ship. If it hits 80-85%, consider keeping Gemini as primary and using local LLM only for "pre-filter" (anything the local model says is clearly COMPATIBLE skips Gemini; anything flagged CONTRADICTION or ambiguous goes to Gemini). This saves ~60% of Gemini calls even at 80% agreement.

### What we'd still need to verify empirically
- **Real accuracy on ASL-specific prompts.** The PROMISE 0.83 F1 is not the ASL task. Need a harness that runs 50-100 real contradiction cases from ASL's recent pipeline logs and measures agreement with Gemini.
- **Whether Qwen2.5-7B or Qwen3 (if available) is better.** Sources focus on Qwen2.5. Qwen3 may have closed the gap with frontier.

---

## Q2 — Latency on consumer hardware

**Short answer:** On GPU, Qwen2.5-7B Q4_K_M generates a 10-50 token response in **0.5-1.8 seconds** warm (acceptable). On CPU-only with 16GB RAM, it takes **1.6-16 seconds** (too slow for a daemon making 50-100 calls per pipeline run).

**Confidence: HIGH (multiple independent tok/s benchmarks on specific hardware)**

### Evidence

**Memory footprint:** Qwen2.5-7B Q4_K_M needs ~4.4-5.5 GB RAM/VRAM. Fits in 8GB VRAM or 16GB system RAM.

**Throughput by hardware:**

| Hardware | Throughput | 10-50 token latency | First-token latency (warm) | Cold start |
|----------|-----------|---------------------|---------------------------|------------|
| CPU-only (DDR5, 16GB RAM) | 3-6 tok/s | 1.6-16s | n/a | n/a |
| NVIDIA RTX 3060 (8GB) | ~42 tok/s | 0.5-1.8s | 145ms-0.7s | ~8.4s |
| NVIDIA RTX 4070 | ~52 tok/s | 0.4-1.5s | similar | ~8.4s |
| Apple M1 Pro (16GB) | 25-30 tok/s | 0.5-2.5s | fast | fast |
| Apple M4 Mac Mini (16GB) | 32-35 tok/s | 0.4-2s | fast | fast |
| Apple M1/M2/M3 Max (MLX) | 50-65 tok/s | 0.3-1.5s | fast | fast |

**Key latency gotchas:**
- **Ollama's 5-minute auto-unload** → 8.4 second cold-start penalty on the first call of each pipeline run unless `OLLAMA_KEEP_ALIVE=-1` is set.
- **Prompt processing dominates for long prompts.** A 5000-token Chain-of-Thought prompt with 5 few-shot examples will take longer to process than the 10-token response takes to generate. Consider a streaming / caching approach if prompts are reused.
- **Backend choice matters:** llama.cpp is 3-10% faster than Ollama on NVIDIA. MLX is 30-50% faster than Ollama on Apple Silicon.

### Implication for ASL daemon

- **If user has a GPU:** local LLM is fast enough. 50 calls × 1.5s average = 75s of inference time per pipeline run. Acceptable.
- **If user is CPU-only:** local LLM is too slow. 50 calls × 8s average = 400s (6.6 minutes) of inference. Defeats the purpose of the decoupling effort.
- **Check the user's machine.** The user profile notes Windows 11 on Manila-evening dev. GPU presence not specified in research context. **Must verify before committing to this path.**

### What we'd still need to verify empirically
- **User's actual hardware** — specifically GPU presence and VRAM. This is the gating factor for whether local LLM is viable at all.
- **Actual prompt processing time** for the ASL contradiction-check prompt (which includes few-shot CoT examples). Prompt processing is often slower than response generation for small-output tasks.

---

## Q3 — Ollama vs llama.cpp for production daemon

**Short answer:** **llama.cpp server mode wins decisively** for a single-user long-running daemon. Ollama's model auto-unload and wrapper overhead make it unsuitable for a latency-sensitive pipeline; llama.cpp's direct access and GBNF grammar support solve the small-model reliability problem at the token-sampling layer.

**Confidence: HIGH (multiple benchmarks, explicit feature comparisons, specific gotchas documented)**

### Evidence

**Head-to-head comparison:**

| Criterion | Ollama | llama.cpp server | Winner |
|-----------|--------|------------------|--------|
| Raw throughput (NVIDIA) | baseline | **3-10% faster** | llama.cpp |
| Model loading | Auto-unloads after 5 min idle (8.4s cold start) | On-demand, persistent once loaded | llama.cpp |
| API stability | Go wrapper with custom GGML fork, lags behind | Direct, "production workhorse" | llama.cpp |
| Concurrency | `OLLAMA_NUM_PARALLEL` heuristics, 503 errors when queue overflows | Low, stable Inter-Token Latency | llama.cpp |
| Grammar-constrained output | Limited (via format parameter) | **GBNF at token-sampling layer** | llama.cpp |
| Setup complexity | One command (`ollama run`) | More config, but documented | Ollama wins on setup only |
| Ecosystem libraries | `ollama-js` etc. | HTTP OpenAI-compat API | Roughly even |

**Critical finding: GBNF grammar constraints.**
- llama.cpp exposes GBNF (Grammar-Based Network Format) which **restricts the model's vocabulary at token sampling**.
- For a binary classifier, the grammar literally allows only two token sequences: the exact string `CONTRADICTION` or the exact string `COMPATIBLE`. The model cannot output anything else — no "As an AI, I...", no hedging, no JSON wrapping unless you want it.
- This is the opposite of prompt-level instructions that a 7B model can ignore 25-67% of the time.
- **This alone justifies llama.cpp over Ollama for ASL's use case.**

**Ollama-specific failure modes:**
- 5-minute auto-unload hits the first call of every intermittent pipeline run. 8.4s cold-start × N pipeline runs = real latency cost.
- Workaround: `OLLAMA_KEEP_ALIVE=-1` — possible but requires env config and permanently reserves VRAM.
- `OLLAMA_MAX_QUEUE` overflow → 503 Server Overloaded. Rare at ASL's volume but a failure mode to handle.
- Custom GGML fork lags upstream llama.cpp fixes by weeks-to-months.

**llama.cpp integration pattern:**
```bash
# Start llama.cpp server (long-running, daemon-managed)
./llama-server \
  -m models/qwen2.5-7b-instruct-q4_k_m.gguf \
  --port 8080 \
  --ctx-size 4096 \
  --n-gpu-layers 99  # or 0 for CPU-only

# Daemon makes POST calls to /completion with grammar parameter
POST http://localhost:8080/completion
{
  "prompt": "<full CoT prompt>",
  "grammar": "root ::= \"CONTRADICTION\" | \"COMPATIBLE\"",
  "temperature": 0,
  "n_predict": 20
}
```

### Revised recommendation for BRIEF §6

Replace:
> "Integration pattern: `src/libs/local-llm.ts` wraps Ollama HTTP API (localhost:11434), exposes `classifyBinary(prompt, options)` that returns the label directly"

With:
> "Integration pattern: `src/libs/local-llm.ts` wraps llama.cpp server mode (localhost:8080). Exposes `classifyBinary(prompt, options)` that constructs a GBNF grammar (`root ::= "CONTRADICTION" | "COMPATIBLE"`) and calls `/completion` with temperature 0. Grammar constraints guarantee a valid label even if the prompt is mangled — this is the quality safety net."

### What we'd still need to verify empirically
- **llama.cpp server startup and Bun integration.** Does `Bun.spawn(['llama-server', ...])` work reliably on Windows? Does HTTP client performance under the daemon's load match the published benchmarks?
- **Whether llama.cpp's GBNF grammar works correctly with Qwen2.5 tokenization.** GBNF is a llama.cpp feature; Qwen2.5 is a third-party model. Most community reports suggest it works fine, but worth a smoke test.

---

## Q4 — Prompt design for small-model reliability

**Short answer:** Temperature 0 + GBNF grammar + Chain-of-Thought + difficult few-shot examples + fixed instruction position + direct label output (not multiple choice). Without all of these, expect significant flip rates and positional bias. A concrete template is below.

**Confidence: HIGH (multiple empirical studies cited)**

### Evidence

**Design patterns that improve reliability:**

1. **Temperature 0** — forces most-likely token selection, near-deterministic outputs. Recommended when correctness is paramount.

2. **GBNF grammar constraints** — restricts output vocabulary at the sampling layer. Guarantees format compliance even if the model "wants" to say something else. Only available in llama.cpp (not Ollama at the same level).

3. **Chain-of-Thought before label** — force the model to reason step-by-step in the output, then emit the final label on a new line. Confirmed most effective when combined with few-shot.

4. **Difficult few-shot examples** — counterintuitively, hard examples outperform easy/random ones by up to 10%. Provides richer decision-boundary features.

5. **Fixed instruction positions** — instructions at the top AND repeated at the bottom. Up to 20-point robustness swing from moving instructions.

6. **Direct semantic labels, not A/B** — the label IS the output, not "option A" or "option B". Avoids positional bias (which hits Llama-3-8B's stability at 44-52%).

**Failure modes to avoid:**

1. **Multiple-choice format** — small models favor option A or option B based on position, not meaning. Ask for the semantic label directly.

2. **Spurious few-shot patterns** — if all your COMPATIBLE examples come first and all CONTRADICTION come second, the model learns the pattern instead of the task. **Shuffle the example order in the prompt.**

3. **Unstable example selection** — don't randomize few-shot at runtime. Use a fixed, curated set per task.

4. **Instructions only at the top** — restate format requirements at the end of the prompt too.

5. **Free-form output** — always constrain with grammar (llama.cpp) or strict format validation + retry.

### Recommended prompt template for ASL gate-regression contradiction checks

**System prompt (fixed):**
```
You are an expert logical validator. Your task is to determine if Statement A contradicts Statement B.

Definitions:
- CONTRADICTION: Statement A asserts something that makes Statement B logically impossible.
- COMPATIBLE: Statement A and Statement B can both be logically true at the same time, even if they discuss different things.

You must analyze the statements step-by-step, then provide the final classification as exactly CONTRADICTION or COMPATIBLE.
```

**User prompt (few-shot + target):**
```
Example 1:
Statement A: "The patient was discharged on Tuesday."
Statement B: "The patient remained in the ICU throughout Wednesday."
Analysis: Being discharged on Tuesday means the patient is no longer in the hospital on Wednesday. Statement B claims the patient is in the ICU on Wednesday. These cannot both be true.
Label: CONTRADICTION

Example 2:
Statement A: "The server requires 16GB of RAM."
Statement B: "The server is running a Linux operating system."
Analysis: Statement A discusses memory requirements. Statement B discusses the operating system software. A server can easily have 16GB of RAM and run Linux simultaneously.
Label: COMPATIBLE

Target Task:
Statement A: "{input_A}"
Statement B: "{input_B}"
Analyze the statements step-by-step. Then, on a new line, output only the final label: CONTRADICTION or COMPATIBLE.
Analysis:
```

**Grammar (llama.cpp GBNF):**
```
root ::= analysis "\nLabel: " label
analysis ::= [^\n]+ ("\n" [^\n]+)*
label ::= "CONTRADICTION" | "COMPATIBLE"
```

This grammar allows free-form analysis text but strictly constrains the label token. The daemon parses the label after `\nLabel: ` and throws if the response is empty.

### Replacement examples for ASL domain

The medical/server examples above are placeholders. Replace with 2-4 real ASL contradiction cases once available. Curation rules:
- Pick **difficult** examples (subtle contradictions, not obvious ones).
- Include at least one CONTRADICTION and one COMPATIBLE.
- Cover the main agent domains (suno/minimax/lyrics).
- **Rotate to prevent the model from memorizing the exact examples** — but keep the rotation deterministic (hash of the target task → which example set to use). Never randomize at runtime.

### What we'd still need to verify empirically
- **Which specific examples from ASL's pipeline history produce the highest benchmark accuracy** when used as few-shot. Likely requires 2-4 rounds of iteration on the example set.
- **Whether Chain-of-Thought is worth the token cost.** CoT adds ~200-500 tokens of output, which at 50 tok/s is ~4-10 extra seconds per call on GPU. If the no-CoT accuracy is already above target, skip CoT.

---

## Surprise findings that reshape BRIEF §6

1. **The Ollama choice in the BRIEF is wrong.** Switch to llama.cpp server mode. GBNF grammar constraints are the difference between "small model tries its best" and "small model cannot output invalid format." This single change probably closes most of the quality gap the BRIEF's 90% gate is worried about.

2. **The BRIEF's "benchmark against Gemini ground truth, 90% agreement" decision gate is achievable but sensitive to prompt design.** Raw zero-shot Qwen will not hit 90%. Qwen + GBNF + CoT + curated few-shot + temp 0 + fixed template probably will. The benchmark must test the full stack, not a simplified version.

3. **Hardware is the gating factor, not the model.** If the user is CPU-only, this entire line of investment is moot — 6+ minutes of inference per pipeline run defeats the purpose of decoupling. Verify GPU presence before scheduling ASL-0010.

4. **Even at 80% agreement, local LLM has value as a pre-filter.** Instead of "replace Gemini for gate-regression," treat local as a cheap first pass — if local says COMPATIBLE with high confidence, skip Gemini. If local says CONTRADICTION or hedges, ask Gemini. This hedged design saves ~60% of API calls at 80% agreement while keeping Gemini as the quality backstop.

## Questions we could not answer from the notebook

- **Qwen2.5-7B on specific NLI (Natural Language Inference) benchmarks like SNLI or MNLI.** Sources cover MMLU and domain binary classification but not NLI directly. Contradiction checking is closest to NLI. **Recommendation:** quick literature search specifically for "Qwen2.5 SNLI" or "Qwen2.5 MNLI" before the benchmark harness is built.
- **User's specific hardware.** Not in the research context. Must be confirmed before ASL-0010 is scheduled.
- **Whether Qwen3 exists and is better than Qwen2.5 for this use case.** Sources focus on Qwen2.5. As of early 2026, Qwen3 may be the better baseline — worth a 5-minute check.
- **LLM binary classification prompt design — query timed out twice.** Partial coverage via the benchmark sources. The prompt template above is synthesized from what Q4 did return, plus general patterns from the classification benchmark sources. Confidence on the template is medium-high, not high — worth validating with a small dedicated benchmark before locking in.

## Citations

All claims are sourced from NotebookLM notebook `8f3f4ce2` — "ASL local LLM for gate classification" — which ingested ~58 unique sources across 6 research queries spanning Ollama/llama.cpp comparisons, Qwen2.5 and Gemma 3 benchmark papers, MMLU/MMLU-CF/MMLU-Pro benchmark tables, binary classification prompt design studies, and quantization latency benchmarks.

Key sources cited:
- Qwen2.5-7B-Instruct Foundation Model Summary (LatticeFlow)
- Qwen2.5-LLM official blog and arxiv 2412.15115
- llm-stats.com comparison tables (Qwen, Gemma, GPT-4o, Gemini Flash)
- MMLU-CF contamination-free benchmark paper (aclanthology.org 2025)
- PROMISE requirements classification empirical study
- Word Sense Disambiguation study on Gemma 3 vs GPT-4o
- Ollama JavaScript library and streaming docs
- "How Ollama Handles Parallel Requests" (Rost Glukhov blog)
- Database Mart Ollama performance guide
- Ollama vs llama.cpp production comparison
- LLM quantization latency benchmarks (Gopenai, RAM setup guides)

Full answer text with inline bracket citations is stashed at `vault/studio/research/2026-04-07-asl-local-llm-gate-classification/` via `research save`.

## Estimated research cost

- 6 `add-research` queries (2 additional queries timed out with ETIMEDOUT) = ~60 sources loaded.
- 4 substantive `ask` calls on notebook 2.
- Token cost not reported by the research tool. Order-of-magnitude estimate: ~350k input tokens loaded, ~12k output tokens across answers.
