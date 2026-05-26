# BUILD_PLAN.md

Architecture, design decisions, and enhancement candidates. Read this when planning non-trivial changes — it captures the "why" behind the current shape so you don't accidentally undo a deliberate choice.

## Architecture

```
┌────────────────┐    ┌──────────────────┐    ┌──────────────┐
│  index.html    │───▶│  Cloudflare      │───▶│  Anthropic   │
│  (faculty SME) │    │  Worker proxy    │    │  Claude API  │
│                │◀───│  (auth + audit)  │◀───│  (sonnet-4)  │
└────────────────┘    └──────────────────┘    └──────────────┘
```

Three components, two of which live elsewhere:

1. **`index.html`** — the only thing in this repo. Single-file, no build, runs anywhere static.
2. **Cloudflare Worker proxy** — separate repo. Validates the team passphrase, attaches the Anthropic API key (server-side secret), and logs the request (reviewer name + timestamp) for audit.
3. **Anthropic Claude API** — generates the brief.

## Design decisions and the why behind them

### Single-file static HTML

**Decision:** Keep the entire app in one `index.html`. No bundler, no framework, no build step.

**Why:** Audience is the Purdue CDD team — internal users on managed devices. Optimizing for "anyone can open this file and use it" trumps any DX win from a framework. Also makes the app dead-simple to deploy anywhere, including GitHub Pages and Cloudflare Pages with zero config.

**When to revisit:** If the app grows past ~1500 lines or needs shared components with another CDD tool. Until then, the simplicity is load-bearing.

### Passphrase auth, no per-user accounts

**Decision:** Single shared team passphrase, validated server-side by the proxy. No identity provider, no SSO, no user accounts.

**Why:** The tool is for ~10–30 reviewers on the CDD team. Real auth (Purdue Career Account + OIDC) would be more correct, but the operational cost (managing app registration, refresh tokens, session storage) isn't justified for the user count. The proxy logs reviewer name on each request, which is enough for audit.

**When to revisit:** If access expands to faculty self-serve at scale, or if Purdue IT requires SSO for any tool that touches API keys.

### No persistence

**Decision:** All state is in-memory. Refresh loses everything.

**Why:** Privacy stance is "we don't store your free-text responses." Adding `localStorage` would technically be local-only, but it complicates the messaging and creates the question of when to clear it. A 15–20 minute one-sitting completion is workable.

**When to revisit:** If completion drop-off is high (need analytics first to know) or if faculty consistently request "save my progress."

### Proxy in a separate repo

**Decision:** The Cloudflare Worker lives in its own repo, not this one.

**Why:** The proxy is shared across multiple tools the same author maintains. Versioning it independently lets one update apply to all consumers. The flip side: the proxy's contract (`tool`, `passphrase`, `model`, `messages`, etc.) is implicit — there's no schema in this repo to validate against.

**When to revisit:** If proxy changes start breaking consumers silently. Adding a tiny `proxy-contract.md` to either repo would help.

### Cosmetic generation progress

**Decision:** The "Reading interview responses → Synthesizing learning outcomes → ..." steps advance on a 4.5s timer, not on real signals from the model.

**Why:** Anthropic streaming wasn't wired up; users want a sense of progress; an honest timer that lines up with the typical ~25s response gives the same psychological effect without the engineering cost.

**When to revisit:** If streaming becomes worth the wiring (e.g., for showing partial briefs as they generate) or if the model's response time becomes variable enough that the timer feels wrong.

## Enhancement candidates

Roughly ordered by leverage — concrete value per unit effort. Not commitments, just options.

### High leverage

- **Add `wrangler.toml`-style config file or `<meta>` tags** to externalize `PROXY_URL` and the model name. Lets non-coders point a fork at a different proxy without diving into the script. Small change.
- **Accessibility pass** to WCAG 2.2 AA — semantic landmarks, focus management between screens, ARIA on the progress track, color contrast spot-check on the gold palette. Purdue cares about this and the page is close already.
- **Live demo URL in README** — once deployed somewhere stable, fill in the `TBD` placeholder. (See [STATUS.md](STATUS.md).)

### Medium leverage

- **Multiple interview templates.** Today there's one `SECTIONS` array. A small refactor could let the page accept `?template=mba` or similar and load a different question set. Useful if CDD wants intake variants for exec ed, MOOC, undergrad, etc.
- **Edit brief before sharing.** Make the rendered brief `contenteditable` so faculty can correct anything before copying. Low effort; high "feels professional" payoff.
- **Auto-email brief to CDD inbox.** Worker-side: on a successful generation, also POST to a SendGrid/Postmark/Microsoft Graph endpoint. Avoids the copy-paste step.
- **Save brief as `.md` download** in addition to copy/print. One-click "Download as Markdown" button.

### Lower leverage (worth doing only if a real signal emerges)

- **localStorage autosave** with a clear "Clear my draft" button and explicit messaging about what's stored. Trade-off: privacy clarity vs. convenience.
- **Streaming brief generation** so faculty see text appear as it's produced. Nice but adds proxy and frontend complexity.
- **Analytics.** Time-to-complete, drop-off-by-section. Privacy-respecting analytics (no PII, aggregate only) via the proxy.
- **Multi-language support.** Probably premature, but worth noting that all UI strings are inline in index.html — extracting them to a single object would be the prep work.

## Open questions

- **Hosting target.** Where does this actually deploy long-term? GitHub Pages works but doesn't have password protection at the page level (the passphrase only gates generation, not page visibility). Cloudflare Pages + Access could gate visibility too if needed.
- **Brief storage.** Should generated briefs ever be archived (not the responses, just the output) for the CDD team to retrieve later? Currently no — faculty are responsible for saving the PDF they print.
- **Versioning the system prompt.** As the brief format evolves, do older briefs reference the version that produced them? Today no. Easy to add once it matters.
