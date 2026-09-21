# AGENTS.md

Guidance for AI coding agents (and humans) working on **kikaphotos.com**.
Everything here was derived from the repository contents as of commit `c0dd9e2` (2026-09-17).

---

## 1. Project overview

| | |
|---|---|
| **What it is** | Personal portfolio site of **Kristína Kaszbeková**, an amateur photographer in Slovakia (shooting since ~2022). Portraits, sports, events/concerts. |
| **Not** | A company, agency, store, or blog. There is no backend, API, CMS, or database. |
| **Live URL** | https://kikaphotos.com |
| **Repo** | https://github.com/2Peti/kikaphotos (`origin`, default branch `main`) |
| **Type** | Fully static site: hand-written HTML + one CSS file + small inline vanilla JS. |
| **Hosting** | Cloudflare Pages (commit `2a3fb27`: "align URLs with Cloudflare Pages"). No `wrangler`/`_headers`/`_redirects` in repo, so it is deployed straight from the git branch with no build step. |
| **Build step** | **None.** No `package.json`, bundler, framework, or generator. Edit files, commit, push. |
| **Analytics** | Google Analytics 4, tag `G-NXZSSNFPJQ`, inlined in every page's `<head>`. |
| **Owner contact** | `info@kikaphotos.com`, Instagram `@_kikaphotos_` |

---

## 2. Languages

The site is **bilingual**: Slovak (default) and Hungarian.

| Language | `lang` attr | Path | Files |
|---|---|---|---|
| **Slovak** (default, `x-default` in practice) | `sk` | `/` | root `*.html` |
| **Hungarian** | `hu` | `/hu/` | `hu/*.html` |

- There is **no i18n framework**. Each Hungarian page is a full, hand-translated copy of its Slovak counterpart. **Any change to a Slovak page must be mirrored manually in the matching `hu/` page** (and vice versa).
- **URL/filenames are Slovak in both languages** (`moja-praca`, `cennik`, `o-mne`, `kontakt`); only the visible text is translated.
- Language switching is a plain `SK / HU` link pair (`.lang-switch`) that points to the same page in the other language. It is not JS-driven and stores no preference.
- Every page declares `rel="alternate" hreflang="sk"` and `hreflang="hu"` plus a self-referencing `rel="canonical"`. There is no `x-default` hreflang tag.
- **Copy-editing convention:** em dashes (`—`) were deliberately removed (commits `2b24984`, `86ae469`, `7efd134`). Use a plain hyphen with spaces (` - `) instead. Currently zero em dashes remain in any HTML file. Keep it that way.
- Slovak typography uses low-high quotes `„...“`; Hungarian uses the same in the About page quote.
- Prices are in **euros**, written as `25 €` (number, space, symbol).

### Page map (identical structure in both languages)

| Purpose | Slovak file | Slovak nav label | Hungarian file | Hungarian nav label |
|---|---|---|---|---|
| Home | `index.html` | Úvod | `hu/index.html` | Kezdőlap |
| Portfolio | `moja-praca.html` | Moja práca | `hu/moja-praca.html` | Munkáim |
| Pricing | `cennik.html` | Cenník | `hu/cennik.html` | Árlista |
| About | `o-mne.html` | O mne | `hu/o-mne.html` | Rólam |
| Contact | `kontakt.html` | Kontakt | `hu/kontakt.html` | Kapcsolat |

Gallery category names: **Portréty / Portrék** (portraits), **Športy / Sportok** (sports), **Akcie / Események** (events).

---

## 3. Repository layout

