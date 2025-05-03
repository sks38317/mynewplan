# Climate-Driven Societal Collapse Simulation Plan

This document outlines a roadmap for constructing a simulation to predict how global climate change from 2025 to 2045 may impact abnormal weather patterns, natural disasters, and the collapse of social systems across various regions.

The simulation links regional climate projections → frequency of extreme events → propagation of social unrest, incorporating not only climate data but also socio-structural variables such as food, politics, and trade.

---

## 1. Classification of Regions by Climate Zone and Historical Data Collection

### Objective
- Collect information on cities, states, and countries belonging to the 31 Köppen-Geiger climate zones.
- Build regional climate profiles using observed data from 2005–2024.

### Method
- Use **Köppen-Geiger 1 km resolution GeoTIFF map** to classify the climate zone of each location.
- Match cities to climate zones using **GeoNames city coordinate data**.
- Extract monthly 2-meter temperature from ERA5-Land and convert to annual averages.
  - Apply **land mask** to exclude ocean tiles.
- Aggregate exogenous variables annually: CO₂, CH₄, TSI, SAOD, ENSO, PM₂.₅.
- Outputs:
  - `cities_by_zone.json`: Cities mapped to climate zones.
  - `annual_temp_zone.csv`: Annual average temperatures for 31 climate zones over 20 years.
  - `features_global.csv`: Annual values of exogenous variables.

---

## 2. Constructing Annual Temperature Prediction Equations (Including Nonlinear Terms)

### Objective
- Develop realistic predictive equations that account for acceleration, interaction, and seasonality among variables.

### Method
- Create derived features such as ΔCO₂, (ΔCO₂)², ΔCH₄, ENSO³, ΔCO₂×ENSO.
- Incorporate **Fourier seasonal terms (SIN/COS 1st and 2nd order)**.
- Use **PolynomialFeatures + LassoCV** to train equations per climate zone.
  - Automatically eliminate less important features.
- Outputs:
  - `temp_model_coefs.json`: Equation terms and coefficients per zone.
  - `temp_model_metrics.csv`: Performance metrics (RMSE, MAPE, R²).

---

## 3. Model Evaluation and Structural Improvement

### Objective
- Validate how well the equations match real temperature changes and modify structure if needed.

### Method
- Measure error using RMSE, MAPE, and R².
- For regions with high error:
  - Adjust model complexity (increase polynomial degree).
  - Test alternative models like XGBoost or Gaussian Processes.
- Add seasonal terms to correct winter bias in high-latitude areas.
- Analyze temporal and spatial autocorrelation of residuals.
- Output:
  - Updated equations for all zones with improved accuracy.

---

## 4. Predicting Frequency of Extreme Weather and Disasters

### Objective
- Estimate regional frequency or probability of extreme events based on temperature predictions and exogenous variables.

### Method
- Inputs: temperature projections, ENSO, CO₂, SAOD, PM₂.₅, resilience index.
- Outputs: frequency or probability of events like heatwaves, heavy rainfall, and strong winds.
- Use XGBoost regression models for zone-level prediction.
- Normalize results to generate **hᵢ(t)** for RFIM input.

---

## 5. Constructing External Stimuli Vector hᵢ(t) for RFIM Input

### Objective
- Combine extreme event intensity and social resilience to create input values for the RFIM model.

### Method
- hᵢ(t) is a weighted sum of:
  - Normalized extreme event frequency
  - Damage cost or damage-to-GDP ratio
  - Resilience index (e.g., ND-GAIN)
  - Greenhouse gas and temperature indicators
- Output:
  - `h_field.npy`: Stimulus time series for 31 climate zones (2025–2045).

---

## 6. RFIM-Based Simulation of Social Unrest and Collapse

### Objective
- Simulate the probability of regional collapse due to climate shocks and influence from neighboring zones.

### Method
- Nodes: 31 climate zones (expandable to states).
- States: Stable (+1) / Unstable (–1).
- Interaction matrix Jᵢⱼ:
  - Built from climate distance + trade flows + migration data.
- Use sensitivity parameter β to adjust neighbor influence.
- Simulation runs annually or biennially from 2025 to 2045.
- Output:
  - `CollapseProb_2045.xlsx`: Collapse probabilities by region.

---

## 7. Incorporating Socioeconomic and Geopolitical Factors into RFIM

### Objective
- Integrate societal structure variables into RFIM to influence collapse transitions.

### Method
- Add the following variables per node:
  - **Food self-sufficiency ratio**: Production/consumption.
  - **Trade dependency**: Total imports + exports / GDP.
  - **Political system and instability index** (e.g., WGI).
  - **Military strength and conflict history**.
  - **Cultural cohesion index** (social capital, language divides).
  - **Diplomatic network strength**.
- Application:
  - External field hᵢ(t): Includes resilience components from these variables.
  - Interaction matrix Jᵢⱼ: Adjusted based on trade and diplomatic ties.
- Output:
  - `CollapseProb_2045_social.csv`: Simulation results with societal variables.

---

## 8. Forecast Validation and Sensitivity Analysis

### Objective
- Validate forecasts using historical data (e.g., 2005–2020).
- Improve model robustness by identifying sensitive variables and parameters.

### Method
- Backtesting: Exclude certain years and compare predictions with actual outcomes.
- Evaluation metrics:
  - AUC (binary classification accuracy).
  - Brier Score (probabilistic prediction accuracy).
- Sensitivity analysis:
  - Vary β, α and observe effects on results.
- Outputs:
  - `evaluation_report.md`: Validation report.
  - `sensitivity_matrix.csv`: Sensitivity of key parameters.
