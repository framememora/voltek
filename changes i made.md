# Changes I Made — Voltek Industries Site

All changes are in `index.html` (working copy, not yet committed to git) plus a new `assets/` folder. This note covers every change made across the whole session, with the evidence that backs each claim.

---

## 1. Mobile responsiveness

**Problem:** The page had media queries but no `<meta name="viewport">` tag, so phones rendered the whole page at desktop width (~980px) and scaled it down — the breakpoints never actually triggered on a real phone.

**Changed:**
- Added `<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">` (`index.html:3`)
- Added 16px minimum font-size on form inputs inside the phone breakpoint, to stop iOS Safari auto-zooming on focus
- Resized the service carousel cards from a fixed 320px to `78vw` and narrowed the edge-fade mask from 48px to 20px, so cards aren't clipped on a 375px-wide phone
- Added `text-size-adjust`, tap-highlight removal, and `touch-action: manipulation` for snappier mobile taps

**Evidence:**
- Grep confirms the tag is live: `index.html:3` → `<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">`
- Browser test at a 390×844 viewport (iPhone-sized): brand name correctly hides, hero/eligibility/services/trust/cert/CTA/footer sections all reflowed to single-column layouts, confirmed visually section-by-section during the session
- Browser test at 768×900 (tablet): trust grid correctly switched to 2 columns, matching the `@media (max-width:860px)` rule

---

## 2. SEO metadata

**Problem:** No meta description, no Open Graph/Twitter tags, no canonical URL, no structured data — effectively invisible to search engines and link previews.

**Changed:** Added to `<head>`:
- `meta description`, `keywords`, `robots`, `author`, `copyright`
- `link rel="canonical"` → `https://voltekindustries.co.in/` (domain you confirmed)
- Full Open Graph block (`og:type`, `og:site_name`, `og:title`, `og:description`, `og:url`, `og:image`, `og:locale`)
- Twitter Card block (`summary_large_image`, title, description, image)
- `LocalBusiness` JSON-LD structured data (name, phone, founding date 1998, Kerala service area)

**Evidence (grep against the live file):**
```
3:<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
6:<meta name="description" content="Voltek Industries has engineered solar installations...">
11:<link rel="canonical" href="https://voltekindustries.co.in/">
17:<meta property="og:title" content="Voltek Industries — Solar & Power Conversion Since 1998">
23:<meta name="twitter:card" content="summary_large_image">
28:<script type="application/ld+json">
```
- Parsed the JSON-LD in-browser to confirm it's valid, not just present: `JSON.parse()` succeeded with no error, and `ldParsed.name === "Voltek Industries"`.

---

## 3. Copyright notice

**Changed:** Added a footer line, separated by a divider, with the year auto-updating via JS so it never goes stale.

**Evidence:**
- `index.html:616` → `<span class="mono">© <span id="copyYear">2026</span> Voltek Industries. All rights reserved.</span>`
- `index.html:653-654` → JS sets `copyYearEl.textContent = new Date().getFullYear()` on load
- In-browser check returned `copyYearText: "2026"` (matches system date 2026-08-05)
- Visually confirmed in a screenshot: copyright line rendered below a divider under the main footer row

---

## 4. Split the 6.6MB single-file site into real, cacheable assets

**Problem:** Every font, photo, and the hero video were inlined as base64 directly in the HTML — a 6,583,806-byte (6.3MB) single file. This meant no browser caching between visits, no CDN benefit, and a slow first load, especially on Indian mobile data.

**Changed:** Wrote a Python script that scanned the file for every `data:...;base64,...` URI, decoded it, and wrote it to `assets/{fonts,images,video}/`, replacing the inline data with a relative path. Duplicate payloads (the service marquee repeats 3 images for its seamless loop) were deduplicated by content hash — reused instead of re-embedded.

**Evidence:**
- File size before/after: `6,583,806 bytes → 61,627 bytes` (a 99% reduction in `index.html` itself)
- 24 unique files extracted into `assets/`:
  - 7 fonts (`plex-cond-600/700.woff2`, `plex-sans-400/500/600.woff2`, `plex-mono-400/500.woff2`)
  - 16 images (hero poster, cert background, 3 service photos, 3 project case photos, 8 client logos)
  - 1 video (`hero-video.mp4`)
- Confirmed via `find assets -type f | wc -l` → `24` (later 23 visible files + the renamed one, all accounted for)
- **Byte-identical verification:** decoded the original inline base64 straight from a copy of the pre-change file and SHA-256-hashed it against the extracted file:
  ```
  orig sha256:      e096dc094e243571cec971ed1895cfc786e057fd88f38239dcba0b8b063e660f
  extracted sha256: e096dc094e243571cec971ed1895cfc786e057fd88f38239dcba0b8b063e660f
  MATCH: True
  ```
  (same test run again just now, independently, with the same result — see section 6)
