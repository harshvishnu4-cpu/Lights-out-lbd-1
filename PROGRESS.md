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
| `#screen-intro` | Title image + Play button (revealed once the preloader hits 100%) |
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
- **Voice-over (`.ogg` clips in `audio/`):** every bot line is spoken — tutorial
  intro (two clips: "These switches are
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
- **2026-08-03:** fixed overlapping audio on the last switch of a pattern. The
  celebration now runs as a strict chain (spoken number → chime *or* victory
  fanfare → current-flow bed → spoken win line) instead of firing everything off
  fixed timers from the same instant. `power_up.ogg` is 8.1 s long and used to
  drone under the whole finale — it is now held 1.25 s and faded out. The finale
  plays `sfxWin` *instead of* `sfxRowComplete` (both are the same ascending
  C-major run, so together they just smeared), and `sfxCorrect` no longer chirps
  over the spoken number. A generation counter cancels an in-flight chain if the
  dialogue is torn down mid-celebration.
- **2026-08-03:** fixed the confetti frame drop. Every particle was a path fill
  with `ctx.shadowBlur = 14`; ~280 blurred fills per frame over 1920×1080 pinned
  the burst at **75 ms/frame (13 fps)**. The glow is now baked once into 18 small
  offscreen sprites (shape × colour) and each particle is a single `drawImage`,
  so the burst runs at the measured ceiling with **zero frames over 40 ms**.
  Also: the canvas backing store is sized to the stage's actual on-screen size
  (clamped 960–1920) instead of always pushing 2.1 M pixels; particles that fall
  past the bottom retire immediately; survivors are compacted in place instead of
  reallocating; and the finale's second burst now joins the live one rather than
  cancelling it and allocating a fresh particle set. `handleAnswer`'s synchronous
  cost on a completed row went 78.5 ms → 11.2 ms.
  *Measured before/after in the real page over CDP (headless Chromium, software
  raster): frames >40 ms during the celebration 52/120 → 8/120, p90 75.7 → 25.4 ms.
  The residual slow frames are a headless rAF-throttling artifact — they appear
  identically on a fully idle page — not game work.*
- **2026-08-03 — sanity pass** (whole-file audit; verified by driving the real page
  over CDP with every `<audio>` element instrumented for play/ended, so the audio
  timeline could be read directly):
  - **Options were tappable before the instruction was spoken.** On level 1 the
    tiles went live ~1.2 s before the tutorial narration was even queued, so a
    quick player could answer over Byte's voice — or finish the whole row before it
    had explained the pattern. Levels 2-4 had the same gap. `_setOptionsLock(true)`
    now lands with the option reveal and the voice queue releases it. Level 1 is
    interactive after the tutorial finishes (~27 s in) rather than mid-sentence.
  - **"Well done! ALL the switches are fixed" could play on level 2 of 4** — it was
    in the random mid-level pool. Now reserved for the final level.
  - **`loadLevelInPlace` spoke over its own SFX**: the 2.2 s instruction started as
    the blast doors opened, under the 1.3 s deploy whoosh and the option pop. It now
    speaks after them, matching the tutorial's box → options → voice order.
  - **Two spoken numbers could stack** when tapping faster than a clip is long
    (~0.9 s); `playNumberVO` now cuts any number still speaking.
  - **A torn-down dialogue line could fire a second level transition.** A queued
    line's watchdog and its autoplay-blocked fallback timer are not cancellable from
    outside, so after `clearVoiceQueue` they could still run `onDone` → `advance()` →
    a second `playTransition`. `finish()` now checks a generation counter
    (`_voiceGen`, formerly `_celebrateSeq`, which already guarded the confetti chain).
  - **`_armBlobFallback` reverted a swapped image to the wrong file**: `#power-btn`
    swaps Red↔Green and `#panel-frame` panel↔panel-green at runtime, but the handler
    captured the path it was armed with — a failed *green* blob would restore *red*.
    It now resolves the path from the blob actually loaded.
  - **Dead code deleted:** the entire never-reachable `row-complete` phase (`phase`
    is never assigned that value) — its `renderQuestion` branches, the `#well-done-msg`
    overlay (`#robot-dance`, `#well-done-text`), and the CSS behind it
    (`greenBorderPulse`, `#main-panel.row-complete` ×3, `#screen-question.celebrating`,
    `.well-done-active`, `wellDonePop`, `robotDance`, `.sw.complete-fill`); the unused
    `playSwitchAppear` + `LEVEL_SWITCH_SOUNDS`; `#main-panel.vent-wrong` (only ever
    removed, never added); `#options-area.invisible`; the unstyled `.next-slot` hook;
    and the dead non-tutorial branch in `handleLevelStart`.
  - **Audio verified:** all 18 `.ogg` files decode and play from their blob: URLs
    (durations 0.61 s–8.10 s, zero failures), all 13 Web Audio SFX run without
    throwing, and a full 4-level playthrough — including a wrong-answer tap — reaches
    the end video with zero JS errors. The only clips now overlapping are
    `electricity` + `power_up`, which are one effect in two layers by design.
- **2026-08-05:** the confetti burst got a sound — `sfxConfettiPop` (a party-popper
  crack: triangle body snapping 940→140 Hz + a high-passed noise snap) and
  `sfxConfettiShower` (noise through a bandpass sweeping 7 kHz→1.3 kHz, plus 7
  scattered glints). Web Audio like every other SFX here, so no new asset.
  **Deliberately split in two:** the crack fires with the visual burst for punch, but
  the shower is broadband noise — exactly what masks speech — so it waits ~1 s and
  joins the row-complete chime once the tapped number has finished. Levels were set by
  re-rendering each SFX through the real master chain in an `OfflineAudioContext`: the
  first attempt peaked at 0.124, quieter than a UI click. Now pop ≈ 0.509 peak (a
  transient — between the chime's 0.332 and `sfxWin`'s 0.862) and shower ≈ 0.286,
  deliberately just under the chime so the melody stays the focus. Zero clipped
  samples for chime+shower (0.492), the finale's win+pop+shower (0.863), and even a
  synthetic five-sound stack (0.917).
- **2026-08-05:** two tutorial-feel fixes.
  - **The tutorial no longer accepts wrong taps.** On level 1 only the glowing correct
    switch responds; the other three render as `.sw.option-inert` (no pointer, no hover
    lift) with no click handler, so a first-time player is walked through the pattern
    instead of being able to get it wrong while it is still being explained. Verified
    by real DOM clicks on all three wrong options — `grid` and `phase` both unchanged —
    and the correct one still works.
  - **"Tap the correct switch." no longer re-types itself every level.** The bar
    already shows it (set at the top of `loadLevelInPlace`), so it was being typed out
    a second time when the line was spoken. `queueBotText` takes an `instant` flag that
    shows the line whole via `_setBotText`; the tutorial's own lines still type. Proven
    with an ordered log of every change to `#bot-text`: exactly one change after the
    tutorial's last line, straight to the full string.
