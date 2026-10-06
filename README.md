# Arrival Delay Classification for Scheduled Flights at AIBD

Interdisciplinary Elective project, M.Sc. Data Science, University of Europe for Applied Sciences, Potsdam.
Author: Mohameth Gueye. Module: Interdisciplinary Elective. Lecturer: Prof. Dr. Raja Hashim Ali.

## Objective

Classify whether a scheduled arrival at Blaise Diagne International Airport (AIBD) reaches its stand (BLOCK_ON) more than 15 minutes after its scheduled time (STA), using only information available before arrival. Three models are compared: a majority-class baseline, logistic regression, and histogram gradient boosting.

## Data

Official AIBD 2025 regular-arrivals ledger (10,863 flights). The data are governed and are NOT included in this repository.
Citation: [FILL: data citation as used in the report, with access date].

Data access: the file is not public. To rerun the notebook, obtain the ledger from the data owner [FILL: contact or agreement], save it as `data/aibd_2025_regular_arrivals.csv` and keep it out of version control (`data/` is git-ignored).

## Protocol

- Label: delayed = BLOCK_ON minus STA greater than 15 minutes (41.6% of all flights).
- Chronological split by scheduled month: January to October train, November validation, December held-out test.
- Features (7, all known before arrival): airline, flight number, aircraft type, origin, scheduled hour, day of week, weekend flag. Actual times, delay codes, passenger and baggage counts, stand, belt, gate, and remarks are excluded to avoid leakage. A guard in the notebook enforces this.
- Model selection on validation macro F1 (ROC AUC as tie-break) with small grids. Final models are refit on January to November and evaluated once on December. Fixed decision threshold of 0.5.
- Uncertainty: day-cluster bootstrap (1,000 resamples over the 31 test days).

## Repository structure

```
01_arrival_delay_classification.ipynb   full pipeline with outputs
results/                                          metrics, confidence intervals, error analysis, robustness, run_info.txt
figures/                                          confusion matrices, precision-recall curves
requirements.txt
```

## Setup and run

```
python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -r requirements.txt
jupyter lab                        # start from the project root
```

Open the notebook and choose Restart and Run All. Start Jupyter from the project root, because the notebook uses the relative paths `data/`, `results/` and `figures/`. Tested with Python 3.12.12 (exact versions in `results/run_info.txt`). Random seed: 42.

## Results (December test, 967 flights, 54.1% delayed)

| Model | Macro F1 (95% CI) | ROC AUC |
|---|---|---|
| Majority baseline | 0.315 | n/a |
| Logistic regression | 0.668 (0.636 to 0.694) | 0.749 |
| Gradient boosting | 0.644 (0.619 to 0.669) | 0.739 |

Both models clearly beat the baseline. Logistic regression is slightly ahead on macro F1, while ROC AUC is practically tied. Details: `results/`.

## Limitations

- One test month; the delayed share shifts from 40.9% (train) and 35.5% (validation) to 54.1% (test), so precision and macro F1 are not comparable across months.
- Fixed 0.5 threshold; a threshold sensitivity table is provided for diagnosis only.
- Schedule-only features give moderate discrimination.
- Whether STA is the original or the updated schedule for rescheduled flights is [FILL: answer or "unresolved"]; excluding them changes results only marginally (`results/robustness_row_exclusions.csv`).
- The bootstrap reflects test-day sampling only, not training variability.
- Airline names are anonymized in the saved tables.

## AI-use statement

The code was drafted with an AI assistant (Claude, Anthropic) in step-by-step form. The author selected the topic and design decisions, ran and checked every cell, interpreted the results, and wrote the report. The report itself was written by the author. [FILL: align wording with the AI-use statement in the report.]