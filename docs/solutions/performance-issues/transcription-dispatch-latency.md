# Solution: Transcription-to-Coaching Latency (Debounce Dispatch)

## Problem
~15-second delay between interviewer finishing a question and coaching response appearing. The `generationComplete` event only fires after Gemini Live API finishes generating its full audio response -- hundreds of audio chunks that are never used (silently discarded by the `modelTurn` filter).

## Root Cause
The app used `generationComplete` as the ONLY trigger to send transcription to the coaching API (Groq/Gemini text). With AUDIO response mode (required by `gemini-2.5-flash-native-audio-preview`), this event is blocked until the model finishes generating its entire audio response.

Cannot switch to TEXT mode -- the model rejects it.

## Solution
Debounce timer on transcription fragments. If no new fragment arrives for 2 seconds, dispatch immediately.

### Key Code Pattern
```js
// Shared dispatch function - prevents double-dispatch
function dispatchTranscription() {
    if (transcriptionDebounceTimer) {
        clearTimeout(transcriptionDebounceTimer);
        transcriptionDebounceTimer = null;
    }
    const text = currentTranscription;
    currentTranscription = '';
    if (text.trim() !== '') {
        // send to coaching API
    }
}

// Reset timer on each transcription fragment
function resetTranscriptionDebounce() {
    if (transcriptionDebounceTimer) clearTimeout(transcriptionDebounceTimer);
    transcriptionDebounceTimer = setTimeout(() => {
        dispatchTranscription();
    }, TRANSCRIPTION_DEBOUNCE_MS); // 2000ms
}
```

### Why 2 Seconds
- <1s: Speaker pauses between words trigger premature dispatch
- 2s: Spans natural speech pauses, responds quickly after question ends
- >3s: Still wastes time

### Double-Dispatch Prevention
Both debounce timer and `generationComplete` call `dispatchTranscription()`. First caller wins (clears timer + resets transcription). Second caller finds empty text and no-ops.

## Prevention Checklist
- When using Live API models that require AUDIO mode, never rely solely on `generationComplete` for time-sensitive dispatch
- Always provide alternative trigger mechanisms (debounce, VAD, silence detection)
- Clean up timers on session close to prevent memory leaks

## Files Changed
- `src/utils/gemini.js`

## Date
2026-02-16
