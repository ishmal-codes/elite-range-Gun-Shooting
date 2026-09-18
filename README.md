# 🎯 Elite Range — Precision Shooting Simulator

**A AAA-quality, browser-based 3D first-person shooting range built with vanilla JavaScript and Three.js — zero build step, zero dependencies to install, one HTML file.**

Elite Range is a tactical marksmanship trainer, not an arcade shooter. Every mechanic — sight alignment, recoil, breathing sway, wind drift — is modeled to reward real trigger discipline over twitch aim, and every stage ends with a coached After-Action Report that reads your shot group and tells you exactly what your hands are doing wrong.

## 🖼️ Visual & Rendering Style

Elite Range targets a **realistic, AAA-tactical-shooter aesthetic** rather than a stylized or arcade look — closer in spirit to the weapon-handling fidelity of modern military shooters than to a typical browser game.

- **Physically-based rendering (PBR):** every material — steel slide, polymer frame, brass casing, concrete floor, gloved hands — is built with `MeshStandardMaterial`, driven by real roughness/metalness values rather than flat colors, so light behaves correctly across matte polymer, brushed steel, and specular brass.
- **Cinematic tone mapping:** ACES Filmic tone mapping and sRGB output encoding are applied at the renderer level, the same pipeline used in modern game engines, giving the scene a filmic, non-washed-out contrast curve instead of raw linear WebGL output.
- **Dynamic soft shadows:** PCF soft shadow mapping from a directional key light plus a cool-toned fill light replicates the mixed hard/soft lighting of an indoor range bay lit by overhead fixtures.
- **Procedural texturing:** concrete floor/wall noise, the B-27 paper target rings, distance markers, and the muzzle-flash sprite are all generated at runtime on `<canvas>` and streamed into the GPU as textures — there are no external image assets to download, which keeps the entire experience to a single file with near-instant load time.
- **Depth-of-field style fog & vignette:** exponential scene fog fades distant lanes realistically at range, and a screen-space vignette sharpens focus toward the crosshair, reinforcing the sense of a narrow, enclosed firing lane.
- **Reflection environment:** a generated PMREM environment map gives metallic surfaces (slide, barrel, steel gongs) believable ambient reflections instead of flat unlit metal.

The overall visual language leans into a **dark, cyberpunk-tactical control room** identity — near-black backgrounds, brushed-gold accent lighting, angular clipped-corner UI panels, and an Orbitron/Rajdhani type pairing chosen to read as precision military-tech rather than generic sci-fi.

---

## ✨ Features

### 🏛️ Elite Armory & Main Menu
- Cyberpunk-tactical dark-gold dashboard themed around an indoor shooting bay
- Full weapon selection screen with real-firearm attributes: magazine capacity, caliber, cyclic rate, recoil intensity, and mechanical spread
- Three authentic weapon profiles:
  | Weapon | Caliber | Magazine | Fire Rate |
  |---|---|---|---|
  | Glock 17 Gen5 | 9×19mm Parabellum | 17 rounds | 330 RPM |
  | SIG Sauer P320 XFive | 9×19mm Parabellum | 21 rounds | 360 RPM |
  | AR-15 Carbine 14.5" | 5.56×45mm NATO | 30 rounds | 700 RPM |

### 🖱️ True FPS Mechanics with Pointer Lock
- Native Pointer Lock API for seamless 360° mouse-look, matching the input model of PUBG, Valorant, and CS
- Automatic **cursor-aim fallback mode** if pointer lock is refused by the host environment (e.g. embedded iframes) — the game degrades gracefully to a cursor-driven aim scheme instead of breaking
- Fully rigged 3D weapons **constructed procedurally from Three.js primitives — no sprites, no billboards, no flat 2D stickers**:
  - A reciprocating slide assembly with individually modeled serrations, an ejection port, extractor, and takedown pin
  - Gloved hands built joint-by-joint — four independently curled fingers per hand wrapping the grip, a modeled thumb, wrist, and trailing sleeve — rather than a single static mesh
  - A glowing, emissive front-sight dot and rear sight notch, mathematically calibrated per-weapon so the sight picture lines up with the center-screen crosshair to sub-pixel accuracy

