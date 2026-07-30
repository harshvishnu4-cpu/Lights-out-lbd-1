# Lights Out — Project Progress

A single-screen, browser-based educational mini-game where the player completes a
repeating number **pattern** by tapping the correct switch. Sci‑fi control-panel
theme (robot guide, glowing pipes, power button, energy current).

- **Status:** Playable end-to-end (4 levels → completion screen). Visuals match the
  Figma design (`node 956-7170`) and success screen (`node 975-16315`).
- **Stack:** Single file — `index.html` (HTML + CSS + vanilla JS). No build step, no
  dependencies. Fonts (Exo 2 400/700/900, Lilita One) load from Google Fonts.
- **Media formats:** WebM (VP9+Opus) video, Ogg (Opus 64–96k) audio, WebP images
  (incl. animated) — supported by Chrome, Edge, Firefox and recent Safari.
- **Last updated:** 2026-07-27

---

## How to run
Serve the folder over HTTP (any static server) **or** open `index.html` directly —
on `file://` the streaming preloader detects that fetch is blocked and falls back
to direct asset URLs automatically. A themed loading bar fills where the Play
button will appear; the button pops in at 100%. (Tapping it also unlocks the Web
Audio context.)

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
| `#screen-intro` | Title image + Play button (title VO plays when the button reveals) |
| `#screen-question` | Main gameplay |
| `#screen-complete` | End video — **the game ends here**, holding on the last frame (no redirect) |
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
- **Pattern-complete audio (files):** on a completed pattern, `audio/electricity.ogg`
  (zap) then `audio/power_up.ogg` (power-up) play as the current sweeps the pipes;
  both are cut at the next transition so they don't bleed over.
- **Voice-over (`.ogg` clips in `audio/`):** every bot line is spoken — **title VO**
  (`title.ogg`, plays when the Play button reveals;
  if autoplay is blocked it retries on the first non-Play tap — NOTE:
  currently a Windows-TTS placeholder saying "Lights Out!", replace the file with
  the real recording, same name), tutorial intro (two clips: "These switches are
  in a pattern." → "Let us read the pattern together."), tutorial instruction,
  "Tap the correct switch" (also plays as each new level's blast doors open),
  level-1 win pair, and the three random win lines for levels 2–4. Number voices
  (3, 4, 5, 7, 10, 11, 13) play as switches cascade in, on correct taps, and
  during the level-1 pattern read-along. The typewriter (**90 ms/char**,
  `TYPE_MS`) mutes its talk blips while a real VO clip speaks; all clips degrade
  gracefully if missing and are stopped by `clearVoiceQueue()` on transitions.

### Inactivity nudge (Levels 2+ only)
- If the player is idle **~10s** during their turn (no tap/click/key), the option
  tiles do a friendly **looping bounce wave** for **~5.5s** + a soft blip to draw
  them back, then rest. Repeats every 10s while idle; any real interaction resets the
  timer (mouse-move alone does not). Only fires when the options are tappable, and
  **not during the Level 1 tutorial** (the correct-option glow already guides there).

---

## Assets
All raster art is **WebP** (converted from PNG/JPG — ~9.9 MB → ~0.35 MB total);
vector frames stay **SVG**. Unused legacy assets have been removed.
- `assets/` — `background.webp` (sci-fi panel-wall game background, 1920×1080).
- `assets/figma/` — `robot.webp`, `connector.webp` (pipes), `tile.webp`,
  `panel.svg`, `panel-green.svg` (success), `options-box.svg`, `options-line.svg`,
  `textbox.svg`.
- `assets/` — `Red button.webp`, `Green button.webp`, `robot dance.webp`
  (animated WebP, re-encoded with `gif2webp` for correct frame disposal —
  ffmpeg's converter left ghost frames), `title screen.webp` (3D title art) and
  `play button.svg` (the **Play** button, used for every playthrough).
  `loader.gif` is present but **not wired** into the game yet.
- `audio/` — number `.ogg` clips (3, 4, 5, 7, 10, 11, 13), bot voice-over `.ogg`
  lines (tutorial, instructions, win lines), and `electricity.ogg` +
  `power_up.ogg` for the pattern-complete current sweep. All Ogg **Opus** (64–96k),
  all present and wired up.
- `_dev/` — quarantined, **never deployed** (see `.gitignore` / `.deployignore`):
  dev scratch files and the pre-conversion originals (`end-video.mp4`,
  `robot dance.gif`, `*.mp3`, Vorbis voice clips).

---

## Performance & delivery (2026-07-27 optimization pass)
- **Media formats:** `end-video` → WebM (VP9 + Opus, constrained quality: CRF 33 +
  bitrate cap at 75% of the H.264 source), all audio → Ogg Opus (voice 64k,
  SFX/music 96k), `robot dance.gif` → animated WebP (q80). Every converted file
  verified smaller than its source; originals quarantined in `_dev/originals/`.
- **Boot preloader:** streams all 32 assets via `fetch` + ReadableStream for
  byte-accurate progress, weighted by a real on-disk size table (regenerate
  `ASSET_SIZES` in index.html if assets change), refined by Content-Length,
  smallest-first queue, concurrency 5, 60s per-transfer abort timeout. Fetched
  files are swapped onto their elements as typed blob: URLs; every element has a
  one-time `error` fallback back to its original URL. Failures/stalls/file://
  count as done — loading can never block. The Play button is hidden (and
  `handleStart` gated) until 100%, then pops in where the loading bar was.
- **Stuck-proof media gates:** the end video and every spoken line advance via
  three paths — the media's own `ended` event, an `error` handler, and a
  duration-based watchdog timer — so no screen can wait forever on media.
- **GPU compositing:** no 3D transforms/backface anywhere (audited); screens swap
  via `display:none` so hidden screens hold no layers; celebration-occluded chrome
  now gets delayed `visibility:hidden` after its fade; the full-screen confetti
  canvas is `visibility:hidden` while idle.
- **Removed dead weight:** unused level-intro screen, `.btn-start` block, end-panel
  CSS, `#wrong-msg` + unreachable `phase==='wrong'` branches, `sfxBarClose`,
  `sfxCurrentFlow`, Bebas Neue font + Exo 2 weights 600/800; inline favicon added
  (no more 404). Dev scratch files (`_a_crop.png`, `_arrow.html`) quarantined.
- **Payload:** 3.30 MB → 2.15 MB (−35%): video 1.37→0.89 MB, images (incl. the
  old GIF) 1.12→0.70 MB, audio 0.70→0.46 MB.
- **Verified with Playwright** (headless Chromium over local HTTP): loading bar
  monotonic and observable under a throttled network, button gated until 100%,
  full 4-level playthrough, end video plays, every `<img>` healthy, zero JS
  errors, zero 4xx/5xx, and blocked-asset failure modes still boot and play.

---

## Pending / ideas
- [x] Add the number-voice `.ogg` files for spoken numbers. *(done — plus full bot VO)*
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
- **2026-07-27:** end-to-end optimization pass — WebM/Opus/animated-WebP media,
  streaming byte-accurate preloader with gated Play button, media watchdogs,
  GPU layer hygiene, dead code removal, junk quarantine (see section above).
- **2026-07-29:** typewriter slowed to 90 ms/char; title VO wired (placeholder
  clip — replace `audio/title.ogg` with the real recording); fixed robot-dance
  ghosting (gif2webp re-encode, 510 KB, still 27% under the source GIF); removed
  the **Play Again** button state (plain Play button returns after the end
  video); tutorial intro split into its two new VO clips; new/restored Vorbis
  clips re-encoded to Opus.
