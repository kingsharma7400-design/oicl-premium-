# NOTICE — provenance and what you may be publishing

**What this is.** An opinionated rebuild of the OICL premium calculator: the rating engine, modules,
GST logic, PDF/share and saved-quote behaviour are an **unmodified copy** of project "A"; the visual
design system and the extra chrome (dark mode, Excel export, history analytics, shortcuts) come from
project "B". Nothing in `src/engine/` or `src/styles/a*.css` was edited — `tools/build.js` aborts if
a single byte of that content differs from the baseline, and `tests/` re-proves it.

**Sources.**
- A (engine, UI markup, all logic): <https://nvkoicl.github.io/PREMIUM-CALCULATOR/>
- B (design system + features that inspired `styles/skin.css` and `chrome/extras.js`):
  <https://kingsharma7400-design.github.io/Motor-premium/>
- Rate tables themselves are the insurer's published tariff/GR schedules, which both upstream pages
  transcribe. This repo adds no rates of its own.

**Before you make this repo public, read this part.**
1. `reference/A_source.html` is a **verbatim copy of A's published page** (3.3 MB). It is kept only so
   the build can byte-compare and so `tests/selftest.js` can run A's own 718-case suite against
   pristine A and against this build in the same process. If you are not the author of that page, do
   not redistribute it — delete it and the two things that need it:
   `tools/extract.py` (re-extract), `tools/build.js`'s fidelity guard and `tests/selftest.js` /
   `tests/fidelity.js`. Everything else keeps working.
2. The committed `index.html` at the repo root **is derived from A** (same bytes, plus two files).
   The same permission question applies to it. If in doubt, link to A instead of hosting a copy.
3. Authorship/attribution of A and B is theirs; keep their credits intact — A's own comment blocks
   (e.g. the designer-credit block) are still present verbatim inside `src/engine/`.

**No warranty.** Premium figures must be checked against the currently notified tariff before any
quote is issued. Use this as a working aid, not as an authority.
