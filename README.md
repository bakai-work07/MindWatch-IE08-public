# MindWatch ML

MindWatch predicts a user's next-day self-reported stress level on a 0–100 scale using a multimodal machine learning model。

## Models

MindWatch provides two complementary machine learning models for stress monitoring and prediction.

### 1. Next-Day Stress Prediction Model

The prediction model predicts the user's next-day self-reported stress level (0–100) when the current self-reported stress value is available. The model uses median imputation for missing measurements followed by Linear Regression.

six features:
- Current self-reported stress
- Participant historical mean stress
- Daily steps
- Overall sleep score
- Deep sleep duration
- HRV (RMSSD)

performance on the time-based test set:
- MAE: 15.483
- RMSE: 21.498
- R²: 0.264

stored at: `models/stress_predictor_multimodal_v1.joblib`

### 2. Current Stress Monitoring Model

The monitoring model estimates the user's current stress level (0–100) without requiring the current self-reported stress value as an input. The model uses median imputation for missing measurements followed by Linear Regression.

five features:
- Participant historical mean stress
- Daily steps
- Overall sleep score
- Deep sleep duration
- HRV (RMSSD)

performance on the time-based test set:
- MAE: 16.789
- RMSE: 22.126
- R²: 0.237

stored at: `models/stress_monitoring_multimodal_v1.joblib`

## Model Roles

| Model | Current Self-Report Required | Purpose | Output |
|---|---|---|---|
| Next-Day Stress Prediction | Yes | Stress forecasting | Next-day stress (0–100) |
| Current Stress Monitoring | No | Passive monitoring | Current stress (0–100) |

## App Integration

MindWatch provides two independent model interfaces. The application team can select either model depending on the intended application design.

### 1. Next-Day Stress Prediction

Use `src/predict.py` to predict the user's next-day stress level when the current self-reported stress value is available.

test example:
```python
from src.predict import predict_from_user_history
prediction = predict_from_user_history(
    current_stress=60,
    stress_history=[35, 50, 42, 38, 55],
    daily_steps=8000,
    overall_score=80,
    deep_sleep_in_minutes=70,
    avg_rmssd=45
)
print(prediction)
```

The function returns the predicted next-day stress level as a float between 0 and 100.

`stress_history` should contain only stress scores recorded before the current day. Wearable parameters may be set to `None` when unavailable.

### 2. Current Stress Monitoring

Use `src/monitor.py` to estimate the user's current stress level.

test example:
```python
from src.monitor import estimate_from_user_history
stress_estimate = estimate_from_user_history(
    stress_history=[35, 50, 42, 38, 55],
    daily_steps=8000,
    overall_score=80,
    deep_sleep_in_minutes=70,
    avg_rmssd=45
)
print(stress_estimate)
```

The function returns the estimated current stress level as a float between 0 and 100.

`stress_history` should contain previous self-reported stress scores. Wearable parameters may be set to `None` when unavailable.

An empty stress history can also be provided for a new user. In this case, the missing historical stress feature is handled by the model's imputation pipeline.

## Project Structure

```text
MindWatch-ML/
    data/
        processed/
            daily_model_v1.csv
    notebooks/
        04_final_multimodal_model.ipynb
        05_current_stress_monitoring.ipynb
    models/
        stress_predictor_multimodal_v1.joblib
        stress_monitoring_multimodal_v1.joblib
    src/
        predict.py
        monitor.py
    requirements.txt
    .gitignore
    README.md
```