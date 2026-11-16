# mmWave Radar Gesture Configuration & Signal Processing Benchmarker

[![Python 3.8+](https://img.shields.io/badge/python-3.8%2B-blue.svg)](https://www.python.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/mit.svg)

An empirical benchmarking and signal processing framework for evaluating TI mmWave radar (AWR1843/DCA1000) TDM-MIMO chirp configurations for micro-gesture recognition (Palm Swipe, Finger Click). 

This repository provides an end-to-end pipeline to ingest raw LVDS binary radar data, parse complex data layouts, perform static clutter removal, extract frame-level micro-Doppler maps and sliding-window STFTs, and quantitatively compare trade-offs between frame rates, duty cycles, velocity resolution, and high-velocity transient smearing.

---

## 🚀 Key Features

* **DCA1000 Data Ingestion Engine:** Parses raw binary files correctly handling RX-major block interleave layouts.
* **Signal Processing Pipeline:**
  * Static clutter / background removal via median-absolute-deviation (MAD) filtering.
  * 2D Range-Doppler FFT generation ($N_{\text{loops}} = 128$, $N_{\text{samples}} = 256$).
  * Intra-frame sliding Short-Time Fourier Transform (STFT) for high-resolution micro-Doppler spectrograms.
* **Automated Metric Extraction:** Computes gesture span, RMS speed, $v_{99}$ velocity percentile, peak-to-median SNR, and envelope energy spectra against Nyquist limits.
* **Comparative Visualization Suite:** Generates side-by-side comparative plots for multi-fps configurations (Old 30 fps vs. New 20 fps vs. Proposed Optimal 40 fps).

---

## 📂 Repository Structure

```text
mmwave-gesture-config-benchmarker/
│
├── data/                    # Expected directory layout for trial binaries
│   ├── oldconfig/           # 30 fps (10 µs idle, 33.33 ms period)
│   ├── newconfig/           # 20 fps (70 µs idle, 50.00 ms period)
│   └── optconfig/           # 40 fps (10 µs idle, 25.00 ms period)
│
├── src/
│   ├── loader.py            # DCA1000 binary parsing & RX-major reordering
│   ├── processor.py         # Clutter removal, Range-Doppler FFT, STFT
│   └── metrics.py           # RMS speed, v99, and spectral energy analysis
│
├── notebooks/
│   └── config_comparison.ipynb # Full exploratory analysis & plotting notebook
│
├── assets/                  # Generated figures for README & report
├── requirements.txt         # Python dependencies (numpy, pandas, matplotlib)
└── README.md
```

---

## 🛠️ Getting Started & Installation

1. **Clone the repository:**
   ```bash
   git clone https://github.com/your-username/mmwave-gesture-config-benchmarker.git
   cd mmwave-gesture-config-benchmarker
   ```

2. **Install dependencies:**
   ```bash
   pip install -r requirements.txt
   ```

3. **Run the analysis notebook:**
   Open `notebooks/config_comparison.ipynb` in Google Colab or Jupyter Lab, point `DATA_ROOT` to your trial dataset directory, and execute cells to reproduce all comparative figures and quantitative tables.

---

## 📊 Summary of Findings: Why Configuration Choice Matters

| Metric / Parameter | Old Config (30 fps) | New Config (20 fps) | Proposed Optimal (40 fps) |
| :--- | :--- | :--- | :--- |
| **Chirp Idle Time / PRI** | 10 µs / 180 µs | 70 µs / 360 µs | 10 µs / 180 µs |
| **Active Integration Window** | 23.0 ms | 46.1 ms | 23.0 ms |
| **Duty Cycle** | 69% (10.3 ms gap) | 92% (3.9 ms gap) | **92% (2.0 ms gap)** |
| **Finger Click Transient SNR** | **High (−5 dB plateau)** | Low (−15 to −22 dB smear) | **High (−5 dB plateau)** |
| **Envelope Nyquist Limit** | 15.0 Hz | 10.0 Hz | **20.0 Hz** |

* **The Core Trade-off:** Increasing idle time to 70 µs achieved a 92% duty cycle in the New Config but doubled the active frame window to 46.1 ms. This caused severe **Doppler smearing** during impulsive $15\text{–}25\text{ ms}$ finger clicks, suppressing high-velocity transient contrast by $10\text{–}15\text{ dB}$.
* **The Optimal Fix:** Keeping idle time at 10 µs while tightening the frame period to 25.0 ms (40 fps) preserves the crisp 23.0 ms integration window, eliminates high-velocity smearing, and achieves a 92% duty cycle with minimal dead time.

---

## 💼 Resume Bullet Points (Copy & Adapt)

* **Developed an empirical radar signal processing framework in Python/NumPy** to benchmark TI mmWave AWR1843 chirp configurations, analyzing raw LVDS binary streams (DCA1000) for gesture recognition.
* **Identified and resolved a critical 10–15 dB Doppler smearing bottleneck** in high-velocity transient micro-gestures (finger clicks) caused by active integration window over-extension ($46\text{ ms}$ vs $23\text{ ms}$).
* **Designed an optimized 40 fps TDM-MIMO configuration** achieving a 92% duty cycle and 20 Hz envelope Nyquist limit, eliminating inter-frame blind spots without compromising velocity resolution or transient fidelity.