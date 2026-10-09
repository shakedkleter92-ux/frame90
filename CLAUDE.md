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
  chroma, `interlace`, analog noise, optional `tapeFx`, `kbps`, `audio`). **Sound (user, 2026-10-09: "authentic
  to the camera")** — `AUDIO_PROFILES` per recording format, assigned by each video format's
  `audio`: `afmStereo` (Hi8: TR2000E, TRV87, Hi8, NightShot), `afmMono` (Video8, Canon
  UC-X10Hi — "Hi-Fi monaural" per Canon), `vhsLinear` (VHS-C/S-VHS-C linear mono SP ≈100 Hz–
  10 kHz: PV-L859, GR-SZ7, VHS '92), `vhsLinearSLP` (≈5 kHz), `vhsWorn`, `dv16` (16-bit 48 kHz:
  VX1000, XL1, TRV900, GL1), `dv12` (12-bit 32 kHz, consumer: GR-DV1, PC1, TRV110, MiniDV),
  `super8` (stripe ≈75 Hz–10 kHz, 18 Hz claw). `buildAudioChain(ac, input, prof, stop)` (works
  on Offline contexts too) = camera mic (hp/eq/lp, mono or narrow stereo) → motor whir
  (pre-AGC) → AGC compressor + makeup → format: wow/flutter delay, noise floor, bandwidth,
  12-bit → limiter → soft clip. `buildAudioGraph()` wraps it for Live recording and uploaded-
  video export (film presets pass sound through). The Live mic is requested **raw**
  (echoCancellation / noiseSuppression / autoGainControl false, channelCount ideal 2) — the
  phone's processing made it sound modern. **24-camera library (user's spec, 2026-10-09)** —
  `CAMERA_LISTS`: **Photo** (12) = Nikon F90 · Superia 200, Canon EOS 5 · Portra 160NC, EOS 500 ·
  Royal Gold 100, Contax G1 · Reala 100, Leica Minilux · Portra 400VC, Olympus mju-II · Superia
  400, Kodak Gold 200 (35mm compact), FunSaver · Gold 400, Instax Mini 10, Kodak DC25, Sony
  Mavica MVC-FD5, Kodak DC290; **Video** (12) = Sony CCD-TR2000E (Hi8 PAL, 1994–95), Canon
  UC-X10Hi (Hi8, 1997), Sony CCD-TRV87 (Hi8 XR, 1999), JVC GR-SZ7 (S-VHS-C, 1994), Panasonic
  PV-L859 (VHS-C, 1999), Sony DCR-VX1000 / JVC GR-DV1 / Sony DCR-PC1 (1998) / Canon XL1 /
  Sony DCR-TRV900 / Canon GL1 (MiniDV), Sony DCR-TRV110 (Digital8). Years/formats checked;
  unverifiable spec models were swapped for documented ones (GR-DVX → VX1000; unnamed VHS-C /
  S-VHS-C / Canon Hi8 → PV-L859 / GR-SZ7 / UC-X10Hi). Spec rules: restrained and believable,
  keep detail (**the user dropped the earlier "nothing may look sharp" rule for this**), no
  film grain on digital, no tape noise on DV, flash / timestamps / borders / tape FX off by
  default, no intensity slider (user: keep UI as is). Film = Kodak Photo CD scan 3072×2048
  (`FILM_SCAN`); analog tape at full SD (640×480 / PAL 768×576) limited by `tape` bandwidth +
  `chromaSub` smear; DV 720×480 anamorphic. The earlier presets are **back** (user, 2026-10-09:
  "I like the old filters"), after the 12 new ones in each list — Photo: Polaroid, Holga X-Pro,
  DC20, QV-10, Cyber-shot, QuickCam, QuickTake (19 in all); Video: VHS '92, VHS SLP, VHS Worn,
  Super 8, Video8, Hi8, MiniDV, NightShot (20). The picker scrolls. The picker shows the current
  tab's list (`activeCameraList()`: Live → capture mode; Upload → a loaded video = Video, else
  Photo); each list remembers its pick (`ACTIVE_IN_LIST`, `syncCameraList()`).
  **Flash button** `#mob-flash` — top row, left of the menu button (user: aligned to the top,
  not under the menu), on **every tab and camera**: a camera's own flash, else
  `FLASH.external` (clip-on/hot-shoe) for stills, `FLASH.videoLight` (continuous halogen)
  for camcorders. Toggles `PRESET_TOGGLES[key].flash`; in Live it drives the phone's torch
  where `getCapabilities().torch` exists (Android Chrome; never iOS): fired for the shot in
  Photo, kept on in Video (`syncVideoTorch()`). **Back camera** = torch (tried even when
  `getCapabilities()` doesn't list it — iOS 17.5+/18 Safari can switch it but reports it
  unreliably); **front camera** = the screen: `#screen-flash` (whole screen warm white for the
  shot, frame taken 300 ms in) / `#screen-ring` (white ring around the live frame in Video). Added 2026-10-08: `polaroid` (600 instant film, 1:1, flash on, no
  forced white border), `holga` (cross-processed slide film, 1:1, plastic-lens corners),
  `super8` (Kodachrome 40 cine film, 18 fps frame hold via `fps`, film grain, gate `weave` +
  `flicker` look params, audio `S8`). A film may set its own `px`/`formats` (instant and
  120 aren't Picture CD scans of 35mm); `FORMAT_RATIO` has `1:1`. `vhsworn` (VHS
  Worn, 2026-10-08, from a sunset-over-rails clip) = hot saturated warm tape, long red bleed and
  constant oxide dropouts (look param `dropouts`, shader stage 5b — part of that preset's look,
  separate from the optional `tapeFx` layer); no OSD. The user's "CAMERA1 / PLAY / SOURCE IPHONE"
  references are the same filter as `vhs92` — not added again. (History: on 2026-10-08 the user removed Mavica FD5, DC290, Gold 200 and Superia 400 as
  too sharp and ruled "nothing may look sharp"; on 2026-10-09 their 24-camera spec brought
  those back and replaced that rule with "realistic, keep detail".) One picker, titled **CAMERA TYPE** (no number), showing the current tab's list — the
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
  Live photo captures. All effects are deliberately gentle (user, 2026-10-09): master `out`
  gain 0.15 (was 0.85; 0.38 was still too loud) + a −5 dB high shelf at 3.5 kHz and a 9 kHz low-pass. Each preset names its sound (`sound`; unlisted → `digicam`); the 24-camera library has
  its own mechanisms (user, 2026-10-09: "authentic sound for each button"): `nikonf90`
  (mirror + metal curtain + built-in winder), `eos5` (Canon AF double beep, quiet damped
  mirror), `eos500` (beeps, plasticky mirror, louder winder), `contaxg1` (buzzy lens AF, crisp
  metal shutter), `minilux`, `mju2` (AF, leaf tick, motor wind); FunSaver/Instax/Mavica/DC25/
  DC290 use `disposable`/`instax`/`mavica`/`earlydigi`/`digicam`. **REC button sounds**
  `shutterSound.rec(key, start)` (start/stop of Live video recording), by format: `8mm` (beep,
  pinch roller, capstan spin-up; 2 beeps on stop), `vhsc` (heavier clunk), `dv` (soft click),
  `super8` (motor spin-up / wind-down); beep pitch by brand (approximate). **They never reach the recording** (user,
  2026-10-09 — the raw mic heard the speaker): the record graph always exists (passthrough
  for presets without a profile) and has a gate — start = `hold(sound length + 0.3 s)` then a
  50 ms fade-in; stop = `mute()` before the stop sound plays; the film-camera profiles are
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
