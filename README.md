# MindWatch V2 — Personalized Stress Estimation

## Overview

MindWatch V2 estimates a user's same-day subjective stress level (0–100) from physiological measurements using Personalized Ridge Regression.

The system is designed for individual calibration rather than universal prediction.

## Model

- Algorithm: Ridge Regression (`alpha=100`)
- Preprocessing: Median Imputation → StandardScaler → Ridge
- Training target: Same-day self-reported stress (`stress`)
- Output: Estimated same-day stress score (0–100)

### Input Features

| Feature | Description |
|---|---|
| `avg_rmssd` | Current-day HRV |
| `overall_score` | Current-day sleep quality score |
| `daily_steps` | Current-day step count |
| `avg_rmssd_dev` | Current-day HRV minus the mean of preceding available HRV measurements |

Self-reported stress is used **only as a training label**, never as an inference feature. Anxiety scores are not used.

## Personal Calibration

MindWatch requires personal calibration before producing individualized estimates.

- Fewer than 30 valid records: Collect calibration data.
- 30–59 valid records: Initial personalized model available.
- 60 or more valid records: Recommended calibration threshold reached.

A valid calibration record includes a same-day stress label and at least one physiological measurement. The training pipeline handles missing individual features when sufficient observations are available.

The 30/60-record thresholds are engineering choices, not experimentally established optimal calibration periods.

## Model Usage

The ML module provides:

- `calibration_status(history)`
- `train_personalized_model(history, template=None)`
- `estimate_stress(trained_user, avg_rmssd, overall_score, daily_steps, hrv_history)`
- `save_personal_model(trained_user, path)`
- `load_personal_model(path)`
- `load_shared_template(path)`

`mindwatch_v2.joblib` stores the shared model template. Each user must calibrate their own model, and the resulting personal parameters must be retained separately.

## Evaluation Results

Personalized Ridge, evaluated on 14 eligible participants using chronological train/test splits:

| Metric | Result |
|---|---:|
| Mean participant MAE | 17.243 |
| Personal mean baseline MAE | 17.547 |
| Participants outperforming baseline | 9/14 |

## Limitations

Due to the personalized training model, developers should pay particular attention to the calibration and model integration process. Each user requires an individual Ridge Regression model trained on their own historical physiological measurements and corresponding self-reported stress labels. Developers must ensure that user data and model parameters are properly separated and that self-reported stress scores are used strictly as training labels rather than inference inputs.

The observed relationships between physiological indicators (HRV, sleep quality, and daily steps) and subjective stress levels are relatively weak and vary considerably across individuals.The model can only provide approximate stress estimatesgeneral trends.

## Integration Responsibilities

The ML module handles feature preparation, personal calibration, estimation, and model serialization.

The application/backend provides physiological records, optional calibration labels, user-specific history, and persistent storage. It does not need to implement Ridge training or calculate ML features itself.
