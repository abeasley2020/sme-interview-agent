# SME Interview Agent

A structured, AI-augmented intake interview for faculty subject-matter experts (SMEs) at Purdue University Online's Course Production department. Replaces the traditional first kickoff meeting with a 15–20 minute guided conversation that produces a polished Instructional Design (ID) Brief, ready for the production team.

**Live demo:** _TBD_

---

## Overview

The Course Production team at Purdue University Online partners with dozens of faculty SMEs each year to build online and hybrid courses. The first conversation — gathering course identity, outcomes, content, assessments, and production preferences — is high-friction to schedule and inconsistent in what it captures.

This tool front-loads that conversation into a single web page. A faculty member enters their name and a team passphrase, walks through five short sections, and receives a structured ID brief that the design team reviews before their first synchronous meeting. Result: better-prepared kickoffs, consistent intake data, and less back-and-forth.

The app is intentionally minimal — a single static HTML file, a small Cloudflare Worker proxy for API auth, and the Claude API for generation. No framework, no build step, no database.

---

## How it works

```
┌─────────────────┐    ┌──────────────────┐    ┌──────────────┐
│  index.html     │───▶│  Cloudflare      │───▶│  Anthropic   │
│  (faculty SME)  │    │  Worker proxy    │    │  Claude API  │
│                 │◀───│  (auth + audit)  │◀───│  (sonnet-4)  │
└─────────────────┘    └──────────────────┘    └──────────────┘
```

### User flow

1. **Auth** — Faculty enters their name and the team passphrase.
2. **Welcome** — Brief orientation: what to expect, time estimate, tone.
3. **Interview** — Five sections, advancing one at a time with progress indicator:
   1. **Course Fundamentals** — name, audience, structure, prerequisites
   2. **Learning Outcomes** — big question, 3–6 outcomes, real-world application
   3. **Content & Modules** — dynamic module list, common student struggles, SME's unique angle
   4. **Assessment & Activities** — assignment types, what strong work looks like, feedback practices
   5. **Media & Logistics** — video preferences, existing materials, constraints, what excites the SME
4. **Generating** — The interview transcript is sent to the proxy, which calls Claude with a team-specific system prompt.
5. **Brief** — A structured ID brief is rendered with executive summary, outcomes, module table, assessment recommendations, content notes, media guidance, and open ID questions.
6. **Copy / Print** — Results can be copied to the clipboard or printed/saved as PDF.

---

## Features

- **Passphrase-gated access** — Tool is restricted to authorized Course Production reviewers; the passphrase is verified server-side by the proxy.
- **No database** — All state is in-memory in the browser tab. Closing the tab clears the session.
- **Dynamic module list** — SMEs can add or remove module rows as they think through their course structure.
- **Markdown-aware rendering** — The Claude-generated brief is parsed (headers, lists, tables, bold/italic) and styled with Purdue branding.
- **Print-friendly** — A `@media print` style strips chrome so the brief saves cleanly to PDF.
- **Single-file deployment** — Hosting is as simple as serving one HTML file.
- **Branded UI** — DM Serif Display + DM Sans + DM Mono on a Purdue-gold and ink palette.

---

## Tech stack

| Layer | Technology |
|---|---|
| Frontend | Single-file HTML / CSS / vanilla JavaScript (ES5-compatible, no build) |
| Typography | Google Fonts — DM Serif Display, DM Sans, DM Mono |
| API proxy | Cloudflare Worker (separate repo) handling passphrase auth + audit logging |
| Generation | Anthropic Claude API — `claude-sonnet-4-20250514` |
| Hosting | Any static host (GitHub Pages, Cloudflare Pages, Netlify, etc.) |

---

## Local development

The app is a single static file with no build tooling. You can either open it directly or serve it locally.

### Open directly

Double-click `index.html` and it will run in your default browser. The proxy URL is hard-coded, so the API call will work as long as you have internet access and a valid passphrase.

### Or serve locally

Some browsers restrict certain APIs when opened via `file://`. To avoid that:

```bash
# Python 3
python -m http.server 8000

# Node.js (no install)
npx serve .
```

Then visit `http://localhost:8000`.

---

## Deployment

Any static host works. Below are the two simplest options.

### GitHub Pages

1. In repo **Settings → Pages**, set the source to `main` branch / root.
2. Save. GitHub will publish at `https://<your-user>.github.io/sme-interview-agent/`.

### Cloudflare Pages

1. Connect this repo in the Cloudflare dashboard.
2. Set build command: _(none)_ — Build output directory: `/`.
3. Deploy.

---

## Configuration

All configuration lives at the top of the `<script>` block in `index.html`.

### Proxy URL

```js
var PROXY_URL = 'https://anthropic-proxyandrebeasley2012workersdev.andrebeasley2012.workers.dev';
```

This points to a separately-deployed Cloudflare Worker that:

- Verifies the team passphrase
- Forwards the request to the Anthropic API with the server-side API key
- Logs the request (reviewer name, tool, timestamp) for usage tracking

### Model

```js
model: 'claude-sonnet-4-20250514'
```

Adjust `max_tokens` if briefs are getting truncated.

### Interview sections

The five interview sections are defined as a `SECTIONS` array. Each section has a tag, title, and an array of fields. Field types: `input` (single line) or `textarea` (multi-line). Add a field by appending to the array:

```js
{
  id: 'myNewField',
  label: 'Your question here?',
  hint: 'Optional helper text shown under the label.',
  type: 'textarea',
  rows: 4
}
```

The `id` becomes the key under `state.answers` and appears in the transcript sent to Claude.

### System prompt

The `systemPrompt` variable inside `generateBrief()` defines the structure and tone of the ID brief. Adjust headings, required sections, or output format here. The user message is appended as `'Generate the ID brief from this interview:\n\n' + transcript`.

---

## Privacy and access

- **No persistent storage.** Interview responses live only in browser memory until the page is closed or refreshed.
- **No analytics or trackers** are loaded by the page itself.
- **Audit logging** happens at the proxy layer — reviewer name and timestamp are recorded for internal usage tracking, but free-text responses are not stored long-term.
- **Passphrase distribution** is handled out-of-band by the Course Production team lead. Do not commit the passphrase to this repo.

---

## File structure

```
sme-interview-agent/
├── index.html      # Entire app — markup, styles, and logic
└── README.md       # This file
```

---

## Acknowledgments

Built for the **Purdue University Online Course Production** team. Generation powered by **Anthropic Claude**. Proxy auth and audit logging via **Cloudflare Workers**.
