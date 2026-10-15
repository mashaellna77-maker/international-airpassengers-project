# Air Passenger Forecasting — Management Report

Prepared by: Mashael Alotaibi

## Recommendation

Use seasonal AutoARIMA for monthly planning: its rolling-origin MASE of 0.647 versus 1.313 for Seasonal Naive approximately halves the scaled forecast error. However, improve its prediction intervals before relying on them as calibrated risk limits.

## Data and historical pattern

Passenger numbers rise over time, with peaks and troughs repeating approximately every 12 months. Seasonal swings grow as passenger numbers increase, suggesting that a log transformation or multiplicative seasonality may be useful.

The dataset contains 144 monthly observations from January 1949 to December 1960, with no missing months. Passenger counts are measured in thousands.

STL decomposition shows that trend explains long-term growth, seasonality captures recurring annual variation, and the remainder is comparatively small. Trend strength rounds to 1.00 and seasonal strength to 0.99. The raw-series autocorrelation declines slowly, with annual peaks supporting models that represent both growth and seasonality. These historical patterns do not guarantee future accuracy.

## Model comparison and evaluation

AutoGluon compared Naive, Seasonal Naive, AutoETS, AutoARIMA, and Theta, and produced a weighted ensemble. Theta failed and fell back to Seasonal Naive; its score therefore does not represent an independently successful Theta model.

AutoGluon reports negative MASE, so values closer to zero are better. WeightedEnsemble led on one internal validation window, while AutoARIMA performed best on the separate final 12-month holdout. This ranking was used to shortlist a model, rather than establish its reliability across periods.

The shortlisted seasonal AutoARIMA was evaluated through StatsForecast against Seasonal Naive and Naive. All models used the same eight rolling origins, a 12-month forecast horizon, and a 12-month step. Each fit and MASE scaling denominator used only the training history available at its origin.

| Model | MASE | Scaled CRPS | 80% coverage |
|---|---:|---:|---:|
| Seasonal Naive | 1.313 | 0.062 | 51.0% |
| AutoARIMA | 0.647 | 0.034 | 69.8% |
| Naive | 2.119 | 0.110 | 68.8% |

MASE measures forecast error relative to seasonal changes in training history. Scaled CRPS evaluates the predictive distribution. Lower values are better for both.

AutoARIMA beats the floor on both measures and has coverage closer to the intended 80%. Naive has worse point and distributional scores than the floor, although its coverage is closer to the target. AutoARIMA’s window-level MASE ranges from 0.330 to 1.594, showing that its average advantage is not a guarantee for every period.

## Prediction intervals and their reliability

An 80% prediction interval should contain approximately 80% of observations over repeated forecasts. None of the tested models achieves that coverage in this backtest.

| Model | Average 80% interval width, thousands | Coverage |
|---|---:|---:|
| Seasonal Naive | 77.758 | 51.0% |
| AutoARIMA | 44.076 | 69.8% |
| Naive | 142.947 | 68.8% |

AutoARIMA produces the narrowest intervals and the highest observed coverage. Nevertheless, 69.8% remains below the nominal 80%, indicating underestimated uncertainty. Naive’s much wider intervals demonstrate that width alone does not ensure adequate coverage.

Use AutoARIMA’s central forecast with explicit recognition of this limitation. The measured coverage applies to 12-month forecasts; it does not validate the notebook’s 24-month illustration.

## What the errors reveal

Seasonal Naive repeats last year’s monthly values without extending growth. Its in-sample residuals average +31.77 thousand passengers and show strong short-lag correlation, indicating that growth and short-term structure remain unexplained.

AutoARIMA’s out-of-sample forecast errors average +5.88 thousand passengers, indicating average underprediction. Errors cluster above and below zero, with positive short-lag correlation and negative correlation at lag 12.

These are multi-step errors from annual origins. Dependence may arise within forecast windows, so this ACF alone does not establish correlation in one-step model residuals. The two error averages also use different periods and forecast settings and should not be treated as a like-for-like comparison.

## One specific next step

Test log-transformed AutoARIMA using the same rolling-origin design, correctly converting predictions and quantiles back into passenger units. This may better represent seasonal variation that increases with the series level.

Retain the change only if it improves distributional accuracy and moves 80% coverage closer to its target without materially worsening point accuracy. Any transformation choices must use training data within each origin.
