# <no value>
# Skill Improvements — Ideation Coach v2

**Date:** 2026-04-04
**Analysis:** Based on evaluation of 8 model variants (Dense 27B and MoE 35B A3B at 4 quantization levels each)

---

## Summary of Recurring Issues

### Issue Frequency by Model

| Issue | Models Affected |
|-------|-----------------|
| Accepts "I don't know" without pushback | Q4K_M, Q4K_XL (both architectures) |
| Missing or shallow stress questions | Q4K_XL (MoE) |
| Stacked multiple questions per turn | Q4K_M, Q4K_XL |
| Vivid language sanitization | Q4K_M, Q4K_XL |
| No success metrics defined | Q4K_XL (Dense), Q4K_XL (MoE) |
| Session ends prematurely | Q4K_XL (MoE) |

---

## N1. Accepting Weak Answers Without Pushback

**Affected models:** Q4K_M (both), Q4K_XL (both)

When users say "I don't know" or "haven't tested it," models at lower quantizations tend to accept this as a final answer and move on. Higher quantizations (Q80, Q8K_XL) respond with "What's your best guess?" or "What would need to be true?"

**Fix:** Add to **Turn rules** section:

> **"I don't know" handling (MANDATORY):** When user says "I don't know," "haven't tested," or "need to research," you MUST respond with one of:
> - "What's your best guess?"
> - "What would need to be true for you to have an opinion?"
> - "If you had to guess, what would you expect?"
> 
> Never accept "I don't know" as a final answer. Treat it as a weak assumption to be marked as ⚠ and explored further.

---

## N2. Missing or Shallow Stress Questions

**Affected models:** Q4K_XL (MoE), some Q4K_M variants

The stress question phase (Reversal, Worst Customer, Second-order, Kill shot) is either skipped entirely or answered with generic responses.

**Fix:** Add to **Phase 3 — Stress questions** section:

> **Stress question enforcement:** After surfacing assumptions, you MUST ask all four questions in order. If user gives one-word answers, follow up with:
> - "Tell me more about that"
> - "What would that look like in practice?"
> 
> Do NOT proceed to wrap-up until all four lenses are explored. If user pushes back (e.g., "doesn't matter"), mark it as conviction data and note it in the document — do not skip the question.

---

## N3. Stacking Multiple Questions

**Affected models:** Q4K_M (both architectures)

Lower-quantized models sometimes ask 2-3 questions in one turn, violating the "one question per turn" rule.

**Fix:** Add to **Turn rules** section:

> **One question per turn (ENFORCED):** If you find yourself asking multiple questions in one turn, you are failing this rule. Split into separate turns. Example of FAILURE:
> > "What do you think about X? And how does Y affect Z? Also, do you have evidence for W?"
> 
> This must be three separate turns. After user answers, then ask the next question.

---

## N4. Vivid Language Sanitization

**Affected models:** Q4K_M (both), Q4K_XL (Dense)

User's vivid phrases like "walking with my dog," "hold a leash," "1 million dollar idea" are sanitized to generic language in the verbatim bank.

**Fix:** Add to **Phase 4 — Verbatim capture** section:

> **Vivid language preservation (MANDATORY):** Use the user's EXACT words in the verbatim bank. Do NOT sanitize. Examples:
> - ❌ "user wants to capture notes while walking" (sanitized)
> - ✓ "walking with my dog" (verbatim from user)
> - ❌ "user has limited time for typing" (sanitized)  
> - ✓ "hold a leash and I can't type fast with single hand" (verbatim)
> 
> If you are tempted to paraphrase the user's words, you are failing this rule. Copy their exact phrasing even if it seems informal.

---

## N5. Missing Success Metrics

**Affected models:** Q4K_XL (Dense: 39/50 had no metrics in doc)

Success criteria are often left empty or marked as "Nothing surfaced."

**Fix:** Add to **Phase 2 — Assumption stack** section:

