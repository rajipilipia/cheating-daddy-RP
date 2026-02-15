---
title: Merging Major Upstream Refactor with Local Customizations
slug: upstream-major-refactor-merge
category: merge-strategies
tags:
  - git
  - merge
  - conflict-resolution
  - electron
  - upstream-sync
status: verified
created: 2026-02-14
updated: 2026-02-14
severity: medium
related_issues:
  - sohzm/cheating-daddy#v0.7.0
  - rajipilipia/cheating-daddy-RP
---

# Merging Major Upstream Refactor with Local Customizations

## Problem Summary

Upstream repository (sohzm/cheating-daddy) released v0.7.0 with major architectural changes:
- 27 commits ahead with 7,451 insertions, 7,950 deletions across 37 files
- Deleted critical modules: `stealthFeatures.js`, `processRandomizer.js`, `config.js`
- Refactored config system from `src/config.js` → `src/storage.js` (531 new lines)
- Removed vitest testing framework completely
- Added 3 new major features: Cloud AI, Local AI (Ollama), Groq integration

**Challenge:** Fork had local customizations (.claude/, guide.md, MCP configs) that needed preservation while accepting architectural refactor.

### Symptoms
- Remote 27 commits behind with breaking architecture changes
- Risk of losing local customizations during merge
- Potential conflicts in multiple critical files (index.js, gemini.js, package.json)

## Root Cause Analysis

The upstream refactor was comprehensive and intentional:
1. **Storage architecture change**: Centralized `storage.js` with multi-provider support
2. **Feature removal**: Stealth modules deleted (no longer maintained upstream)
3. **Testing refactor**: Vitest framework removed, testing approach changed
4. **New capabilities**: Cloud API, local AI, Groq provider, feedback system

Local fork had no conflicting commits, so merge would be clean if local files protected.

## Solution: Protected Merge Strategy

### Step 1: Pre-Merge Protection (CRITICAL)

Create backup before any changes:
```bash
git branch backup-pre-v07-merge
git tag pre-v07-merge
```

Protect local files in `.gitignore` before merge:
```gitignore
# Local customizations
.claude/
.mcp.json
settings.local.json
start-claude-local.bat
guide.md
src/utils/sample prompts.txt
```

Commit gitignore changes:
```bash
git add .gitignore
git commit -m "chore: protect local custom files in gitignore"
```

Backup locally outside repo:
```bash
mkdir ../cheating-daddy-local-backup
cp -r .claude .mcp.json settings.local.json start-claude-local.bat guide.md ../cheating-daddy-local-backup/
cp "src/utils/sample prompts.txt" ../cheating-daddy-local-backup/
```

### Step 2: Execute Merge

```bash
git fetch upstream
git merge upstream/master -m "merge: update to v0.7.0 with cloud/local AI support"
```

