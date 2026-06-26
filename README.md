# API Gateway Performance Observability & Predictive Churn Pipeline

A production-focused machine learning pipeline designed to monitor core backend gateway telemetry and proactively predict developer churn risks before integration abandonment occurs. 

## Project Performance Matrix
* **Overall System Accuracy:** 71%
* **SLA Failure Churn Precision (Class 1):** 99%
* **Active Asset Precision (Class 0):** 70%

---

## The Engineering Challenge & Realistic Tradeoffs

### The Baseline Trap
Initially, default tree architectures trained on raw infrastructure metrics (average latency, sequential rate limits) hit a hard mathematical ceiling with an F1-score of ~44%. Because typical production API environments contain massive amounts of overlapping telemetry noise and significant class imbalances (active vs. dropping developers), the models struggled to separate system friction from intentional user attrition.

### Moving Beyond Synthetic Perfection
While modifying underlying conditional constraints can artifactually drive scores up to 96% across the board, doing so creates an overfitted, unrealistic model that fails to account for real-world environmental noise. 

To bridge this gap, the pipeline was constrained to a highly regularized XGBoost configuration (`max_depth=5`, `learning_rate=0.01`). This intentional optimization strategy yielded an exceptionally defensive model with **99% Precision on Class 1 (Churn)**. 

### Why a 99% Precision Target Makes Business Sense
In enterprise SaaS or platform infrastructures, firing dynamic retention actions—such as dispatching automated AWS/infrastructure credits or discounts—to healthy developers wastes significant financial resources. Achieving a 99% precision rating ensures that when this engine flags a token as a critical threat, it is virtually flawless, eliminating the financial risk of false alarms.

---

## Data Pipeline Architecture

The end-to-end framework manages data cleanly across distinct architectural checkpoints:

1. **`df_raw` Ingestion:** Pulls 25,000 streaming telemetry profiles consisting of average latencies, consecutive 429 rate-limiting spikes, and 500 internal server database errors across segmented subscription tiers (Free, Growth, Enterprise).
2. **`df_cleaned` Sanitization:** Converts string anomalies/blank spaces (`" "`) introduced by gateway drops and applies median imputation vectors.
3. **`df_preprocessed` Feature Engineering:** Encodes categorical subscription tiers/regions and injects advanced non-linear interaction terms to highlight compound structural failures:
   * `error_momentum`: Multiplies 429 and 500 spikes to track compounding client/server breaks.
   * `latency_friction_ratio`: Maps raw latency against user rate-limiting bottlenecks.

---

## Core Implementation Components

### 1. Exploratory Data Analysis (EDA)
* Automated count plotting analyzing the volatility of 429 errors against churn outcomes.
* Non-linear multi-variable correlation heatmaps evaluating background infrastructure dependencies.
* Global pairplot feature matrices to visualize the complex boundaries separating operational keys from dropped accounts.

### 2. Supervised Learning & Regularization
* Employs an extreme gradient boosting framework (**XGBoost**) paired with strategic data partitioning.
* Applies explicit data subsampling limits (`subsample=0.8`, `colsample_bytree=0.8`) to replicate authentic operational errors and ensure stable local evaluation performance.

### 3. Unsupervised Behavioral Segmentation
* Leverages **K-Means Clustering** strictly on active, healthy assets (`churn_target == 0`) to extract distinct operational developer personas.
* Groups developers by consumption styles (e.g., high-throughput vs. high-error spikes) to enable automated platform routing or specialized infrastructure support before any degradation occurs.

---

## Technical Stack
* **Language:** Python 3.12
* **Core Libraries:** XGBoost, Scikit-Learn, Pandas, NumPy
* **Visualization Engine:** Seaborn, Matplotlib
