# waveform-editor

An Ableton Live extension that lets you **redraw the waveform of an audio clip**
with a pencil tool — a ProTools-style quality-of-life feature for spot-fixing
pops, clicks, and clipping without leaving Live.

## How it works

Live's extension API exposes no way to read or write a clip's raw samples
in place, so the extension does a **render → edit → new-clip** round-trip:

1. Right-click an audio clip (or an arrangement time-selection on an audio
   track) → **Redraw waveform…**.
2. The clip's pre-FX audio for that region is rendered to a temporary WAV and
   decoded.
3. A modal canvas editor opens with the waveform. Edit it with:
   - **Pencil** — drag to freehand-redraw samples.
   - **Heal** — drag across a span; it's smoothly bridged (Catmull-Rom),
     the fastest way to kill a click or de-clip a peak.
   - Zoom/pan (wheel, Ctrl/Cmd+wheel, Fit), per-channel lanes, undo/redo.
4. **Apply** writes the result as a lossless 32-bit-float WAV, imports it, and
   drops it on a **new audio track**, time-aligned under the original.

The original clip is never modified — the edited copy lives on its own track so
you can A/B and keep it or discard it.

### Limitations

- **Max 4 s per pass.** Longer regions are capped to the first 4 seconds (a
  banner warns you). Use a time-selection for fine click repair.
- The edited clip is **unwarped sounding audio**; the original's warp markers
  and fades are not carried over.
- The render captures everything sounding on that track in the range, so the
  target clip should sit alone over the edited span.
- No in-editor audio playback yet (visual editing only).

## Get Started

Learn about building extensions: https://ableton.github.io/extensions-sdk/

## Setup

The path to Ableton Live's Extension Host module is stored in `.env` as
`EXTENSION_HOST_PATH`. The generator filled this in for you; edit it if your
install moves.

## Scripts

```sh
npm start                  # build + run in Live's Extension Host
npm run build              # production bundle of src/extension.ts
npm run build:dev          # dev bundle (sourcemaps, not minified)
npm run package            # build for production + create a .ablx archive
```
