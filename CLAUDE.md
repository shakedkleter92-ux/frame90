# Film Camera

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
Bump `CACHE` in `sw.js` (`film-camera-vN`) **and** `__BUILD` / `__SW_URL`'s `?v=` near INIT in
`index.html` together on every deploy; `?__sw_reset=1` force-clears the service worker.

## Map of index.html

- **CAMERAS · FILMS · LENSES** — the look is **fixed**: there are no user adjustment sliders
  (removed on purpose — the user wants it to work like the original's palette picker). The
  panel has exactly two pickers, **01 Camera** and **02 Film**, each a 4-column grid capped to
  3 rows that scrolls internally with the custom square `.grid-scrollbar` (same pattern as the
  original's palette bar). The right-edge side slider is the **lens** (`LENSES`: 8mm fisheye,
  24, 28, 35, 50, 85, 135mm; default 50mm) and is the *only* thing that changes it — **picking a
  camera never changes the lens or the framing** (user-reported bug: cameras used to switch to
  "their" lens, which read as the canvas zooming in). **A lens never crops/zooms** — the full
  frame always stays in view; it changes the feel of space: barrel distortion for wide lenses
  (corners stay put, center bulges), shader depth-of-field falloff with bokeh-weighted
  highlights for long ones (`uDof`), round mask for the fisheye. No pincushion for tele lenses:
  keeping the full frame with pincushion needs pixels from outside the frame (smeared edges).
  A centered `#lens-toast` names the lens and its character while the slider moves.
  There is **no other zoom control** — the original's zoom-presets row and pinch-to-zoom dial
  markup were removed on request (their JS is still present but inert: it's all guarded on those
  elements existing). Live base zoom is always 1×.
  `FILM(...)` entries = chemistry (color, contrast, grain, halation, tone curve, split tone,
  B&W mix; instant films also carry their print frame). `CAMERA(...)` entries = optics/print
  (vignette, soft lens, leaks, fringing, frame, date stamp, grain multiplier) plus small color
  nudges added to the film's. `combineLook()` merges camera + film + lens into the settings
  object `film.render()` takes; `activeFilmSettings()` caches it — the setters
  (`setActiveCamera/Film/Lens`) null `__filmSettingsCache`. To add a camera or film, add one
  `CAMERA(...)`/`FILM(...)` call; pickers are built from `CAMERA_ORDER`/`FILM_ORDER`.
- **FILM ENGINE (WebGL)** — `film.render(source, {outW, outH, crop, mirror, settings, time})`
  draws one frame into `film.canvas` with a single fragment shader (`FILM_FRAG`). Crop/zoom/
  mirror is a UV transform, not a 2D draw. Grain size scales with output height so a 1080×1920
  capture matches the preview. Falls back to a rough Canvas2D `ctx.filter` version if WebGL is
  unavailable. `drawFilmOverlays()` draws the Polaroid/Instax frame and orange date stamp in 2D
  on top of renders (preview, every recorded video frame, exports).
- **renderView() / renderUpload() / refreshView()** — `renderView` paints the on-screen canvas
  (Live: latest film frame, cover-fit; Upload: the filtered photo, contain-fit).
  `renderUpload` re-runs the film pass for the uploaded photo at preview size. Call
  `refreshView()` after any settings change.
- **LIVE CAMERA** — unchanged camera plumbing from the original (facing defaults, ultra-wide
  0.5×, zoom dial, Android rotation fix). `liveLoop()` renders the preview (long side capped by
  `LIVE_PREVIEW_MAX`); `liveCoverCanvas` *is* `film.canvas`. The shutter (`captureLivePhoto()`)
  re-renders the current camera frame at full export size rather than upscaling the preview.
  Video recording blits the preview frame each tick (`drawRecordFrame()`) — never a second
  shader pass per tick (that's what made the original's video choppy).
- **SHUTTER SOUNDS** — `shutterSound`, synthesized with Web Audio (no audio files): noise
  bursts, body thumps, motor whirs, spring twangs and ratchets combined into one `PROFILES`
  entry per mechanism (SLR, Leica, compact, disposable, toy, Polaroid eject, Hasselblad…),
  mapped from camera keys in `CAMERA_SOUND` (unlisted → SLR) and loudness-matched by
  `PROFILE_GAIN` (measured from offline renders; Leica deliberately quietest). Played only for
  Live photo captures. `shutterSound.unlock()` runs on the splash tap — iOS only lets an
  AudioContext start inside a user gesture. On iPhone the ringer/silent switch can mute it.
  **Volume-UP = shutter** (keydown `AudioVolumeUp`/`VolumeUp`/175/24, Live only, key-repeat
  ignored; Down is left alone). Works only where a browser forwards volume keys to pages (some
  Android); **never on iOS** — WebKit doesn't expose them to web content at all, so that would
  need a native wrapper.
- **GALLERY** — photos stored as JPEG. The gallery FAB always shows a still preview of the
  latest capture (never a generic icon once something exists; an empty film-frame square before
  that) and "pops" when a new shot lands. Videos get a still `poster` (JPEG data URL grabbed
  from the last recorded frame in `toggleVideoRecording()`) used by the FAB and the grid —
  a `<video>` element as a thumbnail shows black on iOS until played.

## Conventions

Same as `../grid_symbol_maker/CLAUDE.md` (one file; only the mobile layout path is reachable;
iOS quirks are deliberate) — **except the visual style, which the user replaced on purpose**:
a VHS-sleeve theme from a reference image. Cream paper `--cream #f2ebe0`, deep plum ink
`--plum #4a3942`, and the sunset stripe run (`--stripes`: lime → yellow → orange → red → pink →
magenta → purple) as the only accent. **Every control is rounded** (`--radius` soft squares,
`--pill` for labels/chips) **and see-through** with a frosted `--fab-blur` — the opposite of the
original's square/opaque rule. Font is Outfit (heavy 800 for titles, light spaced caps for small
labels, like "BACK TO THE / 80s"); IBM Plex Mono is loaded only for the canvas date stamp. The
theme lives in the "VHS SLEEVE THEME" block at the end of the `<style>` — restyle there.
**Logo** (`icon.svg`, chosen by the user) = the shutter button *as it looks over the camera
feed*: a frosted warm-beige rounded square (`#bfa48f`→`#a88b77`, the translucent cream ring
sampled from a screenshot — an opaque cream ring vanished on the cream splash) holding the
8-color stripe run, same proportions as `#mob-shutter`. `icon-180.png` (apple-touch
— iOS ignores SVG touch icons) and `icon-512.png` are full-bleed PNG renders of it; regenerate
both if the SVG changes. `icon_logo.svg` / `Asset 1.svg` are the user's earlier film-strip logo,
no longer used.
