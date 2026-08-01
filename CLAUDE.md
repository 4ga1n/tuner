# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the App

No build step. Open `app/index.html` directly in a browser, or serve with any static file server:

```sh
cd app && python3 -m http.server 8080
```

The app requires microphone access (`getUserMedia`) and loads two CDN scripts: `sweetalert2@9` and `aubiojs@0.1.1`.

## Architecture

Vanilla JS, no framework, no bundler. Five constructor-style classes wired together in `app.js`:

- **`Tuner`** (`tuner.js`) — Core engine. Wraps Web Audio API (`AudioContext`, `AnalyserNode`, `ScriptProcessorNode`) and the aubio.js pitch detector. `Tuner.init()` opens the audio context and starts the microphone; `onNoteDetected` callback fires on each detected pitch. Provides `play(frequency)` / `stopOscillator()` for manual note playback. Key math: `getNote(freq)` converts Hz → MIDI note number; `getCents(freq, note)` returns deviation in cents.

- **`Notes`** (`notes.js`) — Renders the horizontal scrollable note grid (octaves 1–8). Has two modes: **auto** (scrolls to detected note) and **manual** (click a note to play it via `Tuner.play()`). `toggleAutoMode()` switches between modes and stops any playing oscillator when switching to auto.

- **`Meter`** (`meter.js`) — SVG-like pointer that rotates ±45° to show cents deviation. `update(deg)` receives a value pre-scaled from cents to degrees by `Application.update()`: `(cents / 50) * 45`.

- **`FrequencyBars`** (`frequency-bars.js`) — Canvas histogram of the low-frequency spectrum (first 64 FFT bins). Updated via `requestAnimationFrame` loop in `Application`.

- **`Application`** (`app.js`) — Orchestrator. Handles A4 tuning reference (persisted in `localStorage`), wires up the `onNoteDetected` callback with a one-repeat filter (note must be detected twice in a row before updating the display), and drives the animation loop.

## Key Behaviours

- A4 reference frequency is stored in `localStorage` under the key `"a4"` and defaults to 440 Hz.
- Auto mode requires the same note name detected on two consecutive callbacks before updating the display (debounce via `this.lastNote`).
- Switching to manual mode clears the active note and stops the oscillator.
- The frequency bars canvas is sized once at startup (`document.body.clientWidth/Height`) and not resized on window resize.
