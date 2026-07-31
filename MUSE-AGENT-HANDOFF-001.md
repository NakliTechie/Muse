# Muse — Agent Handoff (rev 001)

Implements MUSE-VISION-ROADMAP-001.md. Milestones M0 → M1 → M2. Nothing advances
past a gate with its verifier red or its fix-list open.

## 1. Repo / build / deploy

- Repo: `NakliTechie/Muse`, public.
- One file: `index.html`. Vanilla JS, no npm, no bundler, no framework, no build
  step. Inline CSS + JS. Shaders as template literals.
- Deploy: Cloudflare Pages → `muse.naklitechie.com`. M1 and M2 are two deploys of
  the same codebase.
- Visible version string in UI footer + `<meta name="version">`, bumped every push.

## 2. Browser floor

- WebGL2 required. Absent → full-viewport error state (see §6), no partial UI.
- getUserMedia required for M2 camera path only; absence/denial degrades to upload.
- File System Access API primary for save; `<a download>` fallback. No OPFS/IDB
  needed for images (nothing persists except presets/settings).
- Targets: latest-2 Chrome/Edge/Firefox/Safari, desktop + mobile. Mobile Safari is
  the perf floor for M2 — test there, not just desktop Chrome.

## 3. Design tokens & UI (inline UX reference)

- CSS custom properties from day one, house palette mapped onto:
  `--bg`, `--surface`, `--ink`, `--accent`, `--muted`, `--danger`.
- Dark default (photo tool). Editor chrome stays out of the image's way.
- **Surfaces (2):**
  - **Editor** — image viewport center; look deck as horizontal thumbnail strip
    (each thumb = live preview of that look on the loaded image, rendered small);
    slider panel in a right drawer (desktop) / bottom sheet (mobile), grouped by
    pass with per-pass bypass toggles; export + save-preset in top bar.
  - **Camera** (M2) — full-viewport live preview with active look applied, look
    strip overlaid bottom, shutter button, flip control, ✕ back to editor.
- Icons: inline SVG only, single style family, no icon font, no external fetch.
- Sliders: native `<input type=range>` restyled via tokens — keyboard operable for
  free.
- Before-and-after: press-and-hold on the viewport shows the original.

## 4. Empty / error UX

| State | Treatment |
|---|---|
| No image loaded | Drop-zone hero: drop hint + "open camera" (M2) + sample image button. Never a blank canvas. |
| WebGL2 unavailable | Full explanation card: what's missing, which browsers work. No dead sliders. |
| Camera denied | Inline notice in camera surface + auto-return to upload path. Never re-prompt loop. |
| Huge image (>24MP) | Downscale for preview, warn, full-res on export if memory allows; else export at capped res with notice. |
| Unreadable file | Toast: format not supported; accepted: PNG/JPEG/WebP. |
| Recipe found in dropped image | Prompt: "This image carries a Muse recipe — restore the edit?" (in-app confirm, not native `confirm()`). |

## 5. Persistence rules

- **localStorage allowed:** custom presets (`muse.presets`), UI language
  (`muse.lang`), intro-modal-seen flag (`muse.intro`), agent-face toggle
  (`muse.dev`). Nothing else.
- **Forbidden:** image data in any storage tier; canvas fingerprint-adjacent data;
  any network transmission of image bytes, ever.

## 6. CSP & security posture

- CSP meta: `default-src 'self'; img-src 'self' blob: data:; connect-src` GA
  endpoint only; no other external origin. No CDN dependencies in v1 (no models,
  no fonts — system font stack).
- GA: pageview-only, per Grain precedent. No event tracking touching image
  content or filenames.
- Camera stream never leaves the WebGL texture path; stopped (tracks released)
  the moment the camera surface closes.

## 7. a11y & keyboard

- All sliders labelled, grouped in `fieldset`s per pass; drawer/sheet focus-trapped.
- Shortcuts: `Space` (hold = before/after), `1–4` (looks), `Cmd/Ctrl+S` (export,
  preventDefault), `?` (help), `Esc` (close drawer/camera). No conflicts with
  browser-reserved combos; document in help modal.