```
kikaphotos/
├── index.html            # SK home (hero slideshow + intro + 3 teasers)
├── moja-praca.html       # SK portfolio (tabbed masonry galleries + lightbox)
├── cennik.html           # SK pricing
├── o-mne.html            # SK about
├── kontakt.html          # SK contact
├── hu/                   # Hungarian mirror of the 5 pages above
│   ├── index.html  moja-praca.html  cennik.html  o-mne.html  kontakt.html
├── css/
│   └── style.css         # The ONLY stylesheet (928 lines, hand-written)
├── assets/
│   ├── hero{1..4}-{640,1280,1920}.webp   # homepage slideshow, 3 sizes each (12 files)
│   ├── ja.webp                            # About-page portrait (900x1350)
│   ├── instagram.webp                     # Contact-page icon (64x37)
│   ├── portrety/{full,thumb}/portrety-NN.webp   # 19 portraits   (NN = 01..19)
│   ├── sporty/{full,thumb}/sporty-NN.webp       # 14 sports      (NN = 01..14)
│   └── akcie/{full,thumb}/akcie-NN.webp         # 13 events      (NN = 01..13)
├── favicon.png           # 64x64 PNG
├── apple-touch-icon.png  # 180x180 PNG
├── robots.txt            # Allow all + sitemap pointer
├── sitemap.xml           # 10 URLs (5 SK + 5 HU)
├── llms.txt              # LLM-oriented site summary
├── .gitignore
└── AGENTS.md             # this file
```

Totals (excluding `.git`): **124 files, ~14 MB** (`assets/` is ~14 MB; `hu/` 56 KB; `css/` 20 KB).
File-type counts: 106 `.webp`, 10 `.html`, 2 `.png`, 2 `.txt`, 1 `.xml`, 1 `.css`, 1 `.md`, 1 `.gitignore`.

---

## 4. Image formats and pipeline

### 4.1 Formats

- **All photos are WebP** (RGB, static, non-animated). No JPEG/PNG photos are used.
- The only PNGs are the two icons at the repo root (`favicon.png`, `apple-touch-icon.png`, RGBA).
- The `instagram.webp` icon and `ja.webp` portrait are also WebP.
- Source/RAW/JPEG originals are **not** in the repo (one filename hint: `portrety-19` came from `IMG_1463_edited`). Do not expect to regenerate from originals here.

### 4.2 Two-tier gallery images (`full` + `thumb`)

Every gallery photo exists twice, with the **same filename** in sibling folders:

| Tier | Folder | Purpose | Target size |
|---|---|---|---|
| **thumb** | `assets/<cat>/thumb/` | Shown in the masonry grid and homepage teasers | **700 px wide** (height follows aspect ratio) |
| **full** | `assets/<cat>/full/` | Loaded by the lightbox on click | **~2000 px long edge** (1920 for sports) |

Every `full` file has a matching `thumb` (verified: no unpaired files in any category).

### 4.3 Observed sizes per group

| Group | Files | Dimensions | Avg size | Notes |
|---|---|---|---|---|
| `hero*-640` | 4 | 640 px wide | ~27 KB | landscape |
| `hero*-1280` | 4 | 1280 px wide | ~67 KB | landscape |
| `hero*-1920` | 4 | 1920 px wide | ~118 KB | landscape |
| `portrety/full` | 19 | 1200-2000 px long edge | 203 KB | 8 landscape / 11 portrait |
| `portrety/thumb` | 19 | 700 px wide | 45 KB | |
| `sporty/full` | 14 | 1920-2000 px, all landscape | 244 KB | |
| `sporty/thumb` | 14 | 700x466 | 46 KB | |
| `akcie/full` | 13 | 2000 px long edge | 234 KB | 7 landscape / 6 portrait |
| `akcie/thumb` | 13 | 700 px wide | 54 KB | |
| `ja.webp` | 1 | 900x1350 | 37 KB | About portrait |
| `instagram.webp` | 1 | 64x37 | 1 KB | icon |

Category totals: portraits 19, sports 14, events 13 = **46 gallery photos** (92 gallery files incl. thumbs).

### 4.4 Known quirks

