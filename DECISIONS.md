# DECISIONS.md

A log of meaningful decisions made during this project — what was debated, what was tried, and where it landed.

---

## 2026-05-05 — Output format for the assessment

**Context:** The source material was a research document (docx) covering community policing practices. The goal was a working document to share with a Stratford Police Department officer who could mark which practices they currently use.

**Debated:**
- **DOCX** — familiar, printable, no account required. Rejected: filling it out is awkward, formatting breaks, hard to get back in a usable state.
- **Google Form** — easy yes/no collection. Rejected: strips all context, becomes a survey rather than a working document.
- **Google Doc / Notion** — collaborative, link-shareable. Rejected: requires the officer to have an account.
- **PDF with form fields** — fillable but limited interactivity.
- **Single HTML file with localStorage** — opens in any browser, no install, toggles for yes/no, notes persist between sessions, exportable as PDF via print. **Selected.**
- **Hosted web app with backend** — real persistence, multi-officer response collection. Deferred as overkill for one officer; can be revisited if use expands.

**Landed:** Single `index.html` deployed to Netlify at `cp.stratfordcare.org`. No build step, no framework, no accounts required. All state in localStorage.

---

## 2026-05-05 — Deployment and DNS

**Context:** Needed a shareable URL, not a local file. User had an existing Netlify account and DNS already managed by Netlify for `stratfordcare.org`.

**What happened:**
- GitHub repo created at `corticoop/community-policing-assessment`
- Netlify project created in the "Dot Think" team, linked via CLI (`netlify link`)
- `netlify deploy --dir . --prod` used for all deployments (no Git-triggered CI configured)
- Custom domain `cp.stratfordcare.org` was added via `netlify api updateSite`. Because `stratfordcare.org` DNS was already managed in Netlify, a NETLIFY-type record for `cp.stratfordcare.org` was automatically created — no manual DNS entry needed.

**Landed:** `https://cp.stratfordcare.org` pointing to Netlify CDN. Git-triggered deploys not configured; deploy is manual via CLI from the project folder.

---

## 2026-05-05 — Password protection

**Context:** Client wanted the site password-protected with the password "lofton."

**Tried:**
1. **Netlify MCP `update-visitor-access-controls`** — returned 422 Unprocessable Entity.
2. **`netlify api updateSite` with `{"password": "lofton"}`** — returned 422 Unprocessable Entity.
3. **Direct Netlify REST API PATCH** — site `has_password` remained `false`.

**Why it failed:** Netlify's site-level password protection (Visitor Access) requires the Pro plan ($19/month). The project is on the free (`nf_team_dev`) plan.

**Alternatives considered:**
- Upgrade to Netlify Pro — not done; out of scope for now.
- Cloudflare Access (free tier, 50 users) — not done; would require Cloudflare in the DNS chain.
- Client-side password gate in the HTML — **selected** as pragmatic for the use case.

**Landed:** Password check runs in JS on page load. Correct password (`lofton`, stored as `atob('bG9mdG9u')`) sets `cp_v1_auth = '1'` in localStorage; incorrect entry animates a shake and clears the field. Once authenticated, the officer is not prompted again on that device.

**Tradeoff acknowledged:** Client-side password is not cryptographically secure — someone who views page source can bypass it. For a single-officer form with no sensitive data, this is acceptable. If the project grows or sensitivity increases, upgrading to Netlify Pro and removing the client-side gate is the right path.

---

## 2026-05-05 — "Save Answers" / export flow

**Context:** The original design had a "Copy summary" button (clipboard text export) and a separate "Print" button. Client wanted to consolidate and make the export action clearer.

**Debated:**
- Keep clipboard copy — useful for email, but non-obvious how to turn it into a PDF.
- Keep separate Print button — works, but "Print" is ambiguous.
- Single "Save Answers" button that opens a brief modal explaining to choose "Save as PDF" in the print dialog — **selected**.

**Landed:** "Save Answers" opens a modal with the instruction "set the destination to Save as PDF." Clicking "Open print dialog" in the modal closes the modal and fires `window.print()`. The clipboard copy function was removed entirely.

---

## 2026-05-05 — Custom practices per section

**Context:** Client wanted officers to be able to add practices that weren't listed.

**Design:** A dashed "+ Add a practice" button at the bottom of each section creates a new card with an editable title input, yes/no toggles, and a notes field. Custom items are saved to `cp_v1_custom` in localStorage, keyed by section ID. They survive page refresh and appear in the print output. A × button removes them (with confirmation).

**Tradeoff:** Custom item titles are stored and rendered with `escHtml()` to prevent XSS from stored values. The title renders as plain text in print — no description field (unlike built-in items) since the officer is writing both.

---

## 2026-05-05 — Content and editorial

**Decisions made without debate:**

- **Definition** — replaced the original research-derived definition with the exact text provided by the client. Grammar fix: none needed beyond minor punctuation.
- **Em dashes** — removed from all content (client preference). Replaced with commas, "such as," or rephrased. Affected: item descriptions, reference descriptions, addendum title, footer text.
- **Department field** — removed from the officer info card. Only name and date remain.
- **Intro paragraph** — replaced the generic subtitle with a CARE-voiced paragraph explaining the collaborative intent. Fixed "research across country" → "researched...from across the country."
- **Note save indicator** — added per-item `✓ Saved` text that appears below each notes field after a 600ms typing pause, addressing feedback that the global save pill was easy to miss.
- **Headline sizes** — increased by approximately 2px across the board (site h1, section h2, item title, addendum h2).
- **Body copy color** — darkened from `#64748b` to `#475569` for improved readability.
