# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Electron-based real-time AI assistant providing contextual help during video calls, interviews, and meetings via screen/audio capture and Google Gemini 2.0 Flash Live API integration.

Fork of [sohzm/cheating-daddy](https://github.com/sohzm/cheating-daddy) with planned migration to TypeScript/React and shadcn/ui components.

## Key Commands

### Development
```bash
npm install              # Install dependencies
npm start                # Start Electron app in dev mode
npx prettier --write .   # Format code (required before commits)
```

### Building & Packaging
```bash
npm run package          # Package app for current platform
npm run make             # Build distributable for current platform
```

### Platform-Specific Builds
- **Windows**: Squirrel installer with desktop/start menu shortcuts
- **macOS**: DMG installer (code signing/notarization commented out in `forge.config.js`)
- **Linux**: AppImage via `@reforged/maker-appimage`

## Code Architecture

### Main Process (`src/index.js`)
Entry point coordinating:
- Window creation and management (`src/utils/window.js`)
- Multi-provider AI integration: Gemini, Groq, Cloud, Local (`src/utils/gemini.js`, `src/utils/cloud.js`, `src/utils/localai.js`)
- Configuration and credential storage (`src/storage.js`)

**Key IPC Handlers:**
- `get-config`, `set-config`, `get-credentials`, `set-credentials` - Storage management
- `quit-application`, `open-external` - System operations
- `update-keybinds`, `update-content-protection` - Window behavior
- AI provider switching (Gemini/Groq/Cloud/Local)

### Renderer Process
**Component Structure:**
- `src/components/app/` - Main app shell (`CheatingDaddyApp.js`, `AppHeader.js`)
- `src/components/views/` - View components (`MainView.js`, `AssistantView.js`, `CustomizeView.js`, `AICustomizeView.js`, `HelpView.js`, `HistoryView.js`, `OnboardingView.js`, `FeedbackView.js`)
- `src/components/views/sharedPageStyles.js` - Shared styling utilities
- `src/components/index.js` - Component exports

**Utilities:**
- `src/utils/gemini.js` - Multi-provider AI API integration (Gemini/Groq), audio capture, conversation management
- `src/utils/cloud.js` - WebSocket cloud API with BYOK support
- `src/utils/localai.js` - Ollama integration, Hugging Face Transformers, local whisper, VAD
- `src/utils/window.js` - Window creation, positioning, transparency, keyboard shortcuts
- `src/utils/renderer.js` - Renderer-side utilities
- `src/utils/prompts.js` - System prompts for different profiles (Interview, Sales, Business Meeting, etc.)
- `src/audioUtils.js` - PCM to WAV conversion, audio analysis, debug logging

### Configuration System (`src/storage.js`)
Centralized persistent storage for config and credentials:
- **Windows**: `%APPDATA%\cheating-daddy-config\config.json`
- **macOS**: `~/Library/Application Support/cheating-daddy-config/config.json`
- **Linux**: `~/.config/cheating-daddy-config/config.json`

**Storage Categories:**
- **Credentials**: `apiKey`, `groqApiKey`, `cloudToken`
- **User Preferences**: `customPrompt`, `selectedProfile`, `language`, `ollamaHost`
- **App State**: `onboarded`, `layout`, `keybinds`
- **AI Provider**: Selected provider (gemini/groq/cloud/local)

### Audio Capture Architecture
**Platform-Specific:**
- **macOS**: SystemAudioDump binary (`src/assets/SystemAudioDump`)
- **Windows**: Loopback audio capture
- **Linux**: Microphone input (limited support)

**Audio Processing:**
- Captures raw PCM audio at 24kHz, mono, 16-bit
- Converts to WAV for debugging (`audioUtils.js`)
- Streams to Gemini API for real-time transcription
- Debug audio saved to `~/cheddar/debug/` with analysis metadata

### Conversation Management
Session tracking in `src/utils/gemini.js`:
- `currentSessionId` - Unique timestamp-based session ID
- `currentTranscription` - Accumulated transcription text
- `conversationHistory` - Full conversation context array
- Speaker diarization support (Interviewer/Candidate labels)

## Migration Strategy (TypeScript + React + shadcn/ui)

Target architecture inspired by [Gatecrashah/transcriber](https://github.com/Gatecrashah/transcriber).

### Migration Guidelines
- **TypeScript strict mode** - No `any`, explicit interfaces
- **React functional components** with hooks and error boundaries
- **shadcn/ui components** in `src/components/ui/`
- **Path aliases** - Use `@/` prefix for imports from `src`
- **Secure IPC** - Validate all renderer/main boundary data
- **Non-blocking audio** - Keep heavy processing off UI thread

### shadcn/ui Integration
```bash
npx shadcn@latest add <component>  # Never hand-roll components
```

Component pattern: `React.forwardRef` + `cn()` helper for class names
Theming: CSS variables and `@/utils/tailwind` utilities

### Future Features (from transcriber)
1. **Local transcription** - `whisper.cpp` integration for offline speech-to-text
2. **Dual audio capture** - Simultaneous microphone + system audio
3. **Speaker diarization** - tinydiarize integration (`[SPEAKER_TURN]` markers)
4. **Voice activity detection** - Skip silent segments before API calls
5. **Note management** - Local transcription storage with meeting notes

## Code Standards

### Formatting
Prettier config (`.prettierrc`):
- 4-space indentation
- 150 character print width
- Single quotes, semicolons, ES5 trailing commas
- LF line endings

**Always run before commits:**
```bash
npx prettier --write .
```

### IPC Security
- Validate and sanitize all parameters crossing renderer/main boundary
- Use `ipcMain.handle()` for async responses with error handling
- Return structured responses: `{ success: boolean, data?, error? }`

### Audio Processing Principles
- **16 kHz resampling** for whisper.cpp compatibility
- **Dual-stream architecture** for separate mic/system audio
- **Quality preservation** - Maintain sample fidelity
- **Memory efficiency** - Stream large files, avoid loading all at once
- **Error recovery** - Handle device failures gracefully

## Security & Privacy

### Privacy by Design
- **Local processing** - Transcriptions can happen locally via Ollama/Hugging Face
- **User control** - Clear options for data retention/deletion
- **Transparency** - Document what is stored and where
- **Minimal data** - Only persist required data

### AI Provider Options
- **Google Gemini** - Cloud API with streaming support
- **Groq** - Alternative cloud provider
- **Cloud API** - WebSocket-based BYOK service
- **Local AI** - Ollama/Hugging Face models for offline operation

## Platform Requirements

- Electron-compatible OS (macOS 10.13+, Windows 10+, Linux)
- **AI Provider** (choose one or more):
  - Gemini API key ([Google AI Studio](https://aistudio.google.com/apikey))
  - Groq API key ([Groq Console](https://console.groq.com))
  - Cloud token (cheating-daddy.com)
  - Ollama installation for local AI
- Screen recording permissions
- Microphone/audio permissions

## Keyboard Shortcuts

Window management built into `src/utils/window.js`:
- **Ctrl/Cmd + Arrow Keys** - Move window
- **Ctrl/Cmd + M** - Toggle click-through mode
- **Ctrl/Cmd + \\** - Close window/go back
- **Enter** - Send message to AI

Shortcuts dynamically updated via `update-keybinds` IPC handler.

## Upstream Sync

Cherry-pick PRs from [sohzm/cheating-daddy](https://github.com/sohzm/cheating-daddy):
1. Inspect diff, use short commit messages (`feat:`, `fix:`, etc.)
2. Test locally after merge: `npm install && npm start`

## Extra Resources

- Electron Forge config: `forge.config.js`
- Asset bundling: `src/assets/` (logo variants, SystemAudioDump binary, external libs)
- External libs: `highlight.js`, `lit`, `marked` (in `src/assets/`)
- Onboarding SVGs: `src/assets/onboarding/`
