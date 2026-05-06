# Coach Circle — Marketing Site

## What This Is
Public marketing website for Coach Circle — a members-only community of Filipino coaches. Converts visitors into coach applicants and coach members. Every CTA routes to GoHighLevel (GHL).

**Stack:** Plain HTML/CSS/JS — static site deployed on Vercel.
**Domain target:** coachcircle.ph (or equivalent)
**Vercel project:** coach-circle-site

---

## Commands
- No build step — static files served as-is by Vercel
- `vercel dev` — local preview with Vercel config applied (security headers, rewrites)
- `vercel --prod` — deploy to production

---

## File Structure
```
coach-circle-site/
├── index.html          # Home page
├── thank-you.html      # Post-form thank-you page
├── vercel.json         # Security headers, rewrites, redirects
├── .env.example        # Env var template (no real values)
├── .gitignore          # Blocks .env, secrets, OS junk
├── CLAUDE.md           # This file
├── BUILD_SYSTEM.md     # Universal build discipline
└── Assets/             # Images, logos, brand assets
```

---

## Architecture Rules
- **No secrets in HTML files** — GHL URLs, form IDs, embed codes that should be configurable go in `.env.example` and are documented there
- **No inline JavaScript** that logs user data or PII to console
- **No third-party scripts** added without review (tracking pixels, analytics, chat widgets)
- **All CTA links configurable** — document GHL URLs in `.env.example` so they can be swapped without hunting through HTML

---

## Brand System (Design Tokens)

```css
--navy-deep:    #0A1628;   /* primary background */
--navy-mid:     #14253E;   /* alt sections, glass cards */
--ivory:        #F5EFE2;   /* primary text on dark */
--gold:         #D4A857;   /* accent, borders, dividers */
--gold-soft:    #E8C97A;   /* italic gradient stop */
--teal-script:  #5FD4C4;   /* italic gradient start */
--emerald:      #10B981;   /* primary CTA buttons */
--emerald-glow: #34D399;   /* CTA highlight */
```

**Fonts:** Fraunces (display, headings) + Manrope (body, UI)
**Italic accent:** `<span>` with teal→gold gradient on emotionally-loaded phrases
**CTA buttons:** Emerald with green glow shadow — never gold
**Cards:** Glass-morphism (backdrop-blur + gradient border) — never flat

---

## Content Sources of Truth
- Coach data: hardcoded in HTML for Phase 1 — do NOT duplicate across files; extract to a content object or separate JS file if data is repeated
- GHL URLs: `.env.example` documents the variable names; real URLs are pasted into HTML only during deploy or managed via Vercel env vars
- Site name, tagline, contact: `index.html` `<head>` meta tags are the single source

---

## CTA Routing (All go to GHL)
| CTA | Destination |
|-----|-------------|
| Apply Now (coach) | GHL application form |
| Sign In | GHL hosted login |
| Book a Call | GHL Calendar |
| Join Community | GHL Community |
| Newsletter | GHL form embed |

---

## Security Rules (Non-Negotiable)
- **Never hardcode** GHL tokens, webhook URLs, or form secrets in committed HTML
- **Never add** tracking pixels without checking they don't leak PII
- **Always** keep `.env` out of git — `.gitignore` already blocks it
- **Check before every commit:** `git diff --cached | grep -iE "(api_key|secret|password|token|webhook)"`
- **`vercel.json` controls security headers** — do not remove or weaken them

---

## What Claude Should Never Do
- Add dependencies (scripts, CDN links) without asking
- Hardcode GHL form action URLs that should be configurable
- Log user input or form field values to console
- Remove security headers from `vercel.json`
- Push to main without review
- Use default AI-generic layouts: centered hero + 3 cards + footer = rejected
- Use Inter, Roboto, or Playfair Display — brand fonts are Fraunces + Manrope

---

## Build Discipline
Read `BUILD_SYSTEM.md` before any non-trivial change.
Follow the sequence: brainstorm → plan → execute → review → verify.

---

## Open Items
- [ ] Confirm final domain (coachcircle.ph?)
- [ ] Get real GHL URLs and add to Vercel env vars (not to source)
- [ ] Set up `/contribute` rewrite in `vercel.json` once Contribute site URL is confirmed
- [ ] Add GA4 or analytics tag (non-PII, approved by client)
- [ ] Lighthouse audit before go-live (target: Performance ≥ 90, Accessibility ≥ 95)
