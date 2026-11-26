# Audio Fingerprint Lab

<img width="2844" height="1530" alt="Screenshot 2026-11-26 090216" src="https://github.com/user-attachments/assets/1a500ec2-aac9-461d-9a51-3f3d7a2f9303" />

<img width="2840" height="1528" alt="Screenshot 2026-11-26 090258" src="https://github.com/user-attachments/assets/71fce8a7-a972-4c9c-a60d-58ed7a895cf0" />

**Live demo:** (https://audio-fingerprint-lab.vercel.app/)

Audio Fingerprint Lab is a static, browser-based demonstration of constellation audio fingerprinting. It turns microphone or uploaded audio into spectral peaks, hashes pairs of peaks, and votes for songs whose stored hashes align at the same time offset.

The project contains 10 prebuilt tracks and can also fingerprint local audio files in the browser. It does not require a backend, API key, account, or build step.

## Quick Start

1. Open the live demo or serve this folder from a local web server.
2. Press the microphone button or `Space` to start listening.
3. Play a featured track or another track from the built-in library near the microphone.
4. Watch the live FFT, fingerprint dots and hash lines update.
5. Wait for the matching card to gain enough aligned votes.
6. Optional: add background noise with the noise slider or add your own local audio files.

Microphone access requires HTTPS or localhost. Press `D` to open the diagnostics panel and `Space` to start or stop listening. Typing `shazam` activates an easter egg.

## Run Locally

No package installation is required. Serve the folder because browser microphone access and several browser APIs require a secure context or local server.

```powers
python -m http.server 8000
```

Then open `http://localhost:8000`.

## Features

- Live FFT spectrum for the 0–4 kHz band
- Constellation peak detection and hash visualization
- 51 built-in song fingerprints
- Local audio upload and fingerprinting
- Top-three match panel with aligned-vote confidence
- Adjustable generated background noise
- Five-second rolling matching window
- Hidden diagnostics and eight-clip self-test
- Guided interface tour
- Responsive desktop and mobile layouts
- Local-only saved user songs

## How It Works

The app uses a static page with original JavaScript in `index.html`.

1. **FFT:** A radix-2 FFT converts short blocks of audio into a frequency spectrum.
2. **Peak picking:** The strongest local peaks in the 0–4 kHz band become constellation anchors.
3. **Hashing:** Each anchor is paired with nearby target peaks. The pair stores two frequencies and the time gap.
4. **Database:** Built-in fingerprints are indexed by hash value. Each hash points to one or more stored timestamps.
5. **Matching:** Query hashes are compared with the database and collect votes in 100 ms time-offset bins.
6. **Decision:** A rolling window looks for strongly aligned, distinct hashes across several time buckets before reporting a match.

The built-in library contains up to 3,000 selected hashes per track, while the generated noise control demonstrates that matching can remain stable as local peaks disappear.

## Project Pages

- `index.html` — the interactive demo, FFT logic, peak detection, hash matching, song database and browser UI
- `index.css` — demo layout, visual panels, responsive rules and animation styles
- `about.html` — explanation of the algorithm, project statistics, database and further reading
- `about.css` — styling for the explanatory page
- `README.md` — project documentation

There is no package manifest, server directory, data API or build output. The built-in fingerprints and interface logic are embedded in `index.html`.

## Resources

### Audio Fingerprinting

- [An Industrial-Strength Audio Search Algorithm — Avery Wang (PDF)](https://www.ee.columbia.edu/~dpwe/papers/Wang03-shazam.pdf) — the original constellation-mapping fingerprinting paper.
- [Shazam algorithm article on Toptal](https://www.toptal.com/algorithms/shazam-it-music-processing-fingerprinting-and-recognition) — an accessible explanation of peaks, hashes and alignment.
- [Fast Fourier transform — Wikipedia](https://en.wikipedia.org/wiki/Fast_Fourier_transform) — background on the transform used by `spectrum()`.
- [Discrete Fourier transform — Wikipedia](https://en.wikipedia.org/wiki/Discrete_Fourier_transform) — the frequency-bin concepts behind an FFT.
- [Spectrogram — Wikipedia](https://en.wikipedia.org/wiki/Spectrogram) — how frequency, time and magnitude are represented.

### JavaScript and Web Audio

- [MDN: Web Audio API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API) — browser audio graphs and real-time processing.
- [MDN: `AnalyserNode`](https://developer.mozilla.org/en-US/docs/Web/API/AnalyserNode) — reading audio frequency data.
- [MDN: `AudioContext.decodeAudioData()`](https://developer.mozilla.org/en-US/docs/Web/API/BaseAudioContext/decodeAudioData) — decoding uploaded audio into PCM.
- [MDN: `MediaDevices.getUserMedia()`](https://developer.mozilla.org/en-US/docs/Web/API/MediaDevices/getUserMedia) — requesting microphone access.
- [MDN: `Map`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map) — the hash-to-timestamp index.
- [MDN: Canvas API](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API) — drawing the spectrum and fingerprint visuals.

## Attribution

Web Audio API handles decoding and real-time audio input. Canvas handles the spectra and visualizations. The FFT, peak selection, hash construction, database logic and match voting are implemented in this project.

Google Fonts is the only external stylesheet loaded by the pages. 
The project is documented as a Global Virtual Hackathon 2026 entry by Team Techrotians Squad.
