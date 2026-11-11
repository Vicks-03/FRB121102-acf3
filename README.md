# FRB121102-acf3
### Frequency Autocorrelation Analysis of FRB 121102: Investigating Spectral Structure and Scintillation Bandwidths

[![Python](https://img.shields.io/badge/Python-3.x-blue.svg)](https://www.python.org/)
[![Astrophysics](https://img.shields.io/badge/Field-Radio%20Astronomy-purple.svg)](https://en.wikipedia.org/wiki/Radio_astronomy)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

**Research question:** Can frequency-autocorrelation measurements of spectral structure, similar to those employed by Spitler et al. (2018), be reproduced using public 4–8 GHz Green Bank Telescope observations of FRB 121102? How reliable are the resulting characteristic frequency widths?

This repository contains an independent Python-based analysis pipeline developed to investigate frequency-dependent spectral structure in FRB 121102. It implements frequency autocorrelation analysis, Gaussian fitting, injection-recovery calibration and noise controls to examine the reliability of the measured spectral correlation widths.

The analysis focuses primarily on burst 11A and compares the resulting measurements with previously published studies of FRB 121102.

The repository is intended to document both the analysis procedure and its limitations, rather than establish a definitive detection of scintillation.

---

## Abstract

Fast Radio Bursts (FRBs) are millisecond-duration radio transients whose observed spectra can contain frequency-dependent structures arising from both intrinsic emission processes and propagation through intervening plasma.

One possible origin of such spectral structure is scintillation, in which propagation through turbulent plasma introduces frequency-dependent intensity variations. The characteristic frequency scale over which these variations remain correlated can provide information about the intervening medium.

This project investigates the frequency correlation structure of FRB 121102 using publicly available 4–8 GHz observations obtained with the Green Bank Telescope (GBT) through the Breakthrough Listen digital backend.

A Python pipeline was developed to process calibrated burst spectra, remove broad spectral envelopes, compute frequency autocorrelation functions (ACFs), and estimate characteristic correlation widths through Gaussian fitting. The dependence of the measurements on spectral-envelope subtraction was investigated using three smoothing scales: 80, 150 and 300 MHz.

To assess measurement reliability, the pipeline was tested using injection-recovery experiments in which artificial scintillation-like spectral patterns of known characteristic widths were introduced into real burst-free spectra. Additional tests were performed using empty off-pulse windows and synthetic white-noise spectra.

For the late portion of burst 11A, the calibrated characteristic frequency widths obtained at the three envelope scales are 8.45, 10.04 and 11.06 MHz, respectively. Although the raw measurements exhibit a dependence on the envelope scale, injection-recovery calibration reduces this variation. The measurement also passes the implemented noise-control tests.

The measured width is broadly comparable in scale to some previously published measurements, although differences between analysis methods and width definitions complicate direct comparisons. In particular, a reanalysis of the same GBT observations by Levkov et al. (2022) reports a substantially smaller decorrelation bandwidth.

The results demonstrate that a characteristic frequency-correlation width can be recovered from the available data using this analysis procedure. However, the analysis does not independently establish that the observed spectral structure originates from scintillation rather than intrinsic spectral variations.

---

## 1. Introduction and Scientific Motivation

Fast Radio Bursts are extremely energetic, short-duration radio transients originating at cosmological distances. Their observed signals can be modified by propagation through ionised material between the source and the observer.

One such propagation effect is scintillation.

Scintillation arises when radio waves encounter fluctuations in the electron density of intervening plasma. These fluctuations introduce differences in the propagation paths of the waves, producing interference and frequency-dependent intensity variations in the observed signal.

The resulting spectrum can exhibit structures over characteristic frequency scales. One way to investigate these structures is through the frequency autocorrelation function (ACF), which measures how strongly spectral fluctuations at one frequency are correlated with those at nearby frequencies.

The characteristic width of the ACF can provide an estimate of the frequency scale associated with the observed spectral structure.

This makes frequency autocorrelation analysis a useful tool for investigating propagation effects in FRBs.

### 1.1 Motivation from previous observations

Spitler et al. (2018) investigated FRB 121102 using observations obtained with the Effelsberg 100-m radio telescope at approximately 4.85 GHz. Their analysis reported a scintillation bandwidth of 6.4 ± 1.6 MHz.

Subsequent observations of FRB 121102 using the Green Bank Telescope extended the available frequency coverage to 4–8 GHz.

Gajjar et al. (2018) reported high-frequency detections of the source using the Breakthrough Listen digital backend. Their analysis also examined frequency-dependent spectral structure in the bursts.

A later study by Levkov et al. (2022) re-examined the spectral properties of the same GBT observations and reported different characteristic decorrelation bandwidths.

These studies provide an opportunity to investigate how frequency-correlation measurements depend on the analysis procedure, the definition of the characteristic width and the treatment of broad spectral structure.

### 1.2 Objectives

The primary objectives of this project are:

1. Investigate the frequency-correlation structure of FRB 121102 burst spectra using public GBT observations.
2. Estimate the characteristic frequency-correlation width of burst 11A near 6 GHz.
3. Examine how the measured width changes with the choice of spectral-envelope smoothing scale.
4. Develop an injection-recovery procedure to quantify measurement bias.
5. Test the analysis against burst-free data and synthetic noise.
6. Compare the resulting measurements with previously published studies.
7. Examine whether the measurement remains consistent across different bursts and time windows.

An important distinction throughout this work is that measuring a characteristic frequency-correlation width is not equivalent to establishing its physical origin. In particular, the presence of a narrow ACF feature alone does not demonstrate that the feature is caused by scintillation.

---

## 2. Observational Data

### 2.1 Data source

The analysis uses publicly available observations of FRB 121102 obtained using the Green Bank Telescope through the Breakthrough Listen digital backend.

The observations cover the 4–8 GHz frequency range and are available in calibrated PSRFITS format.

**Data source:** [Breakthrough Listen – FRB 121102 Technical Data](https://seti.berkeley.edu/frb121102/technical.html)

### 2.2 Dataset characteristics

| Parameter | Description |
|---|---|
| Source | FRB 121102 |
| Telescope | Green Bank Telescope |
| Backend | Breakthrough Listen digital backend |
| Observing frequency | 4–8 GHz |
| Data format | Calibrated PSRFITS |
| Polarisation data | Stokes I |
| Main burst | 11A |
| Main analysis window | 2.7–5.0 ms |
| Approximate snapshot duration | 20 ms |

The primary dataset used is:

`11A_16sec.calib.4p`

This file contains a short snapshot of approximately 20 ms containing burst 11A.

Other bursts within the available observations, including 11E, 11K, 11C and 11B, were also screened to investigate whether similar measurements could be obtained.

### 2.3 Data availability

The original observational data are not included in this repository because of their size (approximately 322 MB per file).

Users must download the required files independently from the Breakthrough Listen data page and place them in the repository's `data/` directory.

### 2.4 Scope of the comparison with Spitler et al. (2018)

Although this project is motivated partly by the analysis of Spitler et al. (2018), it does not reproduce their original measurement directly.

Their Effelsberg observations are not the dataset analysed here.

Instead, this project applies a similar frequency-autocorrelation approach to an independent, publicly available GBT dataset, allowing the resulting measurements to be compared with published work.

---

## 3. Analysis Methodology

The analysis is implemented as a modular Python pipeline. The processing steps are designed to isolate fine frequency-dependent spectral structure and investigate the stability of the resulting ACF measurements.

### 3.1 Burst selection and time-window definition

The first step is to identify the relevant burst emission and select a time interval for spectral analysis.

For the primary measurement, burst 11A is divided into two windows:

- **Early window:** 1.0–2.7 ms
- **Late window:** 2.7–5.0 ms

The late window is used for the main reported result because it provides a more stable measurement across the different envelope-smoothing scales.

The other bursts in the dataset are screened to determine whether their signal quality permits a similar analysis.

### 3.2 Channel normalisation

The calibrated Stokes I data are read from the PSRFITS file.

Each frequency channel is normalised using its corresponding off-pulse noise level. This helps account for differences in the noise properties of individual channels.

The off-pulse region also provides the burst-free spectra required for subsequent calibration and control experiments.

### 3.3 Frequency-channel masking

Channels with unusual noise properties and problematic band edges are masked before further processing.

This reduces the influence of poorly behaved channels on the resulting spectrum and its autocorrelation function.

For the main burst 11A analysis, 1817 channels are masked.

### 3.4 Time and frequency averaging

The selected burst window is averaged in time to obtain a representative frequency spectrum.

The spectrum is additionally averaged over groups of eight frequency channels.

This reduces noise fluctuations while retaining the frequency-dependent structures relevant to the analysis.

### 3.5 Spectral-envelope subtraction

The observed burst spectrum can contain broad frequency-dependent variations in addition to finer spectral features.

These broad variations may influence the resulting ACF and complicate the interpretation of its characteristic width.

To isolate finer spectral structure, a smoothed version of the spectrum is constructed and subtracted from the original spectrum.

This smoothed spectrum acts as an estimate of the broad spectral envelope.

The analysis investigates three envelope-smoothing scales:

| Envelope scale | Purpose |
|---|---|
| 80 MHz | Removes relatively narrow broad-band spectral variations |
| 150 MHz | Intermediate smoothing scale |
| 300 MHz | Removes broader spectral variations |

Using multiple smoothing scales allows the sensitivity of the measured ACF width to the treatment of the broad spectral envelope to be investigated.

The three measurements are performed on the same underlying burst data. Consequently, their agreement or disagreement is a measure of methodological robustness rather than independent statistical confirmation.

### 3.6 Frequency autocorrelation analysis

After envelope subtraction, the residual spectrum is used to calculate its frequency autocorrelation function.

The ACF measures the relationship between spectral fluctuations separated by different frequency intervals.

A strong central correlation feature indicates that spectral fluctuations remain correlated over a particular frequency separation.

The analysis examines the resulting ACF after excluding zero lag and fits a Gaussian profile to the positive-lag region.

The fitting range extends up to 60 MHz.

The characteristic width reported by this implementation is the half-width at half-maximum (HWHM) of the fitted Gaussian.

This is an operational measurement definition used consistently throughout the pipeline.

It should not automatically be treated as interchangeable with every definition of decorrelation bandwidth used in the literature.

### 3.7 Detection criteria

A measurement is considered to pass the implemented detection rule when:

- The fitted Gaussian amplitude is at least 0.1.
- The fitted HWHM is at least 3 MHz.
- Both conditions are satisfied at all three envelope-smoothing scales.

These criteria are used to identify measurements that remain reasonably stable across the adopted processing choices.

Importantly, the thresholds were selected after inspecting burst 11A. They should therefore be interpreted as analysis consistency criteria rather than independently established statistical significance thresholds.

---

## 4. Calibration and Validation

A characteristic frequency width recovered from an ACF is not necessarily identical to the underlying width of the spectral structure.

The measurement can be affected by the finite frequency resolution, noise, spectral-envelope subtraction and the fitting procedure.

To investigate these effects, the pipeline includes injection-recovery experiments and additional noise controls.

### 4.1 Injection-recovery calibration

The injection-recovery procedure is designed to quantify how accurately the pipeline can recover a known characteristic frequency width.

Artificial scintillation-like spectral patterns with specified correlation widths are generated and introduced into real burst-free spectra obtained from the same observational file.

The resulting spectra are then processed using the same analysis pipeline as the real burst.

This includes:

- Channel normalisation.
- Frequency-channel masking.
- Frequency averaging.
- Spectral-envelope subtraction.
- ACF computation.
- Gaussian fitting.
- Application of the detection criteria.

The recovered widths are compared with the widths used to generate the injected patterns.

This provides an empirical estimate of the measurement bias introduced by the analysis procedure.

The resulting injected-versus-recovered relationship is then used to correct the widths measured in burst 11A.

The calibration is performed separately for the three envelope-smoothing scales.

### 4.2 Injection-recovery results

The following table shows the detection fractions obtained for different injected characteristic widths.

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

The recovered widths are generally smaller than the injected widths, indicating a systematic measurement bias.

The injection-recovery calibration attempts to correct for this effect.

Within the tested injection configurations, the measurement appears most reliable for characteristic widths of approximately 6–20 MHz. This range includes the widths obtained from the main burst 11A analysis.

The calibration nevertheless depends on the injected spectral patterns being sufficiently representative of the actual spectral structure.

### 4.3 Empty-window controls

The same measurement procedure is applied to burst-free time windows to investigate whether the detection criteria can incorrectly identify characteristic widths in the absence of burst emission.

Results:

- Empty windows tested: 5
- Windows passing the detection criteria: 0

None of the five tested empty windows produces a measurement satisfying the adopted detection rule.

### 4.4 White-noise controls

Additional tests are performed using synthetic white-noise spectra.

These spectra are passed through the same measurement procedure to investigate whether random fluctuations can produce apparently significant ACF widths.

Results:

- White-noise spectra tested: 200
- Spectra passing the detection criteria: 0
- Detection fraction: 0

### 4.5 Alternative normalisation

The analysis is repeated using an alternative normalisation procedure to investigate whether the measured widths are particularly sensitive to the adopted channel-normalisation method.

The resulting raw widths are:

| Envelope scale | Adopted normalisation (MHz) | Alternative normalisation (MHz) |
|---|---:|---:|
| 80 MHz | 5.8 | 5.78 |
| 150 MHz | 7.7 | 7.74 |
| 300 MHz | 9.4 | 9.38 |

The two procedures produce very similar raw measurements.

This suggests that the observed envelope-scale dependence is not primarily introduced by the choice between these two normalisation procedures.

### 4.6 Channel-ordering validation

The file-reading procedure includes a check of the calibration-array ordering.

The adopted ordering produces a bandpass roughness measure of 0.0587, compared with 0.4778 for the alternative ordering.

This check helps identify the appropriate interpretation of the calibration array when reading the observational data.

It is an instrumental data-handling validation and should not be interpreted as evidence for scintillation.

### 4.7 Interpretation of the validation tests

Together, the injection-recovery experiments, empty-window controls, white-noise tests and alternative-normalisation comparison provide checks on the internal consistency of the analysis.

However, these tests do not constitute a formal false-alarm probability or establish the astrophysical origin of the measured spectral structure.

---

## 5. Results: Burst 11A

### 5.1 Main measurement

The primary result is obtained from the late portion of burst 11A, using the time interval 2.7–5.0 ms.

The analysis is centred near 6 GHz.

The approximate signal-to-noise ratio of the selected window is 34.3.

A total of 1817 frequency channels are masked during processing.

The resulting raw and calibrated characteristic widths are:

| Envelope scale | Raw HWHM (MHz) | Fit amplitude | Calibrated width (MHz) | 16th–84th percentile calibration interval (MHz) |
|---|---:|---:|---:|---:|
| 80 MHz | 5.8 | 0.31 | 8.45 | 7.66–9.74 |
| 150 MHz | 7.7 | 0.34 | 10.04 | 8.56–12.03 |
| 300 MHz | 9.4 | 0.35 | 11.06 | 9.25–13.16 |

The mean calibrated characteristic frequency width is approximately:

**9.85 MHz**

The range across the three envelope-smoothing scales is:

**8.45–11.06 MHz**

### 5.2 Dependence on envelope smoothing

The raw ACF width increases from approximately 5.8 MHz to 9.4 MHz as the envelope-smoothing scale is increased from 80 MHz to 300 MHz.

This indicates that the inferred width is sensitive to how broad spectral variations are removed.

After injection-recovery calibration, the resulting widths are closer together.

The three calibrated measurements lie between approximately 8.5 and 11.1 MHz.

This suggests that the calibration reduces the dependence of the measurement on the adopted envelope scale.

However, the three values are not independent measurements because they originate from the same burst spectrum.

Their spread should therefore be interpreted as a robustness range rather than an independent statistical confidence interval.

Similarly, the calibration intervals describe the injection-recovery procedure and do not represent the complete uncertainty in the astrophysical measurement.

### 5.3 Dynamic spectrum

The dynamic spectrum provides a visual representation of the burst emission across time and frequency.

It is useful for identifying the burst structure and selecting the time interval used for the frequency-domain analysis.

![Dynamic spectrum of FRB 121102 burst 11A](frbacf/figures./11A_dynamic_spectrum.png)

### 5.4 Frequency autocorrelation functions

The following figure shows the frequency ACFs obtained using the three envelope-smoothing scales.

Black points represent the measured ACF values with zero lag omitted.

The orange curves represent the fitted Gaussian profiles, using lags up to 60 MHz. The dotted lines indicate the fitted HWHM.

A negative dip following the central correlation peak is also visible. This feature is expected to be partly influenced by the subtraction of the smoothed spectral envelope.

A broader feature around 85–100 MHz is visible, particularly in the 150 and 300 MHz envelope-scale panels.

Its origin is not established by this analysis and is discussed further in the limitations section.

![Frequency ACF and Gaussian fits](frbacf/figures./11A_acf_fits.png)

### 5.5 Injection-recovery calibration

The following figure shows the relationship between injected and recovered characteristic frequency widths for the three envelope-smoothing scales.

The difference between the injected and recovered values illustrates the measurement bias addressed by the calibration procedure.

![Injection-recovery calibration](frbacf/figures./11A_injection_recovery.png)

---

## 6. Comparison with Previous Work

The measured characteristic frequency width is compared with published measurements of FRB 121102.

These comparisons provide context for the result, but differences in observational setup, analysis methodology and width definitions must be considered.

### 6.1 Spitler et al. (2018)

Spitler et al. (2018) reported a scintillation bandwidth of:

**6.4 ± 1.6 MHz at 4.85 GHz.**

The measurement was obtained using observations with the Effelsberg 100-m radio telescope.

For an approximate comparison with the present measurement near 6 GHz, the published value can be extrapolated using an assumed frequency dependence of the characteristic bandwidth.

Using frequency-scaling indices between 4 and 4.4 gives an extrapolated bandwidth of approximately 15–16 MHz at 6 GHz, with an uncertainty of roughly 4 MHz from the original measurement alone.

The mean calibrated width obtained here is approximately 9.85 MHz.

The two measurements are of the same general order of magnitude, although the present value is lower than the extrapolated Effelsberg result.

This comparison should be treated cautiously because:

- The observations were obtained using different telescopes.
- The datasets are independent.
- The frequency scaling is assumed rather than measured here.
- The exact measurement procedures and width definitions need not be identical.

The present analysis therefore does not constitute a direct reproduction of the Spitler et al. measurement.

### 6.2 Gajjar et al. (2018)

Gajjar et al. (2018) reported high-frequency observations of FRB 121102 using the Breakthrough Listen digital backend at the Green Bank Telescope.

Their ACF analysis reports a characteristic width of:

**11.5 ± 1.2 MHz**

for burst 11A in a sub-band centred at 6.407 GHz (Table 2 of their paper).

For comparison, the mean calibrated width obtained here can be scaled from approximately 6.0 GHz to 6.4 GHz using the same assumed frequency-scaling range.

This gives approximately:

**12.8–13.1 MHz at 6.4 GHz.**

This is reasonably close to the value reported by Gajjar et al.

Gajjar et al. also discuss fine-scale spectral structure of approximately 10–50 MHz across bursts, which they associate with Galactic diffractive scintillation.

An important qualification is that both studies use the same underlying GBT observation. The comparison therefore provides a check against a published analysis of the same observational data, rather than independent observational confirmation.

Differences in the exact spectral windows, preprocessing, ACF fitting and measurement definitions may also contribute to the remaining discrepancy.

### 6.3 Levkov et al. (2022)

Levkov et al. (2022) re-examined the spectral properties of FRB 121102 using the same GBT observations.

They reported a decorrelation bandwidth of:

**3.3 ± 0.6 MHz at 6 GHz.**

This is approximately three times smaller than the mean calibrated characteristic width obtained in the present analysis.

The discrepancy is particularly important because it demonstrates that the inferred characteristic frequency scale can depend substantially on the adopted analysis procedure and the definition of the measured width.

The two values should not be treated as directly interchangeable without matching their measurement conventions and processing procedures.

Levkov et al. also report spectral peaks separated by approximately 95 ± 16 MHz in the same GBT data. This is potentially relevant to the broader ACF feature around 85–110 MHz observed in the present analysis.

However, a direct connection between these features has not been established.

### 6.4 Summary of the literature comparison

| Study | Observational data | Reported characteristic width |
|---|---|---:|
| Spitler et al. (2018) | Effelsberg, 4.85 GHz | 6.4 ± 1.6 MHz |
| Gajjar et al. (2018) | GBT, 6.407 GHz | 11.5 ± 1.2 MHz |
| Levkov et al. (2022) | GBT, 6 GHz | 3.3 ± 0.6 MHz |
| Present analysis | GBT, approximately 6 GHz | 9.85 MHz mean calibrated width |

The comparisons indicate that the measured characteristic frequency scale is broadly comparable to some published results, while differing substantially from others.

This reinforces the importance of carefully distinguishing between the various definitions and measurement procedures used in frequency-correlation studies.

The frequency scaling adopted in these comparisons is an assumption and has not been tested using the present dataset.

---

## 7. Other Bursts and Time Windows

To investigate whether the measurement is specific to burst 11A, other bursts in the observational data were screened.

### 7.1 Comparison across bursts

| Burst | Approximate S/N | Outcome |
|---|---:|---|
| 11A | 34.3 | Passes detection criteria at all three envelope scales |
| 11E | 17 | Fails at one envelope scale; fitted amplitude of 0.07 |
| 11K | 11 | Fails the detection criteria |
| 11C | ~7 | Screened out due to low S/N |
| 11B | ~5 | Screened out due to low S/N |

Only burst 11A provides a sufficiently stable measurement across all three envelope-smoothing scales.

The remaining bursts do not provide reliable characteristic widths under the adopted detection criteria.

Consequently, the main result is based on a single burst rather than a statistically meaningful sample of bursts.

### 7.2 Early and late portions of burst 11A

The early portion of burst 11A was also analysed independently.

The resulting characteristic widths are:

| Envelope scale | Early-window width (MHz) |
|---|---:|
| 80 MHz | 3.8 |
| 150 MHz | 13.6 |
| 300 MHz | 31.2 |

These measurements show substantially greater variation across envelope scales than the late-window measurements.

The early window therefore does not provide a stable characteristic width under the adopted procedure and is not used for the primary result.

This comparison also highlights the importance of time-window selection when investigating frequency-dependent spectral structure.

---

## 8. Discussion

The analysis demonstrates that frequency-autocorrelation measurements can be performed on the public 4–8 GHz GBT observations of FRB 121102 and that the resulting characteristic widths can be calibrated using simulated spectral structures.

For burst 11A, the calibrated measurements remain within a relatively narrow range across the three envelope-smoothing scales.

The injection-recovery tests indicate that the pipeline systematically underestimates the widths of the injected structures, making calibration an important part of the analysis.

The noise controls also suggest that the adopted detection criteria are not readily satisfied by the tested empty windows or synthetic white-noise spectra.

Nevertheless, the physical interpretation of the observed spectral structure remains uncertain.

A frequency-correlation feature can arise from propagation effects, but intrinsic spectral variations in the burst may also produce correlated frequency structure.

Furthermore, the discrepancy between the measurements reported by Gajjar et al. and Levkov et al. demonstrates that different analysis procedures can produce substantially different characteristic widths from the same observational data.

The present analysis therefore provides a measurement and an investigation of its methodological stability, rather than a definitive identification of scintillation.

---

## 9. Limitations and Future Work

Several limitations should be considered when interpreting the results.

### 9.1 Limited number of usable bursts

Only burst 11A produces a stable measurement across all three envelope-smoothing scales.

The remaining bursts either fail the detection criteria or have insufficient signal-to-noise ratios.

Consequently, the present result cannot establish whether the measured characteristic width is representative of FRB 121102 more generally.

### 9.2 Dependence on analysis choices

The raw ACF width changes with the adopted spectral-envelope smoothing scale.

Although injection-recovery calibration reduces this dependence, the measurement remains sensitive to the analysis procedure.

The adopted Gaussian fitting prescription and fitting range may also influence the resulting characteristic width.

### 9.3 Detection thresholds

The amplitude and HWHM thresholds used in the detection rule were chosen after inspecting burst 11A.

They should therefore not be interpreted as independently calibrated significance thresholds.

Passing these criteria establishes consistency under the adopted procedure, not a formal statistical detection.

### 9.4 Injection model assumptions

The injection-recovery calibration uses idealised scintillation-like spectral patterns with Gaussian correlation properties and full modulation.

Real burst spectra may not follow these assumptions.

The accuracy of the calibration therefore depends on how well the injected patterns represent the actual spectral structures being measured.

### 9.5 Intrinsic spectral structure

The ACF analysis does not independently distinguish between intrinsic emission structure and propagation-induced spectral modulation.

Both can produce frequency-dependent correlations.

Additional observations and independent physical diagnostics would be required to establish the origin of the measured structure.

### 9.6 Broad ACF feature

A broad feature around 85–110 MHz is visible in the ACFs of both analysed windows of burst 11A but is absent in the tested empty windows.

Its origin remains unexplained.

Levkov et al. (2022) report spectral peaks separated by approximately 95 ± 16 MHz in the same GBT data.

The similarity in frequency scale is interesting, but whether the two observations are physically related has not been investigated in this project.

### 9.7 Frequency scaling

The comparisons with measurements at different observing frequencies use an assumed frequency dependence of the characteristic bandwidth.

The present dataset does not independently establish the frequency-scaling index.

### 9.8 Statistical uncertainty

The calibration intervals obtained from injection-recovery experiments do not capture every source of uncertainty in the astrophysical measurement.

In particular, the present analysis does not provide a complete uncertainty budget incorporating all possible systematic effects, model assumptions and intrinsic spectral variations.

### 9.9 Possible future investigations

Several extensions could improve the analysis:

1. **Additional bursts:** Extend the pipeline to other sufficiently bright bursts to investigate the consistency of the measured spectral widths across observations.

2. **Alternative ACF fitting procedures:** Investigate how different fitting ranges, fitting models and baseline treatments influence the recovered characteristic widths.

3. **Injection-model sensitivity:** Test how the calibration changes when the injected spectral structures differ from the assumed Gaussian correlation model.

4. **Broad spectral features:** Investigate the origin of the broad ACF feature around 85–110 MHz and its possible relationship to the spectral peaks discussed by Levkov et al. (2022).

5. **Independent observations:** Apply the analysis to other FRB observations to investigate whether similar frequency-correlation structures are present.

These investigations could help distinguish the robustness of the measured spectral structures from the physical interpretation assigned to them.

---

## 10. Reproducing the Analysis

The repository is organised as a modular Python project with separate components for data reading, preprocessing, ACF computation, calibration, pipeline execution and figure generation.

### 10.1 Requirements

The analysis requires:

- Python 3
- NumPy
- SciPy
- Matplotlib
- Astropy

Development and testing additionally use the packages listed in `requirements-dev.txt`.

### 10.2 Clone the repository

```bash
git clone https://github.com/Vicks-03/FRB121102-acf3.git

cd FRB121102-acf3
```

### 10.3 Install dependencies

```bash
pip install -r requirements.txt
```

For development and testing:

```bash
pip install -r requirements-dev.txt
```

### 10.4 Download the observational data

Download the required calibrated PSRFITS file from:

[Breakthrough Listen – FRB 121102 Technical Data](https://seti.berkeley.edu/frb121102/technical.html)

For the primary analysis, download:

`11A_16sec.calib.4p`

Place the file inside the repository's `data/` directory.

The expected location is:

```text
FRB121102-acf3/
└── data/
    └── 11A_16sec.calib.4p
```

The analysis commands should be executed from the repository root.

### 10.5 Run the analysis

First, screen the burst data and inspect potential analysis windows:

```bash
python scripts/screen_burst.py data/11A_16sec.calib.4p
```

Run the main analysis:

```bash
python scripts/measure.py configs/11A.json
```

This generates the measurement summary:

```text
results/11A_summary.json
```

Generate the figures:

```bash
python scripts/make_figures.py configs/11A.json
```

The figure-generation script produces PNG and PDF outputs.

### 10.6 Configuration

The analysis parameters are specified in JSON configuration files.

The configuration for the main burst 11A analysis is:

```json
{
  "label": "11A",
  "file": "data/11A_16sec.calib.4p",
  "burst_window_ms": [2.7, 5.0],
  "early_window_ms": [1.0, 2.7],
  "off_window_ms": [8.0, 21.0]
}
```

The time windows are specified in milliseconds from the beginning of the approximately 20 ms observational snapshot.

The off-pulse window is particularly important because it is used to estimate channel noise levels and provides burst-free spectra for calibration and control experiments.

It should therefore be selected to avoid burst emission.

### 10.7 Calibration settings

The standard analysis uses:

- 8 injection trials.
- 200 synthetic white-noise spectra.

For a quicker, less thoroughly sampled calibration, the number of injection trials can be reduced using:

```bash
--n-trials 4
```

The injection-recovery procedure involves random simulations, so repeated executions can produce slightly different calibration results.

The numerical results documented in this README correspond to the latest documented run.

### 10.8 Running in Google Colab

The repository can also be used in Google Colab.

After cloning the repository and installing the dependencies, upload the required observational data file and execute the same analysis commands from the repository root.

---

## 11. Automated Tests

The repository includes automated tests using synthetic data.

Run the tests using:

```bash
pytest -q
```

The tests cover:

- Synthetic signals with known frequency-correlation widths.
- Noise-only spectra.
- Data-reader array handling.
- Calibration-array ordering using a stand-in for Astropy.

These tests check individual components of the analysis pipeline.

They do not replace running the complete pipeline on the original observational data, nor do they independently validate the astrophysical interpretation of the measured widths.

---

## 12. Repository Structure

```text
FRB121102-acf3/
│
├── frbacf/
│   ├── figures./
│   │   └── Report figures used in the README
│   │
│   ├── params.py
│   │   └── Analysis settings
│   │
│   ├── io.py
│   │   └── PSRFITS reader and calibration-array ordering check
│   │
│   ├── clean.py
│   │   └── Normalisation, masking and rebinning
│   │
│   ├── acf.py
│   │   └── Window spectrum, envelope subtraction,
│   │       ACF computation, Gaussian fitting and detection rule
│   │
│   ├── inject.py
│   │   └── Injection-recovery, empty-window and white-noise tests
│   │
│   ├── pipeline.py
│   │   └── Full analysis of an individual time window
│   │
│   ├── report.py
│   │   └── Text report generation
│   │
│   ├── figures.py
│   │   └── Figure generation
│   │
│   ├── screen.py
│   │   └── Analysis-window suggestions for additional bursts
│   │
│   └── config.py
│       └── Configuration-file loading
│
├── scripts/
│   ├── screen_burst.py
│   ├── measure.py
│   └── make_figures.py
│
├── configs/
│   └── Individual burst configuration files
│
├── tests/
│   └── Automated tests
│
├── data/
│   └── Downloaded observational data
│
├── conftest.py
├── requirements.txt
├── requirements-dev.txt
├── LICENSE
└── README.md
```

---

## 13. References

1. **Spitler, L. G., et al. (2018).** *Detection of Bursts from FRB 121102 with the Effelsberg 100-m Radio Telescope at 5 GHz and the Role of Scintillation.* The Astrophysical Journal, 863, 150.  
   https://doi.org/10.3847/1538-4357/aad332

2. **Gajjar, V., et al. (2018).** *Highest Frequency Detection of FRB 121102 at 4–8 GHz Using the Breakthrough Listen Digital Backend at the Green Bank Telescope.* The Astrophysical Journal, 863, 2.  
   https://doi.org/10.3847/1538-4357/aad005

3. **Levkov, D. G., Panin, A. G., & Tkachev, I. I. (2022).** *Propagation Effects in the FRB 20121102A Spectra.* The Astrophysical Journal, 925, 109.  
   https://doi.org/10.3847/1538-4357/ac3250

4. **Breakthrough Listen.** FRB 121102 Technical Data.  
   https://seti.berkeley.edu/frb121102/technical.html

---

## 14. License

This project is distributed under the MIT License.

See [LICENSE](LICENSE) for details.



