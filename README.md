# The dots behind Shazam

**Live demo:** *Add the deployed GitHub Pages link here*

A browser app that shows the core idea behind Shazam: it does **not** listen to the melody. It finds a handful of loud points in the sound, then remembers each pair of points by its two frequencies and the time gap between them. That compact relationship is a fingerprint hash that can survive noise even when the waveform itself does not.

## Try it

1. Open the live demo or run it locally.
2. Use a featured track, or play a track in the database near your microphone.
3. Watch amber dots appear for loud peaks and amber lines appear for hashes.
4. When a line matches the stored library, the matching card lights up.
5. Raise the noise slider. Some dots disappear, but surviving lines can keep matching.

The microphone button is optional. It lets you try a live voice or music recording.

## Project overview

This is a static, client-side JavaScript project. It has no backend, API key, database server or audio-upload service:

- `index.html` contains the application UI and fingerprinting implementation.
- `index.css` styles the interactive demo.
- `about.html` explains the science and contains the project database.
- `about.css` styles the science page.
- Built-in track names and hash indexes are embedded in `index.html`.

There is no npm setup, package manifest, build step, or automated test command. The browser runs the app directly from these files.

User-added fingerprints are stored as hash data in browser `localStorage`, under `shazamSongs9`. Audio files themselves are not uploaded.

## Run locally

1. Install a recent browser such as Chrome, Edge or Firefox.
2. Start a local HTTP server directly (no npm command is needed):

   ```powershell
   cd D:\Ojas\Engineering\GitHub\audio-fingerprint-lab
   python -m http.server 8000
   ```

3. Open `http://localhost:8000`.

Do not rely on `file://` for microphone access. Browsers provide reliable microphone permission for localhost or HTTPS. HTTPS or localhost is required to request a microphone.

## Code walkthrough

Everything that creates, stores and compares fingerprints is written in `index.html`.

### 1. Capture a short FFT window

`startEngine()` creates a Web Audio `AudioContext`, `AnalyserNode`, gain node and mixed noise source. `startMic()` connects a `MediaStreamSource` from `getUserMedia()` to the same analyser.

The application uses a fixed 2048-sample window:

```js
const FFT_N = 2048;
const WIN = new Float32Array(FFT_N); // Hann window
```

`binWidth()` converts FFT bin numbers into frequencies:

```js
frequency = binIndex * (sampleRate / fftSize);
```

`spectrum(samples, offset)` applies a Hann window, performs an in-place radix-2 Cooley–Tukey FFT and returns 1024 magnitude bins. It converts magnitude to decibels, then maps the range into 0–255 for drawing and peak detection.

### 2. Find constellation peaks

`pickPeaks(buf)` processes only bins below 4000 Hz. It calculates an adaptive threshold from the spectrum mean, standard deviation and maximum magnitude:

```js
const mean = sum / count;
const std = Math.sqrt(sumOfSquares / count);
const threshold = Math.max(16, mean + std * 1.05, max * 0.32);
```

A bin becomes a candidate when it is greater than or equal to its neighbours and above the threshold. Parabolic interpolation estimates a peak position between adjacent bins. The eight highest-magnitude peaks are returned as:

```js
{ bin, mag, freq }
```

Almost every waveform frequency is discarded. Only stable local peaks survive.

### 3. Form a pair-based hash

`computeHash(f1, f2, dt)` connects two peaks. It quantises:

- `f1` and `f2` into 7-bit values,
- `dt`, the time between peaks, into 5-bit values.

The 19-bit value is packed and displayed as a five-character hexadecimal string:

```js
return (((q1 << 12) | (q2 << 5) | qd) >>> 0)
  .toString(16)
  .padStart(5, '0')
  .toUpperCase();
```

The anchor-and-target pattern is:

```text
peak 1: frequency f1 at time t1
peak 2: frequency f2 at time t2
hash:  q(f1) | q(f2) | q(t2 - t1)
```

Relative relationships are more robust than an exact waveform comparison. A volume or microphone difference is less likely to change a pair of strong peaks than the complete recording.

### 4. Build each song fingerprint

`fingerprintPass()` is used to turn stored PCM into a library fingerprint. Every 100 ms it:

1. Computes an FFT for the next 2048-sample window.
2. Picks up to eight peaks.
3. Uses each peak as an anchor.
4. Searches later peaks within 0.1–1.2 seconds.
5. Keeps up to five target peaks for each anchor.
6. Maps each hash to a song-relative anchor timestamp.

The indexed structure is a JavaScript `Map`:

```js
hash -> [timestamp1, timestamp2, timestamp3, ...]
```

`buildHashesFromPCM()` trains one library entry from six transformed copies of the track:

- clean audio,
- low noise,
- stronger noise,
- small-speaker low-pass filtering,
- bright high-pass filtering,
- an 18 ms echo.

