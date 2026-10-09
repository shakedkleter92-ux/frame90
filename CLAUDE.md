# 90'FRAME (formerly Film Camera)

Single-file browser PWA: a camera app that shoots through film-stock / vintage-camera looks —
live from the camera, or applied to an uploaded photo. No build step, no dependencies, no
framework. Built on the exact app shell of `../grid_symbol_maker` (Pixelart Maker): same
splash → install prompt → Live flow, same panel/FAB/gallery/zoom-dial UI and conventions —
only the content changed (pixel grid + palettes → WebGL film engine + film presets).

## Running locally

```bash
python3 -m http.server 8765     # then open http://localhost:8765
```

Camera needs `localhost` or HTTPS. A wide desktop window redirects to `frame.html` (a
phone-shaped iframe shell), same as the original app — the app always renders its mobile layout.
Bump `CACHE` in `sw.js` (`90frame-vN`) **and** `__BUILD` / `__SW_URL`'s `?v=` near INIT in
`index.html` together on every deploy; `?__sw_reset=1` force-clears the service worker.

## Map of index.html

- **CAMERAS · LENSES** — a **1990s** camera library (user, 2026-10-08: "we're in the 90s now,
  not the 80s"), built from the user's technical spec (pasted 2026-10-08; its rules: no generic
  filter, film grain ≠ sensor noise ≠ tape artifacts, no VHS artifacts on digital, no forced
  timestamps/leaks/scratches/blur, defaults that keep quality, everything adjustable and
  non-destructive, restrained documented approximations). Each preset is assembled by
  `CAMERA(key, label, name, desc, parts)` from independent parts: `BODIES` (optics + `flash`),
  and exactly one of `FILMS` (organic grain, color `matrix`, highlight `shoulder`; scanned at
  `FILM_SCAN` = Kodak Picture CD 1536×1024, 3:2), `SENSORS` (native `px`, matrix, hard clip
  `white`/`black`, sensor noise, `sharpen`, `jpegQ`) or `VIDEO_FORMATS` (tape bandwidth,
  chroma, `interlace`, analog noise, optional `tapeFx`, `kbps`, `audio`). `AUDIO_PROFILES`
  A/B/C(/C32) drive `buildRecordAudio()` (Web Audio chain on the mic for video only; film
  presets keep the original sound). **Two lists (user, 2026-10-09)** — `CAMERA_LISTS`: **Photo** =
  old digital (DC20, QV-10, Cyber-shot DSC-F1, QuickCam, QuickTake) then film stills (FunSaver,
  Polaroid, Holga X-Pro); **Video** = Super 8, Video8, Hi8, MiniDV, the user's VHS references
  (`vhs92`, `vhsslp`, `vhsworn`), NightShot. The picker shows the current tab's list
  (`activeCameraList()`: Live → capture mode; Upload → a loaded video = Video, else Photo);
  each list remembers its own pick (`ACTIVE_IN_LIST`, synced by `syncCameraList()` on tab
  change / upload load / clear). QuickCam uses look param `levels` (16 grays, shader 5a). Added 2026-10-08: `polaroid` (600 instant film, 1:1, flash on, no
  forced white border), `holga` (cross-processed slide film, 1:1, plastic-lens corners),
  `super8` (Kodachrome 40 cine film, 18 fps frame hold via `fps`, film grain, gate `weave` +
  `flicker` look params, audio `S8`). A film may set its own `px`/`formats` (instant and
  120 aren't Picture CD scans of 35mm); `FORMAT_RATIO` has `1:1`. `vhsworn` (VHS
  Worn, 2026-10-08, from a sunset-over-rails clip) = hot saturated warm tape, long red bleed and
  constant oxide dropouts (look param `dropouts`, shader stage 5b — part of that preset's look,
  separate from the optional `tapeFx` layer); no OSD. The user's "CAMERA1 / PLAY / SOURCE IPHONE"
  references are the same filter as `vhs92` — not added again. **Removed by the user
  as too sharp (2026-10-08): Mavica FD5, DC290, Gold 200, Superia 400**. **Rule from the
  user: nothing in the app may look sharp — a preset that renders crisp doesn't belong here**
  (Hi8/Video8/MiniDV were softened for this: Video8 320×240, Hi8 400×300, MiniDV soft 1.2;
  Video8 stays the softest, Hi8 between).  — and the 2000s models
  before that. One picker, titled **CAMERA TYPE** (no number), showing the current tab's list — the
  panel has nothing else. **The 02 Adjust tab was removed on the user's request (2026-10-08)**: no
  sliders/toggles in the UI; `CONTROLS` / `PRESET_TOGGLES` remain only as fixed defaults that
  `combineLook()` reads (effect 100, grain 100, color 100, WB auto, tape FX off, camera sound;
  flash/date = each preset's own default: FunSaver flash on, VHS OSD on). Hold the picture =
  the original (`state.comparing`), a gesture with no UI.
  **Authenticity**: renders at each format's native pixels, the preview itself goes through a
  real JPEG at the camera's quality (`pumpLiveJpeg()`), stills are saved as that JPEG
  (`cameraJpeg()`, enlarged after the JPEG pass if < 960px), video records the small frame at
  `kbps`. Removed as fake: rolling tracking band (now only an optional intermittent
  `tapeFx`), dark scanlines, CCD smear, light leaks, Game Boy. **Formats per camera**
  (`formats`, cycled by the top-center pill `#mob-ratio`): film/DC290 3:2, Mavica/tape 4:3,
  MiniDV 4:3/16:9 **anamorphic** 720×480 (`cameraDims()` = stored pixels, `cameraAspect()` =
  displayed shape; stills resampled by `squarePixels()`). **Flash** has no depth data:
  distance is approximated by frame position, ambient drops, the room's color cast is taken
  out of the flash-lit part, dark things stay dark, shadows only on the far side of edges.
  **Lens = field of view** (`LENSES` `focal`; phone main = 26mm, ultra-wide 13mm; crop by
  focal/feed, hardware zoom first). No fake depth-of-field. Fisheyes `globe` (whole frame in
  the circle) and `fish` (bulging center) are real optics. Default lens 28mm. Picking a
  camera never changes the lens. **Zoom stays** (presets row + pinch dial, never remove):
  effective focal = lens × zoom. To add a camera, add parts + one `CAMERA(...)` call.
- **FILM ENGINE (WebGL)** — `film.render(source, {outW, outH, native, aspect, crop, mirror,
  settings, time, still})` draws one frame into `film.canvas` with one fragment shader
  (`FILM_FRAG`), in separate stages: resolution/optics → flash → color (matrix, WB) → tone
  (film shoulder / digital clip) → noise (type 0 film grain, 1 sensor, 2 analog; temporally
  coherent — cross-faded at 24 fps, never re-randomized per render) → optional tape FX.
  `native` = the camera frame size the render stands for (pixel-scale effects and noise are
  measured in its pixels). Two source textures (current + previous frame) for interlacing;
  `still: true` (uploads) means no previous frame. Crop/zoom/mirror is a UV transform.
  Falls back to a rough Canvas2D `ctx.filter` version if WebGL is unavailable.
- **renderView() / renderUpload() / refreshView()** — `renderView` paints the on-screen
  canvas (Live: the native-size frame + OSD scaled up into its frame rect; Upload: the
  processed photo, contain-fit). `renderUpload` renders the photo cropped to the camera's
  frame shape (landscape/upright following the photo) and the lens. Call `refreshView()`
  after any settings change.
- **Live framing = the camera's own frame**, not full screen (user, 2026-10-08: "the picture
  being full screen is weird"; this replaces the earlier full-screen 9:16 canvas). Tape/digital
  cameras are 4:3, shown upright as 3:4 (Polaroid/Holga are square) (`getLiveTargetRatioValue()` from the camera's
  `px`/`vpx`), contain-fit between the top buttons and the bottom controls
  (`getLiveFrameRect()`, `LIVE_FRAME_TOP/BOTTOM`) on a dark surround. The back camera is
  requested at 3:4 so the stream is the whole sensor. First launch opens at 1× zoom (the old
  0.5× default only existed to undo the 9:16 crop of the sensor). An earlier *landscape*
  4:3 contain-fit frame was rejected as "wide" — keep it upright.
- **UPLOAD VIDEO** (user, 2026-10-08) — Upload accepts photos *and* videos. A video plays
  muted on a loop (`state.srcVideo`); each new frame is copied into the `state.srcImage`
  canvas (≤1280px), so every photo path (crop, lens, compare, camera switch) works unchanged,
  rendered at the camera's *video* size (`uploadFrame()`). The save FAB re-shoots the whole
  clip in real time like Live recording (`exportUploadVideo()`: second `<video>` with sound →
  native frame + OSD → enlarged → MediaRecorder at `kbps`; sound via `buildAudioGraph()` with
  passthrough for film presets); progress in the centered `#export-progress` panel (REC, big %, bar, "please wait" — user asked
  for it mid-screen so it's obvious), tap save again to cancel.
  **iPhone (2026-10-09, user couldn't upload a video from the phone)**: iOS Safari loads nothing
  until `play()`, Low Power Mode refuses even muted autoplay, and a detached `<video>` may not
  render — so `loadVideoFile()` puts the video in the page (hidden), calls `play()` at once,
  readies on any of loadeddata/canplay/seeked/playing, forces a frame with a tiny seek, retries
  the draw while paused (WebKit reports a frame before it's drawable), renders a paused frame as
  a still (no interlace with an empty previous frame), shows `#video-tap-hint` (TAP TO PLAY)
  when autoplay is refused, and gives a message after `UPLOAD_VIDEO_TIMEOUT`. Tested in
  Playwright WebKit with iPhone emulation, incl. a refused-autoplay run — not on a real iPhone.
- **LIVE CAMERA** — unchanged camera plumbing from the original (facing defaults, ultra-wide
  0.5×, zoom dial, Android rotation fix). `liveLoop()` renders the preview at the camera's
  native size (video size in Video mode, still size in Photo; big still sensors capped by
  `LIVE_PREVIEW_MAX`); `liveCoverCanvas` *is* `film.canvas`. The shutter
  (`captureLivePhoto()`) re-renders the current frame at the full native still size.
  Video recording blits the native frame + OSD (`liveComposite`), enlarged by a whole factor,
  each tick (`drawRecordFrame()`) — never a second shader pass per tick (that's what made the
  original's video choppy).
- **SHUTTER SOUNDS** — `shutterSound`, synthesized with Web Audio (no audio files): noise
  bursts, body thumps, motor whirs, spring twangs and ratchets combined into one `PROFILES`
  entry per mechanism (SLR, Leica, compact, disposable, toy, Polaroid eject, Hasselblad…),
  mapped from camera keys in `CAMERA_SOUND` (unlisted → SLR) and loudness-matched by
  `PROFILE_GAIN` (measured from offline renders; Leica deliberately quietest). Played only for
  Live photo captures. Each preset names its sound (`sound`: `mavica`, `digicam`, `compact`, `disposable`,
  `camcorder`, `earlydigi`; unlisted → `digicam`); the film-camera profiles are
  still there, unused. `shutterSound.unlock()` runs on the splash tap — iOS only lets an
  AudioContext start inside a user gesture. On iPhone the ringer/silent switch can mute it.
  **Volume-UP = shutter** (keydown `AudioVolumeUp`/`VolumeUp`/175/24, Live only, key-repeat
  ignored; Down is left alone). Works only where a browser forwards volume keys to pages (some
  Android); **never on iOS** — WebKit doesn't expose them to web content at all, so that would
  need a native wrapper.
- **SAVING FILES** — `isMobile` is by device, not width (the desktop iframe is narrow and its
  share sheet can't open there); shared files use `baseMime()` (no `;codecs=`, iOS refuses it);
  `frame.html`'s iframe allows `web-share`.
- **GALLERY** — photos stored as JPEG. The gallery FAB always shows a still preview of the
  latest capture (never a generic icon once something exists; an empty film-frame square before
  that) and "pops" when a new shot lands. Videos get a still `poster` (JPEG data URL grabbed
  from the last recorded frame in `toggleVideoRecording()`) used by the FAB and the grid —
  a `<video>` element as a thumbnail shows black on iOS until played.

## Conventions

Same as `../grid_symbol_maker/CLAUDE.md` (one file; only the mobile layout path is reachable;
iOS quirks are deliberate) — **except the visual style, which the user replaced on purpose**.
Current style (2026-10-08) = **PHOSPHOR HUD**: the interface blends into the picture like an
old camera's on-screen display (user reference: a green-phosphor radar terminal — thin glowing
green lines and text on black). Black surround (`--void`), phosphor green `--hud` with dimmer
`--hud-mid`/`--hud-dim`, red `--rec` only for recording; thin 1.5px outlines, no filled blocks,
no bevels, no rounding, no blur; text = VT323 with a phosphor glow plus a hairline dark halo so
it reads over any image; selected = inverted (solid green, black text); viewfinder corner
brackets + center cross drawn around the live frame (`drawViewfinderMarks()`, screen only).
The theme is the "PHOSPHOR HUD THEME" block at the end of the `<style>` + the `:root` vars —
restyle there; old variable names (`--cream`, `--plum`, `--lcd`, `--ink`…) are remapped onto it.
The shutter is the same 44px square as the Upload key (the 64px one was out of proportion);
mode labels (VIDEO / PHOTO / UPLOAD) are small, 15px. **Typing animation only on the splash** (`typewrite()`; user: typing everywhere was too much).
Splash = BIOS-style boot lines typed out → logo → "90'FRAME" in DSEG14 (LED segments,
bundled in `fonts/`, OFL) → blinking "TAP TO START". **Rejected, don't bring back**: the
opaque olive-LCD + Winamp beveled-metal look ("too much"), and before it the cream/plum
VHS-sleeve theme (80s; the app is 90s now).
**Logo** (`icon.svg`) = a 16×16 pixel-art camera in glowing phosphor green on black inside
viewfinder corner brackets. `icon-180.png` (apple-touch — iOS ignores SVG touch icons) and
`icon-512.png` are full-bleed PNG renders of it; regenerate both if the SVG changes (render the
SVG in headless Chrome). When the logo changes, bump the `?v=` on every icon reference together (`index.html`
icon/apple-touch-icon/manifest links, `frame.html`, the `icons` in `manifest.json`) — iOS and
Android cache home-screen icons hard. The 512 icon is `purpose: any`, not maskable (a circle
mask would clip the viewfinder corners). Repo: `shakedkleter92-ux/frame90`, served by GitHub
Pages at `https://shakedkleter92-ux.github.io/frame90/`.
The earlier film-strip logo files (`icon_logo.svg`, `Asset 1.svg`) were deleted by the user.
