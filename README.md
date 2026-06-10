# World Cup Personality Quiz — NewsBreak H5

A mobile-first, shareable World Cup personality quiz. 12 questions → 1 of 6 World Cup
identities, with **dynamically computed** stats. Self-contained static site (vanilla
HTML/CSS/JS, no build step) in the same engineering style as `memorial-day-tribute`.

## Run

It's a static site — just serve the folder:

```bash
cd the-world-cup-quiz
python3 -m http.server 8000
# open http://localhost:8000
```

(Open over http, not `file://`, so the `<img>`/share APIs behave. Deploy = upload the
folder to any static host, e.g. `staticfiles.particlenews.com/nwb/...`.)

## Structure

```
index.html          the entire app — config + scoring engine + 3 screens + sound, all inline
assets/
  opening.jpg       landing poster
  q1.jpg … q12.jpg  question artwork (NewsBreak logo kept top-left)
  maestro.jpg …     6 result heroes — cropped above the stat bars, logo/SHARE cleared,
                    so the stats can be re-rendered live in HTML
*.png               original full-res source art (kept for re-cropping; not loaded by the app)
```

## What's interactive (vs. the static artwork)

- **Landing** — full poster, pulsing START overlay.
- **Questions are stitched from modules, not letterboxed.** Each `qN.png` is sliced (in
  `assets/q/`) into a question **panel** + 3 **card** images (cut at the whitest gap row so players
  aren't clipped). A recreated HTML top bar (logo + progress + back arrow) sits above them, and the
  modules flex to **fill any phone full-bleed** — no white bars. Slice rects come from `QRECTS`.
- On tap: a physical **press** + **click sound** (Web Audio, synthesized) + **checkmark** (no border).
- **Back arrow** in the top bar (from Q2 on) returns to revise; the prior answer is pre-highlighted.
  Scores derive from the `answers[]` array, so editing is safe.
- **Landing** is full-bleed (`opening_land.jpg`, the poster cropped above its baked CTA) with a
  recreated HTML **START** button matching the drawn red pill.
- The sound toggle is **hidden** (sound stays on by default).
- **Result** — hero image blends seamlessly into an HTML panel whose **stat bars are
  generated from the answers** (not baked into the image), plus traits, similar-style names,
  share + take-again. Progressive reveal, bars animate, numbers count up.

## Scoring engine (in `index.html`)

- 6 dimensions: `lead, vis, cre, clu, rel, fla`.
- `Q[]` maps every answer to dimension points. `P{}` gives each personality a weight vector;
  the result is the **highest weighted match** (deterministic — never random).
- `statValues()` maps the user's dimensions to each personality's 4 displayed stats (80–99,
  aspirational descending order).

### Tuning

- Edit answer→dimension points in `Q`, or personality weights in `P[key].w`.
- The weight set was balanced by brute-forcing all 3¹² answer combinations; current spread is
  roughly maestro 12% / captain 22% / clutch 14% / creator 20% / guardian 11% / wildcard 21%.
- Option hit-zones are positioned per question by `QCENTERS` (fraction of image height). If you
  swap the artwork, re-measure those centers.

## Crowd share ("X% got this result")

- The result page shows **"X% of players are THE …"**. Reuses the **Memorial Day Google Apps
  Script counter** (same deployed web app) — no new backend. Each finished quiz fire-and-forget
  POSTs `action=hero&id=WCQ_RES_<personality>`; per-result totals are read back via JSONP on load.
  Keys are namespaced `WCQ_RES_` so they coexist with the Memorial `HERO_`/`GLOBAL` rows.
- Shows a **mock** share (`RES_MOCK`) until total results reach **≥50** (`REAL_THRESHOLD`), then the
  live proportion. Endpoint is `COUNTER_API` in `index.html`.

## Sharing (card + link)

- Reuses **Memorial Day's `unifiedShare` cascade**: native share with the **card image File** →
  native share URL-only → clipboard → legacy `execCommand` copy.
- Each personality's result card (`assets/share/<key>.jpg`) is pre-fetched as a `File` on the
  result page (`prefetchCard`) so the share panel carries the **card image + message + link**.
- Falls back to copying the link (toast) when files/native share aren't available.

## Notes

- The result page itself is also the share visual — users can screenshot it.
