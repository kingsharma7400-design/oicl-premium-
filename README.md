# OICL Premium Calculator — B's UI on A's engine

Aaj ka build: **A ka poora calculator (rates, logic, modules, GST, PDF, saved quotes) jaisa tha waisa hi,
bin ek byte badle — upar B ka design system aur B ke behtar features.**

Deliverable: **`preview/index.html`** (aur uski identical copy `dist/index.html`) — ek hi file,
3,291 KB (gz 1,187 KB), koi CDN/network nahi.

> `dist/` aur `node_modules/` workspace snapshots se exclude hote hain, isliye build har baar
> identical copy `preview/` me bhi likhta hai (persist hoti hai). Preview server bhi `preview/`
> serve karta hai. `cmp dist/index.html preview/index.html` → identical.
Baseline jo reproduce hui: `reference/A_source.html` = A's live page (3,334,985 B).

```
A: 3,334,985 B
+ skin.css   14,411 B   (B's design tokens + rail/hero/CTA rules)
+ extras.js  20,695 B   (B's chrome: theme, Excel, history stats, shortcuts, stamp)
+ 32 B                   (<style>/</style> + <script>/</script> wrappers)
= 3,370,125 B            (dist/index.html — exact)
```

## Kya kya kis taraaf se aaya

| | Source | Kaise |
|---|---|---|
| Rate tables, OD/TP/NCB/discount/Nil-Dep math, GST split | **A** | `src/engine/*.js` — A ke inline blocks, verbatim |
| Motor · Health · Fire · WC · CAR/EAR · CPM · PAP's · Shopkeepers · Office Umbrella — 9 modules | **A** | verbatim |
| PDF (jsPDF + artwork), WhatsApp share, image share, saved quotes + drawer, drafts, A's own 718-case self-test | **A** | verbatim |
| Form markup, ids, `switchProduct()`, `calculate()`, step stepper | **A** | verbatim (skin sirf CSS hai, markup nahi badla) |
| Design language: indigo/amber palette, slate neutrals, 16/12/8 radii, shadow stack, Inter stack, dark rail, hero result card, toast | **B** | `src/styles/skin.css` |
| Dark mode + persistence, Excel export, history analytics + CSV/JSON, keyboard shortcuts, provenance stamp | **B** | `src/chrome/extras.js` |

**Rate authority A hai — isliye engine ko haath nahi lagaya.** B ke "har rate field editable" wale idea
ko jaan-boojh kar port nahi kiya: wo A ki correctness (jo already verified hai) ko weak karta.

## Tree

```
src/index.html          A ka document, inline blocks ki jagah <link>/<script src> + 2 nayi tags
src/engine/*.js         A ke 20 <script> blocks          (verbatim)
src/styles/a*.css       A ke  7 <style> blocks           (verbatim)
src/styles/skin.css     MERA — B ka design system
src/chrome/extras.js    MERA — B ke features
src/manifest.json       har asset ka size + sha256 (fidelity ka record)
reference/A_source.html A ka page (baseline; isse extract hota hai)
tools/extract.py        splitter   ·  tools/build.js  inliner + byte-guard  ·  tools/make.sh  ek command
tests/fidelity.js       8 checks   ·  tests/chrome.js  50 checks  ·  tests/selftest.js  8 checks
dist/index.html         BUILT FILE
```

## Rebuild

```sh
curl -sL https://nvkoicl.github.io/PREMIUM-CALCULATOR/ -o reference/A_source.html
npm i -D jsdom     # sirf tests ke liye; app ko kuch nahi chahiye
npm run make       # extract → build → saare tests
```

Alag se: `npm run extract` · `npm run build` · `npm test` · `npm run test:dist` · `npm run test:preview` ·
`npm run test:all` (dono target par poori suite) · `npm run serve` (port 8124, `preview/` se)

## Proof — ye "same rates" ka matlab kya hai

`npm test` aaj: **8/8 fidelity · 50/50 chrome · 8/8 behaviour** (sab `dist/index.html` par).

1. **Byte fidelity (`tests/fidelity.js`, 8/8).** Har extracted asset apne recorded sha256+size se
   match karta hai; `src/` ko wapas jod-ne par `reference/A_source.html` **byte-for-byte** banta hai;
   built file se `skin.css` + `extras.js` hatao to **wahi A ka document** milta hai (diff = 0 bytes).
   Build isi wajah se fail ho jata hai agar koi asset mutate ho ya koi external URL aa jaye.
2. **Behaviour (`tests/selftest.js`, 8/8).** Do JSDOM instances — ek pristine A, ek hamari build —
   dono me 224 cases (200 golden + CNG/electricals/pax/IMT-23 variants) chalaye: **sab tuples identical**.
   Fir **A ka apna `runSelfTest()`** hamari build ke andar: `Regression: 718/718 · Document-verified: 269/274`.
   A apne file me bhi **exact yahi** print karta hai — poora result text character-for-character same,
   including A ke apne 5 headless document-check failures (jsdom me API na hone se).
   Spot check: `OD 26264 / TP 3416 / total 21919` (renewal, no invoice) — A ke barabar.
3. **Chrome (`tests/chrome.js`, 50/50).** Skin last stylesheet hai; `#resultCard` hero gradient aur
   `.btn-primary` amber CTA computed style me milte hain; rail rule ≥1000px query me hai aur A kahin
   `.product-tabs` ko position nahi karta; theme toggle + persistence + returning-visit (localStorage se
   dark apply hota hai); Excel workbook me `Quote`/`Breakdown`/`Add-ons` sheets, real numbers as
   `ss:Type="Number"`, XML escaping; history mirror har calculate par 4-tile stats strip banata hai,
   CSV/JSON export rows dete hain; Alt+T / Alt+C / Alt+H / Alt+X / Ctrl+Enter wired; A ke 9 tabs aur
   `formCard`/`resultCard` intact; zero external URLs; ek file 3–4 MB.

