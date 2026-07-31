# Muse — Vision & Roadmap (rev 001)

*Take pictures of yourself like you're a Davide Sorrenti muse.*

## 1. What it is

A single-file, browser-native 90s film-look photo tool. Drop a photo (v1.0) or open
the camera (v1.1), pick a look — Sorrenti, Corinne Day, Disposable, Mall Studio —
tune it with deep sliders, export. Every look is a parameter bundle over one
deterministic WebGL shader chain. Nothing ever leaves the device.

**Verb-able:** "muse it" / "musing a photo."

## 2. Audience shape

People who want the 90s editorial film look without a subscription filter app that
uploads their selfies to someone's S3 bucket. The tweet is the spec: *"need someone
to take pictures of me like I'm a davide sorrenti muse."* Who becomes themselves
using this: anyone styling their own photos with taste, offline, owning the output.

## 3. Doctrine posture

| Doctrine | Ruling |
|---|---|
| Sidecar | v1 is AI-free — deterministic core only. v2 sidecar: reference-match (AI proposes slider params from a reference image; editable, removable, no-AI state is v1 itself). |
| Edge-First | Dormant until v2 sidecar. When it lands: L2 WebGPU vision model → BYOK ladder, per doctrine. |
| Build | Default shape, no escalation. One HTML file, WebGL2, no build step, no server, no account. GA pageview-only per Grain precedent; image data never leaves device. |

## 4. The looks (v1 deck)

All four are presets over the same chain (§5). Signatures:

| Look | Curve/cast | Grain | Flash | Halation | Defocus | Trim |
|---|---|---|---|---|---|---|
| **Sorrenti** | Lifted blacks, muted sat, green-yellow midtones | Heavy, chunky | Hot center, hard falloff | Strong, red-shifted | Slight radial | 4:3, no stamp |
| **Corinne Day** | Grey cast, low contrast, cool daylight | Medium, fine | None | Minimal | None | 3:2 |
| **Disposable** | Warm, crushed shadows, oversaturated flash zone | Heavy | Harsh full-frame, deep vignette | Medium | Edge blur | 4:3, orange quartz date stamp |
| **Mall Studio** | Warm, glowy, lifted everything | Light | Soft omni | Bloom-heavy diffusion | Full soft-focus | 4:3, optional oval matte |

Custom presets: user-saved param JSON in localStorage; import/export as `.muse.json`
file (user's own transport, no server).

## 5. The pipeline

Single WebGL2 fragment-shader chain, fixed pass order, every pass parameterised
and bypassable:

```
input → 1 film curve + cast → 2 defocus → 3 chromatic aberration
      → 4 flash falloff/vignette → 5 halation (threshold→blur→screen)
      → 6 grain (luminance-weighted, seeded) → 7 light leak (optional)
      → 8 trim (crop, border, date stamp — Canvas2D compositor)
```

Grain and leak are **seeded PRNG** — same params + same seed + same image =
identical pixels. Determinism is a gate condition, not a nice-to-have.

## 6. Recipe = the artifact

Every export embeds its **recipe** — full param JSON + seed + version — as a PNG
`tEXt` chunk (or JPEG COM marker). The image stands alone; the recipe makes it
reproducible: drop an exported image back in, Muse reads the recipe and restores
the exact edit. Closure lives in the artifact; no model, no server, no session
required to replay.

## 7. Role matrix

Single-user local tool; the enumeration is short but done:

| Actor | Authority | Scope | Ownership | Trust boundary | Attribution |
|---|---|---|---|---|---|
| User | Everything | Own images, presets | Full | Inside (their device) | n/a |
| Agent (`window.muse`) | Apply looks, read looks, export blobs | Same as user | None (proposes blobs) | Inside; dev-gated, off by default | Recipe marks `via:"agent"` |
| System | None (no schedulers) | — | — | — | — |
| External party | The photo subject; no data implications — nothing transmitted | — | — | Outside | — |

No reseller/operator/inspector tiers — nothing to inspect, nothing hosted.

## 8. v1 milestones (one spec, one URL, staged deploys)

**M0 — Fidelity spike (riskiest assumption first).**
Full 8-pass chain on static test images. Riskiest unknown is not plumbing — it's
whether the chain actually *reads* as Sorrenti rather than "Instagram 2012."
Gate: (a) determinism verifier — hash-stable output across runs for fixed
params+seed; recipe round-trip test; (b) comparison strip artifact vs. reference
imagery, Chirag signs off on look quality. Machines gate correctness; human gates
shape.

**M1 — v1.0: Editor (upload).**
Drag-drop/upload, 4-look deck, deep slider panel (per-pass, grouped, bypass
toggles), custom presets, recipe embed/restore, export via FSA (fallback:
download), i18n scaffold (EN baseline), help modal, agent face, version string.

**M2 — v1.1: Camera (selfie mode).**
getUserMedia, live shader preview, front/back toggle, capture → lands in the M1
editor with look pre-applied. Perf floor: ≥24fps median on mid-tier mobile at
preview resolution (full-res applied on capture, not live). Camera denied →
degrade cleanly to upload; upload path is the no-camera first-class state.

## 9. Roadmap (inline, not separate docs)

- **v2.0 — Reference-match sidecar.** Drop a reference photo → AI proposes a param
  bundle. Editable artifact in the tool's own language (the recipe), staged never
  auto-committed, Edge-First ladder for inference. The canonical sidecar.
- **v2.x — Batch mode.** Apply one recipe to a folder (FSA directory handle).
  Agent face gets `applyLookBatch`.
- **v2.x — Pack format.** Community look packs as `.muse.json` bundles — file
  transport, no registry, no server.
- **Deliberately out:** video (different perf class — separate tool if ever),
  cloud sync, accounts, any upload path.