> **Success criteria probing:** Before moving to stress questions, you MUST ask:
> - "What number would tell you this feature is working?"
> - "What's your target for adoption? (e.g., % of users per week)"
> - "How fast does transcription need to be?"
> 
> If user cannot provide metrics, note it as an untested assumption (⚠) and suggest a reasonable default in the next steps.

---

## N6. Premature Session End

**Affected models:** Q4K_XL (MoE: only 22/50, session ended before stress questions)

The session ends after only a few rounds without completing all phases.

**Fix:** Add to **Session pacing** section:

> **Minimum completion check:** Before accepting "wrap it up" or "done" signals, verify:
> - ✓ Phase 1 (ugly sentence) completed
> - ✓ At least 5 assumptions surfaced
> - ✓ All 4 stress questions asked
> 
> If any of these are missing, respond: "We haven't covered [X] yet. Can we finish that first?" Do not generate document until minimum phases complete.

---

## N7. Over-Confirming Weak Assumptions

**Affected models:** Q4K_M (MoE: 14 assumptions but all marked confirmed without evidence)

Models at lower quantizations sometimes mark assumptions as ✓ confirmed without verifying evidence, leading to false confidence in unvalidated assumptions.

**Fix:** Add to **Phase 2 — Assumption stack** section:

> **Evidence verification (MANDATORY):** When user confirms an assumption, ask for evidence:
> - "What's your evidence for that?"
> - "Did you test it or is this a guess?"
> - "Who told you that?"
> 
> If evidence is weak (personal belief, untested), mark as ⚠ even if user confirms. False confidence in untested assumptions is the primary failure mode.

---

## Prioritized Improvements

| Priority | Issue | Expected Impact |
|----------|-------|-----------------|
| 1 | Pushback on "I don't know" | +3-5 points on edge case handling |
| 2 | Vivid language preservation | +2-3 points on verbatim capture |
| 3 | Stress question enforcement | +3-4 points on stress coverage |
| 4 | Minimum completion check | Prevents incomplete sessions |
| 5 | One question per turn | +1-2 points on turn compliance |

---

## Quantization-Specific Recommendations

### For Q4K_M and Q4K_XL variants:
These rules need to be MORE specific because lower-quantized models follow complex instructions less reliably. The fixes above add enforcement language that lower-quantized models are more likely to follow.

### For Q80 and Q8K_XL variants:
These models generally follow the skill rules well. The improvements would primarily help with edge case handling and ensuring no assumptions are missed.

---

## Conclusion

The ideation-coach v2 skill produces good results at Q80+ quantization but degrades significantly at Q4K levels. The primary failure modes are:

1. Accepting weak answers without pushback
2. Missing or shallow stress questions  
3. Sanitizing vivid user language

The fixes add **enforcement language** (MANDATORY, NEVER, MUST) that lower-quantized models are more likely to follow, while maintaining the existing skill structure for higher-quantized models that already work well.
), proper phases | Stacked questions, less pushback |
| 4 | Qwen 3.5 35B A3B Q4K_XL | 22/50 | Quick execution | Session too short, missing contradictions section |

---

## Recommendation

**Winner: Qwen 3.5 35B A3B Q8K_XL**

The Q8K_XL variant produced the most insightful ideation session. It excelled at:

1. **Best insight capture** - "speaking is more natural than typing" - frames feature as primary preference, not just accessibility
2. **Best stress question execution** - identified specific failure modes (slow transcripts, low quality, shy users)
3. **Best root cause hypothesis** - captured the real problem (typing is friction, speaking is natural)
4. **Clean assumption stack** - 5 well-explored assumptions vs 14 shallow ones
5. **Best verbatim capture** - preserved user's framing

The Q80 variant is close second with strong technical specification capture but slightly less insight depth.

The Q4K_XL variant had a critical failure - session ended before stress questions, leaving major section empty.

**Key Finding:** For the 35B A3B MoE models, the quantization gap is extreme between Q4K_XL (22/50, incomplete) vs Q80+ (43-44/50). The Q4K_M (38/50) is adequate but less insightful.
