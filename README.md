# Lending Club Borrower Risk Profile (Python EDA)

Which borrower characteristics are linked to loan default, and do they still matter once the loan grade is taken into account? An exploratory analysis of 100,000 completed Lending Club loans using **Python (pandas, matplotlib, seaborn)**.

This is Project 2 of a finance data analytics portfolio. [Project 1](https://github.com/soundharyasivaramans/lending-club-loan-default-sql) used SQL to study loan grade, term and purpose.

## Business question

Project 1 showed that loan grade and loan term are strongly linked to default. This project looks at the **borrower profile** and asks:

1. Which borrower characteristics (income, debt, employment, home ownership, credit history) are linked to higher default?
2. Do those patterns still hold **within the same loan grade**, or were they already captured by the grade?
3. Which characteristics separate low-risk from high-risk borrowers the most?

## Dataset

- **Source:** Lending Club loan data (Kaggle), cleaned in Project 1 into `loan_clean.csv`.
- **Scope:** 100,000 completed loans (Fully Paid or Charged Off), 28 columns.
- **Target:** `is_default` = 1 if the loan was Charged Off, 0 if Fully Paid. Overall default rate: **21.82%**.
- The dataset is not included in this repository. See "How to reproduce".

## Approach

1. **Data checks:** duplicates, missing values, extreme values.
2. **Feature creation:** turned raw numbers into groups (bands) for income, debt-to-income (DTI), credit utilization and loan-to-income, plus numeric employment length.
3. **Distributions:** loan amount, interest rate, income, DTI.
4. **Default rate by borrower characteristic:** 10 characteristics, with group sizes shown on every chart and groups under 500 loans removed.
5. **Correlation screen** of the numeric columns.
6. **Within-grade check:** default rate for every grade x characteristic combination, to see whether a pattern survives once grade is held constant.
7. **Ranking** of the drivers by the gap between the highest-risk and lowest-risk group.

## Data quality notes

- `emp_length` is missing for 6.62% of loans, `revol_util` for 0.06%.
- There are extreme values: `dti` up to 999, `annual_inc` up to 9,300,000 and `revol_util` up to 165.8. They were kept and handled by grouping into bands, so they do not distort the charts.
- No duplicate loan IDs.

## Key findings

**1. Single columns explain little.** Among borrower columns, no correlation with default is above 0.1 in absolute value (DTI 0.093, credit utilization 0.071, loan amount 0.060, annual income -0.044). Interest rate is higher (0.278) but Lending Club sets it from its own grade, so it is a risk assessment, not a borrower characteristic.

**2. Loan-to-income is the strongest steady pattern.** Default rises from 14.19% (loan under 10% of income) to 18.35%, 23.36% and 31.02% (30% or more). It holds in **every grade**: the gap between the lowest and highest group is 4.7 points in grade A, 9.6 in C and 20.8 in E.

![Default by loan-to-income](images/09_loan-to-income.png)

**3. Debt-to-income (DTI) rises steadily too.** From 16.21% (DTI 0-10) to 19.21%, 24.79% and 31.66% (30-40). The 40+ group (31.92%) has only 780 loans, so it is not reliable. Within grade, DTI still matters: in grade A default goes from 4.8% to 10.5%, in grade D from 28.2% to 39.8%.

![Default by DTI](images/07_debt-to-income.png)

**4. Higher income means lower default, but less so within grade.** 28.21% for incomes under 30k down to 15.61% for 150k+. Inside grades A, C, E and F the pattern holds (for example grade C: 29.1% down to 19.6%); inside B and D it is flat and noisy.

**5. Renters default more, in every grade.** RENT 25.94%, OWN 22.10%, MORTGAGE 18.51%. The order MORTGAGE < OWN < RENT appears in every grade, and the RENT vs MORTGAGE gap grows with risk: 2.8 points in grade A, 6.1 in C, 10.3 in E, 15.2 in F.

**6. Income verification looks backwards, but grade mix explains most of it.** Verified borrowers default *more* (27.47%) than Not Verified (16.16%). The reason is who gets verified: grades D-G make up 33.9% of Verified loans but only 13.2% of Not Verified loans, and 29.7% of Not Verified loans are grade A against 9.6% of Verified. Within each grade the gap shrinks from 11.3 points to between 1.7 (grade A) and 7.5 (grade E). Why verification is more common for riskier loans was not tested here.

![Default by income verification](images/04_income_verification.png)

**7. Missing employment information is a warning sign.** Borrowers with no employment length (6,620 loans) default at 29.59%, against 19.98% to 22.62% for every known group. Among known groups, years of employment is a weak driver (10+ years 19.98%, under 1 year 22.62%).

**8. Credit utilization is mostly captured by the grade.** Overall, default rises from 16.78% (0-25% utilization) to 25.16% (75-100%). Inside a grade it is nearly flat above 25% (grade C: 24.0%, 24.8%, 25.1%, 24.7%).

**9. Past delinquencies and public records matter modestly.** No delinquencies 21.34% vs 3 or more 25.41%. No public records 21.13% vs 2 or more 25.43%.

**10. Loan purpose matches Project 1.** Car loans are lowest (15.25%), small business loans highest (33.06%, but only 989 loans).

### Does the pattern hold within the same loan grade?

| Characteristic | Overall gap | Within grade |
|---|---|---|
| Loan-to-income | 16.8 pts | Holds in every grade |
| Debt-to-income | 15.7 pts | Holds in every grade |
| Home ownership | 7.4 pts | Holds in every grade |
| Annual income | 12.6 pts | Holds in some grades, noisy in others |
| Income verification | 11.3 pts | Mostly explained by grade mix |
| Credit utilization | 8.4 pts | Weak inside a grade |

![Default by grade and DTI](images/14_grade_by_debt_to_income.png)

### Ranking of drivers

Gap in default rate between the highest-risk and lowest-risk group of each characteristic:

| Driver | Lowest-risk group | Highest-risk group | Gap (pts) |
|---|---|---|---|
| Loan purpose | car (15.25%) | small_business (33.06%) | 17.81 |
| Loan-to-income | under 10% (14.19%) | 30%+ (31.02%) | 16.83 |
| Debt-to-income | 0-10 (16.21%) | 40+ (31.92%) | 15.71 |
| Annual income | 150k+ (15.61%) | under 30k (28.21%) | 12.60 |
| Income verification | Not Verified (16.16%) | Verified (27.47%) | 11.31 |
| Employment length | 10+ yrs (19.98%) | Unknown (29.59%) | 9.61 |
| Credit utilization | 0-25% (16.78%) | 75-100% (25.16%) | 8.38 |
| Home ownership | MORTGAGE (18.51%) | RENT (25.94%) | 7.43 |
| Public records | 0 (21.13%) | 2+ (25.43%) | 4.30 |
| Delinquencies | 0 (21.34%) | 3+ (25.41%) | 4.07 |

This is a rough ranking. The gap depends on how groups were cut, and the top groups for loan purpose (989 loans) and DTI (780 loans) are small. The employment gap is driven by the "Unknown" group; without it the gap is 2.64 points.

![Driver ranking](images/17_driver_ranking.png)

## Suggestions for a lender (to test, based on this data)

- Use **loan-to-income** and **DTI** as extra risk screens on top of the grade, since both separate borrowers inside every grade.
- Treat **missing employment information** as a risk flag.
- Do not use **income verification status** alone as a risk signal; compare borrowers within the same grade.
- Consider **home ownership** in reviews, since renters default more in every grade.

## Limitations

- **Current loans are excluded**, as in Project 1, so recent loans are only partly represented and final default rates are not known for them.
- Results show **association, not causation**.
- Groups with fewer than 500 loans were removed from the charts (for example credit utilization above 100%). In the grade x characteristic tables, cells under 200 loans are blank, which affects grades F and G the most.
- DTI 40+ has only 780 loans.
- Correlation only measures straight-line relationships.
- Columns that are only known after a loan ends (`total_pymnt`, `recoveries`) were not used to explain default.

## Skills demonstrated

Data quality checks, feature engineering with `pd.cut`, `groupby` and `pivot_table`, handling missing values and outliers, charts with matplotlib and seaborn, correlation analysis, and checking for confounding by comparing patterns within loan grade.

## How to reproduce

1. Download the Lending Club data from Kaggle and create `loan_clean.csv` with the cleaning script from Project 1.
2. Open `project2_borrower_risk_eda.ipynb` in Google Colab.
3. Upload `loan_clean.csv` when asked and choose **Runtime > Run all**.

## Repository structure

```
README.md
project2_borrower_risk_eda.ipynb
driver_summary.csv
images/        (charts created by the notebook)
```
