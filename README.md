# FRB121102-acf3

A Python pipeline for measuring frequency autocorrelation widths in bursts from FRB 121102 using public Green Bank Telescope data from the Breakthrough Listen project.

The project applies an autocorrelation-function (ACF) analysis at three frequency-envelope smoothing scales (80, 150 and 300 MHz), followed by injection-recovery calibration and noise-control tests. The name `acf3` refers to the three envelope scales used in the analysis.

I wrote this while learning how scintillation bandwidths are measured, starting from Spitler et al. (2018), who measured one at 4.85 GHz with Effelsberg. Their data are not public, so the aim here is not to reproduce their number. It is to implement the same kind of measurement on a different, public data set, test how reliable it is, and state clearly what the result does and does not show.

**Short version of the result.** For burst 11A (late window, 2.7-5.0 ms, centred near 6 GHz), the calibrated frequency-correlation widths are approximately 8.45, 10.04 and 11.06 MHz across the three scales, with a mean of 9.85 MHz. This is the right order of magnitude for Galactic scintillation, but the data do not by themselves establish that the structure is caused by scintillation. Of the five bursts examined, only 11A was bright enough to give a stable width.

## What the code does

1. Reads the calibrated PSRFITS data and extracts Stokes I.
2. Normalizes each frequency channel by its own off-pulse noise.
3. Masks band edges and channels with unusual behaviour.
4. Averages the data in time and in groups of eight frequency channels.
5. Removes the broad spectral envelope using smoothing scales of 80, 150 and 300 MHz.
6. Computes the frequency autocorrelation function and fits its central structure with a Gaussian, excluding zero lag.
7. Uses injected signals, empty time windows and white-noise spectra to test the measurement and calibrate the recovered widths.

The fitted width is the half-width at half-maximum (HWHM) of the ACF fit. The detection rule requires a fitted amplitude of at least 0.1 and an HWHM of at least 3 MHz at all three envelope scales.

## Figures

### Dynamic spectrum

The frequency-time representation of burst 11A used in the analysis.

![Dynamic spectrum of FRB 121102 burst 11A](frbacf/figures./11A_dynamic_spectrum.png)

### Frequency ACF and Gaussian fits

Frequency autocorrelation of burst 11A (2.7-5.0 ms) after envelope subtraction at each of the three scales. Black points are the ACF with zero lag omitted. The orange curve is the Gaussian fit, which is made over lags up to 60 MHz, and the dotted vertical line marks the fitted HWHM. The negative dip after the central peak is expected to be partly a result of subtracting the smoothed envelope. The broad bump near 85-100 MHz, most visible in panels (b) and (c), is not explained (see Limitations).

![Frequency ACF and Gaussian fits at three envelope scales](frbacf/figures./11A_acf_fits.png)

### Injection-recovery calibration

Injected frequency-correlation widths are compared with the widths recovered by the pipeline at the three envelope scales. This is used to calibrate the measured ACF widths.

![Injection-recovery calibration](frbacf/figures./11A_injection_recovery.png)

## Requirements

- Python 3 (developed and run in Google Colab)
- NumPy
- SciPy
- Matplotlib
- Astropy

## Installation

Clone the repository:

```bash
git clone https://github.com/Vicks-03/FRB121102-acf3.git
cd FRB121102-acf3
```

Install the dependencies:

```bash
pip install -r requirements.txt
```

To install the development dependencies required for testing:

```bash
pip install -r requirements-dev.txt
```

## Data

The analysis uses the public Breakthrough Listen Green Bank Telescope observation:

`11A_16sec.calib.4p`

