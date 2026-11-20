# Mandelbrot Metal movie renderer for Mac

`mm-render` is a native, headless Metal command-line renderer for long-running fractal movies. Explore and compose in the iPhone/iPad app, export a **Mac flight project**, then render the full movie on an awake Mac. Ordinary Mandelbrot Metal bookmark JSON also works.

This developer preview requires macOS 15 or later and targets Apple Silicon Macs. The CLI uses the app's authoritative Metal kernels, compensated coordinate math and built-in palettes, without UIKit, a simulator or an on-screen rendering view. A CLI does not make fractal arithmetic free: deep or high-iteration scenes can still take a long time. It makes that work suitable for a resumable desktop job.

## Getting started

Extract the ZIP and open Terminal in the extracted folder. Keep `MandelbrotMetalRenderer_FractalRenderCore.bundle` next to `mm-render`. See the [quick start](README.md) for a sample render and distribution details.

## Render an exported flight

```sh
./mm-render \
  --project "$HOME/Downloads/My Flight.mmflight.json" \
  --output "$HOME/Movies/My Flight.mp4"
```

An app flight project embeds the precise center, actual palette LUT, rendering and lighting settings, the close-up scale, source canvas dimensions, direction, zoom range, duration and easing. The CLI uses those defaults. It does not depend on the app's current view or on an external palette file remaining unchanged.

Optional flags override the flight's timing, output size or quality:

```sh
./mm-render \
  --project "$HOME/Downloads/My Flight.mmflight.json" \
  --duration 60 --fps 30 --long-edge 1920 --samples 4 \
  --output "$HOME/Movies/My Flight HD.mp4"
```

Movies are silent H.264 MP4, with SDR sRGB/Rec.709 output. Dimensions are even numbers. By default, the long edge is 1,920 pixels and the source canvas proportions are retained. `--samples 1` is useful for a fast proof; `4` is the default, and `9`, `16` or `25` request more antialiasing on shallow GPU paths and CPU Julia rendering. Deep Mandelbrot perturbation retains the app engine's single-sample tier. Higher sample counts increase render cost substantially. Resolution and codec support depend on the Mac; an unsupported H.264 configuration fails clearly.

The current qualified navigation scale remains `1e-9...1e25` points per complex-plane unit. A Mac does not automatically increase the renderer's precision limit. Deep Julia views may need CPU precision and remain particularly expensive.

## Render an ordinary bookmark

```sh
./mm-render \
  --bookmark "$HOME/Downloads/Bookmark.json" \
  --direction out --zoom full --duration 20 \
  --output "$HOME/Movies/Fractal Fly-Out.mp4"
```

The bookmark is the **close-up**. A fly-out begins there; a fly-in ends there. `--zoom full` connects overview scale and the close-up at the same center. It does not pan to the Mandelbrot set's default center. A numbered range, such as `--zoom 256`, covers a 256-fold change in scale. `--linear` gives constant logarithmic zoom speed; the default eases the start and finish.

Raw bookmarks do not contain the source viewport dimensions. Their documented default reference canvas is **390 × 844 points**. If you created the bookmark on an iPad or another window size, specify the actual canvas dimensions with `--canvas-width` and `--canvas-height` to reproduce the framing. An exported Mac flight project is preferable because it carries those dimensions automatically.

```sh
./mm-render \
  --bookmark "$HOME/Downloads/Bookmark.json" \
  --canvas-width 1024 --canvas-height 768 \
  --width 1920 --height 1440 \
  --direction in --zoom 65536 --duration 30 \
  --output "$HOME/Movies/Fractal Fly-In.mp4"
```

If explicit movie dimensions have a different aspect ratio from the reference canvas, the shared export mapping preserves the center and square pixels while extending the view to fill the output. It does not stretch the fractal. Playback duration is rounded to the nearest whole frame and requires at least two frames.

Named built-in palettes are resolved from the bundled app catalog. For a custom palette, use a flight project with an embedded LUT or provide the app's standard palette JSON:

```sh
./mm-render \
  --bookmark "$HOME/Downloads/Bookmark.json" \
  --palette "$HOME/Downloads/My Palette.mandelpalette.json" \
  --output "$HOME/Movies/Custom Colors.mp4"
```

## Pause and resume

Press **Control-C**, or send SIGTERM. The renderer cooperatively stops between bounded tiles, finishes encoding complete frames, saves its checkpoint, and exits with status **130**. The partial movie is not published as the requested output.

```sh
./mm-render \
  --resume "$HOME/Movies/Fractal Fly-Out.mp4.render"
```

The default checkpoint directory is the output path with `.render` appended. `--state DIRECTORY` selects another location. Keep the entire directory, including `manifest.json` and the completed section files. Its manifest embeds the resolved scene, palette and exact flight settings. The CLI rejects a resume with a different engine fingerprint, so it will not silently combine sections from different shader builds.

Each checkpoint has an exclusive process lock. A second job using the same state directory is rejected while the first is running. Independent jobs should use separate output and state paths.

Completed sections are committed atomically after at most one second of movie frames or roughly five seconds of rendering work. A slow frame is checkpointed as soon as it finishes. Force quitting, power loss or a crash can require repeating the current uncommitted section or frame; earlier committed work remains available. The final MP4 is assembled from completed sections without rendering them again. Checkpoint section files are removed only after the final movie and completed manifest have been saved.

`--pause-after N` intentionally checkpoints after N additional frames. This is useful for short work sessions and testing recovery:

```sh
./mm-render \
  --bookmark "$HOME/Downloads/Bookmark.json" --duration 10 \
  --pause-after 150 --output "$HOME/Movies/Test.mp4"

./mm-render --resume "$HOME/Movies/Test.mp4.render"
```

A resume uses its saved settings. Only the output destination, development `.metallib`, overwrite permission, quiet mode and per-run frame budget can be changed while resuming. Existing output files are protected unless `--overwrite` is explicitly provided. Replacement happens only after the new movie is fully assembled.

The renderer holds a macOS activity assertion to prevent **idle system sleep** while working. Display sleep is allowed. Manual sleep, closing a laptop lid, shutdown and power loss can still stop processing. Resume the checkpoint after waking or restarting the Mac.

Copyright © 2025–2026 Mandelbrot Metal. All rights reserved.
