# RP-Changelog

## 2026-02-16 - Fix: Reduce Transcription-to-Coaching Latency

### Summary
**RESOLVED** Eliminated ~15-second delay between interviewer finishing a question and coaching response appearing.

**Type:** Bug Fix / Performance | **Impact:** Response time drops from ~20s to ~4-5s per question

### Changes

| # | File | Lines | Change |
|---|------|-------|--------|
| 1 | src/utils/gemini.js | 50-51 | Added `transcriptionDebounceTimer` and `TRANSCRIPTION_DEBOUNCE_MS` (2000ms) |
| 2 | src/utils/gemini.js | 426-440 | Added `dispatchTranscription()` - shared dispatch logic preventing double-dispatch |
| 3 | src/utils/gemini.js | 442-447 | Added `resetTranscriptionDebounce()` - resets timer on each transcription fragment |
| 4 | src/utils/gemini.js | 497,502 | Transcription handlers now call `resetTranscriptionDebounce()` |
| 5 | src/utils/gemini.js | 510 | `generationComplete` handler uses `dispatchTranscription()` instead of inline logic |
| 6 | src/utils/gemini.js | 1087-1089 | Timer cleanup on session close |

### Details
**Root cause:** `generationComplete` only fires after Live API finishes generating full audio response (~15s of unused audio chunks). Cannot switch to TEXT mode because the `gemini-2.5-flash-native-audio-preview` model rejects it.

**Fix:** Debounce timer on transcription fragments. If no new fragment arrives for 2 seconds, dispatch immediately without waiting for `generationComplete`. Both paths call same `dispatchTranscription()` function which atomically clears timer + transcription to prevent double-dispatch.

---
