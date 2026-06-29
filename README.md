# waveform-editor

An Ableton Live extension that opens an audio clip in a **sample-level waveform
and pitch editor** — a ProTools/Melodyne-style quality-of-life feature for
spot-fixing pops, clicks, clipping, and tuning without leaving Live.

## How it works

Live's extension API exposes no way to read or write a clip's raw samples
in place, so the extension does a **render → edit → new-clip** round-trip:

1. Right-click an audio clip (or an arrangement time-selection on an audio
   track) → **Edit clip** / **Edit selection**.
2. The clip's pre-FX audio for that region is rendered to a temporary WAV and
   decoded. (Session clips, which aren't on the timeline, are read straight
   from their source file.)
3. A modal canvas editor opens with the audio. Edit it, audition with the
   built-in transport, then **Apply**.
4. **Apply** writes the result as a lossless 32-bit-float WAV, imports it, and
   drops it on a **new audio track**, time-aligned under the original.

The original clip is never modified — the edited copy lives on its own track so
you can A/B and keep it or discard it.

## The editor

Three tabs — **Waveform**, **Pitch**, and **Spectral** — share the same
zoom/pan, undo/redo (up to 50 steps), and transport. Press **Space** to
play/stop; the waveform tab plays from the selection (or the whole region), the
pitch tab from a click-set play head.

### Waveform tab

- **Pencil** — drag to freehand-redraw samples.
- **Heal** — drag across a span; it's smoothly bridged (Catmull-Rom), the
  fastest way to kill a click or de-clip a peak.
- **Select** — drag a range (auto-snaps to zero crossings; toggle with **Z**),
  then apply:
  - **Fade In / Out** with selectable curves.
  - **Gain** in dB.
  - **Reverse**.
  - **Pitch bend** in semitones.
- Per-channel lanes with optional **Link** (mirror edits across channels),
  zoom/pan (wheel, Ctrl/Cmd+wheel), and **Ctrl/Cmd+A** to select all.
- **Spectro** (toggle with **G**) — a per-channel spectrogram strip under each
  lane, locked to the same time axis (log-frequency, Nyquist at top). Clicks
  and pops show as bright vertical broadband streaks, so they're far easier to
  find than in the time-domain trace. Click/drag the strip to mark a range,
  then switch to **Heal** to repair it; drag the strip's top edge to resize it.
  It re-renders after each edit and follows zoom/pan.

### Pitch tab

- Detects the **monophonic** pitch contour, segments it into notes, and lays
  them on a piano roll with a vertical keyboard.
- **Drag a note up/down to retune** — audio is transposed via TD-PSOLA
  (duration- and formant-preserving). Snaps to semitone; hold **Alt** for fine.
- **Correct All** snaps every note to the nearest semitone.
- **Reset & Re-analyze** reverts to the original audio and re-detects.

### Spectral tab

A drawable, invertible log-frequency spectrogram per channel — paint directly
on the spectrum to fix problems that are invisible in the waveform. Unlike the
waveform tab's display-only **Spectro** strip, this keeps a full STFT and
resynthesizes (weighted overlap-add) on each stroke, so edits round-trip back
into the audio. The brush is **time-pressed** like a Photoshop brush: dwelling
or overlapping passes deepen the effect, quick swipes barely touch it.

- **Attenuate** (**A**) — paint toward a noise floor to cut unwanted content
  (hum, bleed, broadband clicks).
- **Restore** (**R**) — paint the mask back toward the original (un-erase).
- **Heal** (**H**) — paint over a click/blemish to fill it from the surrounding
  frames.
- **Draw** (**D**) — synthesize fresh tones into empty spectrum.
- **Brush** size (S/M/L/XL) and **Flow** (Low → Instant) set how fast the brush
  bites; **Heat**/**Rainbow** toggle the color scheme.

### Limitations

- The whole region is loaded into the editor, so length is bounded by memory
  and render time — very long clips will be slow. Use a time-selection for fine
  click repair.
- The edited clip is **unwarped sounding audio**; the original's warp markers
  and fades are not carried over.
- The render captures everything sounding on that track in the range, so the
  target clip should sit alone over the edited span.
- Pitch detection is **monophonic** — chordal or polyphonic material will
  mis-segment.

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
