# 12 - V2 Ideation Coach Skill Experiment: Structured Pressure Testing with Local Models<no value>

I ran the improved ideation-coach skill (v2) through 8 model variants to see if the skill redesign fixed the 18 failure modes from v1.

## Why Test v2

The v1 ideation skill had 18 identified failure modes. The main problems were:

- Vivid language sanitized during Q&A, not just at write time
- Contradiction detection too passive
- Round themes bleeding across rounds
- "I don't know" accepted as final
- Stop detection too fragile

I redesigned the skill around a new structure: instead of rounds, it uses phases with explicit constraints. The v2 skill replaced "rounds" with "phases" such as Phase 1 gets the "ugly sentence," Phase 2 builds an assumption stack, Phase 3 runs stress questions, Phase 4 captures verbatim quotes, Phase 5 wraps up.

The key difference: v2 tells the model what NOT to do in each phase, not just what to do.

---

## Test Setup

Same hardware, same task, same evaluation criteria as v1:

| Parameter             | Value   |
| --------------------- | ------- |
| Context Length        | 262144  |
| GPU Offload           | Full    |
| CPU Thread Pool Size  | 16      |
| Evaluation Batch Size | 8192    |
| Thinking              | Enabled |
| Temperature           | 0.6     |

Task: Ideation session for "Add audio notes using Cactus SDK" in Aily app.

Evaluated by MiniMax M2.5 Free in OpenCode Zen.

---

## The v2 Skill Structure

The ideation-coach v2 skill lives in `Skill/ideation-coach/` with these phases:

1. Phase 0 Scan: read workspace context (README.md, docs/)
2. Phase 1 Ugly Sentence: get complete "for X so they can Y instead of Z" frame
3. Phase 2 Assumption Stack: surface 5-7 testable assumptions with evidence status (✓ confirmed, ⚠ untested)
4. Phase 3 Stress Questions: run all 4 lenses: Reversal, Worst Customer, Second-order, Kill shot
5. Phase 4 Verbatim Capture: preserve user's exact phrases
6. Phase 5 Wrap-up: write ideation doc, handoff to create-prd

### Key v2 Rules Added

- **Turn rules:** One question per turn, quote user first, mark assumptions when surfaced, stay short
- "I don't know" handling: Never accept as final. Respond with "What's your best guess?" or "What would need to be true?"
- **Vivid language preservation:** Use EXACT user words, never sanitize
- **Stress question enforcement:** All 4 lenses must be explored before wrap-up
- **Minimum completion check:** Verify Phase 1, 5+ assumptions, all 4 stress questions before writing doc

### What Changed from v1

| v1 (Ideation Skill)                            | v2 (Ideation Coach)                       |
| ---------------------------------------------- | ----------------------------------------- |
| Round-based (Problem → Users → Constraints)    | Phase-based with explicit constraints     |
| "Name contradictions" (passive)                | Contradictions surfaced as questions      |
| "Do not sanitise vivid language" (Step 5 only) | "Use EXACT words" (enforced from Phase 1) |
| "I don't know" handling missing                | MANDATORY pushback language               |
| Round theme bleeding common                    | "FORBIDDEN in this phase" per phase       |
| No success metrics probing                     | Explicit metrics questions added          |

---

## Qwen 3.5 27B Results (Dense)

