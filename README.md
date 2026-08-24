# Blind Narrowband SETI Detection Under Plasma-Like Signal Distortions

This project studies how a conventional straight-line narrowband search loses
detection efficiency when a synthetic extraterrestrial radio signal is
distorted by plasma-like propagation effects.

The simulation is deliberately phenomenological. It is a complex-baseband
numerical experiment, not a full electromagnetic plasma model and not a
`turboSETI` pipeline. Its purpose is to isolate three failure modes of a blind
linear-drift detector:

1. slow displacement of the signal centroid (**frequency wander**),
2. redistribution of signal power across frequency (**spectral broadening**),
3. redistribution of signal power across time (**scintillation**).

A fourth experiment applies wander and broadening simultaneously and measures
their two-dimensional detection-efficiency surface.

## Main result

At fixed signal amplitude and a fixed 1% false-positive threshold, all three
distortions reduce the probability of detection. Frequency wander and spectral
broadening are more damaging than scintillation over the ranges tested.

| Experiment | Physical response metric | 90% detection point | 50% detection point | Lowest measured detection rate |
|---|---:|---:|---:|---:|
| Centroid wander | wander RMS [Hz] | 0.0906 Hz | 0.7762 Hz | 28% |
| Spectral broadening | effective RMS width [Hz] | 0.5382 Hz | 0.8515 Hz | 21% |
| Scintillation | modulation index (dimensionless) | 2.872 | not reached | 61% |

These response metrics are physically different and must not be compared by
their numerical magnitudes alone. Wander RMS measures track displacement,
effective width measures spectral power spread around a track, and modulation
index measures temporal intensity variability.

The combined wander × broadening experiment is broadly consistent with a
multiplicative independent-effects reference. It also shows localized excess
losses of 7–12 percentage points, especially at strong wander and small-to-
moderate broadening. Because many grid cells were inspected and no multiple-
comparison correction was applied, these local deviations are evidence for
follow-up rather than a general interaction law.

## Simulation and detector

Each realization contains a 120 s complex-baseband observation sampled at
256 Hz. The injected signal has a deterministic drift rate of 0.28 Hz/s and
may also contain correlated centroid wander, fast frequency fluctuations,
lognormal scintillation, a scattering halo, multipath, and receiver noise.

`detector.py` receives only an STFT power array and its observed time and
frequency axes. It never receives the injected track, drift rate, distortion
parameters, seed, or signal-presence flag. For every trial track

```text
f(t) = f0 + drift_rate * t
```

the detector sums the nearest-bin power across all time columns and returns the
largest score. Tracks that leave the observed frequency range are discarded so
that all valid scores contain the same number of samples.

The fixed search grid is:

| Parameter | Range | Step | Trials |
|---|---:|---:|---:|
| Starting frequency `f0` | -24 to +22 Hz | 0.25 Hz | 185 |
| Drift rate | -1 to +1 Hz/s | 0.01 Hz/s | 201 |

This gives 37,185 nominal parameter combinations before invalid edge-crossing
tracks are removed.

### Threshold calibration

The complete blind search was run on 1,000 independent noise-only
realizations. The threshold was frozen at the 99th percentile of the resulting
distribution of full-search maximum scores:

```text
fixed threshold = 156666.25361187334
empirical false-positive rate = 10 / 1000 = 1%
signal amplitude = 0.034
```

Calibrating the distribution of search maxima, rather than individual track
scores, incorporates the look-elsewhere effect of searching many trial tracks.
Detection is defined strictly as `best_score > fixed_threshold`. The threshold
and signal amplitude were not retuned after the signal-present sweeps were
examined.

## Experiments and results

### 1. Blind-detector sanity check

The initial 20-seed paired check compared a signal without slow centroid wander
against the original wander model. Receiver noise and all other stochastic
components were held common within each paired seed.

| Condition | Mean normalized maximum score | Standard deviation | Drift-rate RMSE |
|---|---:|---:|---:|
| No slow wander | 20,426.8 | 5,486.8 | 0.0000 Hz/s |
| Original slow wander | 9,717.9 | 3,296.1 | 0.01245 Hz/s |

The normalized statistic is a diagnostic normalization based on only 20
noise-only runs; it is not a universal Gaussian-significance value. The later
efficiency experiments use the independently frozen 1,000-run threshold.

![Sanity-check score distributions](../../codex-chatgpt-perplexity-perplexity-1-2/sanity_outputs/detection_statistic_distribution.png)

### 2. Centroid-wander response

Only the slow centroid-wander multiplier was changed. One hundred paired seeds
were evaluated at each scale.

| Wander scale | Mean wander RMS [Hz] | Detections | Detection rate |
|---:|---:|---:|---:|
| 0.00 | 0.000 | 96/100 | 96% |
| 0.25 | 0.136 | 87/100 | 87% |
| 0.50 | 0.272 | 77/100 | 77% |
| 0.75 | 0.408 | 70/100 | 70% |
| 1.00 | 0.543 | 56/100 | 56% |
| 1.50 | 0.815 | 49/100 | 49% |
| 2.00 | 1.087 | 42/100 | 42% |
| 3.00 | 1.630 | 28/100 | 28% |

