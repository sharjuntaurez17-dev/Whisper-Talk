# WhisperTalk — Build Plan

**Goal:** a voice-dictation app in the same category as Wispr Flow. You hold a hotkey, speak, and clean, formatted text appears in whatever app you're typing in (email, Slack, docs, code editor). A marketing and account website goes with it.

This is a plan only. No app code has been written yet.

---

## 1. What we're building (MVP scope)

| # | Feature | Notes |
|---|---------|-------|
| 1 | **Push-to-talk global hotkey** | Hold a key (e.g. `Fn` / `Ctrl+Space`), speak, let go. |
| 2 | **Speech-to-text** | Whisper-class model (cloud API first, local option later). |
| 3 | **AI cleanup** | An LLM removes filler words ("um", "like"), fixes punctuation, and handles self-corrections ("at 3, no, 4pm" becomes "at 4pm"). |
| 4 | **Insert into any app** | Copy to the clipboard, simulate paste, then restore the user's previous clipboard. |
| 5 | **Floating mini-widget** | A small pill that shows recording, processing, and done states. |
| 6 | **Personal dictionary** | Names, jargon, and product terms the model should spell correctly. |
| 7 | **History** | A local list of past dictations with copy and retry. |
| 8 | **Website** | Landing page, download links, sign-in, and a usage/billing dashboard. |

**Later (v2+):** per-app tone (casual in Slack, formal in email), voice "command mode" ("make this shorter"), snippets/text shortcuts, more languages, mobile keyboard, team plans, and offline/local mode.

---

## 2. Recommended tech stack

| Layer | Choice | Why |
|-------|--------|-----|
| Desktop app | **Tauri 2** (Rust core + React/TypeScript UI) | Small binaries, a native global hotkey, mic and paste support on macOS and Windows. Lighter than Electron. |
| Speech-to-text | **Groq Whisper-large-v3** or **OpenAI / Deepgram** API, with **whisper.cpp** later for local/offline use | Cloud APIs are fast and easy. Local mode is good for privacy and a free tier. |
| Text cleanup | **Claude Haiku** (latest) via the Anthropic API | Cheap and fast enough for sub-second cleanup, and good at following formatting rules. |
| Backend | **Supabase** (auth, Postgres, edge functions) | Holds the API keys server-side, meters usage, stores the dictionary and settings. |
| Website | **Next.js** on **Vercel** | Landing page, docs/instructions, dashboard, Stripe checkout. |
| Payments | **Stripe** | Free tier plus a Pro subscription. |
| Design | **Figma** | Logo, widget, and landing page mockups before coding. |

**Data flow:**
```
Hotkey down → record mic (16 kHz)
Hotkey up   → upload audio → Supabase edge fn → STT API → raw text
            → LLM cleanup (with dictionary + app context) → clean text
            → back to desktop app → paste into the focused app
```
Target end-to-end time: **under 1.5 s** after you release the key.

---

## 3. Tools, connectors, plugins and skills we'll use

These are available in this Claude Code environment:

| Tool | Used for |
|------|----------|
| **GitHub** (connected) | This repo, PRs, issues, CI with GitHub Actions to build the Mac and Windows installers |
| **Supabase** connector | Create the project, DB tables, auth, and edge functions |
| **Vercel** connector | Deploy the website, set env vars, add a custom domain |
| **Figma** connector | Logo, app widget, and landing page design. Design to code. |
| **Canva** connector | Social and launch graphics |
| **Notion** connector | Roadmap and task tracking (optional) |
| Skills: `claude-api`, `frontend-ui-engineering`, `security-and-hardening`, `planning-and-task-breakdown`, `shipping-and-launch`, `test-driven-development` | Guide each phase |

> Note: some plugin servers (Linear, Slack, Asana, GitHub plugin, Figma plugin) failed to connect in this session because of the network proxy. The main connectors listed above are the ones that matter here.

---

## 4. Phased roadmap

