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
  `chromaSub` smear; DV 720×480 anamorphic. **Per-camera tuning to the user's reference renders** (2026-10-12 →, one camera at a
  time: the user sends a camera + the prompt they gave an image AI + its rendered image):
  measure the reference (sky / whites / darkest 5% / foliage hue+sat / flat-area grain std),
  run test patches (gray ramp, sky, foliage, skin, red) through the preset, and adjust the
  film/sensor until the patch outputs match those traits — without over-fitting to the
  reference's scene (e.g. its dry olive foliage). Done: **F90 · Superia 200** (lifted olive
  blacks ≈21,22,14, warm whites ≈251,249,245, sky ≈212° sat ≈0.56, greens toward olive, fine
  visible grain; deep blues kept blue). **EOS 5 · Portra 160NC** (same scene as the F90 reference, so tuned relative to it: warmer,
  sky ×0.8 saturation, softer warm highlights ≈250,248,241, same lifted warm blacks ≈23,22,13,
  grain ×0.8). **EOS 500 · Royal Gold 100** (vs Portra: warmer still — cream whites ≈249,244,236,
  golden roofs, yellow-warm greens — ~20% more saturated, a touch darker/deeper sky, warm
  blacks ≈25,22,13; grain kept at the F90's level although the reference shows more — a
  100-speed film shouldn't read grainier than a 200). Order of the three, coolest → warmest:
  F90, EOS 5, EOS 500. **Contax G1 · Reala 100** (the deepest blacks of the four ≈18,18,12 / most contrast,
  a deep saturated sky like the F90's but darker, neutral whites ≈248,246,242, natural greens,
  fine grain between Portra and Superia; the crispest — body adds micro-contrast). **Leica Minilux · Portra 400VC** (the most vivid and darkest: midtones down via gamma 1.28
  with whites kept, a deep cyan-leaning sky via the matrix, warm low-blue blacks, 400-speed
  grain, body vignette 20 — the reference showed ~27% darker corners, moderated per the spec's
  "no extreme vignetting"; the sky's extra depth via `blueSat` 0.45 so skin stays ≈0.6).
  **Olympus mju-II · Superia 400** (the only cool one: neutral whites ≈248, cool green blacks
  ≈13,17,14, muted cool greens, midtones ≈46%, deep sky via `blueSat` 0.3, the most grain —
  consumer 400; body: compact softness, vignette 20). **`blueSat`** (look param, shader color
  stage) = extra saturation only where blue clearly leads (skies, water) — added because
  deepening the sky with global saturation or the matrix pushed skin orange. **Kodak Gold 200 · 35mm compact** (Royal Gold's family look with softer contrast: the
  most lifted warm blacks ≈28,26,17, creamy warm whites, warmer greens, a touch more grain;
  body `compact35` = the softest film-camera optics, soft 0.9 / blur 10, mild vignette 6). **FunSaver · Gold 400** (the low-fi one: heavy warm golden cast, muddy yellow-brown
  greens, a washed-out gray-cyan sky via `blueSat` −0.6 — `blueSat` works both ways — harsh
  contrast with near-black blacks ≈12,9,5 and near-clipped warm whites, thick grain ≈2.5× the
  SLR films; then **full strength on the user's request** ("I want the FunSaver like the
  reference"): body vignette 92 (near-black corners), soft 4.2, blur 60, chroma 10, and the
  reference's orange flare top-right via look param **`glow`** [x, y, radius, strength] +
  `glowColor` (screen-blended stray light; FunSaver only — the one exception to "no leaks").
  **Instax Mini 10 · Instax Mini** (pastel and dreamy: saturation −45, lifted slightly purple
  blacks ≈39,36,42, cool never-white highlights ≈239,243,244 with a strong shoulder, pale sky,
  muted light teal greens, smooth fine grain; body soft 3.6 / blur 20 — very blurry, like the
  reference). `sensor()` adds a 12-tap disc when `uSoft` ≥ 2 so heavy softness stays smooth
  (the sparse 8 taps left a mesh pattern). **Kodak DC25** (primitive sensor per the reference: whites clip hard at `white` 0.8,
  blacks to 0, dark midtones, an electric blue sky ≈43,132,254 via `blueSat` 1.6 over muted
  crosstalk color, muddy greens, color noise). Sensor flag **`pixelated`** → `cam.pixelated`:
  `cameraJpeg()` enlarges ≈5× (to ~2400px) with smoothing off, and `renderView()` draws the
  preview without smoothing — hard pixels, like the reference. **Date stamp on by default for
  the DC25** (`date: 'dc25'`: white VT323 MM/DD/YY bottom-right, drawn at native size so it
  enlarges as blocks) — the user's reference/prompt include it, an exception to "no stamps". **Sony Mavica FD5** (very soft — blurry, not pixelated — pure saturated digital blue via
  `blueSat` 3.2 over muted crosstalk color, crushed blacks / clipped whites, dark midtones,
  muddy greens, floppy-size JPEG q 0.55). **Kodak DC290** (the sharpest and most vivid digicam:
  Kodak's warm yellow-orange processing, a saturated but natural sky ≈67,135,216, slightly
  warm clipped whites, deep blacks, a little shadow noise). Digital-camera softness must be
  judged at native size (patch/A-B renders at the film scan size understate it).
  **Video, tuned to reference frames (2026-10-12 →):** **Sony CCD-TR2000E** (warm consumer
  video: yellowish highlights ≈254,251,237, warm lifted blacks, a muted sky ≈82,130,189, warm
  greens, analog noise; OSD **`date: 'hi8'`** on by default — "OCT.12 2026" / "04:25:52 PM"
  bottom-left, "Hi8" / "PLAY" top-right, live clock — as in the reference). New tape look
  params: **`blank`** (black capture-blanking bars at the left/right edges, fraction of width;
  left ×1.3) and **`headSwitch`** (the bottom strip shifted and torn, fraction of height).
  **Canon UC-X10Hi** (warmer than the Sony: yellow highlights ≈254,249,230, deeper warm blacks
  ≈9,7,4, darker midtones, a more muted sky via `blueSat` −0.25, warmer greens, a little more
  noise, vignette 30 via the format's `adj`; no OSD, no bars — as in its reference; midtones
  darkened with gamma, not exposure, so whites stay ≈255,250,231). **Sony CCD-TRV87** (the
  warmest Hi8: most yellow highlights ≈254,248,226, lifted warm blacks ≈15,13,7, most saturated
  warm tones over a muted sky, the strongest vignette 40, a little less noise and a touch
  sharper — XR; no OSD). Lesson: darken video midtones with `gamma`; `exposure` dims the whites. **JVC GR-SZ7**
  (like the TRV87's warmth but the deepest tape blacks ≈8,7,3, a touch more saturation, milder
  vignette 20, a little more noise and S-VHS edge crispness; `blank` + `headSwitch` on, as in
  its reference; OSD off). **Panasonic PV-L859** (lifted warm-gray blacks ≈22,21,18, clean slightly warm whites
  ≈253,253,242, a bluer sky than the Hi8s, the heaviest grain; `tape` 0.7 / soft 0.35 — the
  first try at tape 1.1 smeared far more than its reference, which keeps about the same detail
  as the other tapes; bars + head-switching on, OSD off). **Sony DCR-VX1000** (clean digital SD: grain 2, no
  vignette, crisper than tape; natural not-warm greens, a deep less-saturated sky via `blueSat`
  −0.25, slightly warm whites ≈255,254,242, near-neutral blacks; midtones via gamma 1.3). **JVC GR-DV1** (single-CCD DV: neutral hard-clipping whites ≈253,253,251, slightly lifted
  blacks, a lighter muted sky via `blueSat` −0.3, less saturation, grain 12 — several times
  the VX1000's noise — and softer, soft 0.6). **Sony DCR-PC1** (GR-DV1's single-CCD noise and clipped whites, but warmer
  blacks/greens, a lighter cyan-leaning sky via `blueSat` −0.25, and crisper — soft 0.35,
  sharpen 0.35 — near the VX1000). **Canon XL1** (the VX1000's cleanliness with more depth: gamma 1.38,
  a deep saturated sky, warm not-quite-clipped whites ≈252,249,231 — white 0.97 — deep blacks). **Sony DCR-TRV900** (the most restrained 3-CCD: saturation −26, cooler greens, a lighter
  muted sky ≈82,118,168, slightly warm whites, deep blacks; the crispest DV — soft 0.1, sharpen
  0.4 — with visible sensor noise, grain 10, as in its reference). **Canon GL1** (the XL1's Canon color —
  same warm whites ≈252,249,231, deep blacks, earthy greens — with a lighter less saturated sky
  ≈76,116,167 via `blueSat` −0.2, saturation −13, a touch crisper; clean). **Sony DCR-TRV110**
  (Digital8 — the warmest, most saturated single-CCD DV: deep warm blacks, darker midtones,
  clipped slightly warm whites, a natural sky ≈95,134,190, single-CCD noise). **All 12 spec
  video cameras tuned.** **VHS '92 re-tuned** to a new user reference (2026-10-12), replacing the old
  pink-lilac look: warm aged VHS-C — more saturated than the PV-L859, blacks ≈15,13,11, vignette
  38, heavy noise, occasional `dropouts` 0.006, bars + head-switching, 640×480; OSD `cam92` now
  "CAMERA 1" top-left, "PLAY ▶" top-right, "OCT.12 2026" bottom-left, "04:57 PM" bottom-right.
  **VHS SLP re-tuned** (2026-10-12, replacing the violet look): a cool, worn long-play tape —
  the softest picture (320×240, soft 1.4, tape 1.1), lifted bluish blacks, neutral whites, a
  saturated sky via `blueSat` 0.45 over near-colorless greens (saturation −40), heavy snow,
  dropout streaks crowding the top/bottom (new look param **`dropEdge`**, multiplies the
  dropout rate toward the frame's top and bottom 14%), bars + head-switching; **OSD off**
  (its prompt says no text). Lesson: strong `blueSat` + bluish `shadows` + tape smear made
  purple blotches and blue halos — keep `blueSat` ≤ 0.5 and `sharpen` low on smeared tapes.
  **VHS Worn re-tuned** (2026-10-12, replacing the orange-sunset look): a mistracking worn tape —
  SLP's cool palette with deeper bluish blacks and much more snow (grain 48), plus new look
  params **`rgbSplit`** (red alone slips sideways by 2× this fraction of width → red/cyan
  fringes) and **`trackBand`** (a band of torn lines + dashes of static drifting around y≈0.24),
  and a tall wavy head-switching zone (`headSwitch` 0.1; the strip now waves with amplitude
  scaled by its height, so the thin 0.015 strips on other presets barely move). Old
  `glow`-free, `dropouts` 0.02 with `dropEdge` 4; OSD off.
  **Super 8 re-tuned** (2026-10-12 reference; a different scene, so measured by hand): classic
  Kodachrome — crushed blacks, very dark midtones (gamma 1.45), a deep saturated sky
  (`blueSat` 1.0), warm glowing highlights ≈255,252,225, warm earthy greens, heavy film grain
  45, soft, gate vignette 55 (body), weave + flicker kept. **Video8 re-tuned** (2026-10-12): warm and faded — deep
  warm blacks, dark midtones (gamma 1.4), near-neutral whites, a pale muted sky via `blueSat`
  −0.35, warm brownish greens, vignette 44, analog noise, soft (320×240); OSD off. **MiniDV
  (Handycam) re-tuned** (2026-10-12): crushed blacks, very dark midtones (gamma 1.5), neutral
  clipped whites, a pale muted sky via `blueSat` −0.4, warm dark greens, moderate noise, no
  vignette, soft 0.5 (was an over-soft 1.2); OSD off.
  **NightShot re-tuned** (2026-10-12) to **daylight infrared**: mono via `mix` [0.25, 1.75,
  −1.5] (blue sky → near black, clouds stay bright) plus new look param **`irGreen`** (adds
  glow where green leads red and blue — vegetation reflects IR even in shade, the Wood
  effect); phosphor-green `duo` [≈2,7,2 → 230,255,230], IR halation, heavy analog grain,
  vignette; `tape` 0 and `chromaSub` [1,1] — any color smear gets amplified by the strong mix
  into smeared brightness. OSD off. Hi8 (Handycam) is the one earlier preset not re-tuned yet. The earlier **video** presets are back after the 12
  new ones (user, 2026-10-09: "I like the old filters") — VHS '92, VHS SLP, VHS Worn, Super 8,
  Video8, Hi8, MiniDV, NightShot (20 in Video; the picker scrolls). The earlier **photo** presets
  (Polaroid, Holga X-Pro, DC20, QV-10, Cyber-shot, QuickCam, QuickTake) were **deleted** on the
  user's request (2026-10-12) — Photo is exactly the spec's 12. **Print frames**: a camera
  whose pictures were physical prints gets that print around the *saved* photo (`frame` part →
  `printFrame()` in `captureLivePhoto()` / `buildDownloadCanvas()`; the viewfinder shows only
  the picture, like the real camera). Only the Instax has one: the real Instax Mini card, 54×86
  mm around a 46×62 mm image (4 mm sides, ≈6 mm top, ≈18 mm bottom strip), off-white; a
  landscape shot sits in the card held sideways (thick strip on the right). The 35mm films were
  printed borderless and the digicams are files, so they get none. The picker shows the current
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
  shot, frame taken 300 ms in) / `#screen-ring` (white ring around the live frame in Video). Added 2026-10-08: `super8` (Kodachrome 40 cine film, 18 fps frame hold
  via `fps`, film grain, gate `weave` + `flicker` look params). A film may set its own
  `px`/`formats` (Instax isn't a 35mm scan); `FORMAT_RATIO` has `1:1` (unused since the square
  Polaroid/Holga were deleted). `dropouts` (look param, shader stage 5b: oxide-dropout streaks, part of a
  preset's look, separate from the optional `tapeFx` layer) was added 2026-10-08 for VHS Worn. The user's "CAMERA1 / PLAY / SOURCE IPHONE"
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
  4:3 contain-fit frame was rejected as "wide" — keep it upright. **Ratios are fixed per camera** (user,
  2026-10-12, confirmed "keep it as it is now"): each camera's real ratio — 35mm film & DC290
  2:3, DC25 / Mavica / video 3:4 (upright 4:3), Instax 3:4 in its card, MiniDV 4:3 or 16:9 —
  always upright in Live; uploads keep their own orientation. Reference renders being
  landscape is not a reason to change this; tuning touches color/tone/texture only.
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
