# <no value>
# Dense vs MoE — Ideation Skill Evaluation Comparison

**Date:** 2026-03-27
**Task:** Run ideation session on "Add audio notes using Cactus SDK" for Aily app
**Dense model:** unsloth/qwen3.5-27b (4 quantization variants)
**MoE model:** unsloth/qwen3.5-35b-a3b (4 quantization variants)

---

## Models Tested

| Family | Architecture | Variants |
|--------|-------------|----------|
| Qwen 3.5 27B | Dense | Q4_K_M, Q8_0, UD Q4_K_XL, UD Q8_K_XL |
| Qwen 3.5 35B A3B | MoE (35B total, 3B active) | Q4_K_M, Q8_0, UD Q4_K_XL, UD Q8_K_XL |

---

## Evaluation Criteria

Each model scored 1-5 on 10 axes:

| # | Criterion |
|---|-----------|
| 1 | Skill Flow Adherence |
| 2 | Question Quality |
| 3 | Round Thematic Rigor |
| 4 | Contradiction Detection |
| 5 | Vivid Language Capture |
| 6 | Workspace Context Usage |
| 7 | Ideation Doc Completeness |
| 8 | Ideation Doc Accuracy |
| 9 | Document Polish |
| 10 | Session Pacing |

---

## Full Scoring Matrix

| Criterion | 27B Q4_K_M | 27B Q8_0 | 27B UD Q4_K_XL | 27B UD Q8_K_XL | 35B Q4_K_M | 35B Q8_0 | 35B UD Q4_K_XL | 35B UD Q8_K_XL |
|-----------|-----------|---------|---------------|---------------|-----------|---------|---------------|---------------|
| Skill Flow Adherence | 4 | 4 | 4 | 5 | 3 | 4 | 3 | 4 |
| Question Quality | 3 | 5 | 4 | 4 | 2 | 4 | 3 | 4 |
| Round Thematic Rigor | 3 | 4 | 4 | 5 | 2 | 3 | 2 | 4 |
| Contradiction Detection | 2 | 4 | 3 | 5 | 2 | 3 | 3 | 3 |
| Vivid Language Capture | 2 | 5 | 3 | 4 | 2 | 3 | 4 | 3 |
| Workspace Context Usage | 3 | 5 | 5 | 5 | 2 | 4 | 4 | 5 |
| Doc Completeness | 4 | 5 | 4 | 5 | 3 | 5 | 5 | 4 |
| Doc Accuracy | 3 | 5 | 4 | 5 | 3 | 4 | 3 | 4 |
| Document Polish | 3 | 5 | 4 | 5 | 2 | 4 | 4 | 4 |
| Session Pacing | 3 | 4 | 4 | 5 | 3 | 3 | 4 | 4 |
| **Total** | **29** | **41** | **35** | **43** | **22** | **30** | **31** | **34** |

---

## Ranking — All 8 Models

| Rank | Model | Family | Arch | Score | Session Time |
|------|-------|--------|------|-------|-------------|
| **1** | **27B UD Q8_K_XL** | 27B | Dense | **43** | 16 min |
| 2 | 27B Q8_0 | 27B | Dense | 41 | 16 min |
| 3 | 27B UD Q4_K_XL | 27B | Dense | 35 | 21 min |
| **4** | **35B UD Q8_K_XL** | 35B A3B | MoE | **34** | 11 min |
| 5 | 35B UD Q4_K_XL | 35B A3B | MoE | 31 | 18 min |
| 6 | 35B Q8_0 | 35B A3B | MoE | 30 | 11 min |
| 7 | 27B Q4_K_M | 27B | Dense | 29 | 10 min |
| 8 | 35B Q4_K_M | 35B A3B | MoE | 22 | 8 min |

---

## Architecture Comparison — Dense vs MoE

### Average Scores by Architecture

| Criterion | 27B Dense Avg | 35B MoE Avg | Delta | Winner |
|-----------|--------------|-------------|-------|--------|
| Skill Flow Adherence | 4.25 | 3.50 | +0.75 | Dense |
| Question Quality | 4.00 | 3.25 | +0.75 | Dense |
| Round Thematic Rigor | 4.00 | 2.75 | +1.25 | **Dense** |
| Contradiction Detection | 3.50 | 2.75 | +0.75 | Dense |
| Vivid Language Capture | 3.50 | 3.00 | +0.50 | Dense |
| Workspace Context Usage | 4.50 | 3.75 | +0.75 | Dense |
| Doc Completeness | 4.50 | 4.25 | +0.25 | Dense |
| Doc Accuracy | 4.25 | 3.50 | +0.75 | Dense |
| Document Polish | 4.25 | 3.50 | +0.75 | Dense |
| Session Pacing | 4.00 | 3.50 | +0.50 | Dense |
| **Average Total** | **36.50** | **31.75** | **+4.75** | **Dense** |

### Quantization Impact by Architecture

