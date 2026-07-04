# ReelShift

Free, fully client-side video converter, monetized via Google AdSense.
Deliberately **single-purpose** — see "Single-tool strategy" below.

## What this tool does

Converts video files between MP4, WebM, GIF, and MP3 entirely inside the
browser. No file is ever uploaded — everything runs on-device via a real
build of FFmpeg compiled to WebAssembly.

## Stack

- **AstroJS** (static output) + **Tailwind v4** (`@tailwindcss/vite` plugin,
  imported in `src/styles/global.css`)
- **@ffmpeg/ffmpeg** + **@ffmpeg/util** — client-side video transcoding
  (WebAssembly). Core files are fetched lazily from unpkg CDN
  (`https://unpkg.com/@ffmpeg/core@0.12.10/dist/esm/`) — **must be the `esm`
  build, not `umd`**, because `@ffmpeg/ffmpeg`'s worker does
  `await import(coreURL)`, which requires an ES module. Using `dist/umd`
  throws `Error: failed to import ffmpeg-core.js` silently inside the worker
  (no visible error in the main-thread console unless you check
  `preview_console_logs` at `level: all`).
- **@astrojs/sitemap** — sitemap generation for SEO.
- **Netlify** hosting. `netlify.toml` sets
  `Cross-Origin-Opener-Policy: same-origin` +
  `Cross-Origin-Embedder-Policy: require-corp` site-wide (required for
  ffmpeg.wasm).

This project intentionally has a much smaller dependency list than its
sibling tools (no libheif-js, utif2, gifenc, @xenova/transformers,
@upscalerjs, upscaler, qr-code-styling, canvas-confetti) because this tool
only converts video — it does not touch images, transcription, or anything
else. Only install what a given tool actually needs.

## Design system (mandatory — do not deviate)

This is a premium redesign, deliberately away from the generic
white/cyan "AI tool" look:

- Background: `#FAF6EC` (warm cream)
- Card/surface: `#FFFFFF`, with a warm `#E8DFC8` border (never gray)
- Headings text: `#2B2013` (warm espresso brown, not pure black)
- Body text: `#5C4F3D` (warm brown-gray)
- Accent / CTA / links / active states: `#C9982E` (warm gold), hover
  `#B8860B` (darker gold)
- Fonts: **Fraunces** (Google Font, soft-serif/display) for ALL headings —
  editorial/premium feel. **Inter** (Google Font) for body text, labels,
  buttons, and all interactive UI. Both loaded via Google Fonts `<link>` in
  `Layout.astro`'s `<head>` with `display=swap`. Headings use
  font-weight 600–700 and slightly negative letter-spacing
  (`tracking-tight` / `-0.015em`).
- Generous whitespace, soft shadows (never harsh), rounded-xl corners on
  cards/buttons, no gradients, no dark mode toggle.

This must read as a small, confident, premium product — a boutique SaaS
landing page, not a utility dump.

## Single-tool strategy (why there's no nav to other tools)

This is a deliberate split-out from the multi-tool AnyConvert site. This
repo contains **only ReelShift** — no navigation to, or mention of, any
other tool. One keyword-matched domain per tool is the whole SEO strategy:
a visitor searching "convert video to mp4 online" lands on a site that is
*only* about that, with a URL/brand to match, rather than one tool buried
in a 20-tool hub. Do not add a multi-tool nav bar or a tools index back to
the parent site.

## Structure

- `src/layouts/Layout.astro` — shared shell: simple header (brand name
  only, no nav to other tools), footer (About/Privacy/FAQ + copyright),
  SEO meta tags (title/description/canonical/OG/Twitter), cream/gold theme.
- `src/pages/index.astro` — the tool itself. ffmpeg.wasm video format
  conversion (MP4/WebM/GIF/MP3), multi-file batch queue.
- `src/pages/about.astro`, `privacy.astro`, `faq.astro`, `404.astro` —
  required SEO/trust/AdSense-eligibility pages, linked from both the
  homepage body and the footer.
- `public/robots.txt` — sitemap reference for SEO.

## Security

Any user-controlled text (filenames, etc.) rendered via `innerHTML` MUST go
through the `escapeHtml()` helper defined in `index.astro`'s script block
first. This mirrors the fix applied across the AnyConvert security audit —
copy that pattern into any new page that renders file names or other
user-controlled strings via `innerHTML`.

## Dev / test

- `npm run dev` — runs on **port 4333** (distinct from anyconvert's 4325;
  see `package.json`'s `dev` script).
- File-input-driven tools can't be tested with a real OS file picker in
  headless Playwright/preview tooling. Test by constructing a `File` via
  `new File([blob], name, {type})`, wrapping it in a `DataTransfer`, setting
  `input.files = dt.files`, then dispatching a `change` event — then call
  `document.getElementById('convert-btn').click()` directly rather than the
  coordinate-based preview_click tool, which can get intercepted by the
  Astro dev toolbar overlay.
- Check `preview_console_logs` at `level: 'all'` (not just `'error'`) when
  debugging ffmpeg.wasm — some failures only throw inside the
  worker/log callback, not as a top-level page error.

## iCloud sync hygiene

This project lives in iCloud Drive
(`~/Library/Mobile Documents/com~apple~CloudDocs/website/reelshift`).
`node_modules` is renamed to `node_modules.nosync` with a `node_modules`
symlink pointing at it, so iCloud does not try to sync hundreds of
thousands of tiny dependency files. `tsconfig.json`'s `exclude` array
includes **both** `"node_modules"` and `"node_modules.nosync"` — `tsc`
does not recognize the `.nosync` suffix as implicitly excluded, and
`astro check` will otherwise try to type-check the entire dependency tree
and crash.

## Deploy

Netlify site, deployed via the Netlify CLI using the token in
`~/.claude/credentials/netlify.env`:

```
source ~/.claude/credentials/netlify.env
export NETLIFY_AUTH_TOKEN=$NETLIFY_API_KEY
netlify deploy --prod --dir=dist
```

## Still needed before this earns anything (manual, Shreyas)

1. Buy/point a real domain (pick the domain after confirming the build
   works, which it now does).
2. Get real traffic before applying for AdSense.
3. Apply for AdSense, then add `public/ads.txt` with the real
   `pub-XXXXXXXXXXXXXXXX` line and the AdSense script + Auto Ads.
4. Submit to Google Search Console + Bing Webmaster Tools.
