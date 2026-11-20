# Mandelbrot Metal Mac Renderer 1.0.0 Preview

Render a Mandelbrot Metal 3.1 flight project or bookmark as a silent H.264 MP4. The renderer runs locally in Terminal and saves progress so you can resume long jobs.

**Requirements:** Apple silicon (arm64), macOS 15 or later. Extract the ZIP and keep `mm-render` and `MandelbrotMetalRenderer_FractalRenderCore.bundle` side by side.

This developer preview carries a Developer ID signature. This build has no Apple notarization, so Gatekeeper may flag the downloaded executable. See [Apple’s guidance for software downloaded outside the App Store](https://support.apple.com/en-us/102445).

## Run the included example

Open Terminal in the extracted folder and run:

```sh
./mm-render --version
./mm-render \
  --bookmark examples/mandelbrot-boundary.json \
  --duration 2 --long-edge 720 --samples 1 \
  --output "$HOME/Movies/Mandelbrot Boundary.mp4"
```

## Render a project from iPhone or iPad

In Mandelbrot Metal 3.1 with Pro or Legacy Pro, open **More (…) → Movies**. Choose the direction, zoom range and duration. Tap **Export for Mac…**, then **Save or Share Flight Project**. Transfer the `.mmflight.json` file to your Mac and substitute its filename in this command:

```sh
./mm-render \
  --project "$HOME/Downloads/My Flight.mmflight.json" \
  --output "$HOME/Movies/My Flight.mp4"
```

The project carries your scene, colors, canvas proportions and timing. The Mac renderer defaults to 30 fps and a 1,920-pixel long edge. Add `--long-edge 720 --samples 1` for a quick proof.

## Pause and resume

Press **Control-C** and let the renderer save completed work. Keep the entire checkpoint folder. To resume:

```sh
./mm-render --resume "$HOME/Movies/My Flight.mp4.render"
```

Run `./mm-render --help` for all options. See [the full CLI guide](CLI-GUIDE.md) for output settings, bookmark framing and recovery. The [illustrated movie article](https://mandelbrot-metal.com/blog/f/from-a-single-view-to-a-fractal-movie) covers iPhone and iPad setup.

Copyright © 2025–2026 Mandelbrot Metal. All rights reserved.