- **`portrety-19.webp` (and its thumb) is the only image with embedded EXIF + ICC profile.** All other images are metadata-stripped. This is **intentional / accepted by the owner**, so leave it as is. It is also 1280x1920, narrower than the 2000 px norm.
- `akcie` and `portrety` thumbs vary in height (aspect ratio preserved), `sporty` thumbs are uniformly 700x466.
- History: `portrety-01.webp` and `portrety-02.webp` were originally unoptimised (5456x3632 ~2.5 MB and 3632x5456 ~2.0 MB). They were resized to a 2000 px long edge (quality 92, metadata stripped) to match the rest. The full-size versions remain in git history if ever needed.

**Encoding quality:** every WebP in the repo (full, thumb, hero, `ja.webp`) measures **quality 92**. Match this for new images. Recipe (ImageMagick; `cwebp` is not installed on the dev machine):

```bash
convert original.jpg -resize '2000x2000>' -strip -quality 92 assets/<cat>/full/<cat>-NN.webp
convert original.jpg -resize '700x'       -strip -quality 92 assets/<cat>/thumb/<cat>-NN.webp
```

(`2000x2000>` only shrinks, never enlarges. Sports fulls use a 1920 px long edge, so 1920 is also acceptable there.)

### 4.5 Adding a new gallery photo (checklist)

The 3 latest commits add photos by hand, so the process is manual:

1. Export **two WebP files** with identical names: `assets/<cat>/full/<cat>-NN.webp` (~2000 px long edge) and `assets/<cat>/thumb/<cat>-NN.webp` (700 px wide). Use the next free number, zero-padded to 2 digits. Strip metadata.
2. Add a `<figure>` block to the matching `<div class="masonry" id="portraits|sports|events">` in **both** `moja-praca.html` **and** `hu/moja-praca.html` (HU uses `../assets/...` paths, see section 6).
3. Copy the exact existing markup:
   ```html
   <figure><a href="assets/portrety/full/portrety-20.webp" class="lightbox-link" data-full="assets/portrety/full/portrety-20.webp"><img src="assets/portrety/thumb/portrety-20.webp" alt="Portrét - fotografia z portfólia KikaPhotos" loading="lazy"></a></figure>
   ```
   Keep the alt text identical to the other items in that category (Slovak vs Hungarian strings differ, see section 7).
4. `href` and `data-full` must be the same URL (the `href` is the no-JS fallback; the lightbox reads `data-full`).
5. Nothing else needs to change: the lightbox and prev/next navigation discover items automatically.

---

## 5. Front-end architecture

### 5.1 Stack

- **HTML5**, no templating. Boilerplate (header, nav, footer, GA snippet, font links) is duplicated in all 10 pages.
- **One CSS file**: `css/style.css`. No preprocessor, no utility framework, no `*.min.css` (and `*.min.css`/`*.min.js` are gitignored, so do not add minified files).
- **Vanilla JavaScript, inline** in `<script>` tags at the end of `<body>`. **There are no `.js` files** and no dependencies.
- **Icons are inline SVG** (stroke-based, `viewBox="0 0 24 24"`, styled by CSS). No icon font.

### 5.2 Fonts (Google Fonts, non-blocking)

| Variable | Family | Use |
|---|---|---|
| `--serif` | **Fraunces** (opsz 9-144, weights 300/400/500/600, italics 400/500) | headings, italic accents, prices |
| `--script` | **Allura** | logo/brand mark, hero title, footer, "Tešim sa na spoluprácu" note |
| `--sans` | **Work Sans** (300/400/500/600) | body text, UI labels |

Loaded via `<link rel="preconnect">` + a `media="print" onload="this.media='all'"` async stylesheet with a `<noscript>` fallback, so fonts do **not** block first render. Do not convert this to a CSS `@import` (the CSS file has a comment explaining why).

### 5.3 Design tokens (`:root` in `css/style.css`)

Palette: blush ivory + deep aubergine + dusty rose hairlines.

