# FRB121102-acf3

**Can a frequency-autocorrelation measurement of scintillation bandwidth, of the kind used by Spitler et al. (2018), be reproduced on public 4-8 GHz Green Bank Telescope data of FRB 121102, and how reliable is the width it returns?**

This repository implements that measurement as a Python pipeline and tests it with injection-recovery calibration and noise controls. The name `acf3` refers to the three envelope-smoothing scales (80, 150 and 300 MHz) used in the analysis.

## Summary of findings

- In the late part of burst 11A (2.7-5.0 ms, centred near 6 GHz), the calibrated frequency-correlation width is **8.45, 10.04 and 11.06 MHz** at the three envelope scales (mean 9.85 MHz).
- The raw ACF width depends on the envelope scale (5.8 to 9.4 MHz). Injection-recovery calibration reduces this dependence, and the calibrated values agree to within the calibration ranges.
- Empty off-pulse windows (0/5) and pure white noise (0/200) never pass the detection rule.
- The width is lower than a naive extrapolation of the Spitler et al. (2018) value to 6 GHz (about 15-16 MHz), by a factor of roughly 1.6. The frequency scaling used for that extrapolation is assumed, not measured here.
- Only 1 of the 5 bursts examined (11A) is bright enough to give a stable width.
- The result is a measured frequency-correlation width consistent in order of magnitude with Galactic scintillation. These data do not by themselves show that the structure is scintillation.

## Research questions

1. What frequency-correlation width does burst 11A show near 6 GHz?
2. How sensitive is the measured width to the choice of spectral envelope scale?
3. How accurately does the pipeline recover a known injected width, and what bias does it have?
4. Is the result consistent with the Spitler et al. (2018) measurement and with expectations for Galactic scintillation?
5. Does the measurement hold across other bursts and time windows in the same data?

## Data

Public Breakthrough Listen observations of FRB 121102 with the Green Bank Telescope at 4-8 GHz (calibrated PSRFITS, Stokes I). The main analysis uses `11A_16sec.calib.4p`, a snapshot of about 20 ms containing burst 11A. Other bursts (11E, 11K, 11C, 11B) were screened for comparison.