The interpolated 90% and 50% detection points are 0.0906 Hz RMS and
0.7762 Hz RMS, respectively.

![Detection efficiency versus wander RMS](../../codex-chatgpt-perplexity-perplexity-1-2/wander_efficiency_outputs/detection_efficiency_vs_wander_rms_Hz.png)

### 3. Spectral-broadening response

Slow centroid wander was disabled. Independent zero-mean phase modulation
spread the signal across frequency while preserving injected component power.
The truth-side effective width is the clean-signal STFT's power-weighted RMS
about the deterministic track within ±4 Hz; it is measured only after the blind
search returns.

| Broadening scale | Mean effective width [Hz] | Detections | Detection rate |
|---:|---:|---:|---:|
| 0.00 | 0.460 | 94/100 | 94% |
| 0.05 | 0.462 | 95/100 | 95% |
| 0.10 | 0.466 | 93/100 | 93% |
| 0.20 | 0.485 | 92/100 | 92% |
| 0.30 | 0.515 | 93/100 | 93% |
| 0.50 | 0.600 | 82/100 | 82% |
| 0.75 | 0.736 | 66/100 | 66% |
| 1.00 | 0.887 | 45/100 | 45% |
| 1.50 | 1.193 | 21/100 | 21% |

The interpolated 90% and 50% points are effective widths of 0.5382 Hz and
0.8515 Hz. Total injected component power was constant across conditions, so
the loss is attributable to power spreading across frequency bins rather than
to a weaker injected signal.

![Detection efficiency versus effective spectral width](../../codex-chatgpt-perplexity-perplexity-1-2/broadening_efficiency_outputs/detection_efficiency_vs_effective_width_Hz.png)

### 4. Scintillation response

Slow wander and additional broadening were disabled. Only the strength of the
existing lognormal amplitude process was changed. For each paired seed, every
condition was normalized to the same mean carrier intensity as the original
strength-1 realization.

| Scintillation strength | Mean modulation index | Detections | Detection rate |
|---:|---:|---:|---:|
| 0.0 | 0.000 | 99/100 | 99% |
| 0.5 | 0.616 | 97/100 | 97% |
| 1.0 | 1.318 | 96/100 | 96% |
| 2.0 | 2.872 | 90/100 | 90% |
| 5.0 | 6.379 | 80/100 | 80% |
| 10.0 | 9.496 | 74/100 | 74% |
| 100.0 | 19.381 | 65/100 | 65% |
| 1000.0 | 22.157 | 61/100 | 61% |

The measured 90% point is a modulation index of 2.872. The response never
reached 50%, even at strength 1000, so no out-of-range extrapolation is
reported. The nonzero floor may reflect the ability of a few exceptionally
bright time intervals to sustain the summed track score.

![Detection efficiency versus modulation index](../../codex-chatgpt-perplexity-perplexity-1-2/scintillation_efficiency_outputs/detection_efficiency_vs_modulation_index.png)

### 5. Wander × broadening surface

The two strongest single-effect failure modes were then applied together on a
7 × 8 grid. All 56 cells first received the same 30 paired seeds. All 14 axis
cells, cells near the 90% and 50% boundaries, and the largest positive and
negative pilot residual candidates were extended to 100 seeds. This produced
4,060 runs: 34 cells with 100 trials and 22 cells with 30 trials.

| Wander scale \ broadening [Hz] | 0 | 0.1 | 0.2 | 0.3 | 0.5 | 0.75 | 1.0 | 1.5 |
|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| 0.00 | .95 | .98 | .97 | .95 | .84 | .66 | .47 | .21 |
| 0.25 | .93 | .91 | .90 | .86 | .82 | .767 | .60 | .267 |
| 0.50 | .83 | .82 | .76 | .76 | .767 | .733 | .48 | .30 |
| 0.75 | .69 | .72 | .767 | .767 | .767 | .633 | .37 | .40 |
| 1.00 | .63 | .667 | .633 | .667 | .57 | .60 | .40 | .133 |
| 1.50 | .45 | .45 | .48 | .50 | .43 | .31 | .333 | .167 |
| 2.00 | .43 | .34 | .38 | .31 | .29 | .40 | .233 | .233 |

The table entries are measured detection probabilities. Exact trial counts and
95% Wilson intervals are in
`wander_broadening_surface_outputs/wander_broadening_cell_summary.csv`.

The approximate measured contour ranges are:

| Contour | Effective width [Hz] | Wander RMS [Hz] |
|---|---:|---:|
| 90% | 0.464–0.554 | 0–0.177 |
| 50% | 0.513–0.983 | 0–0.817 |

Contours use linear interpolation only between adjacent measured cells, with no
extrapolation outside the grid.

![Two-dimensional detection surface](../../codex-chatgpt-perplexity-perplexity-1-2/wander_broadening_surface_outputs/detection_probability_heatmap_with_contours.png)

For a descriptive independence reference,

```text
P_expected = P_w_axis * P_b_axis / P0_axis
interaction residual = P_observed - P_expected
```