### 🎯 Aim-Down-Sights (ADS) System
- Right-click or Shift smoothly interpolates the weapon and camera from a hip-fire pose to a true sight picture using eased position/rotation blending — not a hard cut
- Field of view narrows per-weapon (pistols vs. rifle optics scale differently) and breathing sway is damped while aiming, replicating how a real shooter's hold stabilizes as they settle into the sights
- The front sight post's screen-space position has been numerically verified to land within a fraction of a pixel of true center when fully aimed, so the "iron sights align with the crosshair" claim isn't cosmetic — it's measured

### 🔫 Authentic Ballistics & Recoil Physics
- Every shot is a real `Raycaster` cast from the camera through the scene, factoring in mechanical dispersion (weapon-specific accuracy cone), muzzle rise, and live wind drift — **your point of aim genuinely determines the point of impact; nothing is randomly rolled behind the scenes**
- A critically-damped spring model drives recoil: each shot injects an impulse into pitch/yaw/roll/backward-travel springs that then decay naturally, so rapid, undisciplined fire visibly climbs and drifts the muzzle while a slow, settled press keeps consecutive rounds on target — recoil control is a real, learnable skill here, not a fixed animation
- Full multi-layered audio-visual feedback per shot:
  - Procedurally synthesized muzzle report, mechanical slide-cycling clack, and ejecting brass ring — **100% generated via the Web Audio API's oscillator/noise/filter graph at runtime — zero external audio files**
  - Additive-blended dynamic muzzle flash (sprite + geometry + point light) that decays over real time
  - GPU-drawn bullet tracers, impact spark particle bursts, and permanent bullet-hole decals that accumulate on paper targets
  - Physically animated steel: gongs swing on a damped pendulum with a metallic multi-partial "CLANG" (additive sine harmonics tuned per-target size), and knockdown plates topple on hit

### 📈 Progressive Level System
| Stage | Name | Challenge |
|---|---|---|
| 1 | Zero and Fundamentals | Stationary paper target, 10m, no time pressure |
| 2 | Marksman | 25m target with slow lateral drift |
| 3 | Steel Transitions | Three gongs at 15/25/35m — clean transitions |
| 4 | Movers | Swinging plates + paper, lead the movers |
| 5 | Wind Call | Long-range steel in a live crosswind |
| 6+ | Tactical Drills | **Procedurally generated forever** — compressed timers, stretching distances, heavier wind, shrinking targets |

### 📊 Live HUD & Shot Tracking
- Real-time side panel: active stage, score, live accuracy %, countdown timer, magazine/reserve count
- Wind speed and direction indicator on wind-affected stages
- Mini tactical shot-grouping diagram plotting your last 24 rounds in real time

### 📋 After-Action Report (AAR) — the signature feature
Every completed stage feeds its full shot log into a statistical analysis engine that generates a coaching dashboard, not just a scoreboard:
- Total shots fired, hits, misses, and final accuracy percentage
- Shot breakdown by precision band: Bullseyes / Good Hits / Needs-Work
- A computed letter grade (S / A+ / A / B / C / D / E) weighted by both accuracy and bullseye rate
- **Real statistical shot-group analysis**, computed in centimeters from actual raycast impact coordinates:
  - Mean point of impact (horizontal/vertical bias from the point of aim)
  - Extreme spread (maximum pairwise distance between any two hits — the standard real-world group-size metric)
  - Average shot-to-shot cadence, and the percentage of rounds fired with sights aligned (ADS usage rate)