## B ke features — exactly kya mila

- **🌙 Dark mode** — `html[data-theme="dark"]` + `localStorage['oicx.theme']`, `<meta name="theme-color">` bhi follow karta hai. Alt+T.
- **📊 Excel export** — SpreadsheetML 2003 (`.xls`), zero dependency; Quote + Breakdown + Add-ons + (unlabelled numeric fields ka "Other fields" sheet, taaki kuch chhoote nahi). Har number A ke quote object se aata hai, koi recompute nahi. Alt+X, aur result card ke andar button.
- **History analytics** — A ke saved-quotes drawer me 4 tiles (calculated · this month · total premium · average) + highest + storage KB + A ki saved list ka count, CSV/JSON/clear tools. Ye **hamari apni mirror** hai (`oicx.history`, last 500) — A ka storage read-only inspect hota hai, likha kabhi nahi jata.
- **Shortcuts** — Alt+H drawer, Alt+C calculator + smooth-scroll, Alt+T theme, Alt+X Excel, Ctrl+Enter calculate (A ke `#calcBtn` par click dispatch, A ka handler).
- **Provenance stamp** — result card me: engine A, UI B, date.

## Jaan-bujh kar jo nahi kiya / caveats

- **Pixel-level visual QA nahi hui** — headless environment me main screenshot nahi le sakta. Jo verify kiya wo cascade + structure hai (jsdom `getComputedStyle`, media queries evaluate nahi karta isliye rail rule source-level se prove kiya). Ek baar browser me dekh lena; skin sirf CSS hai, isliye tweak safe hai.
- jsdom `@media` ko computed style me apply nahi karta, aur usme `URL.createObjectURL`/`fetch` nahi hai — Excel/CSV ka *download* click tests me silently no-op ho jata hai (isliye `saveBlob` try/catch me hai aur tests payload check karte hain). Browser me download normal hai.
- A ke 5 document-check failures hamari wajah se nahi — A khud bhi headless me wahi 5 print karta hai (e.g. "Rs. instead of the rupee symbol" checks jo DOM state par depend karte hain). **Ek finding actually hamari thi**: wo rule `₹` glyph page par allow nahi karta, hamare stats tiles `₹` likh rahe the (`|3|`) — ab `Rs.` use hota hai, A ke rule ke mutabik.
- Service worker add nahi kiya (A ka document untouched rehna tha); offline isliye bhi chalta hai kyunki file self-contained hai.
- `src/engine/*` me kuch block naam A ke apne comment markers se aaye hain (`08-core-rating.js`, `19-…` = motor engine) — ye sirf file naming hai, content nahi badla.

## Preview

`npm run serve` → port **8124** (chalu hai, `preview/` se serve karta hai). `preview/index.html` ko
directly browser me bhi khol sakte ho — wahi deliverable hai.

Last verified run on the persisted copy: `npm run test:preview` → 8/8 fidelity · 50/50 chrome · 8/8 behaviour.

## GitHub par daalna

```sh
npm run zip          # → github-upload/oicl-b-ui-a-engine.zip  (48 files, ~3.7 MB)
npm run zip:no-ref   # same, bina reference/A_source.html ke (NOTICE.md padho)
```

Zip ke andar ek folder aata hai `oicl-b-ui-a-engine/` — uske **andar ki files** drag karo
(github.com → New repository → "uploading an existing file declaration"), ya CLI:

```sh
unzip oicl-b-ui-a-engine.zip && cd oicl-b-ui-a-engine
git init -b main && git add -A && git commit -m "B UI over A engine, byte-verified"
gh repo create oicl-b-ui-a-engine --public --source=. --push      # ya: git remote add + git push
```

**Repo layout jo zip me milta hai**

| | |
|---|---|
| `index.html` (repo root) | built single file — isi liye GitHub Pages “deploy from branch / root” turant chalta hai |
| `src/` | A ke 27 blocks (verbatim) + `styles/skin.css` + `chrome/extras.js` + shell |
| `reference/A_source.html` | A ka baseline copy — byte-guard aur A-ka-apna-self-test isi se compare karta hai |
| `tests/` `tools/` | 8 + 50 + 8 checks, splitter, guarded inliner, `make_zip.py` |
| `.github/workflows/verify.yml` | CI: `npm install` → `npm run build` → `npm run test:dist` → `cmp dist/index.html index.html` → artifact upload |
| `NOTICE.md` | provenance + “publish karne se pehle ye padho” |
| `.gitignore` | `dist/`, `preview/`, `node_modules/` ignore (GitHub ko inki zaroorat nahi; root `index.html` committed hai) |

Do caveats:
1. **`reference/A_source.html` aur root `index.html` A se derived hain.** Public repo se pehle `NOTICE.md`
   padh lo. Rights clear na ho to `npm run zip:no-ref` — us variant me `npm run build` intentionally
   abort ho jata hai (byte-guard ko baseline chahiye) aur `tests/fidelity.js` / `tests/selftest.js`
   khud ko `SKIPPED` bolke skip kar dete hain; `node tests/chrome.js <file>` phir bhi chalta hai.
2. CI me `npm install` sirf **jsdom** laata hai (tests ke liye); site ka koi dependency nahi.