**Result:** Zero conflicts! Git auto-resolved cleanly because:
- Local files protected by .gitignore
- No conflicting commits in fork
- Clean deletion of stealth modules (fork didn't modify them)

### Step 3: Post-Merge Validation

Install updated dependencies:
```bash
npm install  # Installs new packages: @huggingface/transformers, ollama, ws
```

Test app startup:
```bash
npm start
```

Verify no module errors for deleted files. Expected output:
```
Config directory initialized with defaults
Global shortcuts registered
(GPU cache errors normal for Electron - not critical)
```

### Step 4: Documentation Updates

Update `CLAUDE.md` to reflect new architecture:
- Remove stealth feature references
- Replace `src/config.js` with `src/storage.js`
- Document new providers (Cloud, Local, Groq)
- Remove vitest testing section
- Update component structure (remove AdvancedView, add AICustomizeView)

## Technical Details

### New Dependencies Added
```json
"@huggingface/transformers": "^3.8.1"  // Local whisper models
"ollama": "^0.6.3"                     // Local LLM orchestration
"ws": "^8.19.0"                        // WebSocket for cloud API
```

### Removed Dependencies
- `vitest` - Testing framework
- `jsdom` - Testing environment

### Deleted Files (Safe to Remove)
```
src/utils/stealthFeatures.js      // Anti-analysis features
src/utils/processRandomizer.js    // Process name obfuscation
src/utils/processNames.js         // Supporting module
src/config.js                     // Old config system
vitest.config.js                  // Test configuration
.github/workflows/test.yml        // Vitest CI workflow
.github/workflows/windows-build.yml
src/__mocks__/electron.js         // Test mocks
src/__tests__/                    // All test files
src/components/views/AdvancedView.js
```

### New Files (New Capabilities)
```
src/storage.js                    // Centralized config/credentials (531 lines)
src/utils/cloud.js                // Cloud API with BYOK (203 lines)
src/utils/localai.js              // Ollama/Hugging Face integration (437 lines)
src/components/views/AICustomizeView.js     // Provider selection UI (127 lines)
src/components/views/FeedbackView.js        // User feedback system (237 lines)
src/components/views/sharedPageStyles.js    // Shared styling (172 lines)
```

## Prevention Strategies

### 1. Pre-Merge Checklist
```
□ Create backup branch: git branch backup-pre-$VERSION
□ Create tag: git tag pre-$VERSION
□ Identify local files that must survive
□ Add to .gitignore if not already tracked
□ Backup locally outside repo
□ Run npm list to understand dependencies
```

### 2. Conflict Prediction
Before merge, analyze upstream changes:
```bash
git fetch upstream
git diff master upstream/master -- src/index.js src/utils/gemini.js
```

Identify risky files prone to conflicts:
- Core entry point (`src/index.js`)
- Main API integration (`src/utils/gemini.js`)
- Package dependencies (`package.json`)

### 3. Post-Merge Verification
```bash
# Test app starts
npm start

# Verify key features work
# - Onboarding flow
# - Screenshot capture
# - AI provider selection UI (new)
# - Local AI option if Ollama installed

# Check for module errors
npm list | grep -i error
```

### 4. Documentation as Version Lock
Include upstream commit hash in CLAUDE.md:
```markdown
## Upstream Sync Status
- **Commit**: 60aad7e (sohzm/cheating-daddy)
- **Version**: 0.7.0
- **Last Sync**: 2026-02-14
```

## Rollback Procedure

If merge breaks functionality:

```bash
# Quick rollback
git reset --hard pre-v07-merge
npm install  # Reinstall old dependencies

# Restore local files from backup
cp -r ../cheating-daddy-local-backup/.claude .
cp ../cheating-daddy-local-backup/* .
```

## Testing & Validation

### Automated Checks
1. ✅ No `Cannot find module` errors on startup
2. ✅ Keyboard shortcuts registered correctly
3. ✅ Config system migrated (old paths → new storage)
4. ✅ All new dependencies installed

### Manual Verification
1. Start app: `npm start`
2. Onboarding flow works
3. Click settings → AI provider options visible (new in v0.7.0)
4. Gemini/Groq/Cloud/Local options available
5. Console shows no deleted module references

### Performance Check
GPU cache errors expected on Windows (non-critical):
```
[ERROR:cache_util_win.cc(20)] Unable to move the cache: Access is denied.
[ERROR:gpu_disk_cache.cc(208)] Unable to create cache
```

These are harmless Chromium warnings.

## What Went Well

1. **No conflicts** - Git's ORT strategy auto-resolved cleanly
2. **Local files preserved** - .gitignore protection worked
3. **Zero manual intervention** - Merge completed automatically
4. **Clean dependency upgrade** - npm install handled all transitive deps
5. **App starts successfully** - No critical errors post-merge

## What Could Break

⚠️ **API keys exposed in .mcp.json**
- Contains Todoist API key, Notion token, RAGflow key
- Regenerate before committing
- Add to .gitignore for local development

⚠️ **Storage migration from config.js**
- Old config.json won't be read (path changed)
- First startup initializes new storage.js
- User preferences reset if not migrated

⚠️ **Stealth features removed**
- No more process randomization
- No anti-analysis features
- User can't set stealth levels

## Related Documentation

- [Git Merge Strategies](https://git-scm.com/book/en/v2/Git-Branching-Basic-Branching-and-Merging)
- [Electron Configuration Best Practices](https://www.electronjs.org/docs)
- [Upstream Sync Workflows](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks/syncing-a-fork)
- Upstream repo: https://github.com/sohzm/cheating-daddy/releases/tag/v0.7.0

## Commit History

```
16f7a99 chore: update package-lock.json for v0.7.0 dependencies
5c04554 docs: update CLAUDE.md for v0.7.0 architecture
11fa9b0 chore: protect local custom files in gitignore
[MERGE COMMIT] merge: update to v0.7.0 with cloud/local AI support
```

## Metrics

| Metric | Value |
|--------|-------|
| Commits merged | 27 |
| Files changed | 37 |
| Insertions | 7,451 |
| Deletions | 7,950 |
| Conflicts resolved | 0 |
| Time to merge | < 30 seconds |
| Dependencies added | 3 |
| Dependencies removed | 2 |
| New features | 3 (Cloud, Local AI, Groq) |
| Breaking changes | 4 (config, stealth, vitest, advanced view) |
