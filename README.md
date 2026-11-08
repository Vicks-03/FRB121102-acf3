# FRB121102-acf3

A Python pipeline for measuring frequency autocorrelation widths in bursts from FRB 121102 using public Green Bank Telescope data from the Breakthrough Listen project.

The project applies an autocorrelation-function (ACF) analysis at three frequency-envelope smoothing scales (80, 150 and 300 MHz), followed by injection-recovery calibration and noise-control tests. The name `acf3` refers to the three envelope scales used in the analysis.

For burst 11A, the calibrated frequency-correlation widths are approximately 8.45, 10.04 and 11.06 MHz across the three scales, giving a mean of 9.85 MHz. These measurements are consistent with narrow frequency structure, but do not by themselves establish that the structure is caused by scintillation.

## What the code does

1. Reads the calibrated PSRFITS data and extracts Stokes I.
2. Normalizes the frequency channels using the off-pulse noise.
3. Masks band edges and channels with unusual behaviour.
4. Averages the data in time and groups of eight frequency channels.
5. Removes the broad spectral envelope using smoothing scales of 80, 150 and 300 MHz.
6. Computes the frequency autocorrelation function and fits its central structure, excluding zero lag.
7. Uses injected signals, empty time windows and white-noise spectra to test the measurement and calibrate the recovered widths.

The fitted width is the half-width at half-maximum (HWHM) of the ACF fit. The analysis uses an amplitude threshold of 0.1 and a minimum HWHM of 3 MHz at all three envelope scales.

## Figures

### Dynamic spectrum

The frequency-time representation of burst 11A used in the analysis.

![Dynamic spectrum of FRB 121102 burst 11A](figures/11A_dynamic_spectrum.png)

### Injection-recovery calibration

Injected frequency-correlation widths are compared with the widths recovered by the pipeline at the three envelope scales. This is used to calibrate the measured ACF widths.

![Injection-recovery calibration](figures/11A_injection_recovery.png)

The figures are generated locally by the pipeline. The PNG files must be uploaded to the repository's `figures/` directory for them to appear here.

## Requirements

- Python 3
- NumPy
- SciPy
- Matplotlib
- Astropy

The project was run successfully in Google Colab. The exact Python minor version used for the reported run should be checked with `python --version` if reproducing the environment.

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

The expected file size is approximately 322 MB (322,367,040 bytes).

The configuration file `configs/11A.json` specifies the data file and the burst, early-time and off-pulse windows. The off-pulse window is used to estimate channel noise and to select empty windows for control tests. It should not contain detectable burst emission.

## Running the analysis

First, run the burst-screening script:

```bash
python scripts/screen_burst.py data/11A_16sec.calib.4p
```

Run the ACF measurement:

```bash
python scripts/measure.py configs/11A.json
```

The numerical results are written to:

```text
results/11A_summary.json
```

Generate the figures:

```bash
python scripts/make_figures.py configs/11A.json
```

The generated figures are saved in the `figures/` directory in PNG and PDF formats.

## Google Colab

