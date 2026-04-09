# Donor Propensity Prediction

A machine learning pipeline that helps a non-profit organization identify which donors are most likely to respond to a fundraising campaign, so that mailing budgets are spent on the highest-value prospects.

---

## Overview

Non-profits typically send fundraising letters to large donor lists at a non-trivial unit cost. Mailing everyone is wasteful; mailing too few leaves donations on the table. This project builds a **donor propensity model** that scores each donor's likelihood of giving in an upcoming campaign, enabling data-driven targeting.

The pipeline takes raw donor, gift, and campaign history, engineers behavioral and RFM-style features, trains and compares several classifiers, and outputs a ranked list of scored donors for the target campaign.

**Business outcome:** higher expected return per letter sent, by mailing only the top-scoring segment of the donor base.

---

## Data

Three raw tables provided by the non-profit (in `data/raw/`):

| File | Description |
|---|---|
| `donors.csv` | Donor demographics: zipcode, province, region, gender, language, date of birth |
| `gifts.csv` | Historical donations: donor, campaign, amount, date |
| `campaigns.csv` | Campaign metadata: date, letters sent, unit cost |
| `selection campaign <id>.csv` | Target donor lists for campaigns 6169, 7244, 7362 |

The modeling target is response to **campaign 7362**.

> **Note on the data:** the dataset is synthetic and was provided as part of the course. It does not come from a real non-profit organization, and no real donor information is included in this repository.

---

## Approach

1. **EDA & cleaning** — parse European number/date formats, handle missing values, sanity-check distributions.
2. **Feature engineering** — RFM-style metrics (recency, frequency, monetary), donation behavior patterns (spontaneous-donation rate, share of gifts above a €30 threshold), temporal features (average days between donations, donation trend, gifts per year), campaign-level features (cost-per-letter statistics, letters-sent averages), and demographic encodings.
3. **Modeling** — trained and compared **Logistic Regression**, **K-Nearest Neighbors**, a **Neural Network (MLPClassifier)**, and **XGBoost**, with stepwise feature selection used for the linear baseline.
4. **Evaluation** — validation AUC, lift curves, and a profit-based comparison against a "mail everyone" baseline.
5. **Scoring** — apply the best model to the campaign 7362 selection list and export ranked donors.

---

## Project Structure

```
.
├── data/
│ ├── raw/              # Source CSVs (donors, gifts, campaigns, selections)
│ └── processed/        # Cleaned & feature-engineered datasets
├── notebooks/
│ └── donor_propensity_prediction_notebook.ipynb            # End-to-end pipeline
├── reports/
│ ├── donor_propensity_prediction_presentation.pdf          # Results presentation delivered to the stakeholder
│ └── scored_donors_campaign_7362.xlsx      # Final ranked donor list
├── guidelines/             # Project brief and feature-selection notes
├── requirements.txt
└── README.md
```

---

## Getting Started

```bash
git clone https://github.com/romainready/donor-propensity-prediction
cd donor-propensity-prediction

conda create -n donor-env python=3.10 -y
conda activate donor-env
pip install -r requirements.txt

jupyter notebook notebooks/donor_propensity_prediction_notebook.ipynb
```

---

## Results

**Model comparison (validation AUC):**

| Model | AUC |
|---|---|
| **XGBoost** (selected) | **0.764** |
| Logistic Regression | 0.652 |
| Neural Network (MLP) | 0.567 |

XGBoost was chosen for its ability to capture non-linear interactions and handle the heavy class imbalance (only 181 responders out of 25,636 donors, ~0.7% response rate).

**Business impact — campaign 7362 simulation**

Assumptions: €0.80 per letter mailed, €30 average donation per responder.

| Strategy | Letters | Cost | Revenue | Profit |
|---|---|---|---|---|
| Mail everyone (baseline) | 25,636 | €20,509 | €5,430 | **−€15,079** |
| Mail top 2.5% by model score | 640 | €512 | €1,380 | **+€868** |

By targeting only the top 2.5% of donors ranked by the XGBoost score, the campaign moves from a **€15K loss to a small profit** — a swing of roughly **€15,900** on a single mailing, while still capturing ~25% of all responders.

## Outputs

- `reports/scored_donors_campaign_7362.xlsx` — donors ranked by predicted response probability, ready to feed into a mailing-decision rule (e.g. "mail the top N where expected return > unit cost").

---

## Tech Stack

![Python](https://img.shields.io/badge/Python-3776AB?logo=python&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-150458?logo=pandas&logoColor=white)
![NumPy](https://img.shields.io/badge/NumPy-013243?logo=numpy&logoColor=white)
![Scikit-learn](https://img.shields.io/badge/Scikit--learn-F7931E?logo=scikitlearn&logoColor=white)
![XGBoost](https://img.shields.io/badge/XGBoost-EC4A3F?logo=xgboost&logoColor=white)
![Matplotlib](https://img.shields.io/badge/Matplotlib-003366?logo=matplotlib&logoColor=white)
![Seaborn](https://img.shields.io/badge/Seaborn-4C72B0?logoColor=white)
![Jupyter](https://img.shields.io/badge/Jupyter-F37626?logo=jupyter&logoColor=white)
![Conda](https://img.shields.io/badge/Conda-44A833?logo=anaconda&logoColor=white)

---

## Team & Contributions

This project was completed as a group assignment for my Master in AI & Data Analytics for Business at IÉSEG.

**Team members:** Jules Reichard, Juan Sebastian Nuncira, Mehdi Zorkani, Romain Ready

**My contributions (Romain Ready):**
- Defined the business problem and chose success metrics aligned with the non-profit's ROI objective
- Contributed to the data cleaning and exploratory analysis of the donor, gift, and campaign datasets to surface behavioral patterns guiding the modeling phase
- Built the cost/revenue simulation showing how targeted mailing turns a €15K loss into a profitable campaign, and synthesized the findings to provide recommendations to the stakeholder