| Phase | Deliverable | Rough effort (with AI help) |
|-------|-------------|-----------------------------|
| **0. Setup** | Repo structure (`/desktop`, `/web`, `/supabase`), Figma brand and logo, Supabase project | 1–2 days |
| **1. Core loop** | Tauri app: hotkey, record, STT, paste (no cleanup yet), macOS first | 1 week |
| **2. AI cleanup** | LLM prompt for fillers, punctuation, self-corrections, and dictionary | 3–5 days |
| **3. Polish** | Floating widget, settings, history, onboarding (mic and accessibility permissions) | 1 week |
| **4. Website + accounts** | Next.js landing page, instructions/docs, login, usage dashboard, Stripe | 1 week |
| **5. Windows + release** | Windows build, code signing, auto-update, GitHub Actions release pipeline | 1 week |
| **6. v2 features** | Per-app tone, command mode, snippets, local whisper.cpp mode | ongoing |

---

## 5. Pros and cons

### Pros
- **Proven demand.** Voice typing at roughly 3× keyboard speed is a validated product category, and several competitors exist (Wispr Flow, Superwhisper, MacWhisper, Aqua Voice).
- **The building blocks exist.** Whisper (open source) and cheap LLM APIs mean you don't train any models.
- **Fast to build with AI ("vibe coding").** A working MVP is realistic in a few weeks.
- **Room to differentiate:** a privacy-first local mode, a lower price, better support for Indian languages and accents, open-source pieces, or a niche (developers, doctors, lawyers).
- **Low starting cost.** The main costs are pay-per-use APIs, and the free tiers of Supabase and Vercel cover the start.

### Cons and risks
- **OS-level work is the hard part, not the AI.** Global hotkeys, simulated paste, and macOS Accessibility/Microphone permissions are fiddly. Vibe-coded Rust/native code needs careful testing on real machines.
- **Code signing costs money.** Apple Developer is about $99/year, and a Windows certificate is about $100–400/year. Without them, users get scary "unidentified developer" warnings.
- **Latency is everything.** If it feels slower than typing, people quit. That needs streaming, a fast STT provider, and short prompts.
- **API costs grow with users.** STT plus LLM per minute of speech is cheap but not free, so the free tier must be capped.
- **Privacy and trust.** You're recording users' voices and reading what they type, so you need a clear privacy policy, encryption, and no audio retention by default.
- **Crowded market.** Well-funded competitors already exist, so you need a clear reason for people to switch.
- **Vibe-coding risk.** AI-generated code can hide security bugs, for example API keys shipped inside the desktop app. Keys must stay server-side, and the code should be reviewed before launch.

---

## 6. What we **cannot** copy from Wispr Flow

| Can't copy | Why | What we do instead |
|------------|-----|--------------------|
| **Their name, logo, branding, and look** | Trademark. "Wispr" / "Wispr Flow" are their marks. | Our own brand and design. See the naming note below. |
| **Their source code** | It's closed-source and copyrighted. Reverse-engineering their app breaks their Terms of Service. | Write our own code from open building blocks (Whisper, Tauri). |
| **Their website text, screenshots, images, and UI design** | Copyright and trade dress | Original copy and a new design made in Figma |
| **Their AI prompts, fine-tuned models, and training data** | Proprietary trade secrets | Our own cleanup prompt and our own evaluation set |
| **Their user data or customer lists** | Private, and illegal to take | Grow our own users |
| **Any patented techniques** | They may hold patents | Check before launch if you're going big |

**What we *can* do legally:** build the same *kind* of product. Ideas and features like push-to-talk, AI cleanup, and paste-anywhere aren't protected. Hold-to-dictate is a generic pattern that many apps use. Open-source Whisper is free to use (MIT license).

### Naming note
"WhisperTalk" is fine to use as a working name. Before a public launch:
- "Whisper" is also OpenAI's model name, and "Wispr" sounds very close in the *same product category*. That's a possible trademark-confusion risk.
- Do a trademark search (USPTO / WIPO / India IP) and check domain availability. Consider a more distinct final brand.

---

## 7. Immediate next steps
1. **Make this repo private:** GitHub, then **Settings**, then **Danger Zone**, then **Change visibility** to **Private**. It's currently **public**.
2. Confirm the stack (Tauri + Supabase + Vercel + Claude Haiku) or tell me your preference.
3. Decide macOS-first or Windows-first.
4. Phase 0: scaffold `/desktop`, `/web`, and `/supabase` and make the brand in Figma.
