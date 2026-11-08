# frb121102-acf3

Frequency autocorrelation analysis of burst spectra from the repeating fast radio
burst FRB 121102, using the public Breakthrough Listen observations made with the
Green Bank Telescope (4-8 GHz).

I wrote this while learning how scintillation bandwidths are measured, starting from
Spitler et al. (2018), who measured one at 4.85 GHz with Effelsberg. Their data are
not public, so the aim here is not to reproduce their number. It is to implement the
same kind of measurement, find out how reliable it is on a different data set, and
say clearly what the result does and does not show.

**Short version of the result.** In the later part of burst 11A (2.7-5.0 ms), the
spectrum has a frequency correlation width of about 9.9 MHz near 6 GHz after
injection-recovery calibration. The three envelope scales give corrected widths of
8.45, 10.04 and 11.06 MHz, with individual calibration ranges spanning roughly
7.7-13.2 MHz. This is the right order of magnitude for Galactic scintillation, but
these data do not establish that the measured structure is scintillation. Of the five
bursts I looked at, only 11A was bright enough to give a stable width.

## What the code does

For one burst and one time window:

1. Read the PSRFITS snapshot (Stokes I) and normalise every frequency channel by its
   own off-pulse noise. Channels with unusual noise, and the band edges, are masked.
2. Average the burst window in time and groups of 8 channels in frequency.
3. Subtract a smoothed copy of the spectrum (the "envelope"), so that only the fine
   structure is left. This is done at three smoothing scales: 80, 150 and 300 MHz.
4. Compute the frequency ACF and fit a Gaussian to lags up to 60 MHz, leaving out
   zero lag. The half-width at half-maximum (HWHM) is the reported width.
5. Apply a detection rule: fitted amplitude at least 0.1 and HWHM at least 3 MHz, at
   all three scales.

The measured width depends on the envelope scale, so the code also calibrates it:

6. Inject scintillation-like patterns of known width into real burst-free spectra from
   the same file, run exactly the same steps, and use the injected-versus-recovered
   curve to convert the measured width into a corrected one.
7. Run the same fit on empty windows and on pure noise, and repeat the measurement
   with a different normalisation, as checks.

## Setup

Clone the repository:

    git clone https://github.com/Vicks-03/FRB121102-acf3.git
    cd FRB121102-acf3

Install the dependencies:

    pip install -r requirements.txt

For running the tests, also install:

    pip install -r requirements-dev.txt

Download the data as described in [data/README.md](data/README.md). Run everything
from the repository root.

The calibrated observational files are not included in the repository because of
their size. Download the required `.calib.4p` files from:

https://seti.berkeley.edu/frb121102/technical.html

Place them inside the `data/` directory. The expected size of each calibrated file
is 322,367,040 bytes.

## Usage

Look at a burst and get suggested windows (S/N here is approximate):

    python scripts/screen_burst.py data/11E_344sec.calib.4p

Run the full analysis from a config file:

    python scripts/measure.py configs/11A.json

This prints a report and writes `results/11A_summary.json`. The standard run uses
eight injection trials and 200 white-noise spectra. A smaller number of injection
trials can be used for a quicker, less thoroughly sampled calibration:

    python scripts/measure.py configs/11A.json --n-trials 4

Make the report figures (needs `early_window_ms` in the config):

    python scripts/make_figures.py configs/11A.json

This writes five sets of figures in both PNG and PDF formats to `figures/`:

- Dynamic spectrum
- ACF and Gaussian fits
- Injection-recovery calibration
- ACF comparison across envelope scales
- Envelope-scale dependence

A config looks like this:

    {
      "label": "11A",
      "file": "data/11A_16sec.calib.4p",
      "burst_window_ms": [2.7, 5.0],
      "early_window_ms": [1.0, 2.7],
      "off_window_ms": [8.0, 21.0]
    }

The off-pulse window sets each channel's noise level and supplies the empty windows
and background spectra used in calibration, so it must be free of burst emission.
Windows are in milliseconds from the start of the approximately 20 ms snapshot.

## Running in Google Colab

The repository can also be run in Google Colab without installing Python locally.

1. Open https://colab.research.google.com/ and create a new notebook.
2. Clone the repository:

       !git clone https://github.com/Vicks-03/FRB121102-acf3.git

3. Enter the repository directory:

       %cd /content/FRB121102-acf3

4. Install the dependencies:

       !pip install -r requirements-dev.txt

5. Download `11A_16sec.calib.4p` from the Breakthrough Listen data page and upload
   it to Colab:

       from google.colab import files
       uploaded = files.upload()

6. Move the uploaded file into the data directory:

       import shutil
       shutil.move(
           "/content/11A_16sec.calib.4p",
           "data/11A_16sec.calib.4p"
       )

7. Run the automated tests:

       !pytest -v

