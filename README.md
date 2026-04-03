# uigenfix
UIgen is an example program used in training for Claude Code, attached is the documentation of the debugging and the final code that was working on Windows 11.  Some oversights from Mac/Linux to Windows training was overlooked and fixed to help others.
# Anthropic Claude Code Course — Windows 11 Compatibility Report

**Prepared by:** [Owlstorm.ai](https://www.owlstorm.ai)  
**Date:** March 17, 2026  
**Platform Tested:** Windows 11 (clean install)  
**Course:** Anthropic Claude Code in Action — uigen project  
**Total Issues Found:** 17  

---

## ⚠️ Key Finding

> **This course was never tested on Windows.**

A Windows 11 user following this course exactly as written will encounter blocking errors at every major setup stage. The evidence strongly indicates the course developers had no familiarity with the Windows operating environment at the time of publication.

**Most critically — Issue 17 means the app does not work on ANY platform as published.** The uigen project was written for Vercel AI SDK v4/v5 but ships with v6 installed, introducing five silent breaking changes that make component generation completely non-functional for every user on every operating system.

---

## Issues Summary

| # | Issue | Category | Severity | Status |
|---|---|---|---|---|
| 1 | Node.js installer triggers Chocolatey conflict via suggested Windows method | Instructions | Medium | ✅ Resolved |
| 2 | Wrong Claude Code install command provided for Windows | Instructions | High | ✅ Resolved |
| 3 | Claude Code PATH not configured after installation | Instructions | Medium | ✅ Resolved |
| 4 | No guidance to run PowerShell as Administrator | Instructions | Medium | ✅ Resolved |
| 5 | .cjs file association — Windows opens file dialog instead of running app | Instructions | Medium | ✅ Resolved |
| 6 & 7 | NODE_OPTIONS syntax incompatible with Windows — cross-env missing, PowerShell workaround required | Project Files | High | ✅ Resolved |
| 8 | Missing `@ai-sdk/react` dependency — app crashes on load | Project Files | High | ✅ Resolved |
| 9 | `.env` file — no warning about quote characters breaking API key | Instructions | Medium | ✅ Resolved |
| 10 | `zod` version conflict — `@ai-sdk/react` install fails without `--legacy-peer-deps` | Project Files | High | ✅ Resolved |
| 11 | `zod` not in `package.json` at all — cascading module not found errors | Project Files | High | ✅ Resolved |
| 12 | `NODE_OPTIONS` persists in PowerShell session — breaks subsequent npm commands | Instructions | High | ✅ Resolved |
| 13 | Code bug — `input.trim()` called on undefined crashes app on load (all platforms) | Project Files | High | ✅ Resolved |
| 14 | Infinite React render loop — complete Windows system freeze, hard reboot required | Project Files | **Critical** | ✅ Resolved |
| 15 | `#` memory shortcut does not automatically update `CLAUDE.md` — course inaccuracy | Instructions | Medium | ✅ Identified |
| 16 | `Ctrl+V` clipboard image paste unsupported on Windows 11 — core course workflow unavailable | Instructions | Medium | ✅ Identified |
| 17 | AI SDK v6 breaking changes — 5 silent API changes make app completely non-functional on all platforms | Project Files | **Critical** | ✅ Resolved |

---

## Detailed Issues

### Issue 1 — Node.js Installer Triggers Chocolatey Conflict
**Severity:** Medium

The course links to `https://nodejs.org/en/download`. The suggested Windows installation methods on that page trigger a Chocolatey (Windows package manager) installation step. On machines where Chocolatey is already installed this causes a conflict that blocks Node.js from installing.

**Fix:**  
Use the direct native Windows MSI installer and uncheck "Tools for Native Modules":
```
https://nodejs.org/dist/v22.22.1/node-v22.22.1-x64.msi
```

**Recommended course fix:** Link directly to the MSI installer. Add a warning to uncheck "Tools for Native Modules" during setup.

---

### Issue 2 — Wrong Claude Code Install Command for Windows
**Severity:** High

The course provides this CMD command for Windows:
```cmd
curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd && del install.cmd
```
This does not work on Windows 11. The correct method is PowerShell:
```powershell
irm https://claude.ai/install.ps1 | iex
```

**Fix:** Use the PowerShell command above in an elevated terminal (Run as Administrator).

**Recommended course fix:** Replace the CMD command with the PowerShell command. Add a note to run as Administrator.

---

### Issue 3 — Claude Code PATH Not Configured After Installation
**Severity:** Medium

After running the installer, `claude` is not recognised as a command:
```
claude : The term 'claude' is not recognized as the name of a cmdlet...
```
The installer places `claude.exe` in `C:\Users\[username]\.local\bin` but does not register this in the Windows PATH.

**Fix:**
```powershell
# Step 1 — Find claude.exe
Get-ChildItem -Path $env:USERPROFILE -Recurse -Filter "claude.exe" -ErrorAction SilentlyContinue

# Step 2 — Add to PATH permanently
[Environment]::SetEnvironmentVariable("PATH", $env:PATH + ";C:\Users\[username]\.local\bin", [EnvironmentVariableTarget]::User)

# Step 3 — Restart PowerShell
```

**Recommended course fix:** Add a Windows-specific PATH configuration section to the Claude Code setup lesson.

---

### Issue 4 — No Guidance to Run PowerShell as Administrator
**Severity:** Medium

Multiple setup steps require elevated PowerShell. The course makes no mention of this Windows requirement. On macOS/Linux terminals run with sufficient permissions by default — Windows does not.

**Fix:** Right-click Windows Terminal → "Run as Administrator"

**Recommended course fix:** Add this as a prerequisite step at the start of all Windows setup instructions.

---

### Issue 5 — .cjs File Association Dialog
**Severity:** Medium

When the app attempts to load `node-compat.cjs` via `NODE_OPTIONS`, Windows intercepts it as a file-open request and shows an "Open with..." dialog repeatedly. The `.cjs` extension is not registered as a known file type in a default Windows 11 installation.

**Fix:** Dismiss the dialog and apply the `cross-env` fix (Issue 6 & 7).

**Recommended course fix:** Add a note warning Windows users about this dialog and that it should be dismissed.

---

### Issues 6 & 7 — NODE_OPTIONS Incompatible with Windows / cross-env Missing
**Severity:** High

Running `npm run dev` produces:
```
'NODE_OPTIONS' is not recognized as an internal or external command
```

The `package.json` scripts use Unix/macOS syntax for inline environment variables which Windows does not support. `cross-env` was not included in the project.

**Fix — Step 1:** Install cross-env:
```powershell
npm install --save-dev cross-env
```

**Fix — Step 2:** Update all 4 affected scripts in `package.json`:
```json
"dev": "cross-env NODE_OPTIONS='--require ./node-compat.cjs' next dev --turbopack",
"dev:daemon": "cross-env NODE_OPTIONS='--require ./node-compat.cjs' next dev --turbopack > logs.txt 2>&1 & echo 'Server started'",
"build": "cross-env NODE_OPTIONS='--require ./node-compat.cjs' next build",
"start": "cross-env NODE_OPTIONS='--require ./node-compat.cjs' next start"
```

**Note:** Even after this fix, `npm run dev` still did not work on Windows 11 due to the `.cjs` file association issue. The working workaround is:
```powershell
$env:NODE_OPTIONS='--require ./node-compat.cjs'; npx next dev --turbopack
```

**Recommended course fix:** Add `cross-env` to `devDependencies` and prefix all 4 scripts before publishing. Document the PowerShell workaround for Windows users.

---

### Issue 8 — Missing @ai-sdk/react Dependency
**Severity:** High

After startup, the app crashes on every page:
```
Module not found: Can't resolve '@ai-sdk/react'
GET / 500 in 24076ms
```

`@ai-sdk/react` is used throughout the codebase but is not listed in `package.json` dependencies.

**Fix:**
```powershell
npm install @ai-sdk/react --legacy-peer-deps
```

**Recommended course fix:** Add `@ai-sdk/react` to `package.json` dependencies before publishing.

---

### Issue 9 — .env File Quote Characters Break API Key
**Severity:** Medium

The course instructs users to add their API key to `.env`. Windows Notepad users may naturally add quotes:
```
ANTHROPIC_API_KEY="sk-ant-your-key"  ❌
```
This causes authentication failures as the quotes become part of the key value.

**Fix:**
```
ANTHROPIC_API_KEY=sk-ant-your-key  ✅
```

**Recommended course fix:** Add a clear example showing the correct format. Add an explicit warning: "Do not add quote marks around your API key."

---

### Issue 10 — zod Version Conflict
**Severity:** High

Installing the missing `@ai-sdk/react` produces:
```
npm error ERESOLVE could not resolve
npm error peer zod@"^3.25.76 || ^4.1.8" from ai@6.0.116
npm error Found: zod@3.25.67
```

**Fix:**
```powershell
npm install @ai-sdk/react --legacy-peer-deps
```

**Recommended course fix:** Update `zod` to v4 in `package.json` and pin `@ai-sdk/react` at a compatible version before publishing.

---

### Issue 11 — zod Not Listed in package.json
**Severity:** High

The entire `@ai-sdk` ecosystem depends on `zod` but it is completely absent from `package.json`. On macOS, `zod` is typically present from previous projects. On a clean Windows machine it simply does not exist, producing:
```
Module not found: Can't resolve 'zod'
Module not found: Can't resolve 'zod/v3'
Module not found: Can't resolve 'zod/v4'
```

This is the most revealing indicator that the project was never installed on a clean machine.

**Fix — Step 1:** Add to `package.json` dependencies:
```json
"zod": "^4.0.0"
```

**Fix — Step 2:** Delete and reinstall:
```powershell
$env:NODE_OPTIONS=""
Remove-Item -Recurse -Force node_modules
npm install --legacy-peer-deps
```

**Recommended course fix:** Add `zod` v4 to `package.json` before publishing. This is the single most impactful fix in this entire report.

---

### Issue 12 — NODE_OPTIONS Persists in PowerShell Session
**Severity:** High

After starting the dev server with the PowerShell workaround, subsequent `npm install` commands in the same session fail:
```
npm error Error: Cannot find module './node-compat.cjs'
npm error code MODULE_NOT_FOUND
```

The `$env:NODE_OPTIONS` variable set to start the app persists for the entire PowerShell session. Every subsequent Node.js process (including `npm`) also tries to load `node-compat.cjs` which doesn't exist in that context.

On macOS/Linux, inline `NODE_OPTIONS='...' command` syntax scopes the variable to a single command. PowerShell's `$env:` syntax sets it for the whole session.

**Fix:** Clear the variable before running npm commands:
```powershell
$env:NODE_OPTIONS=""
```

**Recommended course fix:** Add a prominent Windows warning to clear `NODE_OPTIONS` before running any npm commands after starting the dev server.

---

### Issue 13 — input.trim() Called on Undefined (All Platforms)
**Severity:** High

The app crashes on load with:
```
TypeError: Cannot read properties of undefined (reading 'trim')
at MessageInput (src\components\chat\MessageInput.tsx:43:41)
```

`input` is `undefined` on initial render. `.trim()` is called without null checking.

**Fix:** Change both instances of `input.trim()` in `MessageInput.tsx` to:
```tsx
input?.trim()
```

The `?.` optional chaining operator safely returns `undefined` instead of throwing if `input` is not yet defined.

**Recommended course fix:** Fix both instances in `MessageInput.tsx` before publishing. This affects all platforms.

---

### Issue 14 — Infinite React Render Loop / Complete Windows System Freeze
**Severity:** Critical

On first successful app load, the entire Windows system froze completely — mouse, keyboard, all programs unresponsive. A **hard reboot using the power button** was required to recover.

**Root cause:** `chat-context.tsx` had `fileSystem` in a `useEffect` dependency array:
```tsx
useEffect(() => {
  if (!projectId && messages.length > 0) {
    setHasAnonWork(messages, fileSystem.serialize());
  }
}, [messages, fileSystem, projectId]); // ❌ fileSystem causes infinite loop
```

`fileSystem.serialize()` creates a new object reference on every call, triggering the effect again infinitely. On macOS this typically causes a tab crash. On Windows with older hardware it consumed 100% CPU and froze the entire OS.

**Fix:** Remove `fileSystem` from the dependency array:
```tsx
}, [messages, projectId]); // ✅
```

**Recommended course fix:** Fix the dependency array in `chat-context.tsx` before publishing. Review all other `useEffect` hooks for similar issues.

---

### Issue 15 — # Memory Shortcut Does Not Update CLAUDE.md
**Severity:** Medium

The course teaches that prefixing a message with `#` in Claude Code automatically updates `CLAUDE.md`. This did not happen. When asked, Claude Code itself explained:

> *"What # actually does: It adds a note into the conversation context. It is not a documented system command that automatically writes to CLAUDE.md. There is no system-level enforcement."*

**Recommended course fix:** Clarify that `#` is a convention the model should act on, not a guaranteed system command. Instruct users to verify `CLAUDE.md` was updated and manually edit it if not.

---

### Issue 16 — Ctrl+V Clipboard Image Paste Unsupported on Windows
**Severity:** Medium

The course demonstrates using `Ctrl+V` to paste screenshots directly into Claude Code. This does not work on Windows 11.

Windows stores screenshots (Win+Shift+S) as raw bitmap data in the clipboard. Terminal applications cannot read this format. macOS handles clipboard images differently, making the workflow seamless on Mac but completely broken on Windows.

**Open GitHub issue:** https://github.com/anthropics/claude-code/issues/26679

**Windows workarounds:**
- Use Snipping Tool → Save As → reference the file path in Claude Code
- Drag and drop an image file directly into the Claude Code window
- Use ShareX which auto-saves screenshots to disk

**Recommended course fix:** Add Windows-specific alternatives for image sharing. Link to the open GitHub issue so users know it's being tracked.

---

### Issue 17 — AI SDK v6 Breaking Changes (All Platforms)
**Severity:** Critical

**This is the most significant finding in the report.** The uigen codebase was written for Vercel AI SDK v4/v5 but `package.json` specifies `"ai": "^6.0.116"` which installs v6. Version 6 introduced five major breaking API changes that make component generation completely non-functional for every user on every platform.

#### Breaking Change 1 — Messages Not Rendering
`Message` type replaced by `UIMessage` in v6. Old code used `message.content` fallback which always returns null in v6 (v6 messages only have `.parts`).

#### Breaking Change 2 — Chat Never Sends Data to Server
In v6, `useChat` no longer accepts `api`, `body`, or `initialMessages` as direct options — they are silently ignored. Every chat submission went nowhere. Fix: use `DefaultChatTransport` with the `transport` option.

#### Breaking Change 3 — Preview Never Updates
`onToolCall` callback in v6 only fires for client-side tools. Since uigen tools are server-side, it was never called and the virtual file system stayed empty — no component ever rendered. Fix: watch `messages` in a `useEffect` and apply tool operations to the client VFS.

#### Breaking Change 4 — Tool Display Broken
In v6, tool parts have type `"tool-{toolName}"` (e.g. `"tool-str_replace_editor"`) instead of `"tool-invocation"`, and state is `"output-available"` instead of `"result"`.

#### Breaking Change 5 — API Route Format Changed
`toDataStreamResponse()` replaced by `toUIMessageStreamResponse()`. Tools updated from `parameters:` to `inputSchema:` with `jsonSchema()`. Hydration errors fixed with explicit panel IDs and `ssr: false`.

**Why this wasn't caught:** The `^` caret in `"ai": "^6.0.116"` installs the latest v6 automatically. Breaking changes produce no errors at install time — they fail silently at runtime.

**Recommended fix:**
- Update entire uigen codebase for AI SDK v6 APIs
- Pin `ai` to a specific version (remove the `^` caret)
- Run `npm test` before publishing any course materials
- Add CI/CD to catch dependency update failures

---

## Root Cause Summary

The missing `zod` dependency is the single most revealing indicator. Zod is a core dependency of the entire Vercel AI SDK ecosystem. Its absence from `package.json` can only be explained by the project never having been installed on a clean machine. On macOS, zod accumulates silently across projects. On a clean Windows machine, it simply isn't there.

**A single `npm install` run on a clean Windows machine before publishing would have caught:**
- Missing `zod`
- Missing `@ai-sdk/react`
- zod version conflicts
- All 5 AI SDK v6 breaking changes (via `npm test`)

---

## Complete Fix Checklist for Anthropic Course Team

### Project File Changes (update uigen.zip)
- [ ] Add `"zod": "^4.0.0"` to `package.json` dependencies
- [ ] Add `"@ai-sdk/react"` to `package.json` dependencies at correct pinned version
- [ ] Add `"cross-env"` to `devDependencies` and prefix 4 scripts
- [ ] Fix zod version conflict — pin compatible versions across all `@ai-sdk` packages
- [ ] Fix `input?.trim()` null safety bug in `MessageInput.tsx`
- [ ] Fix `useEffect` dependency array in `chat-context.tsx` — remove `fileSystem`
- [ ] Update entire codebase for AI SDK v6 — all 5 breaking changes
- [ ] Pin `ai` to specific version (remove `^` caret)

### Course Content Changes (update Skilljar)
- [ ] Replace CMD Claude Code install command with PowerShell command for Windows
- [ ] Add Windows PATH fix instructions for Claude Code post-install
- [ ] Add "Run as Administrator" guidance at start of all Windows setup steps
- [ ] Warn users to uncheck "Tools for Native Modules" during Node.js install
- [ ] Add `.env` format example showing no quotes around API key
- [ ] Add note about `.cjs` file association dialog on Windows
- [ ] Warn users to clear `$env:NODE_OPTIONS` before running npm commands
- [ ] Clarify that `#` memory shortcut is a convention, not a guaranteed system command
- [ ] Add Windows alternatives for `Ctrl+V` image paste with link to GitHub issue #26679

### Process Changes
- [ ] Test all course materials on a clean Windows 11 machine before publishing
- [ ] Run `npm test` against installed dependency versions before publishing
- [ ] Add CI/CD check that runs tests on dependency updates

---

## Why This Matters

Anthropic's target sectors for Claude adoption — manufacturing, fintech, and medical services — are enterprise environments. Enterprise environments are overwhelmingly Windows-based.

A developer course that:
- Completely fails on Windows at every setup stage
- Does not generate components on any platform due to AI SDK v6 incompatibilities
- Causes a complete Windows system freeze requiring a hard reboot

...does not just inconvenience users. It actively undermines confidence in Anthropic's developer tooling at the most critical first impression.

---

## About Owlstorm.ai

Owlstorm.ai is a boutique AI consultancy specialising in responsible AI deployment in manufacturing, fintech, and medical services. We are currently completing the Anthropic partner application process. This report was prepared as part of our technical onboarding and reflects our commitment to contributing practical, real-world feedback to the Anthropic partner ecosystem.

We would be happy to assist Anthropic in validating a corrected version of the course materials on Windows 11.

**Website:** [www.owlstorm.ai](https://www.owlstorm.ai)

---

*Prepared by Owlstorm.ai - Paul Wojnicki— March 17, 2026*
