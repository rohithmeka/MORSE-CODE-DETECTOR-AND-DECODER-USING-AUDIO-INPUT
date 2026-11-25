# Morse Code Detector and Decoder Using Audio Input

**A Python DSP pipeline that listens to a Morse code recording (.wav) and turns it into readable text, adapting automatically to the tone frequency, signal level and transmission speed of each recording.**

## Why this project

Decoding Morse by table lookup is trivial once you have dots and dashes. The hard part is getting dots and dashes out of real audio:

- recordings contain background noise
- signal amplitude differs from one recording to another
- senders transmit at different speeds (WPM)
- human-sent timing is never perfectly regular

A decoder with fixed thresholds and a fixed speed breaks on most real recordings. This project estimates the threshold and the timing unit **from each recording itself**.

---

## Pipeline

```text
WAV input
   │
   ▼
Preprocessing ──── mono conversion, normalization
   │
   ▼
Band-pass filter ── centred on the Morse tone frequency
   │
   ▼
Envelope extraction
   │
   ▼
Dynamic thresholding ── tone ON / OFF signal
   │
   ▼
Tone segmentation ── ON and OFF durations
   │
   ▼
Speed estimation ── timing unit (dot length) and WPM
   │
   ▼
Dot / dash + gap classification
   │
   ▼
Morse sequence ──► lookup table ──► decoded text
```

---

## How each stage works

### 1. Preprocessing
The WAV file is read, converted to mono if it is stereo, and amplitude-normalized so that later thresholds do not depend on recording volume.

### 2. Band-pass filtering
Morse audio is a single tone (typically 500–1000 Hz). The dominant tone frequency is located <!-- e.g. from the FFT peak -->, and a band-pass filter <!-- type/order, e.g. 4th-order Butterworth, ±100 Hz --> is applied around it to remove out-of-band noise.

### 3. Envelope extraction
The filtered signal is converted into a smooth amplitude envelope <!-- e.g. rectification + moving-average low-pass, or Hilbert transform -->, so each tone burst becomes a single raised block.

### 4. Dynamic thresholding
Instead of a hard-coded level, the ON/OFF threshold is computed from the envelope of the current recording:

```text
<!-- write your exact rule, e.g. threshold = (mean(envelope) + max(envelope)) / 2 -->
```

Samples above the threshold are marked **tone ON**, the rest **tone OFF**.

### 5. Tone segmentation
Consecutive ON and OFF samples are grouped into segments, and the duration of each segment is measured in seconds.

### 6. Speed estimation
The basic Morse timing unit (one dot length) is estimated from the ON durations <!-- e.g. shortest-cluster mean / histogram / k-means with k = 2 -->. Transmission speed follows from the standard PARIS relation:

```text
unit time (s) = 1.2 / WPM
```

### 7. Symbol and gap classification
Every segment is expressed in units and classified against the standard Morse timing ratios:

| Segment | Standard length | Decision rule used |
|---|---|---|
| Dot | 1 unit ON | ON < 2 units |
| Dash | 3 units ON | ON ≥ 2 units |
| Gap within a character | 1 unit OFF | OFF < 2 units |
| Gap between characters | 3 units OFF | 2 ≤ OFF < 5 units |
| Gap between words | 7 units OFF | OFF ≥ 5 units |

<!-- update the decision rules if your code uses different boundaries -->

### 8. Decoding
The dot/dash groups are mapped to letters and digits with a standard International Morse lookup table, and word gaps become spaces.

---

## Example

**Input:** `samples/sos_20wpm.wav` <!-- replace with your file -->

```text
Detected tone frequency : 700 Hz
Estimated unit          : 0.060 s  (≈ 20 WPM)
Morse sequence          : ... --- ...
Decoded text            : SOS
```
<!-- paste a real run here -->

<!-- Add a figure: raw waveform, filtered envelope with threshold line, detected ON segments
![Signal stages](docs/signal_stages.png) -->

---

## Results

| Test file | Speed (WPM) | Noise | Expected | Decoded | Character accuracy |
|---|---|---|---|---|---|
| <!-- file --> | | Clean | | | |
| | | Added noise (SNR __ dB) | | | |
| | | Different speed | | | |

<!-- even 5–6 rows make the project far more convincing -->

---

## Installation

```bash
git clone https://github.com/rohithmeka/<repo-name>.git
cd <repo-name>
pip install -r requirements.txt
```

**Requirements:** Python 3.x, NumPy, SciPy, Matplotlib <!-- adjust to your imports -->

## Usage

```bash
python main.py path/to/morse_audio.wav
```
<!-- change to your actual entry point / arguments -->

## Project structure

```text
<repo-name>/
├── main.py            # entry point
├── <!-- module files -->
├── samples/           # test .wav files
├── docs/              # figures used in this README
├── requirements.txt
└── README.md
```

---

## Limitations

- Works on recorded WAV files only; no live microphone input yet.
- Assumes a single Morse tone; overlapping signals or strong tones near the same frequency will confuse the filter.
- Very irregular hand-sent timing can push a gap or a dash across a decision boundary.
- Accuracy drops at low SNR, where the envelope no longer separates cleanly into ON and OFF.

## Future scope

- Real-time decoding from a microphone or SDR input
- Spectrogram display alongside the decoded text
- Speed tracking *within* a recording for senders who change pace
- Systematic evaluation over a noisy test set (character error rate vs SNR)

---

## Author

**Rohith Meka**
B.Tech, Artificial Intelligence & Data Science, Amrita Vishwa Vidyapeetham
GitHub: [@rohithmeka](https://github.com/rohithmeka)

## License

MIT