Full evaluation: [Qwen 3.5 27B Quantization Comparison](https://github.com/MrOnyszko/local-llm-guide/tree/main/aily-ideation-skill-v2-experiment/Evaluation/Qwen%203.5%2027B%20Quantization%20Comparison%20Evaluation.md).

| Rank | Model  | Score | Session Time | Key Strength                                  |
| ---- | ------ | ----- | ------------ | --------------------------------------------- |
| 1    | Q8K_XL | 48/50 | 70 min       | Most assumptions (14), best verbatim capture  |
| 2    | Q80    | 44/50 | 30 min       | Thorough assumption stack (8), vivid language |
| 3    | Q4K_XL | 39/50 | 17 min       | Good assumption probing, clean sessions       |
| 4    | Q4K_M  | 37/50 | 20 min       | Clean phase progression                       |

Q8K_XL won again. It produced 14 assumptions, the most detailed stack of any model. It pushed back on "I haven't tested" multiple times, caught the privacy/offline contradiction (user wanted on-device but mentioned cloud), and preserved "hold a leash and I can't type fast with single hand."

Q80 was the efficiency winner. At 30 minutes, it produced 44/50, nearly as good as Q8K_XL but less than half the time. The verbatim capture was nearly as good: "walking with my dog," "precious ideas" preserved.

Q4 variants improved. The v1 Q4K_M scored 29/50. The v2 Q4K_M scored 37/50, an 8-point improvement from better skill instructions alone. The gap between Q4 and Q8 narrowed.

> v1 Q4K_M: 29/50 → v2 Q4K_M: 37/50. The skill improvements added 8 points without model changes.

---

## Qwen 3.5 35B A3B Results (MoE)

Full evaluation: [Qwen 3.5 35B A3B Quantization Comparison](https://github.com/MrOnyszko/local-llm-guide/tree/main/aily-ideation-skill-v2-experiment/Evaluation/Qwen%203.5%2035B%20A3B%20Quantization%20Comparison%20Evaluation.md).

| Rank | Model  | Score | Session Time | Key Strength                                 |
| ---- | ------ | ----- | ------------ | -------------------------------------------- |
| 1    | Q8K_XL | 44/50 | 21 min       | Best insight capture, clean assumption stack |
| 2    | Q80    | 43/50 | 20 min       | Best real-time transcription probing         |
| 3    | Q4K_M  | 38/50 | 21 min       | Good assumption count (14), proper phases    |
| 4    | Q4K_XL | 22/50 | 11 min       | Session ended before stress questions        |

Q8K_XL captured the best insight: "speaking is more natural than typing", this frames audio notes as a primary preference, not just an accessibility feature. It also identified specific failure modes in stress questions (slow transcripts, shy users, low quality).

Q4K_XL still fails. At 22/50, it produced an incomplete session that ended before stress questions, and left contradictions section empty. This is the same failure mode as v1. The MoE architecture at Q4 quantization cannot handle structured phase transitions.

**Q80 is viable on MoE.** At 43/50, the Q80 MoE variant is close to Q8K_XL. Unlike v1, the v2 skill produces acceptable MoE results at Q80.

> MoE at Q80 works now. v1 MoE Q8_0 scored 30/50. v2 MoE Q80 scores 43/50. That's a 13-point improvement.

---

## Dense vs MoE: v2 Results

Full comparison: [Dense vs MoE Comparison](https://github.com/MrOnyszko/local-llm-guide/tree/main/aily-ideation-skill-v2-experiment/Evaluation/Dense%20vs%20MoE%20Comparison.md).

| Criterion                | 27B Dense Avg | 35B MoE Avg | Delta     | Winner    |
| ------------------------ | ------------- | ----------- | --------- | --------- |
| Skill Flow Adherence     | 4.50          | 4.00        | +0.50     | Dense     |
| Ugly Sentence Quality    | 4.75          | 4.25        | +0.50     | Dense     |
| Assumption Stack         | 4.50          | 4.00        | +0.50     | Dense     |
| Stress Question Coverage | 4.75          | 4.00        | +0.75     | Dense     |
| Turn Rule Compliance     | 4.00          | 3.50        | +0.50     | Dense     |
| Edge Case Handling       | 4.25          | 3.50        | +0.75     | Dense     |
| Verbatim Capture         | 4.50          | 4.50        | 0         | Tie       |
| Doc Completeness         | 4.50          | 4.00        | +0.50     | Dense     |
| Doc Accuracy             | 4.50          | 4.25        | +0.25     | Dense     |
| Session Pacing           | 4.25          | 3.75        | +0.50     | Dense     |
| **Average Total**        | **42.00**     | **38.75**   | **+3.25** | **Dense** |

**Dense still wins**, but the gap narrowed from v1 (+4.75 → +3.25). The v2 skill structure helps MoE models perform closer to their potential.

**Key finding: MoE at Q80 is now usable.** In v1, MoE Q8_0 scored 30/50. In v2, MoE Q80 scores 43/50. The skill improvements had a bigger impact on MoE than Dense.

### Quantization Sensitivity Comparison

| Quant Level | v1 Dense Avg | v2 Dense Avg | v1 MoE Avg | v2 MoE Avg |
| ----------- | ------------ | ------------ | ---------- | ---------- |
| Q8K_XL      | 43           | 48           | 34         | 44         |
| Q80         | 41           | 44           | 30         | 43         |
| Q4K_XL      | 35           | 39           | 31         | 22         |
| Q4K_M       | 29           | 37           | 22         | 38         |

The v2 skill improved Dense by +4.5 points average, and MoE by +7.5 points average. The biggest gains: Q4K_M Dense (+8) and Q4K_M MoE (+16).

> The v2 skill improvements helped lower quantizations more than higher ones. Q4K_M MoE improved by 16 points, from unusable (22) to adequate (38).

---

## v1 vs v2: What the Skill Changes Fixed

The v2 skill addressed 6 of the 8 original failure modes:

| v1 Failure Mode                 | v2 Fix                                  | Result                       |
| ------------------------------- | --------------------------------------- | ---------------------------- |
| Vivid language lost during Q&A  | "Use EXACT words" enforced from Phase 1 | Q4K_M improved 8 pts         |
| No guard against hallucination  | "Every item must trace to session"      | Doc accuracy improved        |
| Contradiction detection passive | Contradictions surfaced as questions    | Better edge case handling    |
| Round themes bleed              | Phase-based with "FORBIDDEN" rules      | Clean phase transitions      |
| "I don't know" accepted         | MANDATORY pushback language             | +3-5 pts edge handling       |
| Stop detection fragile          | Minimum completion check                | Prevents incomplete sessions |

**What still fails at Q4:** The "I don't know" pushback still requires Q80 or higher to work reliably. Q4K_XL MoE still produces incomplete sessions.

---

## What I Think About the Results

The v2 skill delivers what v1 promised: usable ideation sessions at lower quantizations. The Q4K_M improvement (29 → 37 for Dense, 22 → 38 for MoE) is the clearest win. It means I can run the skill on hardware-constrained devices without sacrificing quality.

MoE at Q80 is now viable. In v1, I wrote off MoE entirely. In v2, MoE Q80 scores 43/50, only 1 point behind Dense Q80. If I need the speed advantage (21 min vs 30 min for comparable quality), I can use MoE at Q80.

The insight quality from MoE Q8K_XL is worth the compute cost. "Speaking is more natural than typing" is a better capture than Dense's "hold a leash", it tells me the root cause, not just the use case. For early-stage ideation, that insight quality matters.

The 48/50 score from Dense Q8K_XL is the ceiling for this task. I've pushed the skill as far as it can go with the current structure. Further improvements need either:

1. Better reference files (more examples, more templates)
2. Multi-model orchestration (use Q8K_XL for insight, Q4K_XL for speed)
3. Different evaluation criteria (maybe ideation doesn't need perfect verbatim capture)

---

## Key Points

1. Dense Q8K_XL remains best: 48/50, most assumptions (14), best verbatim capture
2. v2 skill improves all models: average +4.5 pts for Dense, +7.5 pts for MoE
3. Q4K_M is now usable: v1 was 29/50, v2 is 37/50, an 8-point gain from skill alone
4. MoE at Q80 works: v1 was 30/50, v2 is 43/50, a 13-point gain, now viable
5. MoE still fails at Q4K_XL: 22/50, incomplete session, same failure as v1
6. Quantization gap narrowed: v1 had 14-pt gap Q4 to Q8, v2 has 11-pt gap
7. Best insight from MoE: "speaking is more natural than typing" beats Dense's use case details
8. Skill structure matters more than model size: the phase-based approach with explicit constraints outperforms round-based at equivalent model quality
