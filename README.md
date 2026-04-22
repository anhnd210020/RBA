````markdown
# Risk-Based Authentication — Anomaly Detection

This repository contains three Jupyter notebooks that build a full machine learning pipeline for detecting anomalous login behavior and potential **Account Takeover (ATO)** events in authentication logs.

The project uses the **RBA dataset**, available on Kaggle:

**Dataset:** https://www.kaggle.com/datasets/dasgroup/rba-dataset

---

## Repository Structure

```bash
.
├── eda.ipynb
├── supervised_anomaly_detection.ipynb
├── unsupervised_anomaly_detection_v3.ipynb
├── rba-dataset.csv
└── README.md
````

---

## Dataset

Place the dataset file in the project root as:

```bash
rba-dataset.csv
```

Download it from Kaggle:

[https://www.kaggle.com/datasets/dasgroup/rba-dataset](https://www.kaggle.com/datasets/dasgroup/rba-dataset)

### Dataset overview

The dataset contains authentication log features used for **Risk-Based Authentication (RBA)** research, including login context, network information, device/browser details, and security-related labels.

### Main target label

* **`Is Account Takeover`** — primary label for suspicious login / takeover detection

### Example important columns

* `Login Timestamp`
* `User ID`
* `Round-Trip Time [ms]`
* `IP Address`
* `Country`
* `Region`
* `City`
* `ASN`
* `User Agent String`
* `Browser Name and Version`
* `OS Name and Version`
* `Device Type`
* `Login Successful`
* `Is Attack IP`
* `Is Account Takeover`

---

## Notebook 1 — Exploratory Data Analysis (`eda.ipynb`)

This notebook performs exploratory data analysis on the raw authentication dataset.

### What it covers

* Dataset loading and shape inspection
* Schema and datatype checking
* Missing value analysis
* Duplicate checking
* Boolean label distribution
* Timestamp parsing and temporal feature extraction
* Login activity by hour / day / month
* Geographic and network analysis
* Device and software usage analysis
* Per-user behavioral diversity
* Security-oriented comparison across labels

### Main goals

* Understand the structure and quality of the data
* Identify class imbalance
* Explore temporal, geographic, and device-related patterns
* Inspect suspicious behavior associated with takeover events

### Outputs

This notebook is mainly exploratory and is used to understand the data before modeling.

---

## Notebook 2 — Supervised Anomaly Detection (`supervised_anomaly_detection.ipynb`)

This notebook treats account takeover detection as a **supervised binary classification** problem.

### Models used

* Logistic Regression
* Support Vector Machine (SVM)
* Decision Tree
* Random Forest

### Pipeline summary

1. **Preprocessing**

   * Extract login hour from timestamp
   * Convert boolean columns to numeric form
   * Drop weak or highly-missing columns where needed
   * Convert IP addresses to integers
   * Encode categorical / string-based fields

2. **Sampling**

   * Use all positive takeover samples
   * Randomly sample normal logins at a **1:100** ratio

3. **Train/Test Split**

   * Stratified 80/20 split

4. **Feature transformation**

   * Numeric features scaled with `StandardScaler`
   * Categorical features encoded with `OneHotEncoder`

5. **Imbalance handling**

   * `class_weight='balanced'`

6. **Evaluation metrics**

   * Accuracy
   * Precision
   * Recall
   * F1-score
   * ROC AUC

### Saved outputs

* `supervised_metrics_comparison.png`
* `supervised_confusion_matrices.png`
* `rf_feature_importances.png`

### Motivation

The supervised notebook is designed to directly learn the ATO label and compare standard classification models under severe class imbalance.

---

## Notebook 3 — Unsupervised Anomaly Detection (`unsupervised_anomaly_detection_v3.ipynb`)

This notebook applies **unsupervised anomaly detection** to identify suspicious login events without using labels during model fitting.

### Models used

* Mini-Batch K-Means
* One-Class SVM (RBF kernel)

### Pipeline summary

1. **Preprocessing**

   * Same preprocessing logic as the supervised notebook

2. **Sampling**

   * Same 1:100 ratio for practical training

3. **Train/Test Split**

   * Stratified split
   * Preprocessing fit on training set only

4. **Threshold tuning**

   * Models are trained without labels
   * Labels are only used afterward for threshold calibration and hyperparameter selection

5. **K-Means search**

   * Sweep across multiple `k` values and batch sizes
   * Score anomalies by distance to the nearest centroid

6. **One-Class SVM search**

   * Sweep across multiple `nu` and `gamma` values
   * Score anomalies using the negated decision function

7. **Visualization**

   * PCA projection
   * t-SNE projection

8. **Final evaluation**

   * Evaluate tuned models on the held-out test set

### Saved outputs

* `pca_true_labels.png`
* `tsne_true_labels.png`
* `kmeans_search.png`
* `ocsvm_search.png`
* `test_metrics_bar.png`
* `confusion_matrices_test.png`
* `pca_pred_vs_true_test.png`

### Motivation

The unsupervised approach is useful when labels are extremely rare, incomplete, or unavailable during training.

---

## Setup

Install the required Python packages:

```bash
pip install pandas numpy matplotlib scikit-learn scipy
```

Recommended:

* Python 3.8+
* Jupyter Notebook / JupyterLab

---

## How to Run

Run the notebooks in this order:

```bash
jupyter notebook eda.ipynb
jupyter notebook supervised_anomaly_detection.ipynb
jupyter notebook unsupervised_anomaly_detection_v3.ipynb
```

Suggested workflow:

1. Start with **EDA**
2. Run **supervised modeling**
3. Run **unsupervised modeling**
4. Compare the strengths and weaknesses of both approaches

---

## Design Decisions

### Why use a 1:100 sampling ratio?

The original dataset is highly imbalanced, so training on the full dataset is computationally expensive and can cause the models to be dominated by the normal class.

### Why use `class_weight='balanced'` in supervised learning?

Because even after undersampling, the minority class remains much smaller than the majority class.

### Why use labels in the unsupervised notebook at all?

Labels are **not** used to fit the unsupervised models. They are only used afterward to tune thresholds and compare candidate configurations.

### Why encode high-cardinality strings instead of fully one-hot encoding everything?

Some features such as user agent strings can have very high cardinality, making naive one-hot encoding inefficient or impractical.

---

## Notes

* The dataset file is **not included** in this repository.
* Download it separately from Kaggle and place it in the root folder as `rba-dataset.csv`.
* Due to class imbalance, **Recall** and **ROC AUC** are especially important metrics for this problem.

---

## Author

This project explores both **supervised** and **unsupervised** approaches for anomaly detection in risk-based authentication systems using login event data.

```
```