- **Pattern-matched, mechanically specific coaching feedback.** The engine classifies your group's directional bias (low-left, high-right, etc.) against a library of known trigger-control faults and returns the diagnosis a real firearms instructor would give, for example:
  > *"Your group centres 6.8cm low and left of the X ring. Classic anticipation — you're tightening the whole hand and dipping the muzzle before the sear breaks. Dry-fire on a coin balanced on the front sight until it stops falling."*
- A rendered shot-group scatter plot (`<canvas>`-drawn, scaled to the group's actual spread) with a marked mean-impact point, so the feedback is visually verifiable, not just asserted
- One-click progression into the next stage, carrying forward your weapon and difficulty context

### 🎉 Celebration & Motivation System
- Animated bullseye badge and popup celebrations for center-mass hits
- Stage-clear fanfare with ascending audio cue
- Contextual encouragement when accuracy drops ("Settle in — front sight, slow press")

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| 3D Rendering | [Three.js](https://threejs.org/) r128 (WebGL) |
| Audio | Web Audio API (fully synthesized — no audio files) |
| UI / HUD | Vanilla HTML5 + CSS3 (no framework) |
| Input | Pointer Lock API + raw mouse/keyboard events |
| Fonts | Orbitron & Rajdhani (Google Fonts) |

**Zero build tools. Zero npm install. Zero backend.** The entire game — 3D engine, physics, audio synthesis, UI, and coaching AI — lives in a single self-contained `index.html` file.

---

## 🚀 Running Locally

No installation required — it's one file.

### Option 1 — Just open it
Double-click `index.html` and it runs in your browser.

### Option 2 — Local server (recommended, needed for full pointer-lock reliability)
```bash
# Using Python
python3 -m http.server 8000
# then open http://localhost:8000

# Or using VS Code
# Install the "Live Server" extension → right-click index.html → "Open with Live Server"
```

---

## 🎮 Controls

| Action | Input |
|---|---|
| Look around | Mouse |
| Fire | Left Click |
| Aim down sights | Right Click / Shift |
| Reload | R |
| Pause / release cursor | Esc |

---

## 📁 Project Structure

```
elite-range/
└── index.html   ← everything: HTML, CSS, JS, 3D engine, audio engine, coaching logic
```

---

## 🧠 Design Philosophy

Most browser shooting games reward speed and reflexes. Elite Range is built around the opposite premise: **your point of impact should teach you something about your grip, your trigger press, and your breathing** — the same way a real firearms instructor reads a paper target. The After-Action Report engine analyzes the geometry of your shot group (mean bias, extreme spread, cadence, sight usage) and maps recognizable patterns — low-left, high-right, excessive spread — to the specific mechanical fault that causes them.

## ⚙️ Engineering Highlights

A few implementation details that go beyond a typical browser game demo:

- **Single-file architecture with zero external assets.** No textures, models, or audio files are loaded — every texture is generated on `<canvas>` at boot, every 3D model is built from primitive geometry at runtime, and every sound is a synthesized oscillator/noise graph. The only network dependency is the Three.js library itself, loaded from a CDN.
- **Deterministic-feeling ballistics on a randomized core.** Gaussian-distributed dispersion (Box-Muller transform) is used instead of uniform randomness for mechanical spread, so shot groups cluster the way real ammunition does — dense near point-of-aim, sparse at the edges — rather than spreading evenly across a cone.
- **A fully procedural, infinite difficulty curve.** Stages 1–5 are hand-authored; from stage 6 onward, target count, distance, movement amplitude, wind strength, and time limits are all generated algorithmically from the stage index, so difficulty scales indefinitely without hand-authoring new content.
- **Physically-driven animation, not keyframes.** Weapon sway, recoil recovery, gong swing, and reload dip are all spring/damper simulations evaluated every frame — nothing is a baked animation clip, which is why recoil control genuinely responds to firing cadence.

---

## 📄 License

This project is licensed under the **MIT License** 


If you found this project interesting, a ⭐ on the repo is always appreciated.
