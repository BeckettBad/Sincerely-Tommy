# Plan — Blend Che into the homepage hero collage

**Goal:** Make the three worlds read as one brand the moment the homepage opens. Right
now the rotating hero collage shows only shop / editorial product photography. We want
Che (the kitchen + café + space) to appear in that same rotation so the food and
hospitality side is part of the first impression, not a separate page you scroll to.

This doc is written to be pasted straight into the VSCode + Claude build instance. It
references the live file `index.html` (the mockup at https://beckettbad.github.io/Sincerely-Tommy/).

---

## 1. How the hero collage works today (don't break this)

The hero is a 4-column grid. Each column is a `.tile` containing **3 stacked `<img>`s**
that cross-fade on a loop:

```css
.collage{position:absolute;inset:0;display:grid;grid-template-columns:repeat(4,1fr)}
.tile img{position:absolute;inset:0;width:100%;height:100%;object-fit:cover;
          opacity:0;filter:saturate(.96);animation:revolve 18s ease-in-out infinite}
@keyframes revolve{0%{opacity:0}5%{opacity:1}28%{opacity:1}38%{opacity:0}100%{opacity:0}}
@media(prefers-reduced-motion:reduce){.tile img{animation:none}.tile img:first-child{opacity:1}}
```

Key facts:
- One 18s loop. Each image is opaque ~5%→28% then fades out by 38%.
- 3 images per tile, staggered by `animation-delay` 6s apart (18 ÷ 3) so each column always
  shows one image and crossfades cleanly.
- Current delays: tile1 `0,-6,-12` · tile2 `-2,-8,-14` · tile3 `-4,-10,-16` · tile4 `-1,-7,-13`
  (the per-tile offsets keep columns out of sync so the wall feels alive).
- On mobile the grid collapses to 2 columns; `prefers-reduced-motion` shows only the first
  image per tile, so **put a strong still image first in each tile.**

There are 12 image slots total (4 tiles × 3). The integration is about getting Che into
those slots without muddying the timing.

---

## 2. Recommended approach — give every column a Che image (4 images per tile)

Add **one Che image to each of the 4 tiles** so Che is represented in every column and is
guaranteed on screen as the collage rotates. That means 4 images per tile, so the loop
timing must be re-spaced from thirds (6s) to quarters (4.5s), and the fade envelope
tightened slightly so two images don't sit opaque at once.

### 2a. Replace the keyframe + add a 4-up timing rule

```css
.tile img{position:absolute;inset:0;width:100%;height:100%;object-fit:cover;
          opacity:0;filter:saturate(.96);animation:revolve 18s ease-in-out infinite}
/* 4 images per tile: 25% slots, ~28% on-and-fade envelope for a soft crossfade */
@keyframes revolve{0%{opacity:0}3%{opacity:1}20%{opacity:1}28%{opacity:0}100%{opacity:0}}
```

### 2b. Re-space the delays to quarters (4.5s apart)

Within each tile use `0, -4.5s, -9s, -13.5s`. Keep a small per-tile offset so columns stay
out of sync. Suggested:

| Tile | img1 (delay) | img2 | img3 | img4 |
|------|-----|-----|-----|-----|
| 1 | `0s` | `-4.5s` | `-9s` | `-13.5s` |
| 2 | `-1s` | `-5.5s` | `-10s` | `-14.5s` |
| 3 | `-2s` | `-6.5s` | `-11s` | `-15.5s` |
| 4 | `-3s` | `-7.5s` | `-12s` | `-16.5s` |

### 2c. Distribute Che across the tiles for variety

Aim for one Che image per column, mixing subject matter so the wall reads as *shop + food +
space*, not "a product grid with one food photo." Target this spread (swap to match the
best assets you actually pull):

- **Tile 1:** add a **food / plated dish** Che image.
- **Tile 2:** add a **café interior / counter** Che image.
- **Tile 3:** add a **natural-wine / table moment** Che image.
- **Tile 4:** add an **exterior / room (the "Stay" side)** Che image.

Keep each tile's **first** image a strong shop/editorial still (reduced-motion + initial
paint), and place the Che image as 2nd or 3rd in the stack so the column tells a
shop→food→shop story as it cycles.

### Example — Tile 1 after edit (pattern, repeat for all four)

```html
<div class="tile">
  <img loading="eager" style="animation-delay:0s"     src="https://cdn.shopify.com/s/files/1/0622/1337/files/DSC_3611_d7ace9da-15e1-496d-99f4-a0506cd098e4.jpg?v=1762550787&width=900" alt="Editorial — Umbra knitwear">
  <img loading="lazy"  style="animation-delay:-4.5s"  src="/assets/che/che-dish-01.jpg" alt="Che — seasonal vegetable plate">
  <img loading="lazy"  style="animation-delay:-9s"    src="https://cdn.shopify.com/s/files/1/0622/1337/files/DSC_3110.jpg?v=1762550734&width=900" alt="Editorial">
  <img loading="lazy"  style="animation-delay:-13.5s" src="https://cdn.shopify.com/s/files/1/0622/1337/files/F748A144-40DF-4531-B2AB-AD7A8D709D98_1_52b9978f-99da-4f79-a9f7-4a6b8fbfd2b4.jpg?v=1762207287&width=900" alt="Editorial">
</div>
```