| Token | Value | Role |
|---|---|---|
| `--ink` | `#2a2024` | primary text |
| `--ink-soft` | `#5a4a50` | secondary text |
| `--bg` | `#fbf1f0` | page background |
| `--surface` | `#ffffff` | cards / alt sections |
| `--accent` | `#6e1e3d` | brand aubergine (links, buttons, prices) |
| `--accent-2` | `#8a3358` | hover accent |
| `--rose` | `#d9a6b4` | bullets, hero rule |
| `--rose-line` | `#e8d2d8` | hairline borders |
| `--gold` | `#b98b52` | defined but currently unused |
| `--max` | `1180px` | content max-width (`.wrap`) |

### 5.4 Responsive behaviour

- Breakpoints: **780px** (nav collapses to hamburger; pricing/about/intro grids switch column counts), **640px** (brand tag shown, smaller hero arrows), **620px / 980px** (masonry 1 -> 2 -> 3 columns; lightbox padding).
- Masonry is pure CSS `column-count`, not JS.
- Hamburger menu is a **CSS-only checkbox hack** (`#nav-toggle` input + `label.nav-toggle`), no JS.
- Sticky translucent header with `backdrop-filter: blur`.

### 5.5 Inline JavaScript features

| Page(s) | Feature | Notes |
|---|---|---|
| all | **Google Analytics gtag** | Async loader + `gtag('config','G-NXZSSNFPJQ')` in `<head>`. |
| `index.html`, `hu/index.html` | **Hero slideshow** | 4 slides cross-fading via CSS opacity, auto-advance every **6000 ms**, prev/next arrow buttons that reset the timer. Only slide 1 has a real `src`/`srcset` (for LCP); slides 2-4 use `data-src`/`data-srcset` and are swapped in on `window` `load` so they never compete for bandwidth. Slide focal points are tuned per slide with `.hero1..4 { object-position }` in CSS. |
| `moja-praca.html`, `hu/moja-praca.html` | **Tab deep-linking** | Tabs themselves are pure CSS (radio inputs + `:checked ~` sibling selectors). A tiny script maps `#portraits` / `#sports` / `#events` in the URL hash to the correct radio and scrolls to the panel. Homepage teasers link to these anchors. |
| `moja-praca.html`, `hu/moja-praca.html` | **Lightbox** | Clicking a `.lightbox-link` opens `data-full` in `#lightbox`. Prev/next cycle **within the current tab only**. Closes on `Esc`, overlay click, or close button; `←`/`→` keys navigate; sets `body{overflow:hidden}` while open. |

Design principle worth preserving: **the site works without JavaScript** (tabs and menu are CSS-only; gallery links fall back to opening the full image directly).

### 5.6 Performance and accessibility work already done

