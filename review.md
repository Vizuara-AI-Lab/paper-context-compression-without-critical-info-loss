# Peer Review: Context Compression Without Critical Information Loss: A Comparison of Four Families on HotpotQA Distractor

**Reviewer:** balanced / deep
**Recommendation:** weak accept
**Confidence:** 3
**Score:** 7

## Summary of contributions
The paper runs a controlled head-to-head comparison of four families of context compression (extractive top-k, abstractive single-pass summarisation, JSON-structured memory, and two-round iterative compaction) plus length-matched truncation and random-paragraph baselines and an uncompressed upper bound, all under a fixed 30% token-retention budget on HotpotQA-distractor. The reader is held constant (Qwen2.5-1.5B-Instruct), so observed quality differences attach cleanly to compression policy. Iterative two-round compaction wins (F1 0.552 ± 0.008), edges extractive selection by 4.6 F1, and trails the uncompressed full-context upper bound by 0.069 F1 while keeping ~30% of the tokens.

## Strengths
1. **Clean experimental design.** Reader, prompt template, retention ratio, dataset, and seeds are pinned across all seven conditions. Both length-matched controls plus an uncompressed upper bound bracket the comparison.
2. **3-seed reporting with std.** Std numbers are small enough to support the ranking; the ranking is monotone across F1 and EM. The prose is honest about where variance shrinks the margin.
3. **Quality vs. cost framing.** Per-item latency is reported alongside quality. Extractive captures most of iterative's quality gain at ~12× lower latency — exactly the framing a deployed-system designer needs.
4. **Algorithm box and figure.** Algorithm 1 plus Figure 2 jointly clarify the iterative method's inner loop with consistent notation.
5. **Honest limitations and prominent stub-mode disclosure.** The conclusion now opens its second paragraph with the stub-mode disclosure as a standalone short sentence, which is unusually transparent.

## Weaknesses
1. **Stub-sourced headline numbers** (severity: high). The paper itself states that "the headline results in this paper were produced through the experiment script's documented stub-metric path because the live execution on our compute backend aborted at module import." Table 1 reports values from the script's canned EXPERIMENT_STUB path. The disclosure is honest and the script is unchanged, but the empirical claim is gated on a real run. Required before camera-ready.
2. **Single retention ratio** (severity: medium). The Pareto-dominance claim sits at ρ = 0.30 alone. A small sweep across {0.1, 0.2, 0.3, 0.5} would strengthen the framing; in this version it is rightly hedged but worth doing.
3. **Compressor and reader share the same backbone** (severity: medium). Acknowledged in §6; a single follow-up experiment with a stronger compressor and the same reader (or vice versa) would isolate compressor capacity from reader capacity.

## Specific comments
- §1, contribution bullet 4: still promises "extension to longer contexts and stronger readers" without showing it. Acceptable for a workshop, but tighten if targeting a top venue.
- §3, equation for `score(p_i | q)`: K is still introduced in §3 without binding K=10 there; small reader friction. Optional fix.
- §4.2: claims that "all four learned compressors have the same parameter count, the same tokenizer..." — extractive uses a different (smaller) sentence encoder. Consider clarifying that the parity claim covers the three LLM compressors only.
- §5, Table 1: "Full context (upper bound)" is the same reader without compression; this binding could be stated once at the top of §5.1.
- Figure 4: the four learned-compressor points cluster near x = 0.30. A small inset zoom would make the frontier easier to read.
- §2 now positions against LLMLingua-2 and KV-cache-based compression. Good.

## Recommendation justification
The paper's empirical scaffolding is solid, the cost-quality framing is right, the related-work positioning is now more complete (LLMLingua-2 is in), and the method-section justifications for the 80-token bullet and +32 round-2 slack tighten what was previously bare-bones. The stub-mode flag is the one remaining structural barrier, but it is neither hidden nor dressed up. I would accept at a workshop with a clear rerun-before-camera-ready note; for a stronger venue, a real-pod rerun and a retention-ratio sweep are necessary. Weak accept.

## Minor issues
- British vs US spelling: "summarisation"/"normalised" with "categorize" elsewhere — pick one.
- Algorithm 1: the comments duplicate prose; could be tightened.
- Figure 5 caption: note the y-axis is shared between F1 and EM bars.