The repository can also be tested using Google Colab without cloning it onto a personal computer.

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
files.download("figures/11A_injection_recovery.png")
```

## Results: burst 11A

The analysis identified burst 11A at a central frequency of approximately 6000 MHz, with a signal-to-noise ratio of 34.3. A total of 1817 frequency channels were masked.

| Envelope scale | Raw HWHM (MHz) | Calibrated width (MHz) | Calibration range (MHz) |
|---|---:|---:|---:|
| 80 MHz | 5.80 | 8.45 | 7.66–9.74 |
| 150 MHz | 7.75 | 10.04 | 8.56–12.03 |
| 300 MHz | 9.39 | 11.06 | 9.25–13.16 |

The mean calibrated width is 9.85 MHz, with a range of 8.45–11.06 MHz across the three envelope scales.

The calibration ranges describe the recovered-width ranges from the injection-recovery procedure. Since all three envelope scales use the same burst data, their spread is a robustness range rather than an independent statistical confidence interval.

### Control tests

- The adopted frequency-channel ordering gave a bandpass roughness diagnostic of 0.0587, compared with 0.4778 for the alternative ordering. This is an internal data-ordering diagnostic, not a measurement of scintillation.
- None of the five tested empty time windows passed the detection rule (0/5).
- The fraction of white-noise spectra passing the detection rule was 0.0.
- The detection rule was satisfied by burst 11A at all three envelope scales.

### Injection-recovery

The pipeline was tested by injecting artificial frequency-correlation patterns with known widths into real off-pulse spectra.

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

The recovered widths were generally smaller than the injected widths, motivating the calibration applied to the measured ACF widths.

## Comparison with previous observations

Spitler et al. (2018) measured a characteristic small-scale frequency bandwidth of 6.4 ± 1.6 MHz for FRB 121102 at 4.85 GHz using Effelsberg observations.

If this bandwidth is extrapolated to 6 GHz using an illustrative scaling of Δν ∝ ν⁴·⁴, the expected value is approximately 15.5 MHz. The mean calibrated width measured here, approximately 9.9 MHz, is of the same order of magnitude.

Gajjar et al. (2018), using Green Bank Telescope observations of FRB 121102 at 4–8 GHz, also reported fine-scale spectral structures of approximately 10–50 MHz, consistent with the expected scale of Galactic diffractive scintillation.

These comparisons provide context for the measured frequency structure, but do not establish its physical origin. The frequency scaling used above is an illustrative assumption and is not tested by this analysis.

## Other bursts and additional checks

The same analysis was explored for other bursts in the observation. Burst 11E had a lower signal-to-noise ratio (approximately 17) and did not satisfy the detection criterion at every envelope scale. Bursts 11K, 11C and 11B were weaker and were not suitable for the same measurement.

An early-time window in burst 11A (1.0–2.7 ms) produced widths of approximately 3.8, 13.6 and 31.2 MHz across the three scales. These values were not stable across smoothing scales and are not used for the main result.

An additional ACF feature at approximately 85–110 MHz appears in both the main burst window and the early-time window. Its origin is not established in this analysis.

## Limitations

- Only one burst, 11A, provides a sufficiently strong and consistent measurement for the main result.
- The analysis uses one observation and one telescope; it does not establish time evolution or frequency evolution of the measured width.
- The detection thresholds were selected after inspecting burst 11A. Passing the criteria is therefore a consistency check, not an independently calibrated detection significance.
- The empty-window and white-noise tests are useful controls but do not provide a formal false-alarm probability.
- The injection-recovery procedure uses idealized Gaussian correlation patterns and does not reproduce every possible property of astrophysical scintillation.
- Intrinsic spectral structure in the burst could also produce frequency correlations. This analysis cannot distinguish intrinsic structure from propagation effects.
- The additional ACF feature around 85–110 MHz is not explained.
- The frequency scaling used in the comparison with Spitler et al. is assumed rather than measured from this dataset.

The result should therefore be interpreted as a measurement of frequency-correlation width, not as a definitive detection of scintillation.

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

The tests cover synthetic signals with known frequency-correlation widths, noise-only cases and data-reader array handling. They test individual components of the pipeline but do not replace running the complete analysis on the real observational data.

## Repository structure

```text
FRB121102-acf3/
├── frbacf/
│   ├── params.py
│   ├── io.py
│   ├── clean.py
│   ├── acf.py
│   ├── inject.py
│   ├── pipeline.py
│   ├── report.py
│   ├── figures.py
│   ├── screen.py
│   └── config.py
├── scripts/
│   ├── screen_burst.py
│   ├── measure.py
│   └── make_figures.py
├── configs/
│   └── 11A.json
├── tests/
├── data/
├── results/
├── figures/
├── requirements.txt
├── requirements-dev.txt
├── LICENSE
└── README.md
```

## References

1. Spitler, L. G., et al. (2018). *Detection of Bursts from FRB 121102 with the Effelsberg 100-m Radio Telescope at 5 GHz and the Role of Scintillation*. The Astrophysical Journal, 863, 150. [https://doi.org/10.3847/1538-4357/aad332](https://doi.org/10.3847/1538-4357/aad332)

2. Gajjar, V., et al. (2018). *Highest Frequency Detection of FRB 121102 at 4–8 GHz Using the Breakthrough Listen Digital Backend at the Green Bank Telescope*. The Astrophysical Journal, 863, 2. [https://doi.org/10.3847/1538-4357/aad005](https://doi.org/10.3847/1538-4357/aad005)

3. Breakthrough Listen. [FRB 121102 technical data](https://seti.berkeley.edu/frb121102/technical.html)

## License

This project is released under the MIT License.

