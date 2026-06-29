# Clock Wall Virtual

A web-based reproduction of the world clock wall I use as a video-call background, so I can use it when working away from home via OBS Studio compositing.

## Layers

1. `bg.jpg`: photo of the clock faces without hands or glass, 3840×2160.
2. An inline **SVG** of hands drawn on top in real time. This includes a soft drop shadow under the hour/minute hands; second hands have no shadow for performance reasons.
3. `fg.png`: transparent overlay of the glass highlights, 3840×2160.

The graphics are sized to cover the viewport so any excess is cropped.

## Configuration

The `<script>` contains clock definitions, style defaults, and warp/keyframe effect settings; `<style>` defines the drop shadow.

URL parameters can be used for experimentation:
- `?debug` for each clock's origin and radius.
- `?factor` overrides the global hand-width multiplier.
- `?grid` to help find the right warp; `?grid=120` sets the spacing
- Warp params: `?sx=`, `?sy=` (x/y scale), `?tilt=` (wall tilt in degrees), `?persp=` (distance from wall).

## Usage in OBS

1. Set Browser Source to "Local file" and choose this `index.html`.
2. Set its size to 1920×1080 and disable "Shutdown source when not visible".
3. Place it behind the webcam source (with green-screen Chroma Key or the Background Removal plugin).
4. Start Virtual Camera and select "OBS Virtual Camera" in Teams/Zoom.