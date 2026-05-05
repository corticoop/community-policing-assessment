# Community Policing Assessment — CLAUDE.md

## What this is

A single-page assessment tool built for CARE (the client) to share with the Stratford Police Department. An officer reviews a list of community policing practices, marks yes/no on each, adds notes, and can save as PDF. All state persists in the browser via localStorage — no backend, no accounts.

## Deployment

- **Live URL:** https://cp.stratfordcare.org
- **Netlify project:** community-policing-assessment (in the "Dot Think" team)
- **GitHub repo:** https://github.com/corticoop/community-policing-assessment
- **Deploy command:** `netlify deploy --dir . --prod` from this folder
- **DNS:** `cp.stratfordcare.org` is a NETLIFY-type record in Netlify's DNS zone for `stratfordcare.org` — no manual DNS record needed, it's managed automatically

## Tech stack

Single `index.html` — no build step, no framework, no dependencies except the Inter font from Google Fonts. Vanilla JS. Deploy is a direct file upload to Netlify CDN.

## Files

```
index.html       — the entire app (HTML + CSS + JS)
netlify.toml     — tells Netlify to publish from root (publish = ".")
.gitignore       — excludes *.docx, .netlify/, .DS_Store
Community policing is an operating philosophy.docx  — source research doc (not in app)
```

## Password

The site uses a client-side password gate. Password: **lofton**

Stored as `atob('bG9mdG9u')` in the JS. On correct entry, `cp_v1_auth = '1'` is written to localStorage — the officer only needs to enter it once per device/browser.

This is intentionally client-side: the Netlify free plan does not support server-side password protection. Sufficient for this use case (single officer, non-sensitive form data). To add true server enforcement, upgrade to Netlify Pro and set the password in the Netlify dashboard — the client-side gate can then be removed.

## localStorage keys

| Key | Contents |
|-----|----------|
| `cp_v1_auth` | `'1'` if authenticated |
| `cp_v1_responses` | JSON object: `{ [itemId]: { r: 'yes'|'no'|null, n: string } }` |
| `cp_v1_custom` | JSON object: `{ [sectionId]: [{ id, title, r, n }] }` |
| `cp_v1_name` | Officer name field value |
| `cp_v1_date` | Date filled out field value |

## Content structure

19 built-in practices across 4 sections — all defined in the `SECTIONS` array in `index.html`:
- **Patrol** (6 items: p1–p6)
- **Detectives** (4 items: d1–d4)
- **Specialized Units** (5 items: s1–s5)
- **Evaluation and Supervision** (4 items: e1–e4)

Officers can add custom practices to any section. Custom items get IDs like `x_patrol_1234567890`.

9 reference items in the collapsible addendum are defined in the `REFERENCES` array.

## Key functions

- `checkAuth()` — shows/hides the password gate on load
- `render()` — builds the checklist DOM from `SECTIONS` data
- `addCustomItem(sectionId)` — creates a new editable practice card
- `openSaveDialog()` / `doPrint()` — print-to-PDF modal flow
- `resetAll()` — clears state + custom items + re-renders

## Editing content

All practice text lives in the `SECTIONS` array and `REFERENCES` array in `index.html`. No external data files. To add, remove, or edit a practice, edit those arrays directly and redeploy.

## Style notes

- Font: Inter (Google Fonts)
- `--muted: #475569` for body copy (deliberately darker than Tailwind default for readability)
- Section colors are set per-section in the `SECTIONS` array
- No em dashes anywhere in content (client preference)
- Print styles hide the actions bar, password gate, add-practice buttons, and note-saved indicators