> **Minimal alternative (no timing change):** if you'd rather not retune, keep 3 images per
> tile and simply *replace* the current 3rd image in 3–4 tiles with a Che image. Zero CSS
> changes, slightly less Che presence. Use this if the 4-up retiming ever looks busy.

---

## 3. Get the Che images (the assets)

The Che site (`che-brooklyn.square.site`) is a **Square Online** site — fully
client-rendered, so its images can't be scraped by a plain HTTP fetch. Pull them in a real
browser. All Che assets live under a single uploads namespace:
`/uploads/b/99a06440-0c8a-11ef-8f6f-438b31c77f5c/…` and accept a `?width=` query param for
sizing (e.g. `?width=900`).

### Harvest snippet — run in the browser console on the Che site

Open https://che-brooklyn.square.site/ , scroll the whole page (to trigger lazy-loading),
then paste this in DevTools console. Repeat on `/menu` and `/contact-us`.

```js
// Collect every Che-hosted image URL on the current page
const fromImgs = [...document.querySelectorAll('img')]
  .map(i => i.currentSrc || i.src);
const fromBg = [...document.querySelectorAll('*')]
  .map(e => getComputedStyle(e).backgroundImage)
  .filter(b => b && b !== 'none')
  .map(b => b.slice(b.indexOf('http'), b.lastIndexOf('"') > 0 ? b.lastIndexOf('"') : undefined));
const all = [...fromImgs, ...fromBg]
  .filter(u => u && u.includes('/uploads/'))
  .map(u => u.split('?')[0])               // strip size param to get the master
  .filter((v, i, a) => a.indexOf(v) === i); // dedupe
copy(all.join('\n'));                        // now on your clipboard
console.log(all);
```

Pick the 4–8 strongest shots (good light, on-brand, not low-res menu thumbnails):
food/plated dishes, the café counter/interior, a natural-wine moment, and an
exterior/room shot for the "Stay" side.

### Don't hotlink Square long-term — download + host

The shop images in the mockup are hotlinked from Shopify's CDN, which is fine for a concept.
For Che, **download the chosen files, optimize, and commit them to the repo** so the homepage
doesn't depend on Square's CDN (it can change paths or block hotlinking):

```bash
mkdir -p assets/che
# for each chosen URL (request a sensible width):
curl -L "https://che-brooklyn.square.site/uploads/b/99a06440-.../FILENAME?width=1200" \
  -o assets/che/che-dish-01.jpg
# optional: convert/optimize to webp with a jpg fallback
# cwebp -q 82 assets/che/che-dish-01.jpg -o assets/che/che-dish-01.webp
```

Suggested filenames: `che-dish-01.jpg`, `che-cafe-01.jpg`, `che-wine-01.jpg`,
`che-room-01.jpg`. Reference them as `/assets/che/…` in the collage (as in the §2c example).

---

## 4. Make Che images sit consistently with the shop shots

- The collage already applies `filter:saturate(.96)`. If Che food photos read warmer/more
  saturated, nudge just those: add a class, e.g. `.tile img.che{filter:saturate(.92) brightness(.99)}`,
  so the wall stays calm and cohesive (per the agreed "calm, let the photography talk"
  direction — avoid anything that shouts).
- Match aspect/crop: tiles are full-bleed `object-fit:cover`, so favor Che images that
  survive a tall crop (food shot from above, a vertical interior). Avoid wide group shots
  that lose their subject when cropped to a column.
- Always set a meaningful `alt` (e.g. `alt="Che — natural wine and small plates"`); it
  reinforces the "one brand" story for SEO + accessibility.

---

## 5. Verify before commit

- Load the page: every column should show a Che image at some point within one ~18s loop,
  with clean (not abrupt, not double-exposed) crossfades.
- Toggle `prefers-reduced-motion` (DevTools → Rendering → Emulate CSS prefers-reduced-motion):
  each tile should hold its first (shop) still — confirm none default to a weak Che thumb.
- Resize to mobile (≤900px): grid drops to 2 columns; confirm Che still appears and images
  aren't pixelated (that's why we host at ~1200px width).
- Lighthouse/Network: confirm Che images are the hosted `/assets/che/…` files, not Square
  hotlinks, and are reasonably sized (<300KB each ideally).

---

## 6. Known data already in hand (no need to re-pull)

Che day menu (already in the mockup's `#che` section): Che's Cereal · Kitchari & Okra Roti ·
Yuca Colcannon · Turmeric-Ginger Moong Bean · Local Kale & Fried Okra · Apple-Ginger Purée.
Natural wine: NY, South Africa, Spain. Hours: Daily 10AM–5PM · 302 Malcolm X Blvd, Brooklyn,
NY 11233 · Che@sincerelytommy.com · 347.221.0216.

Che's full live catalog (food + wine + cocktails) is enumerable from the sitemap at
`https://che-brooklyn.square.site/sitemap.xml` if you later want to build out a real Che
menu page from source.

---

## 7. Open to-dos (tracked elsewhere)

- Replace remaining hotlinked Shopify images with hosted/optimized assets.
- HEIC re-exports (e.g. Dasol Red Turtleneck) → web-safe jpg/webp.
- Real Che/Grenada photography beyond what's on the Square site (ask the client for the
  good originals — Square copies are compressed).
