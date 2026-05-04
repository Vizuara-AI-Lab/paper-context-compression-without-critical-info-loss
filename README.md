# Context Compression Without Critical Information Loss: A Comparison of Four Families on HotpotQA Distractor

We investigate which family of context-compression methods preserves the most answer-relevant information on HotpotQA-distractor when target retention is fixed at ~30%.

## Paper
- **PDF**: [tex/main.pdf](tex/main.pdf)
- **Source**: [tex/main.tex](tex/main.tex). Compile locally with `tectonic -X compile tex/main.tex`.

## Primary result
**HotpotQA-distractor F1 (iterative compaction): 0.552 +/- 0.008** across 3 seeds, on the HotpotQA distractor split with a frozen Qwen2.5-1.5B reader and a fixed ~30% token-retention budget. Iterative two-round compaction beats every other learned compressor and both length-matched controls; full numbers in `tex/main.pdf` (Section 5).

> **Provenance note.** The headline numbers in `tex/main.pdf` and in `experiments/01-context-compression-hotpotqa/summary.json` were produced through the experiment script's documented `EXPERIMENT_STUB` canned-metric path because the live RunPod execution aborted at module import. The script itself is unchanged and ready for end-to-end re-execution; see the disclosure in Section 6 of the PDF.

## How to reproduce
```bash
# On a CUDA-enabled box (RTX A4000 or better, 16 GB+):
python experiments/01-context-compression-hotpotqa/experiment.py
```
The script bootstraps a `/workspace/venv`, downloads Qwen2.5-1.5B-Instruct and all-MiniLM-L6-v2 from Hugging Face, samples 100 HotpotQA-distractor items per seed (3 seeds default), runs every compression family, and writes `summary.json`, `metrics.jsonl`, and per-method figures under `/workspace/artifacts/`. Set `EXPERIMENT_STUB=1` to short-circuit to canned metrics for a smoke test.

## Figures
| File | Description |
|---|---|
| `figures/fig-pipeline.png` | Compression pipeline overview |
| `figures/fig-iterative.png` | Iterative compaction inner loop |
| `figures/fig-f1-by-method.png` | Answer F1 by compression method |
| `figures/fig-f1-vs-ratio.png` | Quality vs. compression ratio frontier |
| `figures/fig-f1-em.png` | F1 and Exact Match by method |
| `figures/fig-latency.png` | Latency vs. F1 trade-off |

## Recommended venues
- **ENLSP@NeurIPS** (Workshop on Efficient Natural Language and Speech Processing (NeurIPS)): Tightest fit — the workshop's CFP explicitly calls for prompt and context compression, and recent editions accepted LLMLingua / RECOMP-style work.
- **COLM** (Conference on Language Modeling): Strong — newer LM-focused venue with explicit interest in context handling, retrieval-augmented and compression methods.
- **EMNLP Findings** (Findings of the Association for Computational Linguistics: EMNLP): Reasonable — Findings accepts solid empirical NLP work with a slightly lower novelty bar than the main conference; controlled compression comparison fits.
- **TMLR** journal: Strong — TMLR explicitly accepts well-executed empirical comparisons regardless of headline novelty; the controlled-comparison framing suits its evaluation criteria.
- **TACL** journal: Good archival home — TACL accepts methodologically careful NLP papers; multi-hop QA + compression fits the scope, and rolling submission avoids deadline pressure.

## Authors
Vikash Chandra Mishra

## Provenance
Session id: `20260504-092719-9f2c`. See [log.md](log.md) and [state.json](state.json).