It keeps 2500 strong noise-trained hashes first, then fills up to 3000 hashes with useful augmented hashes. This variation makes the live query compare well with several plausible recordings.

### 5. Match hashes by time alignment

The live loop creates hashes from microphone peaks. For every query hash, `hashVariants()` checks exact and neighbouring frequency values:

- an exact hash receives a weight of 3,
- a one-step frequency neighbour receives a weight of 1.

For each database hit, the code calculates:

```text
offset = queryAnchorTime - databaseAnchorTime
```

Hashes that refer to the same recording position have a consistent offset. The rolling 3-second voting window counts exact hashes, neighbouring hashes, distinct hash values and offset buckets. It favours one song with repeated aligned evidence rather than many accidental matches.

A hash match is common; a cluster of consistent hash matches is a song match.

### 6. Add noise and compare the result

The noise slider changes a continuous noise gain in the live audio graph. The engine mixes generated noise with the microphone signal before it reaches the analyser.

The noise does not change the matching algorithm. It changes the peaks available to `pickPeaks()`. Because the strongest peaks remain, some anchor-target relationships survive and continue producing hashes.

## Main code symbols

| Symbol | Purpose |
| --- | --- |
| `startEngine()` | Creates the Web Audio graph and built-in song cards |
| `startMic()` | Requests microphone permission and connects it |
| `spectrum()` | Converts PCM samples into frequency magnitudes |
| `pickPeaks()` | Applies the adaptive threshold and finds anchors |
| `computeHash()` | Packs frequency and time-gap values |
| `fingerprintPass()` | Turns PCM data into a hash → timestamps `Map` |
| `buildHashesFromPCM()` | Adds clean and transformed training variants |
| `hashVariants()` | Adds tolerant one-bin hash matches |
| `loop()` | Runs the live visualisation and matching update |
| `stopMic()` | Stops audio resources and resets live state |

Useful browser APIs are `AudioContext`, `AnalyserNode`, `MediaStreamAudioSourceNode`, `decodeAudioData()`, `getUserMedia()`, `Map`, `localStorage`, Canvas and `requestAnimationFrame()`.

## Learning resources

### Fingerprinting and signal processing

- [An Industrial-Strength Audio Search Algorithm — Avery Wang (PDF)](https://www.ee.columbia.edu/~dpwe/papers/Wang03-shazam.pdf) — the original constellation-mapping fingerprinting paper.
- [Shazam algorithm article on Toptal](https://www.toptal.com/algorithms/shazam-it-music-processing-fingerprinting-and-recognition) — a beginner-friendly walk through peaks, hashes and alignment.
- [Shazam application overview — Wikipedia](https://en.wikipedia.org/wiki/Shazam_(application)) — historical and product context.
- [Fast Fourier transform — Wikipedia](https://en.wikipedia.org/wiki/Fast_Fourier_transform) — mathematical motivation and algorithm overview.
- [Discrete Fourier transform — Wikipedia](https://en.wikipedia.org/wiki/Discrete_Fourier_transform) — understand the frequency-bin calculation behind an FFT.
- [Spectrogram — Wikipedia](https://en.wikipedia.org/wiki/Spectrogram) — how frequency, time and magnitude are represented.

### JavaScript and browser audio

- [MDN: Web Audio API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API) — nodes, audio graphs and real-time browser processing.
- [MDN: `AnalyserNode`](https://developer.mozilla.org/en-US/docs/Web/API/AnalyserNode) — how the demo obtains frequency data.
- [MDN: `AudioContext.decodeAudioData()`](https://developer.mozilla.org/en-US/docs/Web/API/BaseAudioContext/decodeAudioData) — converting uploaded audio files into PCM.
- [MDN: `MediaDevices.getUserMedia()`](https://developer.mozilla.org/en-US/docs/Web/API/MediaDevices/getUserMedia) — requesting and connecting the microphone.
- [MDN: `Map`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map) — the hash-to-timestamps index.
- [MDN: Canvas API](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API) — drawing FFT bars, peaks and hash lines.

### Further practice

- Change `4000` in `pickPeaks()` to test another frequency range.
- Change the peak threshold and observe stability.
- Change the 1.2-second target zone or five-target limit.
- Replace generated noise with an uploaded noise file.
- Instrument `fingerprintPass()` to count peaks and hashes for clean versus noisy PCM.
- Add a different fingerprint database and compare hash counts.

## What is not mine

Web Audio API handles decoding, playback and raw sample capture. Canvas draws the visuals. Everything that constitutes the fingerprint—FFT, peak selection, hash construction, database creation and matching—is original code in this project.

## Files

- `index.html` — the interactive app, Web Audio graph, fingerprint engine and built-in library.
- `index.css` — styles for the demo.
- `about.html` — the science, database and further-reading explanation.
- `about.css` — styles for the science page.
- `README.md` — project setup and code guide.
