# Subliminal Audio

Shift an audio recording (typically speech) into the near-ultrasonic 14–20 kHz band, and recover it again.

The processing is a Python port of a Nyquist (Audacity) subliminal-audio plugin, built on `numpy` and `scipy`.

## How it works

**`modulate.py`**

1. High-pass at 80 Hz.
2. Low-pass at `cutoff`, applied 10 times for a sharper roll-off.
3. Multiply by `2 × sin(2π · carrier · t)`, with a carrier of 17.5 kHz by default.
4. High-pass at the carrier frequency, applied 10 times. This keeps only the upper sideband.
5. Normalize and write a 16-bit WAV.

**`demodulate.py`**

1. Multiply by the same carrier, which shifts the signal back to baseband.
2. Low-pass at `cutoff`, applied 10 times.
3. Remove DC and divide by 2.
4. High-pass at 80 Hz.
5. Normalize and write a 16-bit WAV.

The cutoff is `min(carrier / 2, sample_rate / 2 − carrier)`, which keeps the signal clear of aliasing. It also limits the bandwidth of the audio that survives:

| Sample rate | Carrier  | Cutoff  |
| ----------- | -------- | ------- |
| 44.1 kHz    | 17.5 kHz | 4550 Hz |
| 48 kHz      | 17.5 kHz | 6500 Hz |

## Requirements

- Python 3
- `numpy` and `scipy`

```sh
pip install numpy scipy
```

## Usage

Put a WAV file named `input.wav` in the project directory, then run:

```sh
python modulate.py      # input.wav -> output_subliminal.wav
python demodulate.py    # output_subliminal.wav -> demodulated_output.wav
```

`*.wav` files are gitignored.

## Configuration

`carrier_freq` is set near the top of the processing section in both scripts. It is clamped to 14000–20000 Hz and **must be identical in both scripts**. The input and output file names are also hardcoded in the scripts.

## Limitations

- The sample rate must be at least 44.1 kHz. `modulate.py` raises an error otherwise.
- Input must be an int16 or float32 WAV. Other sample formats (for example int32 or uint8) are not converted.
- Stereo input is downmixed to mono.
- The round trip is lossy: recovered audio is band-limited to `cutoff`.
- Many speakers and lossy codecs (MP3, AAC) attenuate or remove content above roughly 16 kHz, so the track may not survive playback or compression.

## License

[MIT](LICENSE) © Rockberpro