was evaluated on common paired-seed subsets. This is a reference model, not a
physical law. Uncertainty was estimated with a 5,000-repetition paired-seed
bootstrap.

The largest negative residual occurred at wander scale 2.0 and broadening
0.3 Hz: observed 0.31, expected 0.43, residual -0.12, with a bootstrap 95%
interval of [-0.202, -0.046]. The largest positive residual was +0.114, but its
95% interval [-0.056, 0.256] included zero.

![Interaction residuals](../../codex-chatgpt-perplexity-perplexity-1-2/wander_broadening_surface_outputs/interaction_residual_heatmap.png)

## Reproducing the experiments

### Requirements

- Python 3.10 or later
- NumPy
- Matplotlib

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install numpy matplotlib
```

Run all commands from this directory.

### Generate the original diagnostic

```powershell
python plasma_wandering_signal.py
```

This writes a three-panel diagnostic PNG and a matching CSV. Use `--output` to
choose another destination.

### Run the detector sanity check

```powershell
python sanity_check.py
```

### Rebuild the fixed threshold and wander response

```powershell
python wander_efficiency.py benchmark
python wander_efficiency.py noise
python wander_efficiency.py calibrate --amplitudes 0.010 0.012 0.014 0.016 0.018 0.022 0.028 0.034
python wander_efficiency.py sweep --signal-amplitude 0.034 --wander-scales 0 0.25 0.5 0.75 1 1.5 2 3
```

The staged workflow prevents signal-present results from influencing the
noise-only threshold.

### Rebuild the broadening and scintillation responses

```powershell
python broadening_efficiency.py pilot
python broadening_efficiency.py sweep --widths 0 0.05 0.1 0.2 0.3 0.5 0.75 1 1.5

python scintillation_efficiency.py pilot
python scintillation_efficiency.py sweep --strengths 0 0.25 0.5 1 1.5 2 3 5 10 30 100 300 1000
```

### Rebuild the adaptive two-dimensional surface

The completed experiment used a 30-seed base surface followed by an adaptive
extension selected from that pilot surface:

```powershell
python wander_broadening_surface.py sweep --runs 30 --workers 8
python finalize_wander_broadening_adaptive.py
```

`finalize_wander_broadening_adaptive.py` expects the base-surface outputs in
`wander_broadening_surface_outputs/`. Runtime depends strongly on CPU count; the
recorded completed run took about 13 min 52 s with eight workers.

## Repository layout

```text
detector.py                              blind straight-line drift search
plasma_wandering_signal.py               synthetic signal and diagnostic plot
sanity_check.py                          paired detector sanity check
wander_efficiency.py                     threshold, calibration, and wander sweep
broadening_efficiency.py                 spectral-width sweep
scintillation_efficiency.py              temporal-intensity sweep
wander_broadening_surface.py             two-dimensional base experiment
finalize_wander_broadening_adaptive.py    adaptive extension and final products

sanity_outputs/                           sanity-check tables and figures
wander_efficiency_outputs/                threshold and wander results
broadening_efficiency_outputs/            broadening results
scintillation_efficiency_outputs/          scintillation results
wander_broadening_surface_outputs/         2-D runs, tables, figures, and metadata
```

Each experiment directory contains machine-readable CSV and JSON files in
addition to presentation figures. The JSON metadata records thresholds,
amplitudes, trial counts, confidence procedures, runtime, and metric
definitions.

## Reproducibility and audit checks

- `detector.py` remained unchanged during the response experiments.
- Recorded detector SHA-256:
  `7CD4AC1D032336A246120D3956A87F6929C31412DECA33A9316F3AD29440F68C`.
- All final detections satisfy `best_score > 156666.25361187334`.
- The signal amplitude is fixed at `0.034`.
- Truth-derived wander, width, and drift-error values are attached only after
  the blind search returns.
- Paired designs reuse seeds across conditions to reduce irrelevant Monte Carlo
  variation.
- Injected power is constant across the paired single-effect conditions to
  numerical precision.
- Both axes of the 2-D experiment reproduce the earlier one-dimensional results
  within 95% sampling uncertainty; the largest absolute rate difference is
  seven percentage points.

## Interpretation and limitations

The results show a detector-model mismatch: a straight, one-bin-wide integrator
loses score when signal power moves away from its trial line in frequency or is
concentrated into intermittent time intervals. They do not establish the
prevalence or magnitude of these distortions in real extraterrestrial links.

Important limitations include:

- phenomenological OU-process distortions rather than a first-principles plasma
  propagation calculation;
- one observation duration, sampling rate, STFT configuration, detector grid,
  signal amplitude, and noise model;
- a simple nearest-bin linear-track detector rather than a complete survey
  pipeline;
- finite Monte Carlo samples, including 30-trial cells in the adaptive 2-D
  surface;
- pointwise interaction intervals without correction for searching many cells;
- interpolated crossings only inside the measured range.

Accordingly, the quantitative crossing points are properties of this simulated
setup. The robust qualitative conclusion is that fixed-power narrowband signals
can become substantially harder for a blind straight-line search to detect when
their frequency centroid wanders or their power spreads across frequency, and
that strong temporal scintillation also reduces efficiency, though less sharply
over the tested range.
