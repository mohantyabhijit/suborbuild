# SuborBuild Calculator — Design Spec

**Date:** 2026-04-03  
**Domain:** suborbuild.com  
**Status:** Approved

---

## Overview

A single-page calculator at suborbuild.com that helps anyone quickly decide whether to pay for a SaaS subscription or build and maintain the thing themselves. The answer depends on their hourly rate, build time, and ongoing maintenance burden — this calculator makes that math instant.

---

## Layout & Structure

Single `index.html` file. No framework, no build step, no dependencies.

Page structure (top to bottom):

1. **Header** — "SuborBuild" + tagline: *"Should you subscribe or build it yourself?"*
2. **Calculator card** — centered, max-width 600px. Two columns on desktop (inputs left, results right), stacked on mobile.
3. **Inputs panel**:
   - Hourly rate ($/hr) — required
   - Hours to build once (hrs) — required
   - Hours/month to maintain (hrs/month) — required
   - Subscription cost ($/month) — required
   - Months planned (optional — placeholder: "leave blank for ongoing")
4. **Results panel** (live-updating):
   - Monthly DIY cost
   - Monthly subscription cost
   - Break-even month
   - Verdict badge — green "Build it" or blue "Subscribe"
5. **Calculate button** — triggers validation; shows funny message if required fields are empty
6. **Footer** — minimal, just the domain

Visual style: clean/minimal, white background, simple typography, CSS variables for colors.

---

## Calculation Logic

**Inputs:**
- `rate` — hourly rate ($/hr)
- `buildHours` — one-time hours to build
- `maintainHours` — hours/month to maintain
- `subCost` — subscription cost ($/month)
- `months` — planned usage months (optional; defaults to 24 for display if blank)

**Formulas:**

| Metric | Formula |
|---|---|
| Monthly DIY cost | `maintainHours × rate` |
| Total DIY cost (N months) | `(buildHours × rate) + (maintainHours × rate × N)` |
| Total subscription cost (N months) | `subCost × N` |
| Break-even month | `(buildHours × rate) / (subCost − maintainHours × rate)` |

**Verdict logic:**
1. If `maintainHours × rate > subCost` → **Subscribe** (DIY maintenance costs more per month forever)
2. Else if `months` is set and break-even month > `months` → **Subscribe** (won't recoup build cost in time)
3. Otherwise → **Build it**

**Edge cases:**
- Break-even formula only applies when `subCost > maintainHours × rate`; if not, DIY wins long-term
- If required fields are empty, results panel shows: *"Fill in your numbers above to see the verdict"*

---

## Validation & Error UX

When the user clicks "Calculate" with missing required fields:

- Required fields get a subtle border highlight (no aggressive red)
- A friendly, funny inline message appears below the button, e.g.:
  - *"We'd love to help, but we're not psychic. Fill in your numbers first."*
  - *"Even a napkin calculation needs numbers. Try filling those in."*
- The `months` field is never flagged — it is always optional

Live calculation still fires on every `input` event; the button is an additional entry point with validation.

---

## File Structure

```
suborbuild/
├── index.html
└── docs/
    └── superpowers/
        └── specs/
            └── 2026-04-03-calculator-design.md
```

Everything lives in `index.html`:
- `<style>` block in `<head>` — CSS variables, responsive layout via media query
- Semantic HTML markup — `<input>` elements, `<button>`, no `<form>` submit
- `<script>` at bottom — input event listeners (live calc) + button click handler (validation)

---

## Deployment

**Target:** Digital Ocean droplet running nginx  
**Method:** `rsync` or `scp` the `index.html` to `/var/www/suborbuild/html/` (or equivalent nginx webroot)  
**Nginx config:** Server block pointing root at that directory, `index index.html`  
**Domain:** suborbuild.com — DNS A record pointing to droplet IP

No CI/CD required for v1. Manual deploy: edit locally, push file to droplet.
