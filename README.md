# supermarket-transaction-data
Sales Forecasting and Promotional Effectiveness Analysis

##  Project Goal
This project uses PySpark on a Databricks platform to build and evaluate a machine learning model for forecasting retail sales. The objective goes beyond simple prediction by enabling a "What-If" analysis tool to estimate the lift or drop in sales resulting from different promotional strategies (vouchers, in-store displays, and feature placement).

---

## Dataset and Architecture Overview
This pipeline processes four key datasets related to retail operations and promotions.

### Layer Overview

| Layer        | Source Datasets                                | Key Transformations / Outputs                                                                 |
|--------------|-----------------------------------------------|------------------------------------------------------------------------------------------------|
| **Silver**   | Sales, Item, Supermarkets, Promotion          | Data cleaning and feature engineering (time features, location indexing, categorical encoding). |
| **Gold**     | Silver layer views                            | Four-way join (Sales, Item, Supermarket, Promotion) to create the unified feature table for ML training. |
| **ML Model** | Gold layer                                    | Trains machine learning models using the unified feature table. |

---

## Key Features & Technical Highlights

### 1. Advanced Feature Engineering
- **Time Series:** Sales timestamps were decomposed into cyclical features (`hour_sin`, `hour_cos`, `cycle_day_x`) to capture daily and weekly periodicity.
- **Categorical Encoding:** Item properties (`brand`, `type`) and Supermarket location (`postal_code`) were converted to numerical indices for use in the model.
- **Promotion Indexing:** Non-numerical promotional features (`feature`, `display`) were indexed to create `promo_feature_indexed` and `promo_display_indexed`.

### 2. Robust Gold Layer Unification
- The final feature table (`gold_df`) was created using a multi-step Spark SQL process involving the four tables (Sales, Item, Supermarkets, Promotion). 
- A critical final `WHERE` clause ensures that only sales records with a corresponding valid promotion record are retained, effectively performing a secure INNER JOIN on the promotion data.

### 3. Model Training and Evaluation
- **Model Used:** Random Forest Regression (PySpark MLlib)
- **"What-If" Analysis:** The deployed model predicts sales for a target product (`3000005040`) across various simulated scenarios, quantifying sales lift/drop for different promotional mixes.

---

##  Final Prediction Results

| Scenario           | Avg Predicted Sales | Change vs Baseline | Business Impact          |
|-------------------|-------------------|------------------|-------------------------|
| 01_Baseline_OFF    | 2.0797            | Baseline         | Sales without promotion |
| 03_Feature_Only    | 2.1039            | +1.16%           | Minor predicted lift    |
| 02_Voucher_Only    | 2.0121            | -3.25%           | Predicted sales drop    |
| 05_Max_Combined    | 1.9261            | -7.39%           | Largest predicted drop  |

### Key Business Insights
- Promotions were often used reactively to clear poorly performing inventory rather than proactively boosting high-volume sellers.
- The model learned: *"Promotions are associated with low sales."*
- To measure true promotional lift, include a price or discount percentage feature.
- By combining sales forecasts with lead times, the system can flag potential stockouts or overstocks within 7–14 days and help reduce spoilage/waste of perishable goods.

---

## ⚙️ Testing and Verification

### Sales Data Tests
- **Cyclical Feature Test:** Confirms correct decomposition of time variables (`hour`) into sine and cosine components.
- **Null Imputation Test:** Ensures null values in sales-related features are imputed correctly.

### Item Data Tests
- **Size Parsing & Consistency:** Verifies `size_value` and `size_unit` parsing.
- **Categorical Indexing:** Confirms proper string indexing for `brand` and `type`.

### Supermarket Data Tests
- **Postal Code Indexing:** Ensures postal codes are converted to numeric indices (`double` type).

### Promotion Data Tests
- **Vector Assembler Test:** Confirms promotion categorical features are correctly indexed and assembled into a feature vector.

### Gold Layer Join Logic Tests
- **Unit Tests:** Implemented with `unittest` and PySpark.
- **Focus:** Verification of the four-way join and `WHERE` filter behavior.
- **Outcome:** Ensures only records with valid entries in all four tables are used for ML training.

---

## Prerequisites and Setup

### Azure Resources
- Azure Databricks Workspace with an active cluster.
- Azure Data Lake Storage Gen2 (ADLS Gen2) account with the following empty containers:
  - `raw-bronze`
  - `cleaned-silver`
  - `gold`
  - `model`
- An Azure Key Vault instance containing your ADLS Gen2 credentials.

---

### Azure Key Vault Configuration
1. **Create a Key Vault**
   - Ensure it is in the same subscription/region as your Databricks workspace.
   - Add two secrets to store your ADLS Gen2 credentials:
     - `storageaccount` – Name of your ADLS Gen2 storage account
     - `storagekey` – Full access key for your storage account

2. **Grant Databricks Access**
   - Assign the Databricks workspace Managed Identity **"Get"** and **"List"** permissions on the Key Vault secrets.
   - This allows notebooks to read the secrets securely.

---

### Databricks Notebook Setup
1. **Mount ADLS Gen2 Storage Using Key Vault**
```python
# Read secrets from Azure Key Vault
storage_account = dbutils.secrets.get(scope="azure-key-vault-scope", key="storageaccount")
storage_key = dbutils.secrets.get(scope="azure-key-vault-scope", key="storagekey")

configs = {
    "fs.azure.account.key.{0}.dfs.core.windows.net".format(storage_account): storage_key
}

# Mount raw-bronze container
dbutils.fs.mount(
    source = f"abfss://raw-bronze@{storage_account}.dfs.core.windows.net/",
    mount_point = "/mnt/raw-bronze",
    extra_configs = configs
)

# Repeat mounts for silver, gold, and model containers
