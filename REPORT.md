# Site Audit Report — captainfuelright.com

**Date:** 2026-08-11
**Scope:** `index.html` + supporting assets (single-page static site, GitHub Pages)
**Reviewed:** SEO / structured data, social sharing, performance, accessibility, dead code, asset inventory, content consistency

---

## Summary

The site is in good structural shape: clean semantic HTML, strong accessibility fundamentals, valid JSON-LD, responsive layout, and self-contained inline CSS/JS. The issues below are refinements — nothing is broken for a normal desktop/mobile visitor — but three of them (social preview image, iOS touch icon, stale sitemap) have real external-facing impact and are cheap to fix.

| # | Finding | Severity | Effort |
|---|---------|----------|--------|
| 1 | Open Graph image is SVG — won't render on most social platforms | High | Low |
| 2 | `apple-touch-icon` is SVG — ignored by iOS | High | Low |
| 3 | `sitemap.xml` `lastmod` is stale (2026-04-16) | Medium | Trivial |
| 4 | Hero image (1.5 MB) has no `width`/`height` → layout shift (CLS) | Medium | Low |
| 5 | ~2.5 MB of orphaned image assets committed but unused | Medium | Trivial |
| 6 | Dead CSS: `.video-section` / `.video-placeholder` after Video section removal | Low | Trivial |
| 7 | External Google Fonts — render-blocking + contradicts "zero external deps" | Low | Medium |
| 8 | JSON-LD `sameAs` still links YouTube after YouTube card removed | Low | Trivial |
| 9 | Percentage inconsistency: "67%" vs "66.7%" (D2274) | Low | Trivial |

---

## Findings

### 1. Open Graph image is an SVG (High)
`index.html` sets `og:image` and `twitter:image` to `og-image.svg`.

Facebook, LinkedIn, X/Twitter, Slack, and iMessage **do not render SVG** as link-preview images — they require PNG or JPG. As written, shared links to the site will show a blank or fallback preview, undercutting the `summary_large_image` card and the declared `1200×630` dimensions.

**Fix:** Export a `1200×630` PNG (or JPG) version of the OG artwork and point `og:image` / `twitter:image` at it. Keep the SVG only if you also want a vector copy; the meta tags must reference the raster file.

### 2. `apple-touch-icon` is an SVG (High)
`<link rel="apple-touch-icon" href="/favicon.svg" />`

iOS ignores SVG for the home-screen / "Add to Home Screen" icon and falls back to a screenshot of the page. Apple touch icons must be PNG, typically `180×180`.

**Fix:** Add a `180×180` PNG (e.g. `apple-touch-icon.png`) and reference it. The `rel="icon" type="image/svg+xml"` line for the browser-tab favicon is fine to keep as SVG.

### 3. Stale `sitemap.xml` `lastmod` (Medium)
`sitemap.xml` reports `<lastmod>2026-04-16</lastmod>`, but content has changed since (recent commits rewrote FAQs, standardized the phone format, removed the Video section). Today is 2026-08-11.

**Fix:** Update `lastmod` to the last real content-change date whenever the page changes. With a single-page site this is low-cost to keep accurate; a stale date weakens the recrawl signal.

### 4. Hero infographic has no intrinsic dimensions (Medium)
`petro-fractions-v3.png` (~1.5 MB) is inserted with `loading="lazy"` (good) but **no `width`/`height` attributes**. Without them the browser can't reserve space, causing Cumulative Layout Shift (CLS) as the large image loads — a Core Web Vitals penalty.

**Fix:** Add explicit `width` and `height` (the image's natural pixel dimensions) so the `.ladder-figure img { width:100%; height:auto }` rule scales from a reserved box. Consider also serving a compressed/WebP version — 1.5 MB is heavy for the one raster on the page.

### 5. Orphaned image assets (~2.5 MB) (Medium)
These files are committed but referenced **nowhere** in `index.html`:

- `fuel-ladder-canva.png` (~1.24 MB)
- `fuel-ladder-canva-v2.png` (~1.26 MB)
- `fuel-ladder.svg` (~6.5 KB)

Only `petro-fractions-v3.png`, `favicon.svg`, and `og-image.svg` are actually used. The orphans don't ship to visitors (nothing links them) but they bloat the repo and clone size.

**Fix:** Delete them if they're superseded, or keep them intentionally and note why. (Left in place by this report — deletion is a content decision.)

### 6. Dead CSS after Video section removal (Low)
The most recent commit removed the Video section + YouTube contact card, but the CSS rules `.video-section` and `.video-placeholder` (and its `a` child) remain in the `<style>` block with no matching markup.

**Fix:** Remove the unused rules to keep the stylesheet honest.

### 7. External Google Fonts (Low)
The page preconnects and loads Inter from `fonts.googleapis.com` / `fonts.gstatic.com`. Two implications:

- The README states the site has "**zero external runtime dependencies**" — the font load contradicts that.
- It's a render-blocking external request, and serving Google Fonts to EU visitors has known GDPR sensitivities.

**Fix (optional):** Self-host the Inter subset (woff2) locally to make the "zero external deps" claim literally true, remove the render-blocking hop, and sidestep the privacy angle. If you prefer the CDN, soften the README wording.

### 8. JSON-LD `sameAs` still references YouTube (Low)
The Organization schema keeps `"sameAs": ["https://www.youtube.com/@captainfuelright"]` even though the visible YouTube card was removed. This may be intentional (the org still has a channel) — just confirm it's the desired public signal.

### 9. Percentage inconsistency for ASTM D2274 (Low)
0.5 vs 1.5 mg/100 mL is stated as "**67% below**" (JSON-LD, Technical section, Stats) and "**66.7% lower**" (FAQ). Both are correct roundings of 0.333, but pick one form for a consistent read. Relatedly, D2274 is framed as both "long-term storage stability" and "accelerated oxidation-stability" across sections — that's technically consistent (D2274 *is* an accelerated oxidation test used to infer storage stability), but the wording drifts between sections.

---

## What's already solid

- **Accessibility:** skip link, `aria-label`/`aria-labelledby` on sections, visible `:focus-visible` outlines, `prefers-reduced-motion` handling, semantic landmarks (`header`/`main`/`footer` with roles), and a keyboard/ARIA-wired mobile nav toggle.
- **Structured data:** valid `@graph` with Organization + WebSite + three Products, cross-linked via `@id`.
- **SEO basics:** canonical URL, description, Open Graph + Twitter tags (image format aside), `robots.txt` + sitemap present, descriptive `alt` text on the infographic.
- **Performance posture:** inline CSS/JS (no extra round-trips beyond fonts), `loading="lazy"` on the hero image, single HTML document.
- **Consistency:** phone `(888) 388-9211` and `info@captainfuelright.com` are uniform throughout.

---

## Recommended priority order

1. Swap OG image + Apple touch icon to PNG (#1, #2) — external-facing, breaks previews/icons today.
2. Refresh `sitemap.xml` `lastmod` (#3).
3. Add `width`/`height` to the hero image; consider WebP (#4).
4. Remove or justify orphaned assets and dead CSS (#5, #6).
5. Decide on fonts + tidy copy inconsistencies (#7, #8, #9).
