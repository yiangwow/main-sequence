# CLAUDE.md — Main Sequence

Incremental/idle game where the player IS a star: accrete hydrogen, ignite real fusion chains, accumulate an inert iron core, go supernova (prestige), seed the next star with stardust.

**Live artifact:** https://claude.ai/code/artifact/2f5016d6-47f8-432c-bf44-8db95533c08b
(Update flow: edit `main-sequence.html`, then republish via the Artifact tool passing this URL as `url` from any session.)

## Architecture

Single self-contained file: `main-sequence.html` (no build, no deps, CSP-safe — no external requests).

- **Rendering:** full-window `<canvas id="sky">` (starfield + evolving star + particles + supernova shockwave, rAF loop) behind a fixed right-side DOM console (400px; column layout under 720px).
- **Logic tick:** `produce(dt)` every 100ms; UI `refresh()` every 250ms; autosave every 5s.
- **Save:** localStorage key `main-sequence-save-v1`, offline progress capped at 2h (single-pass `produce(away)` — fusion chain cascades only once per load, known approximation).

## Game systems (all constants near top of the script)

- Elements `EL`/`ORDER`: H→He→C→O→Si→Fe, each with real emission wavelength (spectrum strip UI).
- `GENS`: 6 hydrogen generators, cost growth 1.15.
- `PROCS`: 5 fusion converters (real process names: pp-chain, triple-alpha, alpha ladder, oxygen/silicon burning), growth 1.18, `BURN=8` (8:1 in→out). Iron produces nothing — it fills the core.
- `CHANDRA=1500` Fe → supernova enabled. Stardust gain = `floor(6*sqrt(Fe/1500))`.
- Stardust: +8% global each, plus shop `DUSTS` (Metallicity leveled ×2 cost, Population II, CNO Cycle, Degenerate Matter).
- In-run upgrades `UPGS`; stage visuals `STAGES` keyed off highest fusion process owned (`stageIdx()`).
- **Remnants (v2):** each supernova leaves a permanent remnant — the fork is *when you collapse*: Fe < `TOV` (3600) → neutron star, ≥ TOV → black hole. Pulsars grant earned automation (1st: auto-buy generators, 2nd: fusion, 3rd: upgrades; extras speed the pulse; buys only above 2× cost so they never starve the player). Black holes: ×1.3 global each + 2h offline cap each. v1 saves migrate: past `novas` become retroactive remnants.
- **Persistent nebula sky:** every remnant paints a seeded nebula (`drawNebula`, mulberry32 PRNG) into the starfield offscreen canvas. Luminance budget: alpha decays 0.6^age while radius grows — verified clean at 20 remnants. Pulsars/black holes render as live objects in the rAF loop.
- Balance caveat: fusion *output* scales with global multiplier while *consumption* stays fixed — late prestiges accelerate hard. Untuned by real playtesting yet.

## Design language (do not casually break)

Observatory-instrument aesthetic: deep indigo-black ground, per-element spectral colors, monospace data + Georgia-italic log lines, uppercase tracked labels. Signature elements: the living star canvas and the emission-spectrum resource strip. Supernova screen carries the "born to be a star" line — intentional, personal to the user; keep it.

## Verification

No Linux browser here — use Windows Chrome headless from WSL:
```bash
cp main-sequence.html /mnt/c/Users/computer/AppData/Local/Temp/ms-test/
"/mnt/c/Program Files/Google/Chrome/Application/chrome.exe" --headless=new --disable-gpu \
  --window-size=1440,900 --virtual-time-budget=6000 \
  --screenshot='C:\Users\computer\AppData\Local\Temp\ms-test\shot.png' \
  'file:///C:/Users/computer/AppData/Local/Temp/ms-test/main-sequence.html'
```
Seed states by prepending a `<script>` that writes the save key before the game script; simulate clicks by appending one after it. Note: headless clamps window width to ~500px minimum (mobile shots crop, they don't reflow).
