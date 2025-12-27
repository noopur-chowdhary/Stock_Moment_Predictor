**📈 Stock_Movement_Predictor (NVDA)**

This project aims to analyze NVIDIA (NVDA) stock using various technical indicators and predict the stock’s movement over the next 10 days.
The goal is not short-term trading execution, but to demonstrate an end-to-end machine learning workflow for time-series financial data, including feature engineering, feature selection, model training, and evaluation.

The implementation is provided in a Jupyter Notebook (stock.ipynb) to ensure clarity and reproducibility.

Below is a step-by-step walkthrough of the complete process, from data preparation and indicator analysis to model development and final results.

**1)Data Collection**
Historical NVIDIA stock data (January 2022 – December 2025) is downloaded using Yahoo Finance (yfinance).
Rationale for Period Selection
Starting in 2022, NVIDIA entered a new structural growth phase driven by AI infrastructure and data-center demand.
Pre-2022 price behavior reflects a different market regime and is therefore less relevant for modeling current and near-future price dynamics.NVIDIA entered a new structural growth phase starting in 2022, focussing on AI infrastructure.This period captures NVIDIA’s AI-era price dynamics, making it the most relevant window for current and near-future stock movement prediction.

**2. Feature Engineering: Technical Indicators**
Total Indicators Extracted: 60
Library Used:https://github.com/Crypto-toolbox/pandas-technical-indicators
Indicator Categories-Momentum, Volume, Volatility,Trend-based metrics
These indicators are designed to capture different market behaviors and investor sentiment over multiple time horizons.

**3. Target Variable Definition**
The task is framed as a binary classification problem.
Objective:
Predict whether the stock price moves up or down after 10 trading days.
Label Construction
pred = 1  if Close(t) > Close(t-10)
pred = 0  otherwise
This forward-looking target avoids look-ahead bias and aligns with real-world prediction constraints.



**4. Feature Matrix & Labels**
X (Features):
All technical indicators
(Close price and intermediate gain columns are excluded)
y (Target):
pred (binary movement label)


**4Train-Test Split**

Data split using TimeSeriesSplit to data leakage and ensures the model is always evaluated on future, unseen data, which is critical for time-series modeling.


**6. Feature Reduction (Statistical Filtering)**
Before model-based selection, non-informative features were removed:
 ->Zero-variance features
 ->Highly multicollinear features

22 features dropped:
['ATR_14', 'ROC_14', 'CCI_14', 'RSI_26', 'SO%d_26', 'ATR_26',
 'CCI_26', 'RSI_44', 'SO%d_44', 'ATR_44', 'CCI_44', 'Vortex_44',
 'RSI_66', 'SO%d_66', 'ATR_66', 'CCI_66', 'Vortex_66',
 'ema50', 'ema21', 'ema14', 'MACD_12_26', 'MACDsign_12_26']

This step improves model stability and reduces noise.

**6.Sequential Feature Selector(SFS)**
After statistical filtering, Sequential Feature Selection (SFS) was applied using a Random Forest classifier to identify the most predictive subset of features.
Final 7 features:
['EoM_5', 'SO%d_14', 'MFI_14', 'EoM_14', 'EoM_26', 'EoM_44', 'EoM_66']
This step balances model performance and interpretability.

**7.Model Training**
Baseline Model

 We trained a Random Forest Classifier using default values , using only SFS-selected features. This gave us a baseline model 

We were able to acheive 57 accuracy .

**8. Hyperparametre tuning using RandomisedSearchCV**

20 parameter combinations

2-fold cross-validation

Total fits: 40

{
  'n_estimators': 500,
  'min_samples_leaf': 5,
  'max_features': None,
  'max_depth': None,
  'class_weight': None
}

**9.Model Evaluation**
Confusion Matrix & Classification Report.
The best-performing Random Forest model achieved an accuracy of approximately 59% on unseen data. While this is marginally better than random guessing, performance was uneven across classes, with the model showing stronger predictive power for upward price movements than for downward movements.The recall for 0.0 is 18 percent which is disappointing 

**10.Probability-Based Signal Selection**

Instead of relying on hard class predictions, model output probabilities were analyzed and the decision threshold for an upward signal was increased  0.55. Then again we generated the classification report. Recall for 0.0 improved from 18% to 22 %

**Result**s-The final Random Forest model achieved an accuracy of approximately 59% on unseen data. Performance was uneven across classes, with the model consistently predicting upward price movements more effectively than downward movements. Initial evaluation showed low recall for downside signals, limiting the model’s usefulness for risk-aware decision making.To address this issue, predictions were generated using class probabilities instead of hard labels, and the decision threshold for an upward signal was increased from 0.50 to 0.55 This adjustment resulted in a slight improvement in downside recall, as fewer marginal upward predictions were accepted. However, the improvement was modest and came with a trade-off in precision, indicating that threshold tuning alone is insufficient to fully capture downside movements.
Even after probability-based selection, the model struggled to reliably detect downward price movements. This behavior is likely driven by class imbalance, the strong upward market regime during the AI-driven NVDA rally, and the lagging nature of technical indicators. These results highlight the inherent difficulty of directional stock prediction and the importance of evaluating models beyond aggregate accuracy.


**Future Improvements**

Potential improvements include explicit handling of class imbalance, adoption of more expressive models such as gradient boosting or sequence-based architectures, incorporation of regime-aware and market-wide features, and reframing the task as a multi-class or regression problem to better model return magnitude and downside risk.
