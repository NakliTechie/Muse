# Muse

> Take pictures of yourself like you're a Davide Sorrenti muse.

A single-file, browser-native 90s film-look photo tool. Drop a photo (v1.0) or open
the camera (v1.1), pick a look — **Sorrenti · Corinne Day · Disposable · Mall Studio** —
tune it with deep sliders, export. Every look is a parameter bundle over one
deterministic WebGL2 shader chain. **Nothing ever leaves the device.**

- One `index.html`. Vanilla JS, no npm, no bundler, no framework, no build step.
- Every export embeds its **recipe** (param JSON + seed + version) in a PNG `tEXt`
  chunk — drop the image back in and Muse restores the exact edit. Closure lives in
  the artifact.
- Deploy target: Cloudflare Pages → `muse.naklitechie.com`.

## Status

Scaffolded from handoff. Not built yet. Milestones: **M0 fidelity spike → M1 editor
(upload) → M2 camera (selfie)**.

## Handoff docs

The authoritative spec lives alongside this README until the product-facing README
lands at M1:

- [`MUSE-VISION-ROADMAP-001.md`](MUSE-VISION-ROADMAP-001.md) — what it is, the four
  looks, the 8-pass pipeline, roadmap.
- [`MUSE-AGENT-HANDOFF-001.md`](MUSE-AGENT-HANDOFF-001.md) — build/deploy, browser
  floor, UX, CSP, agent face, milestone gates & verifiers.
