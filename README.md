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
- `?tick` — second hands step once per second instead of sweeping smoothly.

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

- second hands sweep every frame,
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

- `CLOCKS` — one entry per clock: `city`, IANA `tz`, top-left `x`,`y`, diameter
  `d`, optional straightness `corr` (degrees, + clockwise / − counterclockwise).
  Dial **center** is computed as `(x + d/2, y + d/2)`.
- `DEFAULTS` — hand lengths/thicknesses (as fractions of each dial's radius) and
  colors. Override per clock with a `style: { ... }` object.
- Per-clock photo hands: add `images: { hour|minute|second: { href, pivot } }`,
  where `pivot` is the fraction of the image height from the rotation center to
  the far tip (0.5 = pivot at the image's middle).

### A placement assumption to confirm

The clock list was given as "x/y from top left, diameter". This is read as the
**top-left corner of each dial's bounding box**, so the center is `x + d/2`,
`y + d/2`. Open `index.html?debug` to verify the green outlines sit on the real
dials. If your `x,y` were actually the dial *centers*, change the two lines in the
build loop from `c.x + r, c.y + r` to `c.x, c.y`.