- Live-served all 24 assets and the page itself: 27 network requests recorded, **all returned HTTP 200**
- Font-face declarations kept their `format('woff2')` suffix intact after extraction (checked directly): 
  `@font-face { font-family: 'Plex Cond'; ... src: url(assets/fonts/plex-cond-600.woff2) format('woff2'); font-display: swap; }`

---

## 5. Favicon

**Problem:** No favicon at all — blank/generic tab icon.

**Changed:** Added `<link rel="icon">` and `<link rel="apple-touch-icon">` pointing at the existing `voltek-logo.jpg`.

**Evidence:**
- `index.html:12` → `<link rel="icon" type="image/jpeg" href="voltek-logo.jpg">`
- Confirmed the logo is a clean square (suitable for a favicon) by reading its actual dimensions: `(640, 640) RGB`

---

## 6. Quote form reliability

**Problem:** The "Send Request" button only did `window.location.href = "mailto:..."`. On mobile, if the visitor has no configured email app (common), the button silently does nothing.

**Changed:** Primary action now opens WhatsApp with the form fields prefilled (matching the channel this business already uses everywhere else on the page). `mailto:` is kept as a secondary "Send by email instead" link.

**Evidence:**
- Filled the live form with test data (Name: "Test User", Phone: "9999999999", Location: "Kochi, Kerala", Requirement: "Residential rooftop") and clicked submit.
- The click opened a new tab whose URL was captured directly:
  ```
  https://api.whatsapp.com/send/?phone=919846081688&text=Hi+Voltek%2C+I%27d+like+a+quote.%0AName%3A+Test+User%0APhone%3A+9999999999%0ALocation%3A+Kochi%2C+Kerala%0ARequirement%3A+Residential+rooftop&type=phone_number&app_absent=0
  ```
  Decoded, the message reads exactly as intended: *"Hi Voltek, I'd like a quote. Name: Test User / Phone: 9999999999 / Location: Kochi, Kerala / Requirement: Residential rooftop"* — this proves the button, the field reads, and the WhatsApp deep-link encoding all work end-to-end, not just that the code looks right.

---

## 7. Investigated the hero video playback concern (no code change needed)

**Background:** In an earlier pass I flagged that the hero `<video>` never reached a "playing" state in this sandboxed test browser and suggested a manual phone check after deploy. You asked me to actually verify it myself instead of just flagging it — so I did, this session.

**What I found, in order:**
1. Re-confirmed the extracted video file is byte-identical to the original working copy (SHA-256 match, shown above) and has a standard, universally-supported codec: inspected the MP4's `stsd` box directly and found `avc1` (H.264) — not an obscure or unsupported codec.
2. Asked the browser itself whether it supports this codec: `video.canPlayType('video/mp4; codecs="avc1.42E01E"')` → `"probably"` (Chrome 150, confirmed via `navigator.userAgent`). Codec support isn't the issue.
3. Fetched the video file with plain `fetch()` (bypassing the `<video>` element entirely): **got all 2,428,736 bytes back correctly in 59ms.** So the network path, the server, and the file are all fine.
4. Took those exact fetched bytes, built a `Blob` URL from them (no network involved at all at this point), and pointed a `<video>` at it. It **still** never reached `readyState ≥ 1` after 4 seconds, with no error event.
5. As a final control, pointed a fresh `<video>` at a well-known external test file (`w3schools.com/html/mov_bbb.mp4`) — completely unrelated to this project. **Same result:** stuck at `readyState: 0`, no error.

**Conclusion:** Step 5 is the decisive test — an unrelated, known-good, externally-hosted video fails identically in this same browser tab. That means *no* video plays in this particular automated/sandboxed browser session, regardless of source. This is a restriction of the test environment itself (likely the media/decode pipeline being disabled for the automated Chrome session), not a defect in your file, your server, or my extraction work. **No code change was made for this item** because there is nothing in the site to fix — the evidence points away from the site and at the test harness. Worth a real-device spot check after you deploy, purely as routine due diligence, not because anything here points to an actual bug.

---

## Not touched (per your earlier instruction)

The eligibility PIN checker (`checkPin()` in the `<script>` at the bottom of `index.html`) — still client-side theater that accepts any 6-digit input. Left as-is on your explicit instruction.

## Not yet committed

All of this is in the working directory (`git status` shows `index.html` modified, `assets/` untracked). Nothing has been committed to git — say the word if you want it committed.
