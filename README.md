# UK-Energy-Demand-Forecasting-2009-2014
This project predicts half-hourly UK electricity demand using machine learning models trained on historical grid data. The goal is to capture both short-term dynamics and long-term structure (seasonality and trends).

The final tuned LightGBM model achieves an RMSE of ~360 MW on unseen 2024 data, demonstrating strong predictive performance for real-world forecasting scenarios.

# Overview

Electricity demand is driven by multiple interacting factors:

- Time of day (daily cycles)
- Day of week (weekday vs weekend behaviour)
- Seasonal variation (winter vs summer demand)
- Recent demand history (autocorrelation)

This project builds a forecasting pipeline that captures all of these effects using feature engineering + tree-based models.

# Dataset
Source: Kaggle (UK National Grid demand data)
Time range: 2009 – 2024
Resolution: Half-hourly demand (MW)

# 1. Long-term Trends in UK Energy Demand

<p align="center"> <img src="images/Demand_by_year.png" width="800"> </p>

The full dataset (2009–2024) shows several important characteristics:

- Strong daily and seasonal cycles
- Consistent winter peaks and summer troughs
- A clear long-term decline in peak demand

This downward trend is likely driven by:

- Improvements in energy efficiency
- Changes in industrial demand
- Increased contribution from distributed generation

This context is important — the model must learn both:

- Short-term structure (daily cycles)
- Long-term behaviour (trend + seasonality)

# 2. Baseline Model - Random Forest

Prediction Performance (Example Day)
<p align="center"> <img src="images/May-2023-day.png" width="700"> </p>

The Random Forest model performs reasonably well:

- Captures the overall daily shape
- Tracks general increases and decreases in demand

However, it tends to:

- Smooth sharp transitions
- Slightly lag behind real peaks

Feature Importance – Random Forest
<p align="center"> <img src="images/May-2023-day-RF-feature-importance.png" width="700"> </p>

This is where the key issue appears:

- The model is almost entirely dominated by lag_1 (previous timestep demand)
- Other features contribute negligibly

The model has effectively learned that "Tomorrow is proportional to today". This works because electricity demand is highly correlated, but it’s neither robust nor insightful.

# 3. Improving the Model - LightGBM

To address this, a LightGBM model was trained with the same feature set but with better handling of feature interactions and boosting.

Prediction Performance (Test Week – Jan 2024)
<p align="center"> <img src="images/Jan-LightGBM-boosted-week.png" width="800"> </p>

The LightGBM model shows:

- Strong alignment with actual demand
- Better handling of peak demand periods
- Improved tracking of daily structure across multiple days

Feature Importance – LightGBM
<p align="center"> <img src="images/Jan-LightGBM-boosted-week-feature-importance.png" width="700"> </p>

Unlike Random Forest, the model now uses a broader set of features:

hour → dominant driver (daily demand cycle)
lag_1 → still important, but no longer overwhelming
rolling_7d_mean, rolling_24h_mean → capture trends
month → seasonal behaviour
lag_168 → weekly repetition

Interpretation

The model now reflects how demand actually works:

- Time of day drives usage patterns
- Demand depends on both recent history and longer-term trends
- Weekly and seasonal cycles are explicitly learned

# 4. Key Takeaways

Electricity demand is highly autocorrelated, but:
- Relying only on lag features leads to shallow models

Random Forest defaulted to a “copy the previous value” strategy
LightGBM successfully:
- Distributes importance across meaningful features
- Captures real-world structure
- Improves both performance and interpretability