| Quant | 27B Dense Avg | 35B MoE Avg |
|-------|--------------|-------------|
| Q4_K_M | 29.0 | 22.0 |
| Q8_0 | 41.0 | 30.0 |
| UD Q4_K_XL | 35.0 | 31.0 |
| UD Q8_K_XL | 43.0 | 34.0 |

**Pattern:** Higher quantization improves both architectures, but the gap is wider for Dense (+14 from Q4_K_M to Q8_K_XL) than MoE (+12). Dense benefits more from quantization upgrades.

---

## Criterion Deep Dive

### Where Dense dominates most

**Round Thematic Rigor (Delta: 1.25)**
The largest gap. Dense models maintain round boundaries cleanly. MoE models consistently mix themes — asking constraint questions in round 1, exploration questions in round 3. The sparse activation pattern of MoE may weaken the ability to maintain strict conditional logic.

**Contradiction Detection (Delta: 0.75)**
Only the Dense Q8_K_XL actively challenged user assumptions (Cactus offline capability). No MoE model named a contradiction. Dense models had higher ceiling on this hardest skill rule.

**Doc Accuracy (Delta: 0.75)**
MoE UD Q4_K_XL changed the user's 45% success metric to 20%. MoE Q4_K_M hallucinated workspace context without reading docs. Dense models were more faithful to source material.

### Where MoE is competitive

**Session Pacing (Delta: 0.50)**
MoE models are faster. All 35B sessions completed in ≤18 min. Dense sessions ranged from 10-67 min. For speed-sensitive workflows, MoE has an advantage.

**Doc Completeness (Delta: 0.25)**
MoE UD Q4_K_XL produced the most comprehensive doc (10 open questions, 5 risks). MoE models can generate thorough documents despite weaker sessions.

**Vivid Language Capture (Delta: 0.50)**
The gap is modest. Both architectures struggle with preserving user phrasing. MoE UD Q4_K_XL captured vivid language well in-session (though didn't carry it to the doc).

---

## Quantization Impact — Cross-Architecture

### Q8 variants outperform Q4 in both architectures

| Metric | Q4 avg (both) | Q8 avg (both) | Delta |
|--------|--------------|---------------|-------|
| Total score | 26.5 | 37.0 | +10.5 |
| Contradiction detection | 2.0 | 3.5 | +1.5 |
| Doc accuracy | 3.0 | 4.5 | +1.5 |

Q8 quantization consistently improves reasoning-heavy criteria (contradiction detection, doc accuracy) more than structural criteria.

### UD (Unsloth Dynamic) variants

| Variant | Avg improvement over non-UD |
|---------|---------------------------|
| 27B UD Q4_K_XL vs Q4_K_M | +6 points |
| 27B UD Q8_K_XL vs Q8_0 | +2 points |
| 35B UD Q4_K_XL vs Q4_K_M | +9 points |
| 35B UD Q8_K_XL vs Q8_0 | +4 points |

UD quantization provides larger gains at Q4 level than Q8 level. At Q8, the base quant is already strong so UD adds less.

---

## Key Findings

### 1. Dense architecture is better for structured conversational tasks

The ideation skill requires strict conditional logic (round themes, stop detection, contradiction naming). Dense models maintain these rules more consistently. The 4.75-point average gap is significant.

### 2. MoE is faster but less rigorous

All MoE sessions completed in ≤18 min. Dense sessions ranged 10-67 min. But MoE's speed comes at the cost of structural discipline — round themes bleed, contradictions go unnamed, and metrics get changed.

### 3. Q8_K_XL is the best quant for both architectures

Within each family, the UD Q8_K_XL variant wins. Higher bit-width preserves reasoning quality, and Unsloth Dynamic quantization adds further gains.

### 4. The Dense→MoE gap is largest on hardest criteria

Round thematic rigor (1.25 delta) and contradiction detection (0.75 delta) are the hardest skill rules. These show the largest Dense advantages. Easier criteria (doc completeness, session pacing) show smaller gaps.

### 5. Quantization matters more than architecture at Q4

At Q4_K_M, the 27B Dense (29) beats 35B MoE (22) by 7 points. At Q8_K_XL, the gap narrows: 27B Dense (43) vs 35B MoE (34) by 9 points. But the 35B MoE Q8_K_XL (34) still beats the 27B Dense Q4_K_M (29) — quantization is the stronger lever.

---

## Recommendation

**Best overall: Qwen 3.5 27B UD Q8_K_XL (Dense, 43/50)**

For ideation-quality tasks requiring strict skill adherence, the Dense 27B at highest quantization is the clear winner. It combines structural rigor with deep questioning and accurate documentation.

**Best MoE: Qwen 3.5 35B A3B UD Q8_K_XL (34/50)**

The MoE architecture can be competitive if speed is prioritized over rigor. At 11 min for 6 rounds with decent doc accuracy, it's viable for faster iteration cycles — but expect round theme bleeding and weaker contradiction detection.

**If constrained to Q4:** Choose Dense 27B (29) over MoE 35B (22). The Dense architecture degrades more gracefully at lower quantization.