8. Run the screening script:

       !python scripts/screen_burst.py data/11A_16sec.calib.4p

9. Run the full analysis:

       !python scripts/measure.py configs/11A.json

10. Generate the figures:

       !python scripts/make_figures.py configs/11A.json

The numerical summary will be saved in `results/` and the figures in `figures/`.

## Results

Numbers from the latest run on burst 11A (late window, 2.7-5.0 ms, centre frequency
about 6.0 GHz):

| Envelope scale | Raw HWHM | Corrected HWHM | 68% calibration range |
|---|---|---|---|
| 80 MHz | 5.8 MHz | 8.45 MHz | 7.66-9.74 MHz |
| 150 MHz | 7.7 MHz | 10.04 MHz | 8.56-12.03 MHz |
| 300 MHz | 9.4 MHz | 11.06 MHz | 9.25-13.16 MHz |

The mean of the three corrected widths is 9.85 MHz. Their range is 8.45-11.06 MHz.

The three scales use the same data, so their spread is a robustness range and not
an independent statistical confidence interval. The individual calibration ranges
describe the injection-recovery calibration, not the complete uncertainty on the
astrophysical measurement.

The alternative normalisation gives raw widths of 5.78, 7.74 and 9.38 MHz,
respectively, very close to the adopted normalisation.

Other diagnostics from the same run:

- Approximate S/N: 34.3
- Centre frequency: 5999.7 MHz
- Masked channels: 1817
- Detection rule: passed at all three scales
- Empty windows passing the rule: 0 of 5
- White-noise spectra passing the rule: 0.0 fraction
- Adopted calibration-array ordering bandpass roughness: 0.0587
- Alternative ordering bandpass roughness: 0.4778

The injection-recovery test detected 75% of injected 4 MHz patterns and 100% of
the injected 6-20 MHz patterns. For 30 MHz injections, the detection fraction was
92.5%.

These checks support the internal consistency of the analysis, but the empty-window
and white-noise results are not a formal false-alarm probability.

The other bursts:

| Burst | S/N (approx.) | Result |
|---|---|---|
| 11E | 17 | Fails the detection rule (amplitude 0.07 at one scale); no stable width |
| 11K | 11 | Fails the detection rule; no usable width |
| 11C, 11B | about 7 and 5 | Screened out at low S/N, not analysed further |

The early part of 11A (1.0-2.7 ms) also gave no stable width: it came out at 3.8,
13.6 and 31.2 MHz for the three envelope scales.

The calibration is random, so rerunning the analysis can give slightly different
numbers. The values above correspond to the latest run documented here.

## Limitations

- One burst gives a usable width, from one night with one telescope. Nothing here
  says anything about change with time or frequency.
- The detection thresholds (amplitude 0.1, width 3 MHz) were chosen after looking at
  11A. Passing the rule is a consistency check for picking candidates, not a
  calibrated significance, and the noise tests are not a false-alarm probability.
- The injected patterns are idealised (Gaussian correlation, full modulation), and
  the calibration only holds as far as they resemble the real burst.
- Intrinsic spectral structure of the burst could produce a similar ACF, and these
  data cannot separate it from propagation effects.
- Both parts of 11A show an extra ACF bump at lags of about 85-110 MHz that I cannot
  explain. It is not present in the empty windows.
- The scaling to other frequencies in the report assumes width proportional to
  frequency to the power 4-4.4, which I did not test.

The measured width should therefore be treated as a frequency correlation width
consistent in order of magnitude with Galactic scintillation, not as a confirmed
scintillation bandwidth.

## Tests

    pip install -r requirements-dev.txt
    pytest -q

The tests use synthetic data: a burst with a known scintillation width, noise-only
spectra, and the array handling in the file reader (checked against a stand-in for
astropy, so no data file is needed). They do not replace running the pipeline on a
real file.

## Layout

    frbacf/
        params.py     analysis settings
        io.py         PSRFITS reader and the calibration-array ordering check
        clean.py      normalisation, masking, rebinning
        acf.py        window spectrum, envelope, ACF, Gaussian fit, detection rule
        inject.py     injection-recovery, empty-window and white-noise tests
        pipeline.py   full analysis of one window
        report.py     text report
        figures.py    report figures
        screen.py     window suggestions for new bursts
        config.py     config file loading
    scripts/          command-line entry points
    configs/          one JSON file per burst
    tests/            automated tests
    data/             put the downloaded burst files here

## Reference

Spitler, L. G., et al. 2018, Detection of Bursts from FRB 121102 with the Effelsberg
100-m Radio Telescope at 5 GHz and the Role of Scintillation, ApJ, 863, 150
(arXiv:1807.03722).

Data: Breakthrough Listen, https://seti.berkeley.edu/frb121102/technical.html

## License

MIT, see [LICENSE](LICENSE).

