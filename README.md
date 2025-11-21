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
