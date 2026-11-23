# The dots behind Shazam

**Live demo:** https://arush4213.github.io/audio-fingerprint-lab/

A browser app that shows the one thing Shazam actually does: it does **not** listen to the melody. It finds a handful of loud points in the sound, and remembers each *pair* of points by their two frequencies and the time gap between them. That small shape is a hash, and it survives noise even when the waveform itself does not.

## Try it (20 seconds)

1. Open the link. No sign-in, no install, no API key.
2. Play one of the four featured tracks near your mic, or any other track in the database.
3. Watch the sound picture: amber **dots** appear (loud peaks). Amber **lines** join pairs of dots — those are hashes.
4. After about five seconds, the matching card lights up and the song name appears.
5. Push the noise slider up. Some dots vanish, but the matching lines keep appearing. That is the point.

Press `D` for the hidden diagnostics panel: peaks/s, query hashes, database lookups that hit, each song's total collisions and top/second offset bins, and the best-vs-second margin. The panel also has an eight-clip self-test button.

## The mechanism, in the code

The page is static. CSS lives in `index.css` and `about.css`; all fingerprint logic is original JavaScript in `index.html`.

- **FFT** — a radix-2 Cooley–Tukey transform (`spectrum()`) turns 4096 sound samples into a frequency spectrum. A Hann window and 44,100 Hz sampling give 10.77 Hz bins. The same function runs on live mic samples and on stored audio.
- **Peak picking** — `pickPeaks()` keeps the two strongest local maxima per 250 ms frame in the 0–4 kHz band. A peak must clear `mean + 2.5σ` and 32% of the loudest bin, sit at least 50 Hz from another peak, and respect a two-frame cool-down. That works out to roughly six peaks per second.
- **Hashing** — `computeHash()` packs each anchor/target pair into a 19-bit value: 7 bits per frequency and 5 bits for the time gap. `F_STEP = 44100/4096` (nearest FFT bin), `DT_STEP = 0.1` (100 ms), DB fan-out 8, query fan-out 5.
- **Database** — the 51 built-in fingerprints were retrained from 14 variants of each track: clean, noise at 0.02/0.05/0.10/0.18/0.28, low-pass at 0.38/0.60/0.78, high-pass at 0.985/0.96, and echoes at 18 ms/0.32, 35 ms/0.22 and 60 ms/0.14. Interference-aware selection keeps up to 3,000 hashes per song: **131,386 hashes total**, 751–3,000 per song, 2,576 on average.
- **Matching** — hashes vote in 100 ms time-offset bins. Exact hashes count five votes, one-step frequency/time neighbours count one. A rolling five-second window must contain at least five aligned hashes, three distinct hashes and three time buckets, and the top bin must be at least 2× the second-best. Otherwise the result is “No match”.
- **Noise** — a generated noise buffer is mixed into the analyser input, with `echoCancellation`, `noiseSuppression` and `autoGainControl` disabled so the mic stays raw.

## Measured training result

A 306-case benchmark over all 51 songs (two start positions, three noise levels: 0, 0.15, 0.30) was used to choose the final model.

| | Before tuning | Final model |
|---|---:|---:|
| Pass rate | 178/306 | **227/306** |
| Correct top-1 song | 276/306 | **295/306** |
| Quiet / medium / noisy passes | 59 / 57 / 62 | **74 / 78 / 75** |
| Median best-vs-second margin | 2.32× | **2.79×** |
| Peaks/s | 5.1 | 6.0 |
| Hashes in library | 75,020 | 131,386 |

The final peak threshold changed from `2.7σ` to `2.5σ`, the offset bin from `150 ms` to `100 ms`, the exact-hash weight from `3` to `5`, and the training set from 6 to 14 variants. Those changes together lifted the pass rate by 49 cases and the median margin by 0.47×.

## Learning resources

### Fingerprinting and signal processing

- [An Industrial-Strength Audio Search Algorithm — Avery Wang (PDF)](https://www.ee.columbia.edu/~dpwe/papers/Wang03-shazam.pdf) — the original constellation-mapping fingerprinting paper.
- [Shazam algorithm article on Toptal](https://www.toptal.com/algorithms/shazam-it-music-processing-fingerprinting-and-recognition) — an accessible explanation of peaks, hashes and alignment.
- [Fast Fourier transform — Wikipedia](https://en.wikipedia.org/wiki/Fast_Fourier_transform) — background on the transform used by `spectrum()`.
- [Discrete Fourier transform — Wikipedia](https://en.wikipedia.org/wiki/Discrete_Fourier_transform) — the frequency-bin concepts behind an FFT.
- [Spectrogram — Wikipedia](https://en.wikipedia.org/wiki/Spectrogram) — how frequency, time and magnitude are represented.

### JavaScript and browser audio

- [MDN: Web Audio API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API) — browser audio graphs and real-time processing.
- [MDN: `AnalyserNode`](https://developer.mozilla.org/en-US/docs/Web/API/AnalyserNode) — reading audio frequency data.
- [MDN: `AudioContext.decodeAudioData()`](https://developer.mozilla.org/en-US/docs/Web/API/BaseAudioContext/decodeAudioData) — decoding uploaded audio into PCM.
- [MDN: `MediaDevices.getUserMedia()`](https://developer.mozilla.org/en-US/docs/Web/API/MediaDevices/getUserMedia) — requesting microphone access.
- [MDN: `Map`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map) — the hash-to-timestamp index.
- [MDN: Canvas API](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API) — drawing the spectrum and fingerprint visuals.

## What is not mine

Web Audio API handles decoding, playback and raw sample capture. Canvas draws the visuals. Everything that constitutes the fingerprint — FFT, peak selection, hash construction, database creation and matching — is original code in this project. The only external request is the Google Fonts stylesheet.

## Files

- `index.html` — app markup, 51 prebuilt song fingerprints, eight test clips and all fingerprint logic.
- `index.css` — demo styling, including the hidden `D` diagnostics panel.
- `about.html` / `about.css` — the explainer page.
- No backend, no data API, no sign-in, no build step.