The data can be obtained from the [FRB 121102 technical data page](https://seti.berkeley.edu/frb121102/technical.html).

Download the file and place it in the repository's `data/` directory:

```text
data/
└── 11A_16sec.calib.4p
```

The expected file size is approximately 322 MB (322,367,040 bytes). The data file is not included in the repository because of its size.

The configuration file `configs/11A.json` specifies the data file and the burst, early-time and off-pulse windows:

```json
{
  "label": "11A",
  "file": "data/11A_16sec.calib.4p",
  "burst_window_ms": [2.7, 5.0],
  "early_window_ms": [1.0, 2.7],
  "off_window_ms": [8.0, 21.0]
}
```

Windows are in milliseconds from the start of the approximately 20 ms snapshot. The off-pulse window is used to estimate channel noise and to select empty windows for control tests, so it must not contain detectable burst emission.

## Running the analysis

Run everything from the repository root.

First, run the burst-screening script, which shows the burst and suggests analysis windows:

```bash
python scripts/screen_burst.py data/11A_16sec.calib.4p
```

Run the ACF measurement:

```bash
python scripts/measure.py configs/11A.json
```

This prints a report and writes the numerical results to `results/11A_summary.json`. The standard run uses eight injection trials and 200 white-noise spectra. For a quicker, less thoroughly sampled calibration, use fewer trials:

```bash
python scripts/measure.py configs/11A.json --n-trials 4
```

The calibration is random, so repeated runs can give slightly different numbers. The values reported below come from the latest run documented here.

Generate the figures (this needs `early_window_ms` in the config):

```bash
python scripts/make_figures.py configs/11A.json
```

The figures are saved to `figures/` in PNG and PDF formats.

## Google Colab

The repository can also be tested using Google Colab without installing Python locally.

1. Open [Google Colab](https://colab.research.google.com/) and create a new notebook.
2. Clone the repository:

```python
!git clone https://github.com/Vicks-03/FRB121102-acf3.git
```

3. Move into the repository directory:

```python
%cd /content/FRB121102-acf3
```

4. Install the dependencies:

```python
!pip install -r requirements-dev.txt
```

5. Run the tests:

```python
!pytest -v
```

6. Download `11A_16sec.calib.4p` from the [Breakthrough Listen data page](https://seti.berkeley.edu/frb121102/technical.html), then upload it to Colab:

```python
from google.colab import files
uploaded = files.upload()
```

7. Move the uploaded file into the data directory:

```python
!mkdir -p data
!mv 11A_16sec.calib.4p data/
```

8. Run the screening and measurement:

```python
!python scripts/screen_burst.py data/11A_16sec.calib.4p
!python scripts/measure.py configs/11A.json
```

9. Generate the figures:

```python
!python scripts/make_figures.py configs/11A.json
```

10. Inspect the results:

```python
!cat results/11A_summary.json
```

The PNG figures can be downloaded from Colab using:

```python
from google.colab import files

files.download("figures/11A_dynamic_spectrum.png")
files.download("figures/11A_acf_comparison.png")
files.download("figures/11A_injection_recovery.png")
```

## Results: burst 11A

Numbers from the latest run on the late window of burst 11A (2.7-5.0 ms), centre frequency about 6000 MHz, approximate signal-to-noise ratio 34.3, with 1817 frequency channels masked.

| Envelope scale | Raw HWHM (MHz) | Fit amplitude | Calibrated width (MHz) | Calibration range (MHz) |
|---|---:|---:|---:|---:|
| 80 MHz | 5.8 | 0.31 | 8.45 | 7.66-9.74 |
| 150 MHz | 7.7 | 0.34 | 10.04 | 8.56-12.03 |
| 300 MHz | 9.4 | 0.35 | 11.06 | 9.25-13.16 |

The mean calibrated width is 9.85 MHz, with a range of 8.45-11.06 MHz across the three envelope scales.

The calibration ranges describe the recovered-width ranges from the injection-recovery procedure, not the full uncertainty on the astrophysical measurement. Since all three envelope scales use the same burst data, their spread is a robustness range rather than an independent statistical confidence interval.

Repeating the measurement with an alternative normalization gives raw widths of 5.78, 7.74 and 9.38 MHz, very close to the adopted normalization.

### Control tests

- **Empty windows:** none of the five tested burst-free windows passed the detection rule (0/5).
- **White noise:** none of the white-noise spectra passed the detection rule (fraction 0.0).
- **Burst 11A:** passed the detection rule at all three envelope scales.
- **Channel ordering:** the adopted ordering of the calibration arrays gave a bandpass roughness of 0.0587, compared with 0.4778 for the alternative ordering. This is an internal check on how the data file is read, not a scintillation measurement.

These checks support the internal consistency of the analysis, but they are not a formal false-alarm probability.

### Injection-recovery

The pipeline was tested by injecting artificial frequency-correlation patterns of known width into real burst-free spectra from the same file.

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

The recovered widths were generally smaller than the injected widths, which motivates the calibration applied to the measured ACF widths.

## Comparison with previous observations

Spitler et al. (2018) measured a characteristic small-scale frequency bandwidth of 6.4 ± 1.6 MHz for FRB 121102 at 4.85 GHz using Effelsberg observations.

Extrapolating this to 6 GHz with an illustrative scaling of Δν ∝ ν^4 to ν^4.4 gives roughly 15-16 MHz, with an uncertainty of about ±4 MHz from the original measurement alone. The mean calibrated width measured here, approximately 9.9 MHz, is of the same order of magnitude.

Gajjar et al. (2018), using Green Bank Telescope observations of FRB 121102 at 4-8 GHz, also reported fine-scale spectral structure of approximately 10-50 MHz, consistent with the expected scale of Galactic diffractive scintillation.

These comparisons give context for the measured frequency structure but do not establish its physical origin. The frequency scaling used above is an assumption and is not tested by this analysis.

## Other bursts and additional checks

| Burst | Approx. S/N | Outcome |
|---|---:|---|
| 11E | 17 | Fails the detection rule at one scale (amplitude 0.07); no stable width |
| 11K | 11 | Fails the detection rule; no usable width |
| 11C, 11B | about 7 and 5 | Screened out at low S/N; not analysed further |

An early-time window in burst 11A (1.0-2.7 ms) produced widths of approximately 3.8, 13.6 and 31.2 MHz across the three scales. These values were not stable across smoothing scales and are not used for the main result.

An additional ACF feature at approximately 85-110 MHz appears in both the main burst window and the early-time window, but not in the empty windows. Its origin is not established in this analysis.

## Limitations

- Only one burst, 11A, provides a sufficiently strong and consistent measurement for the main result.
- The analysis uses one observation and one telescope; it does not establish time evolution or frequency evolution of the measured width.
- The detection thresholds (amplitude 0.1, HWHM 3 MHz) were selected after inspecting burst 11A. Passing the criteria is therefore a consistency check, not an independently calibrated detection significance.
- The empty-window and white-noise tests are useful controls but do not provide a formal false-alarm probability.
- The injection-recovery procedure uses idealized Gaussian correlation patterns with full modulation, and the calibration holds only as far as these resemble the real burst.
- Intrinsic spectral structure in the burst could also produce frequency correlations. This analysis cannot distinguish intrinsic structure from propagation effects.
- The additional ACF feature around 85-110 MHz is not explained.
- The frequency scaling used in the comparison with Spitler et al. is assumed rather than measured from this dataset.

The result should therefore be interpreted as a measurement of frequency-correlation width, consistent in order of magnitude with Galactic scintillation, and not as a confirmed scintillation bandwidth.

## Tests

The repository includes tests using synthetic data, so the basic analysis components can be checked without downloading the full observational dataset.

Install the development dependencies:

```bash
pip install -r requirements-dev.txt
```

Run the tests:

```bash
pytest -q
```

The tests cover synthetic signals with known frequency-correlation widths, noise-only cases and data-reader array handling (checked against a stand-in for Astropy, so no data file is needed). They test individual components of the pipeline but do not replace running the complete analysis on the real observational data.

## Repository structure

```text
FRB121102-acf3/
├── frbacf/
│   ├── params.py      analysis settings
│   ├── io.py          PSRFITS reader and calibration-array ordering check
│   ├── clean.py       normalization, masking, rebinning
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
├── data/              place downloaded burst files here
├── results/           written by scripts/measure.py
├── figures/           written by scripts/make_figures.py
├── conftest.py
├── requirements.txt
├── requirements-dev.txt
├── LICENSE
└── README.md
```

## References

1. Spitler, L. G., et al. (2018). *Detection of Bursts from FRB 121102 with the Effelsberg 100-m Radio Telescope at 5 GHz and the Role of Scintillation*. The Astrophysical Journal, 863, 150. [https://doi.org/10.3847/1538-4357/aad332](https://doi.org/10.3847/1538-4357/aad332)

2. Gajjar, V., et al. (2018). *Highest Frequency Detection of FRB 121102 at 4-8 GHz Using the Breakthrough Listen Digital Backend at the Green Bank Telescope*. The Astrophysical Journal, 863, 2. [https://doi.org/10.3847/1538-4357/aad005](https://doi.org/10.3847/1538-4357/aad005)

3. Breakthrough Listen. [FRB 121102 technical data](https://seti.berkeley.edu/frb121102/technical.html)

## License

This project is released under the MIT License. See [LICENSE](LICENSE).
## License

This project is released under the MIT License.

