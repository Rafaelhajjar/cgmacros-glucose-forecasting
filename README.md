# Personalized 60-Minute Glucose Forecasting

An individual time-series machine-learning project using [CGMacros](https://physionet.org/content/cgmacros/1.0.0/).

**Research question:** How accurately can glucose be forecast 60 minutes ahead, and does calibration using a small amount of an individual's previous data improve forecasts compared with a shared population model?

The project will also examine whether personalization helps more after meals than at other times. The goal is a focused forecasting study with sound temporal evaluation, appropriate baselines, and a reproducible workflow.

## Current status

The topic is selected and data access is confirmed. The complete CGMacros v1.0.0 archive has been downloaded, its SHA-256 checked against the publisher's manifest, and all 45 participant time-series files inspected.

**No forecasting models have been trained and no predictive-performance results are available yet.** Model choices and evaluation details below are an initial plan.

- [Topic and dataset selection](docs/topic-selection.md)
- [Verified data-access check and known issues](docs/data-check.md)

## Prediction task

| Item | Initial definition |
| --- | --- |
| Target | A participant's Dexcom glucose value in mg/dL |
| Forecast horizon | Exactly 60 minutes after the forecast origin |
| Instances | Individual participants, each with their own time series |
| Inputs | Previous glucose measurements, time-of-day features, and meal information available by the forecast origin |
| Main comparison | A shared population model versus the same model with limited personal calibration |
| Main outcomes | Future-period MAE and RMSE, reported overall, by participant, and after meals |

This is a forecasting task. It does not estimate the causal effect of a food or provide treatment recommendations.

## Why this question

A shared model can learn from many people, but the same recent glucose trajectory or meal information may be associated with different subsequent responses across individuals. Training a separate complex model for each person is difficult when only a few days of their data are available.

We will test whether a small personal correction improves a shared forecast, how much history that correction needs, and when it makes predictions worse. Improved accuracy is a hypothesis, not an assumed result.

## Dataset

CGMacros combines glucose recordings from two sensors, meal timestamps and macronutrients, activity measurements, meal photographs, and supplementary participant information. The published study includes 45 people with approximately ten days of observations each, spanning participants without diabetes, with prediabetes, and with type 2 diabetes.

Our local inspection found:

| Check | Result |
| --- | --- |
| Participant time-series files | 45 |
| Released minute-timeline rows | 687,580 |
| Logged meal records | 1,706 |
| Meal records with calories, carbohydrate, protein, fat, and fiber present | 1,705 |
| Duplicate timestamps within a participant | 0 |
| Dexcom values missing from existing rows | 8.40% |

These are data-access and quality checks, not model results. The minute-level rows are not independent measurements: the released glucose series contains interpolation, while the documented native sampling intervals are five minutes for Dexcom and fifteen minutes for Libre. Sampling alignment and the information used by interpolation must be resolved before making claims about leakage-free real-time performance. See [the data check](docs/data-check.md).

Raw data and meal photos are not included in this repository. CGMacros is distributed under [CC BY-NC-SA 4.0](https://physionet.org/content/cgmacros/1.0.0/), and its terms apply separately from this project's documentation.

## Initial methods

1. **Persistence baseline:** predict that glucose will remain at its latest available value.
2. **Shared forecasting model:** train one LightGBM model across training participants, using lagged glucose and other permitted inputs.
3. **Personalized forecast:** apply a low-dimensional correction to the shared model, learned only from the new participant's earlier data. An additive residual-bias correction is a starting candidate.

We will initially compare no personal calibration with calibration based on one and three days. The exact windows depend on usable participant coverage. A small LSTM or a pretrained forecasting model is a possible later comparison if the core study is complete.

## Temporal evaluation

- Separate training, validation, and test participants to distinguish learning population patterns from adapting to a new person.
- Fit preprocessing, feature selection, and model settings using training and validation data only.
- Use an early prefix of each test participant's series for personal calibration. Compare all calibration budgets on the same later test period.
- Update a correction only after its required glucose outcomes have actually become available. A 60-minute target cannot inform an update before those 60 minutes have elapsed.
- Match targets by timestamp, handle gaps explicitly, and avoid shuffling overlapping windows across temporal boundaries.
- Verify sensor alignment and interpolation before choosing the forecast origins and observation grid.
- Use only meal information available at prediction time. Exclude post-meal photos and final amounts consumed from a forecast issued at meal onset.
- Report participant-level performance and uncertainty. Do not count overlapping windows as independent experimental replications.

The primary analysis will compare overall errors with errors in a predefined post-meal window. That window and the evaluation rules will be fixed before final testing.

## Data access

The official versioned dataset is available on [PhysioNet](https://physionet.org/content/cgmacros/1.0.0/). The full archive is approximately 657 MB in decimal units. To obtain a local copy from the repository root:

```sh
mkdir -p data/raw
curl --fail --location --retry 2 \
  https://physionet.org/files/cgmacros/1.0.0/CGMacros_dateshifted365.zip \
  --output data/raw/CGMacros_dateshifted365.zip
curl --fail --location --retry 2 \
  https://physionet.org/files/cgmacros/1.0.0/SHA256SUMS.txt \
  --output data/raw/SHA256SUMS.txt
shasum -a 256 data/raw/CGMacros_dateshifted365.zip
```

Compare the printed archive hash with its entry in `data/raw/SHA256SUMS.txt`. The verified v1.0.0 archive hash is recorded in [the data check](docs/data-check.md). The `data/` directory is excluded from Git.

There is no executable training pipeline in this initial repository. Reproduction instructions, dependencies, and experiment settings will be added alongside the implementation.

## Next steps

- Resolve native sensor sampling, interpolation, gaps, and meal-feature availability.
- Define participant splits and calibration/test windows.
- Implement and verify the persistence baseline.
- Train the shared model and test limited personal calibration.
- Compare results across participants and post-meal periods.
- Document limitations, verification, and the computational workflow in the final paper and video.

## References

- [CGMacros v1.0.0: dataset, documentation, and license](https://physionet.org/content/cgmacros/1.0.0/). Dataset DOI: [10.13026/3z8q-x658](https://doi.org/10.13026/3z8q-x658).
- [Personalized Blood Glucose Forecasting From Limited CGM Data Using Incrementally Retrained LSTM](https://pubmed.ncbi.nlm.nih.gov/39514345/), Shen and Kleinberg, 2025.
- [Evaluating Time-Series Foundation Models and Multimodal Dietary Context for CGM Forecasting](https://arxiv.org/abs/2609.11872), Zhang et al., 2026 preprint.

## AI assistance

OpenAI Codex assisted with brainstorming, literature lookup, dataset inspection, and this repository's documentation. The final paper will describe how generated code, temporal alignment, evaluation procedures, citations, and numerical claims were independently verified and which corrections were required. This initial repository makes no claim that model implementation or predictive results have been verified.