- Contrast: chrome text ≥4.5:1 against `--surface`.

## 8. Agent face

`window.muse`, gated behind `muse.dev` setting, off by default:

```
muse.getLooks() → [{id, name, params}]
muse.applyLook(source, lookIdOrParams, {seed?}) → Promise<Blob>   // source: Blob|ImageData
muse.getRecipe() → current param JSON
muse.setRecipe(json) → applies to loaded image
muse.export({format}) → Promise<Blob>  // recipe embedded, via:"agent" marked
```

Same pipeline the UI drives — one mechanism, two doors. These hooks are also the
headless test surface.

## 9. Milestones, gates, verifiers

Termination conditions are deterministic — never agent self-report. Verification
runs in fresh context (maker–checker); forward-pass / walkthrough / ux-review on a
**different model family** than the builder, per ntkit posture.

**M0 — Fidelity spike**
- Build: 8-pass chain, param schema, seeded PRNG grain/leak, recipe
  serialise/deserialise.
- Gate artifacts: (1) `m0-determinism.html` self-test page — renders fixed test
  image × fixed params × fixed seed twice, SHA-256 of pixel buffers must match, and
  recipe JSON must round-trip byte-identical; result rendered as PASS/FAIL in DOM
  and console. (2) Comparison strip: test portraits × 4 looks, side-by-side PNG.
- Termination: self-test page shows PASS for all 4 looks × 3 test images.
  (Look-quality sign-off is Chirag's, on the strip — human gates shape.)

**M1 — Editor**
- Gate artifacts: deployed URL; `m1-selftest.html` — headless checks via agent
  face: recipe round-trip through a real export (embed → re-read → param-equal),
  applyLook returns hash-stable blob, preset save/load, i18n key coverage (no
  hardcoded strings outside the lang map — checked by scanning).
- Termination: all self-tests PASS + zero console errors on load + forward-pass
  fix-list closed.

**M2 — Camera**
- Gate artifacts: `m2-perf.html` harness — runs 300-frame preview loop, logs
  median/p95 frame time as JSON artifact; capture-to-editor recipe continuity
  test.
- Termination: harness JSON shows median ≤42ms at preview res on the test device
  + camera-denied path renders upload state (automatable via permissions mock) +
  walkthrough repairs complete.

## 10. Loop discipline

- Per-chunk budget cap; same failure ~3× consecutive → stop, write tried-trail to
  `NAKLITECHIE-PROJECT-STATE.md` entry, escalate.
- Escalate ONLY for: locked-decision conflicts, new dependency needs (there should
  be none — zero deps is locked), scope ambiguity that changes the product.
  Internals, shader math, debugging: proceed on own authority.
- Checker screens for reward hacking first: skipped/deleted self-test = classic
  hack; the determinism page must actually hash pixels, not stub PASS.

## 11. README scope & portfolio

- README: what it does for a user, screenshots, the four looks, recipe format
  spec (documented so anyone can write a reader — open format), privacy line
  ("your photos never leave your device — verifiable: no network tab activity"),
  keyboard map. No model names, no line counts.
- Update portfolio site + `NAKLITECHIE-PROJECT-STATE.md` on each milestone ship.

## 12. What NOT to do

- No npm, no framework, no build step, no external CDN, no font fetch.
- No image bytes over the network, ever, under any framing.
- No parallel Canvas2D filter path "for compatibility" — WebGL2 or the error
  state. One pipeline.
- No hand-rolled EXIF library — recipe embed is PNG `tEXt` / JPEG COM only.
- No native `alert/confirm/prompt` — in-app modals.
- No auto-downloading anything at load beyond the file itself.
- Don't copy shader code from GPL-licensed filter projects. MIT/Apache/BSD
  reference is fine with attribution notes in-source.

## 13. i18n

EN baseline, all strings through one lang map from the first commit. Scaffold
ready for additional locales; date-stamp rendering locale-independent (it's a
prop, not a date).
