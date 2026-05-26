# CLAUDE.md

Project conventions and stack for the SME Interview Agent. Read this at the start of any Claude Code session in this repo.

## What this project is

A single-file static web app for **Purdue Course Design and Development (CDD)**. Faculty subject-matter experts (SMEs) complete a 5-section interview; the responses are sent to a Cloudflare Worker proxy, which calls the Anthropic Claude API and returns a structured Instructional Design (ID) brief.

Live at: `index.html` deployed wherever (currently TBD — see [README.md](README.md)).

## Stack

| Layer | Technology |
|---|---|
| Frontend | Single-file HTML / CSS / vanilla JavaScript |
| JS dialect | ES5-compatible (no arrow funcs, no `let`/`const` in the IIFE) — keeps the file open-in-browser friendly |
| Build | None. No bundler, no transpiler, no package.json |
| API proxy | Cloudflare Worker at `anthropic-proxyandrebeasley2012workersdev.andrebeasley2012.workers.dev` (separate repo) |
| LLM | `claude-sonnet-4-20250514` via the proxy |
| Fonts | DM Serif Display + DM Sans + DM Mono (Google Fonts) |

## File map

```
sme-interview-agent/
├── index.html       # Entire app: markup, styles, logic
├── README.md        # User-facing project docs
├── CLAUDE.md        # This file — conventions for Claude
├── STATUS.md        # Current state, recent work, next steps
└── BUILD_PLAN.md    # Architecture decisions and enhancement candidates
```

## Where things live in index.html

| What | Location |
|---|---|
| Interview sections (5) | `SECTIONS` array, [index.html:545](index.html) |
| Generation step labels (UI feedback only) | `GEN_STEPS` array, [index.html:590](index.html) |
| Proxy URL | `PROXY_URL` const, [index.html:599](index.html) |
| In-memory state | `state` object, [index.html:601](index.html) |
| API call + system prompt | `generateBrief()`, [index.html:752](index.html) |
| Markdown → HTML renderer | `mdToHtml()`, [index.html:799](index.html) |
| Event bindings | `init()`, [index.html:878](index.html) |

## Conventions

- **No build step.** The repo is intentionally one HTML file. Resist adding a bundler, framework, or npm tooling — the simplicity is a feature.
- **ES5-compatible JS.** The script is wrapped in an IIFE using `var` and function expressions. Keeps it pasteable, debuggable in any browser, and avoids transpile surprises.
- **No secrets in the repo.** The team passphrase is entered by the user at runtime and validated server-side by the proxy. Do not hard-code passphrases, API keys, or reviewer names.
- **No persistent storage.** State lives in `state` (in-memory) and is intentionally lost on refresh. Don't add `localStorage` without discussing the privacy trade-off first.
- **Purdue brand palette.** Gold `#C8973A`, ink `#1A1A1A`, teal `#1D6B5E`. Defined as CSS custom properties on `:root`.
- **HTML escaping** for any string interpolated into innerHTML — use the `esc()` helper at [index.html:611](index.html).

## How to run / test

Open `index.html` directly in a browser, or serve locally:

```bash
python -m http.server 8000
# or
npx serve .
```

Then walk through the full flow (auth → welcome → 5 sections → generate → brief) at least once before pushing. The generation step requires a working proxy and a valid passphrase, so end-to-end testing needs both.

## Deployment

Currently uploaded via the GitHub web UI (see git log). Any static host works — GitHub Pages, Cloudflare Pages, Netlify.

## When making changes

- **Editing the interview**: add/remove fields in `SECTIONS` (see [index.html:545](index.html)). The `id` becomes a key in the transcript sent to Claude.
- **Editing the brief format**: edit the `systemPrompt` string inside `generateBrief()` ([index.html:759](index.html)). It defines the markdown structure of the output.
- **Changing the model or token budget**: edit the `model` and `max_tokens` fields in the fetch body ([index.html:771](index.html)).
- **Adding a new step to the gen-steps UI**: append to `GEN_STEPS`. Note that this is cosmetic — the steps advance on a 4.5s timer, not on real progress signals.

## What this repo does NOT do

- Store user responses anywhere persistent.
- Authenticate users beyond a shared team passphrase.
- Track analytics or page views.
- Email or auto-share the generated brief — copy/print only.

These are intentional constraints. Revisit them deliberately if changing.
