# supermarket-transaction-data
Sales Forecasting and Promotional Effectiveness Analysis
🎯 Project Goal
This project utilized PySpark on a Databricks platform to build and evaluate a Machine Learning model for forecasting retail sales amount. The core objective was to move beyond simple prediction and create a "What-If" analysis tool to assess the predicted lift (or drop) in sales resulting from different promotional strategies (vouchers, in-store displays, and feature placement).

📊 Dataset and Architecture
This pipeline processes four distinct datasets related to retail operations and promotions.

Layer

Source Datasets

Key Transformation / Output

Silver Layer

Sales, Item, Supermarkets, Promotion

Data Cleaning, Feature Engineering (Time, Location Indexing, Categorical Encoding).

Gold Layer

Silver Layer Views

4-Way Join (Sales, Item, Supermarket, Promotion) to create the unified feature table for ML training.

ML Model

Gold Layer

Trained Random Forest Regression Model for sales prediction.

✨ Key Features & Technical Highlights
1. Advanced Feature Engineering
Time Series: Sales timestamps were decomposed into cyclical features (hour_sin, hour_cos, cycle_day_x) to capture daily and weekly periodicity.

Categorical Encoding: Item properties (brand, type) and Supermarket location (postal_code) were converted to numerical indices for use in the model.

Promotion Indexing: Non-numerical promotional features (feature, display) were indexed to create promo_feature_indexed and promo_display_indexed features.

2. Robust Gold Layer Unification
The final feature table (gold_df) was created using a multi-step Spark SQL process involving four tables (sales, item, supermarkets, promotion). A critical final WHERE clause ensures that only sales records with a corresponding, valid promotion record are retained, effectively performing a secure INNER JOIN on the promotion data.

3. Model Training and Evaluation
Model Used: Random Forest Regression (PySpark MLlib)

Performance: The final model achieved an R 
2
  of approximately 0.29 on the holdout test set. While not perfect, this score demonstrated a non-random correlation and sufficient predictive power to support directional analysis.

Data Quality Note: Initial attempts to include lag features led to severe data leakage, confirming the need for rigorous feature selection.

🚀 "What-If" Analysis: Promotional Effectiveness
The deployed model was used to predict sales for a target product (3000005040) across various simulated scenarios. The goal was to quantify the sales lift/drop for different promotional mixes compared to the baseline.

Final Prediction Results
Scenario

Average Predicted Sales

Change from Baseline

Business Impact

03_Feature_Only

2.1039

+1.16%

Minor Predicted Lift

01_Baseline_OFF

2.0797

Baseline

Sales without any promotion.

02_Voucher_Only

2.0121

-3.25%

Predicted Sales Drop

05_Max_Combined

1.9261

-7.39%

Largest Predicted Drop

Key Business Insights

Promotions were likely used reactively to clear poorly performing inventory, rather than proactively boosting high-volume sellers. The model correctly learned: "Promotions are associated with low sales." To truly measure promotional lift, a price feature or discount percentage feature is required to separate the impact of the price reduction from the baseline sales trend.

Combining the sales forecast with the lead time for product delivery, the system can flag items that are projected to experience a stockout or overstock situation in the next 7-14 days. Identify perishable goods (e.g., fresh produce, baked goods) that have slow-moving trends or are frequently discounted, allowing management to reduce order quantities and minimize spoilage/waste.

⚙️ Testing and Verification
Sales Data Tests:

Cyclical Feature Test: Confirms the accurate decomposition of the time variable (e.g., hour) into the sin and cos components, ensuring correct capture of periodicity.

Null Imputation Test: Verifies that null values in sales-related features are correctly imputed with predefined fallback values.

Item Data Tests:

Size Parsing & Consistency: Ensures that item size strings are correctly parsed into a positive numeric value (size_value) and a consistent unit (size_unit).

Categorical Indexing: Confirms the string indexing process for brand and type features.

Supermarket Data Tests:

Postal Code Indexing Test: Verifies that the postal-code feature is correctly encoded into a numeric index and that the output type is double.

Promotion Data Tests:

Vector Assembler Test: Confirms that promotion categorical features (feature, display) are correctly indexed and assembled into a feature vector for consumption by the ML model.

Gold Layer Join Logic Tests (Pipeline Integrity)
Unit tests were implemented using the unittest library and PySpark to confirm the integrity of the Gold layer:

Test Focus: Verification of the complex four-way join and the final WHERE filter's behavior.

Outcome: Tests confirmed that the filtering correctly retains only records that have a valid entry in all four primary source tables, ensuring the ML model is trained on a c
omplete, unified feature set.

# Prerequisites and Setup
To run this project, you need the following:

Azure Resources
An Azure Databricks Workspace and an active cluster.

An Azure Data Lake Storage Gen2 (ADLS Gen2) Account with four empty containers created:

raw-bronze

cleaned-silver

gold

model

2. Databricks Secret Configuration (Required for Mounting)
The mount-adlsg2-storage.ipynb notebook uses a secure method (Storage Account Key) via Databricks Secrets to authenticate and create permanent mount points.

⚠️ SECURITY WARNING: The actual storage key is not stored in this public repository. You must set up your own secret scope and keys to run the notebook.

Create a Secret Scope: Create a Databricks secret scope (e.g., using the Databricks CLI).

Bash

## Example using Databricks CLI
databricks secrets create-scope --scope <your-scope-name>
Add Required Secrets: Store your ADLS Gen2 credentials in the new scope using the following key names:

Scope: sm-trans-sscope (or your chosen scope name)

Key 1 (storageaccount): The name of your ADLS Gen2 storage account (e.g., mystorageacct123).

Key 2 (strans-key): The full Access Key for your storage account.

3. Execution Steps
Import: Upload the mount-adlsg2-storage.ipynb notebook to your Databricks workspace.

Connect: Attach the notebook to your running cluster.

Run: Execute the mount-adlsg2-storage.ipynb notebook. If successful, you will have four new mount points accessible via DBFS:

/mnt/raw-bronze

/mnt/silver

/mnt/gold

/mnt/model

You can verify the mounts by running dbutils.fs.ls("/mnt/").
