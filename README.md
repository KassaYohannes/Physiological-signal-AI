# Physiological Signal AI

Applied machine learning on physiological signals from intensive-care patients. This repository contains a foundational regression study using the BIDMC PPG and Respiration Dataset, focused on predicting a patient's respiration rate from routine vital signs.

The goal here is not a state-of-the-art model. It is clean, honest, end-to-end work: load real clinical signal data, explore it, fit an interpretable model, and report what it can and cannot do — including where it fails and why that matters clinically.

## Project: predicting respiration rate from vitals

**Question.** Can a patient's respiration rate (breaths/min) be predicted from two routinely monitored vitals — heart rate (HR) and blood oxygen saturation (SpO2)?

**Approach.** Multiple linear regression (ordinary least squares) across 53 ICU patients, with an 80/20 train/test split and a held-out evaluation.

**Pipeline.**
1. Load and concatenate the per-patient numerics files (53 patients, ~25,000 rows).
2. Clean the data: drop missing values and remove physiologically impossible rows (RESP = 0), leaving 24,988 rows.
3. Explore relationships with scatter plots and a correlation matrix.
4. Fit `RESP ~ HR + SpO2` on the training set and evaluate on the test set.
5. Inspect predictions against actual values to understand the model's behaviour.

## Results

| Metric | Value |
| --- | --- |
| Test RMSE | 2.91 breaths/min |
| Test R² | 0.089 |

Fitted relationship:

```
RESP = 42.26 + 0.015 * HR - 0.268 * SpO2
```

Both predictors are statistically significant, but SpO2 is by far the dominant one (its coefficient is ~18x larger than HR's, with a much stronger t-statistic). The negative SpO2 coefficient is clinically sensible: as oxygen saturation falls, patients tend to breathe faster to compensate.

## The honest finding

An R² of 0.089 means the two predictors together explain less than 9% of the variation in respiration rate. The predicted-vs-actual plot shows why: the model collapses toward predicting the population average (~17–18 breaths/min) for almost every patient. It performs acceptably in the normal range but fails at the extremes — exactly the very slow or very fast breathers a clinician most needs to flag.

This is the real takeaway, and it is worth stating plainly: heart rate and SpO2 alone are weak predictors of respiration rate in ICU patients. Respiration is driven by many other factors — underlying condition, medication, lung disease, pain. A two-variable linear model cannot capture that. The value of the analysis is in establishing that limitation rigorously rather than reporting a misleadingly optimistic number.

## Data

This project uses the **BIDMC PPG and Respiration Dataset**, available from PhysioNet. The dataset contains simultaneously recorded PPG, respiration, and ECG waveforms, plus extracted numerics (HR, SpO2, respiration rate, pulse), for 53 critically ill patients.

The raw data is **not** included in this repository. To reproduce the analysis, download the dataset from PhysioNet (search for "BIDMC PPG and Respiration Dataset"), unzip it, and place the `bidmc_csv` folder where the notebooks expect it. The regression notebook reads the per-patient `*_Numerics.csv` files (HR, SpO2, respiration rate); the exploration notebook reads the `*_Signals.csv` waveform files. Update the `data_path` variable at the top of each notebook to point to your local copy.

## Repository structure

```
.
├── notebooks/
│   ├── 01_signal_exploration.ipynb        # EDA: PPG, respiration and ECG waveforms
│   └── 02_project1_linear_regression.ipynb # Main analysis: load, clean, model, evaluate
└── results/                                # Saved figures
```

The exploration notebook (`01`) inspects the raw waveform morphology — individual heartbeats in the PPG signal, the breathing cycle, and the relationship between PPG, respiration, and ECG. The regression notebook (`02`) is the modelling work described above.

## Running it

Requires Python 3.10+. Install dependencies:

```bash
pip install numpy pandas matplotlib scikit-learn statsmodels ISLP
```

Then download the BIDMC dataset (see **Data** above), set `data_path` at the top of each notebook to point to your local `bidmc_csv` folder, and run the cells top to bottom.

## Tech stack

- **pandas / NumPy** — data loading and cleaning
- **statsmodels** — OLS regression with full statistical summary
- **scikit-learn** — train/test split and error metrics
- **matplotlib** — exploratory and diagnostic plots

## Notes

This is a learning-oriented project that prioritises a correct, interpretable, well-documented workflow over predictive performance. Every step in the notebook is commented to explain both the code and the clinical reasoning behind it.
