# OICL Premium Calculator (upload these 8 files)

B ka UI, A ka poora rating engine. Site **ek file** se chalti hai: `index.html` — koi CDN, koi build
step, koi dependency nahi; phone par ya airplane mode me bhi. Baaki 5 chhoti files isliye saath hain
kyunki A ka page khud `./manifest.json`, `./icon-192.png`, `./apple-touch-icon.png` link karta hai aur
`./sw.js` register karta hai (PWA install + offline); `tools/make_pwa.js` inhe banata hai.

**Ye file kaise bani:** poora project (`src/engine/*` = A ke inline scripts, verbatim;
`src/styles/skin.css` + `src/chrome/extras.js` = B ka design system aur chrome) ko `tools/build.js`
ne ek file me inline kiya, aur guard ne verify kiya ki A ke engine bytes **byte-for-byte** wahi hain.

Rebuild + verify (poora repo se):
```sh
curl -sL https://nvkoicl.github.io/PREMIUM-CALCULATOR/ -o reference/A_source.html
npm i -D jsdom && npm run make      # extract → build → 8 fidelity + 50 chrome + 8 behaviour checks
```

Rates A ke notified tables se aate hain; issue karne se pehle current tariff se match karo.
