# Clock Wall (virtual video-call background)

A single self-contained `index.html` that renders a wall of 17 world clocks with
live, animated hands. Built to be used as an **OBS Studio Browser Source** behind
a background-removed webcam feed, then sent out via OBS Virtual Camera.

## Layers

1. `bg.jpg` — photo of the clock faces (no hands, no glass), 3840×2160.
2. An inline **SVG** drawn on top in real time — the hands.
3. `fg.png` — transparent overlay of the glass highlights, 3840×2160.

All three are locked to the images' native 16:9 aspect and sized to **cover** the
viewport (centered, cropping any excess). The images are exactly 3840×2160, so a
16:9 browser source (e.g. 1920×1080) crops nothing — everything lines up
pixel-for-pixel. A non-16:9 source crops the overflow rather than letterboxing.

## Usage in OBS

1. Add a **Browser Source** → *Local file* → this `index.html`.
2. Set its size to your canvas (e.g. 1920×1080) and enable "Shutdown source when
   not visible" off so it keeps animating.
3. Place it **behind** your webcam source (which has a green-screen Chroma Key or
   the Background Removal plugin applied).
4. Start **Virtual Camera**; select "OBS Virtual Camera" in Teams/Zoom and turn
   their own background effects off.

You can also just open `index.html` in any browser to preview.

## URL flags

- `?debug` — overlays each dial's outline, center crosshair, and city label.
  Use this to validate the placement/sizing of every clock.
- `?grid` — overlays a green alignment grid with coordinate labels and
  emphasized center axes. `?grid=120` sets the spacing (image px; default 240).
  The grid lives in the hands layer, so it warps with the hands — line it up
  against the wall's perspective while tuning the warp params.
- `?factor=` — global hand-width multiplier for live tuning (any per-clock
  `factor` still takes precedence). Example: `index.html?factor=0.8`.
- Warp / keyframe params for live experimentation (see "Overall warp" below):
  `?sx=`, `?sy=` (x/y scale), `?tilt=` (rotateX degrees), `?persp=` (perspective px).
  Example: `index.html?sx=1.04&sy=0.96&tilt=5`.

## How the hands are drawn

Two approaches were considered:

- **Drawn (vector) hands** — the JS draws each hand from the dial center using the
  center/radius you provide. This is the default and recommended path: it is
  computationally trivial (SVG transforms are GPU-composited), supports per-clock
  tweaks, and will cooperate with a future "camera angled down" perspective tilt.
- **Image hands** — drop a transparent PNG/SVG per hand and place/rotate it.
  Supported per-hand without changing the engine (see `images:` below); worth it
  only if you want photographic hand textures/shadows on specific clocks.

### Performance

One `requestAnimationFrame` loop drives everything. To minimize DOM writes:

- second hands sweep every frame (only Philadelphia has one by default),
- minute hands advance once per whole second,
- hour hands advance once per 30 seconds,

and a rotation is only written to the DOM when it actually changes. With 17
clocks this is negligible CPU, leaving headroom for OBS compositing + screen
share. (No "even/odd second multiplexing" was needed — the cost isn't there.)

Time zones use **IANA names** and are resolved with the browser's built-in
`Intl.DateTimeFormat` (no external library); UTC offsets are cached and refreshed
every 5 minutes so DST transitions are handled automatically.

## Configuration

Everything lives in the `<script>` in `index.html`:

- `CLOCKS` — one entry per clock: `city`, IANA `tz`, dial **center** `cx`,`cy`,
  diameter `d`, optional straightness `corr` (degrees, + clockwise / −
  counterclockwise). Coordinates and diameter are in native image pixels
  (3840×2160). Open `index.html?debug` to verify the green outlines sit on the
  real dials.
- Second hands are **off by default**. Give a clock a red second hand by adding
  `second: "sweep"` (smooth) or `second: "tick"` (steps each second). Only
  Philadelphia has one out of the box.
- `DEFAULTS` — hand lengths/thicknesses (as fractions of each dial's radius) and
  colors. Override per clock with a `style: { ... }` object; overrides merge
  per-property, so `style: { minute: { color: "#a00" } }` keeps the default
  minute length and thickness.
- `factor` — hand-**width** multiplier, also applied to the **hub dots**. Hand
  thickness and hub radius are fractions of each dial's radius, so large dials get
  fat hands and small ones get spindly hands. Set `DEFAULTS.factor` (the global
  baseline) and/or a per-clock `factor` to even them out (`< 1` slims, `> 1`
  thickens). The factor only scales values that come from `DEFAULTS`; if a clock
  overrides a hand's `thick` (or a `hubGray`/`hubRed`) explicitly, that value is
  used verbatim and the factor is discarded for it.
- Hands cast a light, diffuse drop shadow onto the dials via a single
  `filter: drop-shadow(...)` on the `#hands` SVG (in the CSS). Adjust the offset,
  blur, and opacity there to taste.
- Per-clock photo hands: add `images: { hour|minute|second: { href, pivot } }`,
  where `pivot` is the fraction of the image height from the rotation center to
  the far tip (0.5 = pivot at the image's middle).

### Overall warp / keyframe effect

The warp is applied to the **hands layer only** — not the `bg.jpg`/`fg.png`
photos. The photos already contain the camera's slightly-downward perspective;
the warp bends the flat-drawn hands to conform to it. (Use `?grid` to see the
warp applied to a reference grid and match it against the wall.) Edit the `WARP`
defaults in `index.html`, or experiment live with the URL params above:

- `scaleX` / `scaleY` (`?sx` / `?sy`) — non-uniform x/y scaling.
- `tilt` (`?tilt`) — `rotateX` degrees, leaning the top away from the viewer.
- `persp` (`?persp`) — perspective distance in px (smaller = stronger 3D).

At the defaults (`tilt: 0`, `scale 1`) the transform is visually identity, so the
wall renders flat until you dial something in.
