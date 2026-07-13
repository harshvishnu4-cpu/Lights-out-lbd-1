# Lights Out — Project Progress

A single-screen, browser-based educational mini-game where the player completes a
repeating number **pattern** by tapping the correct switch. Sci‑fi control-panel
theme (robot guide, glowing pipes, power button, energy current).

- **Status:** Playable end-to-end (4 levels → completion screen). Visuals match the
  Figma design (`node 956-7170`) and success screen (`node 975-16315`).
- **Stack:** Single file — `index.html` (HTML + CSS + vanilla JS). No build step, no
  dependencies. Fonts (Exo 2, Bebas Neue, Lilita One) load from Google Fonts.
- **Last updated:** 2026-06-25

---

## How to run
Open `index.html` in any modern browser. (For audio to start, the player taps
**Start** — this unlocks the Web Audio context.)

---

## Layout & rendering
- Fixed **1920×1080** design canvas (`#canvas`), positioned top-left and centered
  with a JS `translate() + scale()` so it fits **any display size** (letterboxed,
  aspect preserved). Rescales on resize / orientation change / mobile address-bar
  (`visualViewport`).
- **Responsive extras:** touch handling (`touch-action`, no tap highlight,
  no user-zoom) and a **rotate-to-landscape hint** overlay for narrow portrait phones.
- All element sizes/positions match the Figma frame exactly (panel `1573.9×453`,
  options box `742×256`, text bar `1499w`, switch tiles `128×174`, option tiles
  `112.2×152.5`, power button `183×178`, etc.).

## Screens
| Screen | Purpose |
|---|---|
| `#screen-intro` | Title image + Play button (becomes **Play Again** after a full playthrough) |
| `#screen-question` | Main gameplay |
| `#screen-complete` | End video; when it finishes, redirects back to the title screen |
| `#transition` | Sci‑fi **blast-door** transition between levels |

---

## Game mechanics
- **Levels** (`ROWS_CONFIG`): 4 levels, each an 8-slot row of a 2-number pattern.
  | Level | Pattern | Options |
  |---|---|---|
  | 1 | 3, 5 | 3 6 5 4 |
  | 2 | 4, 7 | 4 5 8 7 |
  | 3 | 7, 10 | 10 9 11 7 |
  | 4 | 11, 13 | 13 12 11 10 (uses a **scaffold** — only some slots are player-filled) |
- `TOTAL = 8` slots; `PREFILLED = 5` shown at start (levels 1–3). Player taps the
  correct option to fill the remaining slots in order.
- **Scoring:** 15 (first try) / 10 (after a wrong try) / 5 (after a hint).
- **Wrong answer:** feedback shows **only on the tapped option** (gentle shake +
  **amber** glow pulse + gold spark burst) — nothing on the switch row / vent
  circuit, and **no spoken instruction** repeats. Recovers quickly so the player
  can retry without waiting. After **2** wrongs, the correct option glows as a hint.
- **Correct, completed pattern:** success state (see below) → "Well done!" →
  blast-door transition to the next level (or the completion screen after level 4).

---

## Implemented features

### Entrance (per level, staggered "one by one")
robot bounces in → text bar opens → **main box powers on & opens** → blue pipe +
red power button appear together → switches **cascade/drop in** one by one → bot
gives the instruction ("Tap the correct switch.") → options box deploys out of the
panel → option tiles pop in. Each beat has its own sound.
- Switch cascade timing: `SWITCH_OFFSET_MS = 3850`, `SWITCH_STEP_MS = 340`.

### Switches & vent
- Tiles render as a metal rocker (cropped from `tile.png`) + a **Lilita One** number.
- Empty slots: cyan neon border (no glow). The **correct option** glows instead, to
  guide the player.
- **Vent bars** (top of panel) recolor by state: cyan (idle), green (correct).
  Wrong answers are shown on the tapped option only, not the vent/main panel.

### Power button & current (success feedback)
- Power button: **red** (off) → **green** (on) + pulse when the pattern is complete.
- On completion the panel border turns **green** (Figma success screen), the **vent
  bars flow green** (left→right toward the pipe), then a **green current** sweeps
  along the pipes — accompanied by an electric **surge SFX**. Held ~2.4s.

### Celebration & transition
- "Well done!" with a dancing robot, then a **blast-door** power-down/up transition
  swaps in the next screen.

### Dialogue
- Bot lines **typewriter** out with soft talk blips; options lock while the bot
  "speaks". Instruction is spoken right when the switches finish appearing.

### Tutorial guide (Level 1 only)
- The **correct option glows** (pulsing cyan outline + halo) to show which switch to
  tap. The glow follows the correct option across the tutorial's taps. (Levels 2+
  only glow the correct option as a hint after two consecutive wrong attempts.)

### Audio (SFX synthesized via Web Audio API; master gain + compressor)
- No background music (SFX only).
- **SFX:** hover, click, correct, **wrong (gentle "oops", not alarming)**,
  row-complete, entrance pops, panel power-on, options deploy whoosh, power-down/up
  (transition), bot-bar open/close, switch-appear, talk blips, and a
  **happy victory fanfare** on game completion (plays with the confetti).
- **Current-flow audio (files):** on a completed pattern, `audio/electricity.mp3`
  (zap) then `audio/energy.mp3` (sustained energy) play as the current sweeps the pipes.
- Optional number-voice `.ogg` files degrade gracefully if missing.

### Inactivity nudge
- If the player is idle ~10s during their turn (no tap/move/key), the option tiles
  do a friendly **bounce wave** + a soft blip to draw them back. Repeats while idle;
  any activity resets the timer. Only fires when the options are tappable.

---

## Assets
All raster art is **WebP** (converted from PNG/JPG — ~9.9 MB → ~0.35 MB total);
vector frames stay **SVG**. Unused legacy assets have been removed.
- `assets/` — `background.webp` (sci-fi panel-wall game background, 1920×1080).
- `assets/figma/` — `robot.webp`, `connector.webp` (pipes), `tile.webp`,
  `panel.svg`, `panel-green.svg` (success), `options-box.svg`, `options-line.svg`,
  `textbox.svg`.
- `assets/` — `Red button.webp`, `Green button.webp`, `robot dance.webp`,
  `title screen.webp` (3D title art), `play button.svg` (baked **Play** button shown
  on first load) and `button-plate.svg` (blank octagon used for the **Play Again**
  state, with an HTML text label over it after a full playthrough).
- `audio/` — number/voice `.ogg` clips are referenced by name but **not currently
  present**; the game runs fine without them (synth SFX cover everything).

---

## Pending / ideas
- [ ] Add the number-voice `.ogg` files (or remove the references) for spoken numbers.
- [ ] On-screen score/streak display (score is tracked but not shown).
- [ ] Persist player progress / best score (e.g. `localStorage`).
- [ ] More levels / difficulty curve; optional timer.
- [ ] Audit/remove unused legacy assets in `assets/`.

---

## Recent changelog
- Implemented full Figma layout at native 1920×1080 with exact element sizes.
- Made the game responsive across all display sizes (+ portrait rotate hint).
- Added playful staggered entrance + per-beat SFX; typewriter dialogue.
- Power button: lightning-panel iteration → reverted to the **red/green** button per
  the latest Figma.
- Success: green panel border, **green** vent flow + pipe current (confined to the
  exact pipe shape), held longer and sped up for visibility.
- Added the **current-flow electric surge SFX** as an attention grabber.