From the "Fix Lighthouse performance, accessibility, and SEO issues" PR (#2):

- Responsive hero images via `srcset` (640/1280/1920) with `sizes="100vw"`, explicit `width`/`height` on hero images (prevents CLS), `fetchpriority="high"` + `decoding="async"` on the LCP image.
- `loading="lazy"` on all gallery/teaser/about images.
- Async, non-render-blocking Google Fonts.
- `aria-label`s on hamburger, hero arrows, lightbox controls, and language links (translated per language).
- Meaningful `alt` text on every image (see section 7).

---

## 6. Path and URL conventions (important, easy to break)

| Context | Rule | Example |
|---|---|---|
| Root pages -> assets/css | **relative** | `assets/...`, `css/style.css` |
| `hu/` pages -> assets/css | **`../` relative** | `../assets/...`, `../css/style.css` |
| Favicons | absolute from site root | `/favicon.png`, `/apple-touch-icon.png` |
| Internal page links | **extensionless** | `moja-praca`, `cennik`, `o-mne`, `kontakt` (never `moja-praca.html`) |
| Home links | `/` (SK) and `/hu/` (HU) | |
| Cross-language link | SK page `moja-praca` <-> HU `hu/moja-praca` (SK side) / `/moja-praca` (HU side) | |
| Canonical / hreflang | absolute `https://kikaphotos.com/...`, **extensionless** | `https://kikaphotos.com/hu/cennik` |

Notes:

- Extensionless URLs rely on Cloudflare Pages' clean-URL behaviour. Opening the files from disk (`file://`) or via a server that does not do this will 404 on internal links. Use a dev server that supports clean URLs, or test with the `.html` extension locally.
- `sitemap.xml`, `<link rel="canonical">`, and `llms.txt` all use the extensionless form. (`llms.txt` originally used `.html` links; it was aligned to the canonical form.) Keep new links in all three extensionless.
- Inside `hu/` pages, relative nav links like `href="moja-praca"` resolve to `/hu/moja-praca`, which is intended.

---

## 7. Content reference (what each page says)

Keep factual content in sync across both languages.

**Home:** tagline "Portréty, športy, akcie - svetlo zachytené tak, ako naozaj vyzeralo." Hero shows 4 slides (portraits, sports, events x2) with the brand script "KikaPhotos". Three teaser cards link to `moja-praca#portraits`, `#sports`, `#events` using `portrety-10`, `sporty-08`, `akcie-02` thumbs.

**About:** quote *„Každý človek má svoj príbeh.. ja ho zachytím.“*; hobbyist since ~2022; prefers photographing people and natural light; moving into concerts and sports; wants to go professional "in a few years". Portrait image `ja.webp`.

**Contact:** name Kristína Kaszbeková, `mailto:info@kikaphotos.com`, Instagram `https://www.instagram.com/_kikaphotos_/` (`target="_blank" rel="noopener"`). No contact form.

**Pricing (EUR)** - three cards; identical numbers in SK and HU:

| Card | Tier | Price | Includes |
|---|---|---|---|
| Portraits & family | Mini | 25 € | 20-30 min, 10 edited photos |
| | Standard | 45 € | 45-60 min, 20 edited photos |
| | Premium | 65 € | 60-90 min, 30 edited photos |
| | (all tiers) | | photo selection option, delivery within 7 days |
| Sports events | Základ / Alap | 30 € | 10 edited photos |
| | Standard | 55 € | 30 edited photos |
| | Premium | 80 € | 50 edited photos |
| | Custom pack | 90 / 110 / 130 € | 60 / 80 / 100 photos |
| Events & concerts | 1 hour | 50 € | min. 20 edited photos |
| | 2 hours | 90 € | min. 40 edited photos |
| | 3 hours | 130 € | min. 60 edited photos |
| | each extra hour | +30 € | |
| Info strip (all) | | | electronic delivery; colour and light edited; **express delivery within 48 h: +20 €** |

Closing note: "Tešim sa na spoluprácu! ♡" (SK) / "Várom a közös munkát! ♡" (HU).

**Alt-text convention (gallery):**

| Category | Slovak | Hungarian |
|---|---|---|
| Portraits | `Portrét - fotografia z portfólia KikaPhotos` | `Portré - fénykép a KikaPhotos portfólióból` |
| Sports | `Šport - fotografia z portfólia KikaPhotos` | `Sport - fénykép a KikaPhotos portfólióból` |
| Events | `Akcia - fotografia z portfólia KikaPhotos` | `Esemény - fénykép a KikaPhotos portfólióból` |

---

## 8. SEO and discoverability

- **`<title>`:** homepage is just `KikaPhotos`; inner pages use `<Page name> - kikaphotos.com`.
- **`<meta name="description">`** is unique and translated per page.
- **`robots.txt`:** `Allow: /` for all agents, points to `https://kikaphotos.com/sitemap.xml`.
- **`sitemap.xml`:** lists all 10 URLs (extensionless), all with `lastmod 2026-07-23`, `changefreq daily`, priorities descending (SK home 1.0, HU home 0.8, SK inner 0.64, HU inner 0.512). Generated by xml-sitemaps.com (note the `xml-stylesheet` line). **It is hand-maintained; update `lastmod` and add entries when pages are added.**
- **`llms.txt`:** plain-language summary for LLM crawlers, grouped SK/HU, states this is an amateur hobby portfolio with informal rates. Update if pages, prices framing, or contact details change.
- Every page has `canonical` + `hreflang sk/hu`. Add both when creating a page (in both languages).
- No Open Graph / Twitter Card tags, no JSON-LD structured data currently. (Reasonable future additions, not present today.)

---

## 9. Development workflow

- **No install, no build, no tests, no linter.** There is nothing to run for verification beyond opening pages in a browser.
- **Preview locally:** serve the repo root with any static server, e.g. `python3 -m http.server 8000` in the repo root. Extensionless internal links will not resolve there (see section 6); append `.html` manually or use a server with clean-URL support (e.g. `npx serve`, which handles it).
- **Deploy:** push to `main`; Cloudflare Pages deploys automatically (no CI config in repo).
- **Git history:** 19 commits, feature work via small PRs (`testhu` -> Hungarian version, `lighthouse-fixes`, `portfolio-lightbox-reorg`). Direct commits to `main` also occur. Commit messages are a mix of Slovak and English, short and imperative.
- **`.gitignore`** excludes `.claude/`, OS/editor cruft (`.DS_Store`, `Thumbs.db`, `.vscode/`, `.idea/`, swap files), `node_modules/`, logs, `dist/`, `build/`, and `*.min.css`/`*.min.js`. The Node/build entries are precautionary; nothing in the repo uses them.

---

## 10. Rules for agents (do / don't)

**Do**

1. **Mirror every content/structure change in both languages.** SK root file and its `hu/` twin must stay structurally identical (only text, `lang`, asset path prefix `../`, and nav/aria strings differ).
2. Keep internal links **extensionless** and asset paths correct for the file's depth (`assets/` vs `../assets/`).
3. Reuse existing CSS classes and design tokens; add new rules to `css/style.css` (single stylesheet), using the `:root` variables rather than hard-coded colours.
4. Keep gallery images as **paired `full` + `thumb` WebP** with matching zero-padded names.
5. Preserve the no-JS-friendly patterns (CSS tabs, CSS hamburger, `href` fallback on lightbox links).
6. Keep new images `loading="lazy"` (except the LCP hero slide) and give them meaningful, translated `alt` text and explicit dimensions where layout shift is possible.
7. Add `canonical` + both `hreflang` tags and a sitemap entry for any new page.

**Don't**

1. Don't introduce a framework, bundler, `package.json`, or external JS dependency without being asked. The site's value is that it is a simple, dependency-free static site.
2. Don't use em dashes in copy (use ` - `).
3. Don't add JPEG/PNG photos; use WebP.
4. Don't add `.min.css` / `.min.js` files (gitignored).
5. Don't remove or reorder the GA snippet, and don't change the GA ID without being asked.
6. Don't commit RAW/originals or large unoptimised images. Keep full images at ~2000 px long edge and quality 92 (see section 4.4).
7. Don't change prices, contact details, or the personal bio text without explicit instruction; these are the owner's real business information. Note that the site frames itself as an **amateur/hobby** portfolio, so avoid wording that implies a professional studio.
8. Don't switch to the `.html` link form or absolute-path links for internal navigation.

---

## 11. Quick reference

```text
Domain ........ kikaphotos.com          Owner ......... Kristína Kaszbeková
Languages ..... sk (default, /), hu (/hu/)
Pages ......... 5 per language (index, moja-praca, cennik, o-mne, kontakt) = 10 HTML files
Stylesheet .... css/style.css (928 lines)       JS ......... inline vanilla only, no deps
Fonts ......... Fraunces, Allura, Work Sans (Google Fonts, async)
Images ........ WebP only (46 gallery photos x {full 2000px, thumb 700px}) + 4 heroes x {640,1280,1920}
Icons ......... favicon.png 64x64, apple-touch-icon.png 180x180, inline SVG for UI
Analytics ..... GA4 G-NXZSSNFPJQ        Hosting ..... Cloudflare Pages (no build)
Contact ....... info@kikaphotos.com, instagram.com/_kikaphotos_
```
