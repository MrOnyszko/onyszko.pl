# <no value>
# Ideation: Audio Notes with Cactus SDK

**Date:** 2026-03-27
**Status:** Draft | Ready for PRD

---

## Problem

### What problem are we solving?
Users can't capture notes while on-the-go (e.g., walking) because typing isn't feasible. They need to speak their thoughts quickly and refine them later.

### Who has this problem?
People who prefer speaking over typing, especially when mobile or in motion.

### How do they solve it today?
Use external apps (Voice Memos, Google Keep), then copy/paste into Aily notes. This is fragmented and loses context.

### Root Cause Hypothesis
Aily only supports text/rich text notes. No native audio capture or transcription means users must switch apps and manually transfer content.

---

## Solution

### Proposed approach
Add audio note creation to the existing Notes feature. Users tap record, speak, get local transcription via Cactus SDK, then playback audio and/or edit the transcript as a hybrid audio+text note.

### MVP behavior
1. Create Note → Tap Record button
2. Record audio (30s to 10m)
3. Transcribe locally using Cactus SDK (offline-first)
4. Playback audio + view/edit transcript
5. Save as note with both audio file and text content

### What's NOT in scope
- Cross-device sync (Aily doesn't sync anyway)
- Cloud transcription (Claude disabled by default)
- Real-time transcription while recording (can be v2)
- Audio editing (trimming, splicing)

---

## Value

### Why is this worth building?
Captures thoughts faster than typing, keeps everything in Aily, respects privacy with offline-first local transcription.

### Success criteria
- 45% of notes are audio notes within 6 months
- Users can record, transcribe, and edit without leaving Aily

### Early signal metric
Increase in audio note creation rate (track from day 1).

### Risks
| Risk | Likelihood | Impact | How to Validate |
|------|-----------|--------|-----------------|
| Local transcription quality is poor | Med | High | Test Cactus SDK with sample recordings before commit |
| Users don't actually record notes | Med | High | Track adoption rate in first month |
| Audio storage bloats DB | Low | Med | Show storage info, let users manage/delete |
| Recording permissions block adoption | Low | Med | Measure permission grant rate |

---

## Open Questions

- [ ] Cactus SDK integration complexity — does it work seamlessly with Flutter?
- [ ] Audio file storage strategy — embed in DB or separate files?
- [ ] Transcript accuracy expectations — what's acceptable error rate?
- [ ] Should transcript be editable as rich text or plain text?
- [ ] Playback UX — inline player or separate screen?
- [ ] What happens if transcription fails — retry flow?
- [ ] Should audio notes be filterable/searchable like text notes?

---

## Next Steps

- [ ] Evaluate Cactus SDK compatibility with Flutter (iOS/Android)
- [ ] Design audio note data model (audio blob + transcript fields)
- [ ] Prototype recording + transcription flow
- [ ] Write the PRD using the `create-prd` skill
