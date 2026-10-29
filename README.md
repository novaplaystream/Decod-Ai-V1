# Dcode AI — coding agent for VS Code

Use **any OpenAI-compatible API** (NVIDIA build.nvidia.com, OpenRouter, Groq, OpenAI, Ollama, LM Studio) as a real coding agent inside VS Code: it can read your files, search your code, edit files and run commands — always asking permission first, unless you switch it to full-auto.

## Features

- Sidebar chat panel (Activity Bar → **Dcode AI**)
- **Real file access**: `list_files`, `read_file`, `search_code`, `write_file`, `replace_in_file`, `run_command`
- **Attachments**: images, audio, video, and any text/code/binary file
- **Persistent chat history**: every chat is saved to disk and restored after a VS Code restart, until you delete it
- **Permission modes**
  - `Ask before edits` — every edit and command needs an Allow click
  - `Auto edits, ask commands` — edits apply automatically, shell commands still ask
  - `Full auto` — nothing asks (use with care)
- Per-tool **Always allow** for the current session
- Streaming replies, markdown + code blocks, collapsible tool cards
- **Live UI preview** — the agent writes HTML/CSS/JS and a preview panel shows the page instantly, auto-reloading on every change; click any element to send it to chat
- **Copy or resend any message** — hover a message (yours or the agent's) and use the two icons to copy it or run it again
- **Interrupt by typing** — send a new message while the agent is working and the current run stops instantly; the new one starts right away
- **Auto-resume after a restart** — if VS Code is closed in the middle of a task, the extension continues that task by itself the next time VS Code opens (no click needed)
- **Compact icon chat box** — attach, preview, chat mode, new chat, delete, settings, stop and send all sit inside the input box as VS Code-style icons
- **Live running indicator** — spinner, elapsed time and the exact action it is doing, plus a status-bar item; warns you when nothing has happened for 40 s
- `@path/to/file` in a message to attach a file as context
- One-click model lists, grouped by saved provider profile (NVIDIA, Groq, Gemini, and others)
- **Up to 5 saved API profiles** — keep separate provider URLs, API keys and models (NVIDIA, Gemini, OpenRouter, OpenAI, Groq, local servers, or custom endpoints); selecting a model in another provider section switches to that profile. Keys are stored using VS Code SecretStorage.
- Auto-normalizing Base URL — pasting `https://integrate.api.nvidia.com` or a full `/chat/completions` URL both work

## Install

```bash
code --install-extension dcode-ai-0.9.1.vsix
```

Then reload VS Code.

## Setup (NVIDIA)

1. Get a key at <https://build.nvidia.com>
2. Open **Dcode AI** in the Activity Bar → **⚙**
3. Fill in:

   | Field | Value |
   | --- | --- |
   | Preset | NVIDIA (build.nvidia.com) |
   | Base URL | `https://integrate.api.nvidia.com/v1` |
   | API Key | `nvapi-...` |
   | Model | pick from the list, e.g. `moonshotai/kimi-k2.6` |

4. Save. The model list loads automatically.

## Multiple API profiles

Open **⚙ → Saved API profiles**, then use **+ Add** to create profiles for different providers or accounts (up to ten). Give each profile a name, choose a preset or enter a custom Base URL, add its API key, fetch/select its models, and save. Switching the profile loads its own key, model and model list; API keys are stored with VS Code SecretStorage. Existing single-provider settings are imported as the first profile automatically.

## Other providers

| Provider | Base URL |
| --- | --- |
| Gemini (Google AI Studio) | `https://generativelanguage.googleapis.com/v1beta/openai/` |
| OpenRouter | `https://openrouter.ai/api/v1` |
| Groq | `https://api.groq.com/openai/v1` |
| OpenAI | `https://api.openai.com/v1` |
| Ollama (local) | `http://localhost:11434/v1` |
| LM Studio (local) | `http://localhost:1234/v1` |

## Attachments

Click the **paperclip icon**, drag & drop files onto the panel, or paste an image from the clipboard.

| Type | How it is sent |
| --- | --- |
| Image (`png`, `jpg`, `gif`, `webp`, …) | Sent as an image part (`image_url` data URL) — needs a vision model |
| Audio (`mp3`, `wav`, `ogg`, `m4a`, …) | Sent as `input_audio` — needs an audio-capable model |
| Video (`mp4`, `mov`, `webm`, `mkv`, `avi`) | 6 frames extracted with **ffmpeg** and sent as images |
| Text / code | Inlined as a `<file>` block |
| Other (pdf, zip, …) | Only the file name is sent |

Limits: 25 MB per file. If the model rejects images or audio, the request is retried automatically with text only.

Video frame extraction requires `ffmpeg` on your `PATH`. Without it, only the file name is sent.

## Chat history

Every conversation is stored on disk in the extension's global storage
(`<userData>/globalStorage/local.dcode-ai/sessions.json`).

- Upgrading from the old **Universal API Chat** install is automatic: saved chats and settings (API key, base URL, model) are copied over on first start; the old folder is left untouched
- Chats survive window reloads and VS Code restarts
- The chat dropdown in the header switches between saved chats
- **＋** creates a new chat, **trash icon** deletes the current one (with confirmation)
- Chat titles are taken from the first message
- Up to 60 chats, 300 messages and 600 UI items per chat are kept
- Attachments are not stored as raw data after a restart — only a note that they existed (keeps the file small)

## Copy, resend and interrupting

- Hover any message: **copy** copies its exact text (assistant answers keep their markdown), and a check mark confirms it
- **resend** (arrow icon) puts the same message in as a new turn — works on your own messages and on the agent's answers
- While the agent is working the send button stays visible: typing and pressing Enter **stops the current run at that instant** and starts your new message. Nothing already written to your files is undone, and the partial answer stays in the chat
- Both buttons also work on messages restored from older sessions

## Auto-resume after closing VS Code

If VS Code is closed (or crashes) while the agent is still working, the unfinished
task is remembered on disk. The next time VS Code starts, the extension activates
in the background (`onStartupFinished`) and continues that same chat on its own —
no click, no re-typing.

- A `Resuming the task that was interrupted when VS Code closed…` note marks the spot
- Resuming only happens when the interrupted chat still exists, the work is less than **12 hours** old, and the same workspace folder is open (it will never edit a different project)
- Pressing **Stop** during the resumed run cancels it for good
- It retries at most **3 times** if VS Code keeps closing mid-task, so a broken run cannot loop forever
- If the API key or model is missing, the task waits and resumes on a later start once they are set

## Scrolling

The message list has its own scrollbar on the right edge, so long chats never push the input box off-screen.

- Scroll with the mouse wheel, trackpad, or by dragging the scrollbar thumb
- Scrolling up **stops the auto-scroll** — new tokens arriving will not yank you back down
- A round **↓** button appears in the bottom-right corner while you are reading older messages; click it to jump to the newest message
- When you send a message the view snaps to the bottom automatically

## Running indicator

While the agent works you always see what is happening:

- A bar above the input shows a spinner, the current action (`Reading src/file.ts`, `Editing …`, `Running …`, `Waiting for your approval …`) and a live elapsed timer
- If nothing arrives for **40 seconds** the bar turns amber and says `no updates for Ns`, so a frozen request is obvious instead of silently hanging
- A spinning item appears in the VS Code **status bar** (`Working 12s`) and can be clicked to focus the chat
- The activity-bar icon gets a dot badge
- When it finishes: `Finished in 8.4s`, `Stopped after 3.1s`, `Failed after 2.0s` or `Paused at step limit`
- While running, the send button becomes a **stop** button

## Composer icons

The chat box holds every control as a compact icon, VS Code style:

| Icon | Action |
| --- | --- |
| paperclip | attach a file / image / audio / video |
| `Ask` / `Auto-edit` / `Full auto` | permission mode |
| **+** | new chat |
| trash | delete current chat |
| gear | settings (API key, base URL, model list) |
| monitor | open the live UI preview |
| square | stop the run (only while working) |
| ↑ | send (Enter; Shift+Enter for a new line) |

## Live UI preview

Ask the agent to build or change a screen and it writes real HTML/CSS/JS, then opens a **UI Preview** panel next to the editor. The panel is a live window into the page:

- **Auto-open** — the moment the agent writes or edits an HTML page the preview opens by itself (turn it off with `previewAutoOpen`).
- **Auto-reload** — every file the agent changes refreshes the page. CSS edits are hot-swapped without losing scroll or form state; HTML/JS edits do a full reload.
- **Changes list** — the strip at the bottom of the panel shows every UI file the agent touched, with time and *create*/*edit*. Click a row to open that file in the editor.
- **Inspect (element picker)** — press **Inspect**, then click any element in the page. Its selector (e.g. `#save`) jumps into the chat composer as a 🎯 chip. Now just type *"remove this"* or *"make it blue"* — the agent receives the exact file + selector and edits only that element. The preview reloads so you see the result immediately.
- **Browser** — open the same page in your normal browser if you want devtools.

The panel is served from a throwaway `127.0.0.1` server bound to the workspace folder only; nothing is exposed on the network and the server stops when VS Code closes.

## When the provider returns an error

**HTTP 500 "Failed to generate completions"** (common on NVIDIA build.nvidia.com) usually means the saved tool history got out of sync, the model is overloaded, or the chat is too long. The extension now:

- repairs the tool history before every request (missing tool results are filled in, orphan results dropped, `null` content replaced)
- retries the step once **without tools** so you still get an answer instead of a dead end
- tells you what to do next when it still fails — retry, press **+** for a new chat, or switch model

## Permissions

The agent cannot touch anything outside the open workspace folder. Writes, edits and shell commands are gated by the mode selector in the composer. Tool cards show what is about to change — click a card header to expand the details, or **Deny** to stop that step.

## Settings (`dcodeAi.*`)

| Setting | Default | Meaning |
| --- | --- | --- |
| `apiBaseUrl` | `https://integrate.api.nvidia.com/v1` | OpenAI-compatible base URL |
| `apiKey` | — | Provider key |
| `model` | `meta/llama-3.3-70b-instruct` | Model ID |
| `temperature` | `0.3` | Sampling temperature |
| `maxSteps` | `0` | Max tool steps per message. `0` = no limit (recommended) — the agent keeps working until the task is done or you press Stop. |
| `systemPrompt` | — | Override the built-in agent prompt |
| `previewAutoOpen` | `true` | Open the live UI preview automatically when the agent writes an HTML page |
| `previewEntry` | — | HTML page to preview (e.g. `index.html`); empty = auto-detect |

## Notes

- Models without function-calling support fall back to plain chat automatically.
- Streaming with SSE; non-streaming gateways also work.
- There is no step limit by default. A safety guard stops the run only if the model repeats the exact same tool call 11 times in a row, which indicates a stuck loop rather than real work.
- The `run_command` tool runs in the workspace root with a 120 s timeout.

## Logo

The Dcode AI mark is generated, not hand-drawn: `scripts/make-logo.js` renders a 3D extruded "D" with a neon cyan-to-violet gradient on a dark glass tile using signed distance fields, then encodes the PNG itself (no image libraries). Any resolution comes from the same design — edges are anti-aliased analytically, so the 4K file is razor sharp rather than upscaled.

```bash
node scripts/make-logo.js              # 512, 128 and 64 px assets used by the extension
node scripts/make-logo.js --4k         # 4096 px logo
node scripts/make-logo.js --sheet      # contact sheet of variants a, b and c
node scripts/make-logo.js --variant=b  # another mark style
node scripts/make-logo.js --ascii      # print a text preview in the terminal
```

| File | Use |
| --- | --- |
| `media/logo.png` (512) | full logo |
| `media/icon.png` (128) | extension / marketplace icon |
| `media/logo-64.png` (64) | chat panel header |
| `media/mark-24.svg` (24) | Activity Bar icon, monochrome so it follows the theme |
| `media/logo-4k.png` (4096) | print / social / high-DPI |
| `media/logo-sheet.png` | contact sheet comparing the three variants |
