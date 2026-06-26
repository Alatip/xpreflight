# xpreflight

A pre-flight reality-check for X (Twitter) drafts. You paste a draft; the app cleans
typos, fact-checks the claims against the web, and returns a verdict — **CLEARED**
(safe to post) or **HOLD** (has a wrong or risky-unverified claim). Posting stays
manual. The point is to stop me from firing off confident but unverified takes.

- **Live:** https://alatip.github.io/xpreflight/
- **Hosting:** GitHub Pages, served from `main` branch root.
- **Deploy:** push to `main` → Pages redeploys automatically (~1 min). No CI, no build step.

## Architecture (and why it is this way)

- **Single static file: `index.html`.** Vanilla JS in one IIFE, plain CSS with
  `:root` variables, no framework, no bundler, no dependencies except Google Fonts
  via CDN `<link>`. This must keep running as a plain file on GitHub Pages.
  → *Do not add a build system, npm, or a framework unless explicitly asked.*
- **File must stay named `index.html`** so Pages serves it from the repo root.
- **No backend, on purpose.** Hosting is free and the author never pays for anyone
  else's usage because the API key is **bring-your-own (BYO)**, entered by the user
  at runtime. Trade-off: the key is visible in the user's *own* browser — acceptable
  for a personal/BYO tool. → *If this ever becomes a paid product, it needs a backend
  proxy so keys aren't client-side. Until then, keep it backendless.*
- **Two providers**, chosen at runtime:
  - Anthropic → `POST https://api.anthropic.com/v1/messages`, header
    `anthropic-dangerous-direct-browser-access: true`, web-search tool `web_search_20250305`.
  - OpenAI → `POST https://api.openai.com/v1/responses`, web-search tool `web_search`.
  - Model field is a free-text input with suggestions, because provider model names
    change — don't hardcode a single model as the only option.
- **Posting is via X web intent**: `https://twitter.com/intent/tweet?text=<encoded>`.
  → *Do not integrate the X/Twitter API or OAuth.* The web intent keeps a human in the
  loop (you confirm in X's own composer), which is the entire reason the tool exists.
- **i18n:** UI + fact-check note language switch between **EN (default)** and **RU**,
  via the toggle top-right. Strings live in the `I18N` object. Keep EN as default.
- **Persistence:** `localStorage`, all keys prefixed `pf_`:
  `pf_lang`, `pf_provider`, `pf_key_anthropic`, `pf_key_openai`,
  `pf_model_anthropic`, `pf_model_openai`, `pf_websearch`, `pf_draft`,
  `pf_consent` (GA4 analytics consent: `granted` | `denied`).
  All `localStorage` access is wrapped in try/catch so it degrades silently.

## Guardrails — do not break these

- **Never hardcode or commit an API key.** Keys are user-supplied at runtime and live
  only in the browser. The repo is **public** — nothing secret goes in the file.
- Keep it a **single self-contained static `index.html`** (see above).
- Keep **posting manual** via web intent; no X API.
- Preserve **EN-default + RU toggle** and the `localStorage` persistence behavior.
- Web search is a **paid provider tool** — keep it behind the existing checkbox, off-path
  when unchecked.

## Conventions

- Match the existing terse, dependency-free style. No new libraries without a reason.
- Show me a diff before committing. Use clear commit messages.
- After a change, remind me it auto-deploys on push to `main`.

## Known next steps (owner will drive)

- Analytics (GA4 or a cookieless option like Plausible/GoatCounter/Cloudflare). Owner
  will provide the Measurement ID / snippet — a Measurement ID (`G-XXXXXXXXXX`) is public
  and safe in the repo, unlike API keys.
- Optional later: light/dark theme, Open Graph meta tags for nicer X share cards.
