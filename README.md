# Anomaly Detection - Project Summary Report
---

## 1. Executive Summary

This report outlines the approach, methodology, and results of the anomaly detection analysis performed on the provided system log dataset. The objective was to identify anomalous records within over 60,000 log entries containing attributes such as browser, OS, ASN, and user operations.

My analysis employed a robust **Unsupervised Ensemble Learning** approach, combining **Isolation Forest**, **Local Outlier Factor (LOF)**, and **DBSCAN**. This multi-model strategy was chosen to minimize false positives and capture different types of anomalies (point anomalies, contextual anomalies, and collective anomalies).

**Key Findings:**
- **Data Quality:** The dataset contained significant duplicates (~75%) and high missingness in specific fields like `device_os_version` (93%).
- **Anomalies Detected:** 
    - **226** anomalies identified by Majority Vote (at least 2 models agreeing).
    - **17** high-confidence anomalies identified by Unanimous Vote (all 3 models agreeing).
- **Nature of Anomalies:** Detected issues include critical error statuses, unusually high-frequency events, and rare operations performed during non-standard hours.

---

## 2. Data Analysis and Visualization

An extensive Exploratory Data Analysis (EDA) was conducted to understand the data distribution and quality.

### 2.1 Data Quality Assessment
- **Dataset Size:** 63,713 records with 18 columns.
- **Missing Values:** `device_os_version` was missing in 93% of records, suggesting it is an optional or recently added field.
- **Duplicates:** A large portion of the dataset consisted of duplicate log entries. These were handled by aggregating them while preserving the frequency count, ensuring the model focuses on unique behavioral patterns rather than raw volume.

### 2.2 Key Insights & Visualizations
- **Frequency Distribution:** The `frequency` attribute is highly right-skewed (Mean: 4.45, Max: 514), indicating that most logs represent single events, but a few represent massive aggregations of activity.
- **Error Rates:** Approximately 20% of logs indicated an error status.
- **Temporal Patterns:** Analysis of timestamps revealed distinct patterns in user activity, allowing for the creation of time-based features (e.g., business hours vs. night hours).
- **Categorical Diversity:** Fields like `asn` (Autonomous System Number) showed high variability, indicating traffic from diverse networks.

---

## 3. Feature Engineering and Selection

To transform raw logs into a format suitable for machine learning, we implemented a comprehensive feature engineering pipeline.

### 3.1 Feature Selection
I had selected and engineered features that capture **behavioral context** and **rarity**. Key features include:
- **Temporal Features:** `is_weekend`, `is_business_hours`, `is_night_hours`.
- **Rarity Indicators:** `is_rare_operation`, `is_rare_browser`, `is_rare_isp`.
- **Categorical Flags:** `has_browser`, `has_device_os`, `has_country`.
- **Numerical Features:** `frequency` (scaled), `error_status`.

### 3.2 Transformation Process
- **One-Hot Encoding:** Applied to low-cardinality categorical variables to make them machine-readable.
- **Scaling:** `RobustScaler` was used for numerical features like `frequency` to handle outliers effectively without distorting the data distribution.
- **Traceability:** An `original_index` column was preserved throughout the pipeline to ensure every detected anomaly could be traced back to its exact row in the raw CSV file.

---

## 4. Model Architecture and Strategy

Since the dataset lacks labeled "ground truth" for anomalies, we adopted an **Unsupervised Ensemble Approach**. No single model is perfect; by combining three distinct algorithms, we achieve a more reliable detection system.

### 4.1 Models Used
1.  **Isolation Forest:** A tree-based algorithm efficient at isolating anomalies. It works on the principle that anomalies are "few and different," making them easier to isolate in a random tree structure.
    -   *Hyperparameters Tuned:* `contamination`, `n_estimators`, `max_samples`.
2.  **Local Outlier Factor (LOF):** A density-based algorithm that compares the local density of a point to its neighbors. It is effective at finding local anomalies that might look normal in the global distribution.
3.  **DBSCAN (Density-Based Spatial Clustering of Applications with Noise):** A clustering algorithm that groups dense points together and marks points in low-density regions as noise (anomalies).

### 4.2 Ensemble Strategy
-   **Majority Vote:** Flags a record as anomalous if at least 2 out of 3 models agree. This balances precision and recall.
-   **Unanimous Vote:** Flags a record only if ALL 3 models agree. This provides a "high confidence" set of anomalies.

---

## 5. Evaluation and Results

### 5.1 Evaluation Metrics
Without labeled data, I used **Score Separation** and **Silhouette Score** to evaluate model performance.
-   **Score Separation:** Measures the difference in average scores between predicted normal and anomalous points. A higher separation indicates the model is confidently distinguishing between the two.
-   **Silhouette Score:** Used to measure how well-separated the resulting clusters are.

### 5.2 Identified Anomalies
The models successfully identified specific rows with suspicious attributes.

| Metric | Count | Description |
| :--- | :--- | :--- |
| **Majority Vote Anomalies** | **226** | Broader set of potential issues. Includes unusual combinations of browser/OS and rare operations. |
| **Unanimous Vote Anomalies** | **17** | Critical anomalies. These records often feature a combination of high frequency, error status, and rare network origins. |

**Example Anomalous Patterns:**
-   **High Frequency & Error:** Records with `frequency > 200` combined with `error_status = 1`.
-   **Rare Context:** Operations performed from rare ISPs during night hours.
-   **Critical Failures:** Logs explicitly marked with critical error codes.

### 5.3 Deliverables
- `EDA_Report.ipynb`: Exploratory Data analysis and visualization.
- `feature_engineering.ipynb`: Feature engineering details.
- `model_training.ipynb`: Model training and evaluation details.
-   `results/anomalies_majority_vote.csv`: The full list of 226 potential anomalies, where >= 2 models agree.
-   `results/anomalies_unanimous_vote.csv`: The shortlist of 17 critical anomalies, where all 3 models agree.
-   `models/`: Saved `.pkl` files for Isolation Forest, LOF, and DBSCAN for future deployment.

---

## 6. Conclusion

The analysis successfully met the project requirements by building a robust, multi-model anomaly detection system. The ensemble approach ensures that we are not reliant on the assumptions of a single algorithm. The identified anomalies provide actionable insights into potential system issues, security threats, or data integrity problems.
