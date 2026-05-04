# Experiment plan v1 — Context Compression Without Critical Information Loss

## Question

When a downstream reader (a small frozen LLM) must answer multi-hop questions
from a 10-paragraph context that contains 2 gold paragraphs and 8 distractors,
which family of context-compression methods preserves the most answer-relevant
information at a fixed compression ratio?

We compare four compression families against three reference baselines.

## Hypothesis

Iterative compaction (per-paragraph summary → query-conditioned compaction)
will dominate the answer-quality vs. compression-ratio frontier at moderate
ratios (~30%), beating extractive selection by ≥3 F1 points and beating
truncation/random baselines by ≥10 F1 points, with absolute F1 within 5
points of the uncompressed full-context upper bound.

## Dataset + preprocessing

- HotpotQA distractor-validation split via `hotpot_qa` on Hugging Face.
- 100 questions sampled deterministically per seed (`random.Random(seed)`).
- Each item ships with 10 paragraphs (titles + sentence lists). We rebuild a
  flat context as `<title>: <sentences>` per paragraph, joined with blank
  lines.
- Gold answer evaluated with SQuAD-style normalised F1 + Exact Match.

## Models

- Reader (frozen, fp16): `Qwen/Qwen2.5-1.5B-Instruct`. Used both as the
  question-answerer and as the LLM compressor for abstractive / structured /
  iterative methods.
- Sentence embedder for extractive selection:
  `sentence-transformers/all-MiniLM-L6-v2`.

A single 1.5B parameter checkpoint keeps everything inside the A4000's 16 GB
budget and removes the confound of comparing different reader strengths.

## Compression methods

| Family       | Operator                                                                                  | Cost / item            |
|--------------|--------------------------------------------------------------------------------------------|------------------------|
| full         | Identity (uncompressed upper bound)                                                       | 1 QA call              |
| truncate     | First-N tokens of concatenated context, N = ⌊ratio · |full|⌋                              | 1 QA call              |
| random       | Random k of K paragraphs at the same ratio (random control)                               | 1 QA call              |
| extractive   | Score each paragraph by max sentence-question cosine sim; keep top-k                      | 1 QA + emb             |
| abstractive  | Single LLM call: "compress to ≤T tokens preserving entities/dates/relations"              | 1 compress + 1 QA      |
| structured   | LLM extracts JSON {entities, relations, facts}; serialised as text                        | 1 compress + 1 QA      |
| iterative    | Round 1: per-paragraph 2-sentence summary; Round 2: query-conditioned compaction          | 1+10 compress + 1 QA   |

Compression ratio target = 0.30 (kept tokens / full-context tokens). The same
ratio is enforced for all baselines; abstractive / structured / iterative use
the ratio as an upper-bound generation cap and may produce shorter outputs.

## Evaluation protocol

- Three random seeds (0, 1, 2) controlling both the example sample and the
  random-baseline draws. Greedy decoding (`temperature=0`) for reader and
  compressors.
- Per method, per item we record: F1, EM, realised compression ratio, and
  end-to-end latency (compress + QA, ms).
- Headline metric reported in the paper: mean ± std of F1 across seeds, per
  method.
- Variance budget: ≥3 seeds, σ reported alongside means, no significance
  tests claimed beyond "within 1σ of …".

## Expected artifacts

- `artifacts/data/summary.json` — primary metric, per-seed records, per-method
  aggregate.
- `artifacts/data/metrics.jsonl` — one line per seed.
- `artifacts/figures/f1_by_method.png` — bar chart of F1 by method.
- `artifacts/figures/f1_vs_ratio.png` — quality–ratio frontier scatter.
- `artifacts/figures/f1_em_by_method.png` — F1 vs EM bars.
- `artifacts/tables/compression_methods.csv` — full numeric table.

## Compute budget

- A4000 16 GB, fp16 reader.
- Per seed: 100 items × 7 methods × ~1 s LLM call ≈ 12 min QA + 8 min
  compression (iterative dominates) → ~20 min/seed.
- 3 seeds → ~60 min/run, comfortably within the 4-hour iteration cap.

## Out of scope (workshop tier)

- No fine-tuning of the reader or compressor.
- No comparison to >1.5B compressors (deferred — single-model isolation is
  the cleaner ablation for this paper).
- No multi-hop chain prompting; the reader sees only the compressed context.
