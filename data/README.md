# CompPhish Phishing Website Detection

A machine learning project comparing URL-derived and HTML-derived features for detecting phishing websites in the CompPhish dataset.

## Project question

How well do URL-derived, HTML-derived, and combined features distinguish phishing websites from legitimate websites in CompPhish?

## Dataset

CompPhish Version 3 contains 15,358 samples: 8,154 legitimate (label `0`) and 7,204 phishing (label `1`). The feature spreadsheet contains 70 precomputed features, plus the URL, label, and serial number. The features are grouped as 42 URL-based, 5 brand-jacking, and 23 content-based (HTML-derived) features.

Seven repeated URLs with the same label were found. The analysis kept the first row for each URL, leaving 15,351 rows. The constant `brand_subdomain` feature was excluded from modelling, leaving 69 varying features: 46 URL-derived and 23 HTML-derived.

Dataset files are not stored in this repository. Download the feature spreadsheet, mapping file, and data dictionary from the [CompPhish Mendeley record](https://data.mendeley.com/datasets/fmbs4kp9wz/3) and place them in the `data/` folder.

## Method

The data was split into training and test sets using an 80/20 stratified split with `random_state=42`. The same split was used to compare five classifiers:

- Logistic Regression
- Decision Tree
- K-Nearest Neighbors (KNN)
- Linear Support Vector Machine (SVM)
- Random Forest

Features were scaled inside a pipeline for Logistic Regression, KNN, and SVM. Decision Tree and Random Forest were trained without scaling. Performance was measured using accuracy, phishing precision, phishing recall, phishing F1, ROC-AUC, and confusion matrices.

## Classifier results

| Model | Accuracy | Phishing precision | Phishing recall | Phishing F1 | ROC-AUC |
|---|---:|---:|---:|---:|---:|
| Logistic Regression | 0.9547 | 0.9489 | 0.9549 | 0.9519 | 0.9901 |
| Decision Tree | 0.9515 | 0.9486 | 0.9479 | 0.9482 | 0.9513 |
| KNN | 0.9583 | 0.9613 | 0.9493 | 0.9553 | 0.9847 |
| Linear SVM | 0.9570 | 0.9479 | 0.9611 | 0.9545 | 0.9899 |
| Random Forest | **0.9736** | **0.9715** | **0.9722** | **0.9719** | **0.9955** |

Random Forest had the highest scores on this random test split. Its confusion matrix contained 40 false negatives and 41 false positives.

## URL-only, HTML-only, and combined features

| Feature set | Accuracy | Phishing precision | Phishing recall | Phishing F1 | ROC-AUC | False negatives |
|---|---:|---:|---:|---:|---:|---:|
| URL-only | 0.9525 | 0.9481 | 0.9507 | 0.9494 | 0.9880 | 71 |
| HTML-only | 0.9319 | 0.9509 | 0.9014 | 0.9255 | 0.9823 | 142 |
| Combined | **0.9736** | **0.9715** | **0.9722** | **0.9719** | **0.9955** | **40** |

On this split, the combined feature set performed best. URL-only features performed better than HTML-only features on accuracy, recall, and F1. These results describe this dataset and split; they do not establish performance on future websites.

## Additional analysis

- A Welch two-group t-test found a difference in mean URL length between classes: phishing mean = 52.19 characters; legitimate mean = 29.35; `t = 22.05`, `p < 0.001`.
- The Random Forest permutation-importance analysis ranked `count_www`, `shortest_word_length`, and `url_length` among the highest features.
- The model missed 40 phishing samples on the random test split. Those missed samples had shorter median URLs and higher median hyperlink counts than phishing samples overall in the test set.
- A hostname-based split kept each normalized hostname entirely in either training or test data. Random Forest achieved 0.9658 accuracy, 0.9636 phishing F1, and 0.9943 ROC-AUC on that split. Different subdomains may still be treated as separate hosts.

## Figures

![Selected feature correlation heatmap](reports/figures/selected_correlation_heatmap.png)

![Random Forest permutation feature importance](reports/figures/feature_importance.png)

Additional plots and outputs are included in the notebook.

## Run the project

1. Download the three dataset files listed above and place them in `data/`.
2. From the project folder, create an environment and install the required libraries:

   ```powershell
   python -m venv .venv
   .\.venv\Scripts\python.exe -m pip install -r requirements.txt