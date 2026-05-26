# STATUS.md

Living snapshot of where the project stands. Update at the end of meaningful work sessions (`closeout-code` skill handles this automatically).

**Last updated:** 2026-05-25

---

## Current phase

**Stable / in production.** The interview flow works end-to-end, the brief generates against the live proxy, and the page is in use by the Purdue CDD team. No active rewrite or major feature work in flight.

## What's working

- Auth screen with name + team passphrase validation (passphrase verified server-side at the proxy)
- 5-section interview (Course Fundamentals → Learning Outcomes → Content & Modules → Assessment & Activities → Media & Logistics)
- Dynamic module list (add/remove modules in Section 3)
- Progress indicator across sections
- Brief generation via Cloudflare Worker proxy → Claude Sonnet 4
- Markdown rendering of the returned brief (headers, lists, tables, bold/italic)
- Copy-to-clipboard and print/save-as-PDF actions
- Print stylesheet that strips chrome for clean PDFs
- Error screen when generation fails, with restart path
- Responsive layout (single breakpoint at 600px)

## Recent work

| Date | Change |
|---|---|
| 2026-05-25 | Added detailed [README.md](README.md), [CLAUDE.md](CLAUDE.md), [STATUS.md](STATUS.md), [BUILD_PLAN.md](BUILD_PLAN.md) |
| 2026-03-16 | Renamed entry file to `index.html` for default static-host routing |
| 2026-03-15 | Initial app uploaded |

## Known limitations (not bugs — just current scope)

- **No persistence.** Closing or refreshing the tab loses all in-progress responses. Faculty must complete the interview in one sitting.
- **Cosmetic generation progress.** The "gen-steps" UI advances on a 4.5s timer, not on real progress from the model. Looks fine but is not informative.
- **Hard-coded proxy URL.** Changing the proxy means editing [index.html:599](index.html) and redeploying.
- **Single brief template.** The `systemPrompt` produces one ID brief format. No variants for different course types (e.g., MBA vs. undergrad vs. exec ed).
- **No analytics.** No visibility into completion rates, drop-off points, or time-to-complete.
- **No edit-after-generation.** The brief is read-only once produced. Faculty who want to revise must redo the interview.
- **No accessibility audit yet.** Per [purdue-accessibility](https://...), the page should pass WCAG 2.2 AA — needs a formal pass.

## Open questions

- Should responses persist across refresh (localStorage)? Trade-off: convenience vs. the privacy stance currently stated in the README.
- Should the brief be auto-emailed to a shared CDD inbox after generation, or remain copy/print only?
- Should there be multiple interview templates (different question sets) selectable at the start?

## Next steps

Open — no specific work currently planned. Candidates for the next session live in [BUILD_PLAN.md](BUILD_PLAN.md) under "Enhancement candidates."

## Outstanding decisions for the user

- Confirm a live demo URL so the README can be updated (currently `TBD`).
- Decide whether `CLAUDE.md`, `STATUS.md`, and `BUILD_PLAN.md` should remain in the repo or move to `.gitignore` for local-only use.
