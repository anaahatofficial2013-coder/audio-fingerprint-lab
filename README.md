# The dots behind Shazam

**Live demo:** *add your GitHub Pages link here before submitting*

A browser app that shows the one thing Shazam actually does: it does **not** listen to the melody. It finds a handful of loud points in the sound, and remembers each *pair* of points by their two frequencies and the time gap between them. That small shape is a hash, and it survives noise even when the waveform itself does not.

## Try it (15 seconds)

1. Open the link. No sign-in, no install, no API key.
2. Use the four suggested tracks at the top, or play any track in the database near your mic.
3. Watch the sound picture: amber **dots** appear (loud peaks). Amber **lines** join pairs of dots — those are hashes.
4. When a line matches one stored in the library, the matching card lights up and the song name appears.
5. Push the noise slider up. Some dots vanish, but the matching lines keep appearing. That is the point.

The microphone button is optional — it lets you try your own voice or music instead.

## The mechanism, in the code

Everything below is written from scratch in `index.html`. No library does the explaining.

- **FFT** — a radix-2 Cooley–Tukey transform (`spectrum()`) turns 2048 sound samples into a frequency spectrum. The same function runs on live microphone samples and on stored audio, so the two are directly comparable.
- **Peak picking** — `pickPeaks()` keeps the eight strongest local maxima above an adaptive threshold in the 0–4 kHz band.
- **Hashing** — `computeHash()` packs each pair of peaks into a 19-bit value: 7 bits per frequency and 5 bits for the time gap. Displayed as five hex characters.
- **Database** — `buildHashesFromPCM()` reads the decoded audio samples directly and builds the same hashes offline, instantly. No 15-second training wait, no microphone noise baked into the reference.
- **Matching** — a song wins when a rolling three-second window contains aligned votes from at least six hashes, four distinct hashes and three time buckets. Only one card can light up at a time.
- **Noise** — a generated noise buffer is mixed into the analyser input, so you can hear and see the fingerprint degrade and still hold.

## What is not mine

Web Audio API handles decoding, playback and raw sample capture. Canvas draws the visuals. Everything that constitutes the fingerprint — FFT, peak selection, hash construction, database creation and matching — is original code in this single file.

## Files

- `index.html` — the whole app: HTML, CSS, JavaScript and 51 prebuilt song fingerprints. No backend, no data API and no sign-in.