Source: [Breakthrough Listen FRB 121102 technical data](https://seti.berkeley.edu/frb121102/technical.html)

The data are not included in this repository (about 322 MB each). See [Reproducing the analysis](#reproducing-the-analysis).

Spitler et al. (2018) measured a scintillation bandwidth with Effelsberg at 4.85 GHz. Those data are not public, so this work does not reproduce their number. It applies the same kind of measurement to an independent, public data set.

## Method

For one burst and one time window:

1. **Read and normalise.** Read Stokes I and normalise each frequency channel by its own off-pulse noise.
2. **Mask.** Mask band edges and channels with unusual noise.
3. **Average.** Average the burst window in time and in groups of 8 frequency channels.
4. **Remove the envelope.** Subtract a smoothed copy of the spectrum (the envelope) so that only fine spectral structure remains. This is done at three smoothing scales: 80, 150 and 300 MHz.
5. **Compute the ACF and fit.** Compute the frequency ACF and fit a Gaussian to lags up to 60 MHz, excluding zero lag. The reported width is the half-width at half-maximum (HWHM).
6. **Apply a detection rule.** Require a fitted amplitude of at least 0.1 and an HWHM of at least 3 MHz at all three envelope scales.

The measured width depends on the envelope scale, so it is calibrated and tested as follows:

7. **Injection-recovery.** Inject scintillation-like patterns of known width into real burst-free spectra from the same file and run the identical pipeline. The injected-versus-recovered curve converts the measured width to a corrected width.
8. **Controls.** Run the same measurement on empty windows and on pure noise, and repeat it with an alternative normalisation.

## Results: burst 11A

Late window (2.7-5.0 ms), centre frequency about 6000 MHz, approximate S/N 34.3, 1817 channels masked.

| Envelope scale | Raw HWHM (MHz) | Fit amplitude | Calibrated width (MHz) | 68% calibration range (MHz) |
|---|---:|---:|---:|---:|
| 80 MHz | 5.8 | 0.31 | 8.45 | 7.66-9.74 |
| 150 MHz | 7.7 | 0.34 | 10.04 | 8.56-12.03 |
| 300 MHz | 9.4 | 0.35 | 11.06 | 9.25-13.16 |

The mean calibrated width is 9.85 MHz, with a range of 8.45-11.06 MHz across scales. The three scales use the same data, so this spread is a robustness range, not an independent statistical confidence interval. The calibration ranges describe the injection-recovery procedure, not the full uncertainty on the astrophysical measurement.

An alternative normalisation gives raw widths of 5.78, 7.74 and 9.38 MHz, very close to the adopted normalisation.

### Figures

**Dynamic spectrum of burst 11A**

![Dynamic spectrum of FRB 121102 burst 11A](frbacf/figures./11A_dynamic_spectrum.png)

**Frequency ACF and Gaussian fits at the three envelope scales.** Black points show the ACF with zero lag omitted. The orange curve is the Gaussian fit (lags up to 60 MHz) and the dotted line marks the fitted HWHM. The negative dip after the central peak is expected to be partly caused by subtracting the smoothed envelope. The broad bump near 85-100 MHz, most visible in panels (b) and (c), is not explained (see Limitations).

![Frequency ACF and Gaussian fits](frbacf/figures./11A_acf_fits.png)

**Injection-recovery calibration.** Injected widths against recovered widths at each envelope scale.

![Injection-recovery calibration](frbacf/figures./11A_injection_recovery.png)

## Validation

### Injection-recovery

Injected patterns of known width were added to real burst-free spectra from the same file.

| Injected width (MHz) | Detection fraction |
|---:|---:|
| 4 | 0.75 |
| 6 | 1.00 |
| 8 | 1.00 |
| 10 | 1.00 |
| 12 | 1.00 |
| 16 | 1.00 |
| 20 | 1.00 |
| 30 | 0.925 |

Recovered widths are generally smaller than injected widths. This bias is what the calibration corrects. The measurement is most reliable for widths of about 6-20 MHz, which includes the measured value.

### Controls

- **Empty windows:** 0 of 5 burst-free windows pass the detection rule.
- **White noise:** 0 of 200 spectra pass (fraction 0.0).
- **Burst 11A:** passes at all three envelope scales.
- **Channel ordering:** the adopted calibration-array ordering gives a bandpass roughness of 0.0587, against 0.4778 for the alternative ordering. This checks how the file is read and is not a scintillation measurement.

These controls support the internal consistency of the analysis. They are not a formal false-alarm probability.

## Comparison with previous work

Spitler et al. (2018) measured a characteristic frequency scale of 6.4 ± 1.6 MHz for FRB 121102 at 4.85 GHz. Scaling with Δν ∝ ν^4 to ν^4.4 gives about 15-16 MHz at 6 GHz (±4 MHz from the original uncertainty alone). The mean calibrated width here, about 9.9 MHz, is of the same order but lower than this extrapolation. The scaling index is an assumption and is not tested by this analysis. The difference may therefore reflect the assumed scaling, a different epoch or line of sight through the scattering medium, or a difference in method.

Gajjar et al. (2018), using GBT observations of FRB 121102 at 4-8 GHz, also reported fine-scale spectral structure at the 10-50 MHz level.

## Other bursts and time windows

| Burst | Approx. S/N | Outcome |
|---|---:|---|
| 11E | 17 | Fails the detection rule at one scale (amplitude 0.07); no stable width |
| 11K | 11 | Fails the detection rule; no usable width |
| 11C, 11B | about 7 and 5 | Screened out at low S/N; not analysed further |

The early window of 11A (1.0-2.7 ms) gives widths of 3.8, 13.6 and 31.2 MHz at the three scales. These are not stable across scales and are not used in the main result.

## Limitations

- Only burst 11A gives a strong, consistent measurement, from one observation with one telescope. The data say nothing about time or frequency evolution of the width.
- The detection thresholds (amplitude 0.1, HWHM 3 MHz) were chosen after inspecting 11A. Passing them is a consistency check, not a calibrated significance.
- The empty-window and white-noise tests are not a formal false-alarm probability.
- The injected patterns are idealised (Gaussian correlation, full modulation). The calibration holds only as far as these resemble the real burst.
- Intrinsic spectral structure in the burst can produce a similar ACF. This analysis cannot separate intrinsic structure from propagation effects.
- A broad ACF feature at about 85-110 MHz appears in both windows of 11A but not in empty windows. Its origin is unexplained.
- The frequency scaling used in the comparison with Spitler et al. is assumed, not measured.

## Reproducing the analysis

Requirements: Python 3, NumPy, SciPy, Matplotlib, Astropy.

```bash
git clone https://github.com/Vicks-03/FRB121102-acf3.git
cd FRB121102-acf3
pip install -r requirements.txt          # add requirements-dev.txt for tests
```

Download `11A_16sec.calib.4p` (322,367,040 bytes) from the [data page](https://seti.berkeley.edu/frb121102/technical.html) and place it in `data/`. Run everything from the repository root.

```bash
python scripts/screen_burst.py data/11A_16sec.calib.4p   # inspect burst, suggest windows
python scripts/measure.py configs/11A.json               # writes results/11A_summary.json
python scripts/make_figures.py configs/11A.json          # writes figures/ (PNG and PDF)
```

The standard run uses 8 injection trials and 200 white-noise spectra. Use `--n-trials 4` for a quicker, less thoroughly sampled calibration. The calibration is random, so repeated runs can differ slightly. The numbers above are from the latest documented run.

A config file sets the data file and windows (in ms from the start of the ~20 ms snapshot):

```json
{
  "label": "11A",
  "file": "data/11A_16sec.calib.4p",
  "burst_window_ms": [2.7, 5.0],
  "early_window_ms": [1.0, 2.7],
  "off_window_ms": [8.0, 21.0]
}
```

The off-pulse window sets each channel's noise level and supplies the empty windows used in calibration, so it must be free of burst emission.

The code can also be run in Google Colab with the same commands after cloning the repository and uploading the data file.

### Tests

```bash
pytest -q
```

The tests use synthetic data: signals with a known correlation width, noise-only spectra, and the file reader's array handling (checked against a stand-in for Astropy). They test the components and do not replace running the full analysis on real data.

## Repository structure

```text
FRB121102-acf3/
├── frbacf/
│   ├── params.py      analysis settings
│   ├── io.py          PSRFITS reader and calibration-array ordering check
│   ├── clean.py       normalisation, masking, rebinning
│   ├── acf.py         window spectrum, envelope, ACF, Gaussian fit, detection rule
│   ├── inject.py      injection-recovery, empty-window and white-noise tests
│   ├── pipeline.py    full analysis of one window
│   ├── report.py      text report
│   ├── figures.py     report figures
│   ├── screen.py      window suggestions for new bursts
│   └── config.py      config file loading
├── scripts/           command-line entry points
├── configs/           one JSON file per burst
├── tests/             automated tests
├── data/              downloaded burst files go here
├── results/           written by scripts/measure.py
├── figures/           written by scripts/make_figures.py
├── conftest.py
├── requirements.txt
├── requirements-dev.txt
├── LICENSE
└── README.md
```

## References

1. Spitler, L. G., et al. (2018). *Detection of Bursts from FRB 121102 with the Effelsberg 100-m Radio Telescope at 5 GHz and the Role of Scintillation*. ApJ, 863, 150. https://doi.org/10.3847/1538-4357/aad332
2. Gajjar, V., et al. (2018). *Highest Frequency Detection of FRB 121102 at 4-8 GHz Using the Breakthrough Listen Digital Backend at the Green Bank Telescope*. ApJ, 863, 2. https://doi.org/10.3847/1538-4357/aad005
3. Breakthrough Listen. [FRB 121102 technical data](https://seti.berkeley.edu/frb121102/technical.html)

## License

MIT. See [LICENSE](LICENSE).

