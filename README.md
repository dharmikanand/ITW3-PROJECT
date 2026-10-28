# Personal Finance Analyzer

**Course Project – IT Workshop III**

A Python tool that takes a bank statement CSV and shows where the money goes. It cleans the data, categorises every transaction, finds unusual and repeating payments, checks a monthly budget, forecasts next month's spending and stores everything in SQLite.

## Features

- **Cleaning** – removes duplicates and empty rows, fixes dates, descriptions, amounts and debit/credit types
- **Categorisation** – keyword dictionary with 8 categories (Food, Travel, Subscriptions, Shopping, Rent, Health, Entertainment, Bills), everything else goes to `Others`
- **Charts and savings** – spending by category, spending per month, savings rate
- **Anomaly detection** – unusual expenses per category (mean + 2 × standard deviation)
- **Recurring payments** – merchants paid every month for almost the same amount
- **Budget check** – OK / WARNING / ALERT against fixed monthly limits
- **Forecast** – next month's spending with Linear Regression
- **SQLite storage** – cleaned data saved to `finance.db` and queried with SQL

## Tech stack

Python, Jupyter Notebook, pandas, NumPy, Matplotlib, scikit-learn, sqlite3

## Project structure

```
├── finance_analyzer_simple.ipynb   # the full pipeline
├── bank_statement.csv              # input (date, description, amount, type)
├── finance.db                      # created by the notebook
├── Project_Report.pdf              # project report
└── images/                         # charts used in this README
```

## How to run

```bash
pip install pandas numpy matplotlib scikit-learn jupyter
jupyter notebook finance_analyzer_simple.ipynb
```

Run the cells from top to bottom. In the settings cell you can change:

| Setting | Meaning |
|---|---|
| `CSV_FILE` | the bank statement file (a Kaggle statement can be used) |
| `DAY_FIRST` | `True` for dates like 25-12-2025, `False` for 2025-12-25 |

If `bank_statement.csv` does not exist, the notebook creates a small synthetic statement so it can still run.

## Method

```
Clean → Categorise → Charts and savings → Anomaly detection → Recurring payments → Budget check → Forecast → SQLite
```

| Step | Rule |
|---|---|
| Categorise | first keyword match wins (order matters, e.g. `PRIME` is checked before `AMAZON`) |
| Anomaly | debit > mean + 2 × std within its category (categories with fewer than 5 rows skipped) |
| Recurring | seen in 3+ months and std < 5% of mean; yearly cost = mean × 12 |
| Budget | < 80% OK, ≥ 80% WARNING, ≥ 100% ALERT |
| Forecast | Linear Regression, x = month number (1–12), predict month 13 |

## Results

Dataset: 437 transactions, Jan–Dec 2025.

| Measure | Value |
|---|---|
| Money in | Rs. 6,52,775 |
| Money out | Rs. 4,77,966 |
| Savings | Rs. 1,74,809 (**26.8%** savings rate) |
| Highest month | September (about Rs. 69,000, due to a Rs. 28,000 Reliance Digital purchase) |
| Recurring payments | 5 payments, Rs. 1,80,822 per year (37.8% of spending) |
| Forecast for next month | **Rs. 38,990.85** |

![Spending by category](images/spending_by_category.png)
![Total spending per month](images/monthly_spending.png)

### Spending by category

| Category | Debit total (Rs.) | Share |
|---|---:|---:|
| Rent | 1,37,995.32 | 28.9% |
| Shopping | 71,841.06 | 15.0% |
| Others | 58,396.25 | 12.2% |
| Bills | 51,408.97 | 10.8% |
| Health | 50,152.82 | 10.5% |
| Food | 45,783.29 | 9.6% |
| Travel | 35,347.57 | 7.4% |
| Entertainment | 14,232.86 | 3.0% |
| Subscriptions | 12,807.97 | 2.7% |

### Unusual expenses

10 expenses were flagged. Five are genuine one-off purchases: Reliance Digital (Rs. 28,000), Flipkart (Rs. 15,000), a family restaurant bill (Rs. 9,000), Apollo Pharmacy (Rs. 6,800) and an IRCTC ticket (Rs. 4,500). The other five (four KSEB electricity bills and one PVR bill) are false positives or minor outliers, because the Bills category mixes small and large payments.

### Recurring payments

| Merchant | Months | Avg (Rs.) | Yearly cost (Rs.) |
|---|---:|---:|---:|
| HOUSE RENT PAID | 12 | 11,499.61 | 1,37,995.32 |
| FITNESS GYM MEMBERSHIP | 12 | 2,501.55 | 30,018.57 |
| NETFLIX SUBSCRIPTION | 12 | 648.94 | 7,787.22 |
| AMAZON PRIME MEMBERSHIP | 12 | 299.67 | 3,596.07 |
| SPOTIFY PREMIUM | 12 | 118.72 | 1,424.68 |

KSEB is correctly not listed because its amount changes every month.

### Budget check (March 2025)

Shopping is an **ALERT** (150% of its limit, caused by one Rs. 15,000 Flipkart order) and Rent is a **WARNING** (95.3%). The other seven categories are OK.

### Forecast

The trend line is almost flat and slightly downward. With only 12 data points, the forecast is a rough estimate.

![Forecast](images/forecast.png)

## Limitations

- Keyword rules do not know new merchants; they fall into `Others` until a keyword is added
- The forecast uses only 12 points
- Mean + 2 × std works best on roughly bell-shaped data; outliers cause false positives (electricity bills)
- Budget limits are fixed by hand and only one month is checked at a time
- Synthetic sample data is cleaner than a real bank statement

## Future scope

- Web dashboard (Gradio)
- Machine-learning categorisation
- Reading PDF statements
- Per-merchant anomaly detection

## Report

See [Project_Report.pdf](Project_Report.pdf) for the full project report.
