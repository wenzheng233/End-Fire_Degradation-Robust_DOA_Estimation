# End-Fire Degradation-Robust DOA Estimation

This repository contains MATLAB code, real-room recordings, and evaluation outputs for **"End-Fire Degradation-Robust DOA Estimation for Compact Linear Microphone Arrays."** The work studies why compact uniform linear arrays (ULAs) lose direction-of-arrival (DOA) accuracy near end-fire and evaluates two training-free methods designed to reduce that degradation.

## Overview

For a far-field source and an array aligned with the x-axis, the pairwise time-difference-of-arrival (TDOA) is proportional to `cos(theta)`. Recovering azimuth requires the nonlinear inverse `theta = arccos(c * tau / d)`, whose sensitivity grows without bound as `theta` approaches 0 or 180 degrees. Small delay errors can therefore cause large angle errors near end-fire.

The paper proposes two complementary methods:

- **W-SRP-PHAT** adds baseline-length, frequency, and inter-channel-coherence weights to a wideband SRP-PHAT score. The implementation uses a 0.2-degree scan grid and energy-based frame selection.
- **GCC-WLS** refines pairwise GCC-PHAT delays with 16x oversampling, restricts delays to physically feasible values, and fuses normalized delays in the linear cosine domain before applying `arccos` once.

The MATLAB evaluation also includes **SRP-MVDR** and **SRP-PHAT** baselines. All methods are training-free and do not require model checkpoints.

## Experimental setup and paper results

The recordings were made in a reverberant indoor room with a four-microphone linear array. Microphone spacing is 3.5 cm (10.5 cm total aperture); the audio is evaluated at 16 kHz for source distances of 1 m and 2 m. Azimuths span 20 to 160 degrees in 10-degree steps. The end-fire region is defined as 20-40 degrees and 140-160 degrees.

The paper reports mean RMSE over all azimuths (`E_all`), mean RMSE in end-fire regions (`E_EF`), Cauchy soft accuracy in end-fire (`S-ACC_EF`), and the degraded angular span:

| Distance | Method | E_all (deg) | E_EF (deg) | S-ACC_EF | Degraded span (deg) |
| --- | --- | ---: | ---: | ---: | ---: |
| 1 m | SRP-MVDR | 5.31 | 9.01 | 0.51 | 30 |
| 1 m | SRP-PHAT | 3.31 | 4.83 | 0.65 | 30 |
| 1 m | W-SRP-PHAT | 2.45 | 3.18 | 0.82 | 0 |
| 1 m | GCC-WLS | 2.43 | 3.14 | 0.82 | 0 |
| 2 m | SRP-MVDR | 7.53 | 12.34 | 0.48 | 50 |
| 2 m | SRP-PHAT | 5.38 | 7.71 | 0.47 | 40 |
| 2 m | W-SRP-PHAT | 3.71 | 4.46 | 0.70 | 0 |
| 2 m | GCC-WLS | 4.01 | 4.99 | 0.66 | 0 |

These are the paper's metrics, not values recomputed from the CSV files in this repository. `measure.m` currently writes per-angle MAE/RMSE and binary accuracy using `abs(error) < 6 deg`; it does not compute the paper's Cauchy `S-ACC@5 deg` or the aggregate metrics above.

## Repository layout

```text
.
|-- measure.m                       # Batch DOA estimation and per-angle metrics
|-- draw.m                          # Plot RMSE, accuracy, and error distributions
|-- function/                       # STFT, inverse STFT, and synchronization helpers
|-- dataset/
|   |-- wav_split_1m/                # 1 m multichannel WAV recordings
|   `-- wav_split_2m/                # 2 m multichannel WAV recordings
|-- csv/                            # Per-angle metrics and per-recording estimates
`-- mat/                            # MATLAB result tables used by plotting
```

WAV filenames encode the source angle and distance, for example `100d1m_001.wav`. The estimator parses the angle from the filename. Each WAV must contain at least four microphone channels.

## Requirements

- MATLAB (the scripts use MATLAB audio, signal-processing, and statistics functions such as `audioread`, `resample`, `stft`, `kaiser`, `quantile`, and `table`).
- The repository's `function/` folder. `measure.m` currently calls `addpath("func")`, although the checked-in folder is named `function`; see the run instructions below.

## Run the evaluation

1. Open `measure.m` and set `audio_dir` to either `dataset/wav_split_1m` or `dataset/wav_split_2m` using an absolute path. Set `out_dir` to a writable output directory.
2. Correct the helper path near the top of `measure.m` to match this repository:

   ```matlab
   addpath("function");
   ```

3. In MATLAB, change the current folder to the repository root and run:

   ```matlab
   measure
   ```

The script resamples audio to 16 kHz when needed, uses the first four channels and first three seconds, estimates DOA with all four methods, and writes `DOA_results_per_file.csv` and `DOA_metrics_by_angle.csv` to `out_dir`.

To generate plots from a result MAT file, update `mat_file_all` near the top of `draw.m` to its path, then run `draw` from MATLAB. The checked-in `draw.m` also references a machine-specific path and expects a MAT file containing `T1` or `T2`.

## Included data and results

The `dataset/` folder contains the multichannel WAV segments for the 1 m and 2 m conditions. The `csv/` folder contains the checked-in per-angle metric tables and per-recording estimates; `mat/` contains MATLAB result tables. These files let you inspect the supplied results without running the estimator. Running `measure.m` recomputes CSV outputs from the WAV data.

## Citation

If you use this work, please cite the paper:

```text
Zheng Wen, Huayang Wang, Zhongxin Bai, Xin Guo, Gongping Huang, Qiqi Tong, and Yaling Zhang,
"End-Fire Degradation-Robust DOA Estimation for Compact Linear Microphone Arrays,"
Interspeech 2026.
```

## Acknowledgments

This work was conducted at the IASP Lab, School of Electronic Information, Wuhan University. The authors acknowledge Guangdong Murora Intelligent Lighting Co., Ltd. for providing the recording equipment.

## License and data use

No license file is currently included in this repository. Please contact the authors regarding code reuse and redistribution of the recordings until a license and dataset terms are specified.
