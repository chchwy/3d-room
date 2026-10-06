# CLAUDE.md

Static GitHub Pages site (served from `master`, repo root) with three Three.js miniature bedrooms and a gallery page. There is no build step and no package.json.

## Files

- `index.html`: gallery landing page, a normal full HTML document. Its cards link to the room pages and use the images in `thumbs/`.
- `skyline.html`, `girl.html`, `boys.html`: one self-contained room per file.
- `thumbs/{skyline,girl,boys}-{sunset,night}.webp`: 960×720 gallery thumbnails.
- `boys.old.html`: the user's own scratch file. It is untracked; don't commit, edit or delete it.

## Room pages: how they're built

- **No document skeleton.** The room files start with `<meta charset>` / `<meta viewport>` / `<title>` and have no `<!doctype>`, `<html>`, `<head>` or `<body>`. This is deliberate, because the same files are also published as claude.ai Artifacts, which wrap the page in their own skeleton. Keep it that way.
- **Three.js 0.170 from jsDelivr `+esm` URLs**, for example `https://cdn.jsdelivr.net/npm/three@0.170.0/examples/jsm/postprocessing/EffectComposer.js/+esm`. Don't switch to an import map: the claude.ai artifact viewer blocks it, and the scene silently fails to load.
- **The engine is copied into all three files, not shared.** The renderer, camera, city, post-processing, `applyMix` and the loop are duplicated in each room. When you change engine behavior (sway, framing, city, bloom and so on), apply the change to all three files unless the user names one room.
- **Mood blending.** `mix` runs from 0 (sunset) to 1 (night). `applyMix(m)` lerps every light, the bloom, the exposure and the shader uniforms. Materials made with `glow(cSun, cNight, kSun, kNight, neon)` blend automatically. A new light needs an entry in `P` and a line in `applyMix`. The `#night` URL hash starts a room in night mode.
- **The city is clipped to the window in screen space.** `updateWinQuad()` feeds the window's projected outline to the sky, building, car and billboard materials through `clipShader` / `clipBuiltin`, which `discard` fragments outside it. Don't go back to a stencil buffer: it failed on the user's GPU and the city covered the whole room.
- **NaN guard.** A `ShaderPass` before the bloom zeroes out NaN/Inf pixels. Some GPUs produce them, and bloom smears them into big black blocks. Clamp the input to any `pow()` in a shader.

## Gotchas that keep coming up

- **Z-fighting.** The user notices it right away. Never leave two faces coplanar. The inner wall surface is at `-5` (x or z). The wainscot panel covers `-5 … -4.96` and the chair rail covers `-5 … -4.92`. Anything standing against a wall needs its back panel at roughly `-4.89` or further into the room. Decorative bands on objects (the bus stripe, for example) must be slightly bigger than the body, not the same size. The camera near/far is `40 / 280`; keep that range tight.
- **Seeded randomness.** Layout comes from a seeded RNG (`mulberry32(2088)`, used by `R()`, `rand()` and `pick()`). Removing code that calls them shifts every random value that comes after it: city buildings, rug trees, LEGO rotations, leaves. To remove something randomized without disturbing the rest, keep its generation and just don't add it to the scene. That's how the dust motes were hidden.
- **Camera sway** is `sway.x*0.09` / `sway.y*0.05` in the loop. A bigger sway needs a wider city (the `sx` spread and the car `lim`) and more framing margin in `fit()` (`upp` uses 0.9).
- **The user prefers less clutter.** Requests so far have removed floating dust motes, fairy lights and wall stars from the boy's room, and floor clothes, shoes and the fishbowl from the girl's room. Don't reintroduce them.

## Verifying changes (Windows, no Playwright)

Take screenshots with headless Edge, using a temporary copy of the page:

```sh
E="/c/Program Files (x86)/Microsoft/Edge/Application/msedge.exe"
"$E" --headless=new --use-angle=swiftshader --enable-unsafe-swiftshader \
  --window-size=1200,900 --virtual-time-budget=9000 \
  --screenshot=out.png "file:///D:/work3/3d-room/<room>.html#night"
```

- For thumbnails, write a copy to the scratchpad with `#mode{display:none!important} canvas{opacity:1!important;transition:none!important}` appended to the `<style>`, and with `const k = 0;` replacing `const k = reduceMotion ? 0 : 1;` to stop the sway.
- Headless Edge sometimes captures a shifted or dimmed frame with a dark band (pixel color `(26,16,34)`) along the bottom or right edge. Check for it and retake; `--window-size=1280,960 --force-device-scale-factor=1` often helps.
- Static screenshots can't show z-fighting or animation, so tell the user when something wasn't checked visually.
- To make thumbnails, resize to 960×720 with Pillow and save as WebP, `quality=82, method=6`. Retake the thumbnails whenever a room's look changes.

## Git

- Commit and push only when the user asks; they usually say "commit & push". Push to `origin master`.
- Stage specific files and leave `boys.old.html` out.
