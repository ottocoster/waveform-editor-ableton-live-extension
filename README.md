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

Two tabs share the same zoom/pan, undo/redo (up to 50 steps), and transport.
Press **Space** to play/stop; the waveform tab plays from the selection (or the
whole region), the pitch tab from a click-set play head.

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

### Pitch tab (Melodyne-style)

- Detects the **monophonic** pitch contour, segments it into notes, and lays
  them on a piano roll with a vertical keyboard.
- **Drag a note up/down to retune** — audio is transposed via TD-PSOLA
  (duration- and formant-preserving). Snaps to semitone; hold **Alt** for fine.
- **Correct All** snaps every note to the nearest semitone.
- **Reset & Re-analyze** reverts to the original audio and re-detects.

### Limitations

- The edited clip is **unwarped sounding audio**; the original's warp markers
  and fades are not carried over.
- The render captures everything sounding on that track in the range, so the
  target clip should sit alone over the edited span.
- Pitch detection is **monophonic** — chordal or polyphonic material will
  mis-segment.

## Get Started

Learn about building extensions: https://ableton.github.io/extensions-sdk/
