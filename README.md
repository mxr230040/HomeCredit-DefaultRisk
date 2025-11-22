# Credit Risk Modeling (AWS End-to-End Project)
An end-to-end credit risk analytics project using the multi-source **Home Credit Default Risk** dataset. This project simulates how enterprise risk teams (e.g., Capital One, Chase) build datasets, govern data, perform SQL EDA, and prepare for PD (Probability of Default) modeling.

## 📂 Dataset Summary  
Source: Kaggle – *Home Credit Default Risk*  
Link: https://www.kaggle.com/competitions/home-credit-default-risk/data

| File | Description |
|------|-------------|
| `application_train.csv` | Main loan application (demographics, income, credit amount, external credit scores). |
| `bureau.csv` | External credit history at other institutions. |
| `bureau_balance.csv` | Monthly delinquency status for each bureau loan. |
| `previous_application.csv` | Past applications submitted by the customer. |
| `installments_payments.csv` | Installment loan repayment behavior. |
| `credit_card_balance.csv` | Monthly credit card usage & arrears. |
| `POS_CASH_balance.csv` | Point-of-sale loan performance. |

---

## 🟦 Data Ingestion & Governance (S3 + Glue + Athena)

All seven datasets were ingested into an **S3 raw data zone**, establishing a centralized storage layer representing the system of record.  
A dedicated folder structure (`raw/`, `clean/`, `scored/`) separates ingestion from downstream processing.

Each dataset was registered in the **AWS Glue Data Catalog** via automated crawlers, producing governed schemas with column definitions, datatypes, and metadata.  
This enables consistent discoverability and schema management across the environment.
AWS Athena was configured as the SQL query layer, with query outputs stored in S3.  

This setup mirrors enterprise credit risk environments by ensuring:

- centralized, immutable raw data  
- schema consistency  
- auditable SQL access  
- reproducible analysis  
- clear ingestion → catalog → query lineage

- ## 🗃️ Data Model & Relationship Mapping (Athena)

- `application_train` is the primary entity describing each customer’s active application.  
- `bureau` and `bureau_balance` provide a view of the customer’s external credit obligations and delinquency cycles.  
- `previous_application` captures the customer’s historical interactions with the lender, including approvals, refusals, and prior loan amounts.  
- `installments_payments` contains detailed repayment behavior that often drives PD model lift.  
- `credit_card_balance` and `POS_CASH_balance` contribute revolving credit and short-term loan behavior.

This multi-source structure mirrors real-world consumer risk systems, where the borrower’s creditworthiness is assessed using a combination of demographic attributes, external bureau information, and repayment history across various loan products.

                            ┌─────────────────────────────────────────────┐
                            │            application_train                │
                            │        One row per customer application     │
                            │                Primary Key: SK_ID_CURR      │
                            └───────────────────────────┬─────────────────┘
                                                        │
                          ┌─────────────────────────────┼─────────────────────────────┐
                          │                             │                             │
                          ▼                             ▼                             ▼

        ┌────────────────────────────────┐   ┌──────────────────────────────┐   ┌──────────────────────────────┐
        │             bureau             │   │     previous_application     │   │     credit_card_balance      │
        │  All external loans for a      │   │  All past applications the   │   │  Monthly credit card status  │
        │  customer across institutions  │   │  customer submitted to lender│   │  and balances                │
        │  PK: SK_ID_BUREAU              │   │  PK: SK_ID_PREV              │   │  Linked by SK_ID_CURR        │
        │  FK → SK_ID_CURR               │   │  FK → SK_ID_CURR             │   └───────────────┬────────────┘
        └──────────────────┬─────────────┘   └──────────────────┬──────────┘                   │
                           │                                  │                                ▼
                           │                                  │                 ┌──────────────────────────────┐
                           ▼                                  ▼                 │        POS_CASH_balance      │
        ┌────────────────────────────────┐   ┌──────────────────────────────┐   │  Monthly POS loan behavior  │
        │        bureau_balance          │   │     installments_payments    │   │  FK → SK_ID_CURR            │
        │  Monthly history for each      │   │  Repayments for each         │   └──────────────────────────────┘
        │  bureau loan (time series)     │   │  previous loan               │
        │  FK → SK_ID_BUREAU             │   │  FK → SK_ID_PREV             │
        └────────────────────────────────┘   └──────────────────────────────┘

---

## Feature Engineering & Master Modeling Table (SageMaker + Python)

Customer-level credit risk features were engineered across all seven datasets to prepare inputs for Probability of Default (PD) modeling.  
Feature groups included:

- **Application features:** income ratios, loan-to-goods ratios, age/employment indicators  
- **Bureau features:** external loan counts, overdue history, credit sums, prolongations  
- **Bureau balance features:** monthly delinquency patterns and status transitions  
- **Previous application features:** past loan outcomes, approval/refusal behavior, requested amounts  
- **Installment payment features:** late payment frequency, installment-to-payment ratios  
- **Credit card features:** utilization trends, delinquency flags, limit behavior  
- **POS cash loan features:** monthly performance and delinquency indicators  

All features were aggregated to **SK_ID_CURR** and merged into a unified **master modeling table**, stored in another S3 Bucket.

---

## 🟦 PD Model Development (SageMaker)

Baseline Probability of Default (PD) models were trained using the engineered dataset:

- **Logistic Regression** for interpretability and baseline risk separation  
- **XGBoost** for stronger nonlinear performance and interaction handling  

Models were evaluated using standard credit risk metrics:

- **AUC (Area Under Curve)**
- **KS Statistic**
- **Score decile rank-ordering**
- **Lift and capture rate curves**
- **Confusion matrix and event rate checks**

Model outputs included customer-level PD scores, feature importance rankings, and decile performance tables.

---

## 🟨 Risk Insights & Portfolio Analysis

Portfolio-level insights were generated using the model outputs and engineered features.  
Examples include:

- **Risk segmentation** by income band, employment stability, external credit score, and repayment quality  
- **Rank-ordering validation**, confirming high-risk deciles capture most defaults  
- **Behavioral analysis**, highlighting delinquency frequency and utilization as key drivers  
- **Feature importance review**, identifying variables with the highest predictive contribution  
- **Customer-level risk flags**, supporting decisioning and potential scorecard use  

These insights reflect typical analyses performed in credit risk organizations to validate model behavior and portfolio implications.

---

## 🟧 Dashboarding & Visualization (Power BI)

Interactive dashboards were created in Power BI to present both modeling outcomes and portfolio insights.  
Dashboards included:

- **Portfolio Overview:** event rate, customer count, risk distribution  
- **Score Decile Analysis:** PD distribution, rank-ordering, lift  
- **Applicant Profile Segmentation:** income, employment, family size, loan purpose  
- **Behavioral Risk Views:** repayment patterns, credit utilization, delinquency cycles  
- **Feature Importance Visuals:** key drivers of model predictions  

These dashboards allow stakeholders to understand customer risk profiles, monitor portfolio behavior, and evaluate model performance.


