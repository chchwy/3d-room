# Tiny Rooms

Three interactive isometric 3D miniature bedrooms, each built entirely from simple Three.js shapes with no external 3D models. Every room switches between a warm sunset and a night mode.

**Live site:** https://chchwy.github.io/3d-room/

| Room | File | Highlights |
| --- | --- | --- |
| Skyline Bedroom | [`skyline.html`](skyline.html) | Desk with a glowing monitor, a sleeping cat, and a futuristic city with flying traffic outside the window. Sunset / neon night. |
| Tiny Dreamer Bedroom | [`girl.html`](girl.html) | Pastel kid's room with a play teepee, a toy shelf, crayons, a 2×2 bookshelf and fairy lights. Sunset / night. |
| Little Driver Bedroom | [`boys.html`](boys.html) | Blue boy's room with toy cars on a road-map rug (one drives laps), LEGO bricks, a handheld console and a traffic-light night light. Sunset / night. |

[`index.html`](index.html) is the gallery page that links to all three.

## Controls

- **Mode button** (bottom of the screen) or **Space** / **N**: switch between sunset and night.
- **Mouse**: move it to turn the room slightly.
- Add `#night` to a room's URL to open it in night mode, e.g. `skyline.html#night`.

## Run locally

There is no build step. Each room is a single HTML file that loads Three.js from the jsDelivr CDN, so you need an internet connection.

```sh
# open a file directly in a browser, or serve the folder:
python -m http.server 8000
# then visit http://localhost:8000/
```

## How it's made

- **Three.js 0.170** with `EffectComposer`, `UnrealBloomPass` and `OutputPass`, loaded as ES modules from jsDelivr's `+esm` endpoint.
- **Isometric look:** an orthographic camera that frames the room to fit any screen size.
- **Lighting:**
  - A shadow-casting sun comes in through the window. Invisible walls and a ceiling block it everywhere else.
  - A rect-area light makes the window glow, and point lights stand in for the lamps.
- **Textures:** all of them are drawn at runtime on `<canvas>`, including the floor planks, rugs, posters, the road-map rug and the console screen.
- **The city outside:** procedural instanced buildings with a custom shader for windows, neon trims and haze. It is clipped in screen space to the window's outline.
- **Switching modes:** one value from 0 (sunset) to 1 (night) blends every light, material and sky color over about two seconds.

## Repository layout

```
index.html      gallery landing page
skyline.html    Skyline Bedroom
girl.html       Tiny Dreamer Bedroom
boys.html       Little Driver Bedroom
thumbs/         gallery thumbnails (sunset + night per room, WebP)
```

## Note on characters and brands

The girl's room contains a Peppa Pig–style plush. The boy's room has LEGO-style bricks and a Nintendo Switch–style console. All of them are simple fan-made shapes with no logos. Peppa Pig, LEGO and Nintendo Switch are trademarks of their respective owners, and this project is not affiliated with them.
