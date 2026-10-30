# CompPhish Phishing Website Detection

A machine learning project comparing URL-derived and HTML-derived features for phishing website detection.

## Project question

How well do URL-derived, HTML-derived, and combined features distinguish phishing websites from legitimate websites in the CompPhish dataset?

## Dataset

This project uses CompPhish Version 3. The original feature file contains 15,358 samples and 70 features, plus a URL, label, and serial number. Label `0` means legitimate and label `1` means phishing.

| Dataset check | Result |
|:---|---:|
| Original samples | 15,358 |
| Legitimate samples in the original file | 8,154 |
| Phishing samples in the original file | 7,204 |
| Missing values | 0 |
| Repeated URLs removed from the analysis copy | 7 |
| Samples after repeated-URL handling | 15,351 |

The mapping file matched all original samples, with no URL or label mismatches. The analysis kept one row per repeated URL. The constant `brand_subdomain` feature was excluded from modelling.

| Feature group | Features in dictionary | Features used in models |
|:---|---:|---:|
| URL-based and brand-jacking | 47 | 46 |
| HTML-derived content | 23 | 23 |
| Combined | 70 | 69 |

The downloaded Excel files are not included in this repository. Download the feature spreadsheet, mapping file, and data dictionary from the [CompPhish Mendeley record](https://data.mendeley.com/datasets/fmbs4kp9wz/3) and place them in the local `data` folder.

## Method

The primary experiment used an 80/20 stratified train/test split with `random_state=42`. All five classifiers used the same split. Phishing, label `1`, is the positive class.

The classifiers compared were:

- Logistic Regression
- Decision Tree
- K-Nearest Neighbors (KNN)
- Linear Support Vector Machine (SVM)
- Random Forest

Features were scaled inside a pipeline for Logistic Regression, KNN, and SVM. Decision Tree and Random Forest were trained without scaling.

Metrics include accuracy, phishing precision, phishing recall, phishing F1, ROC-AUC, and confusion matrices.

## Classifier results

| Model | Accuracy | Phishing precision | Phishing recall | Phishing F1 | ROC-AUC |
|:---|---:|---:|---:|---:|---:|
| Logistic Regression | 0.9547 | 0.9489 | 0.9549 | 0.9519 | 0.9901 |
| Decision Tree | 0.9515 | 0.9486 | 0.9479 | 0.9482 | 0.9513 |
| KNN | 0.9583 | 0.9613 | 0.9493 | 0.9553 | 0.9847 |
| Linear SVM | 0.9570 | 0.9479 | 0.9611 | 0.9545 | 0.9899 |
| **Random Forest** | **0.9736** | **0.9715** | **0.9722** | **0.9719** | **0.9955** |

Random Forest had the highest scores on the stratified random test split. Its confusion matrix contained 40 false negatives and 41 false positives.

## URL-only, HTML-only, and combined features

Random Forest was evaluated with the same training and test samples for each feature set.

| Feature set | Features | Accuracy | Phishing precision | Phishing recall | Phishing F1 | ROC-AUC | False negatives |
|:---|---:|---:|---:|---:|---:|---:|---:|
| URL-only | 46 | 0.9525 | 0.9481 | 0.9507 | 0.9494 | 0.9880 | 71 |
| HTML-only | 23 | 0.9319 | 0.9509 | 0.9014 | 0.9255 | 0.9823 | 142 |
| **Combined** | **69** | **0.9736** | **0.9715** | **0.9722** | **0.9719** | **0.9955** | **40** |

On this split, combined features performed best. URL-only features performed better than HTML-only features on accuracy, recall, and F1. HTML-only features had slightly higher precision than URL-only features, but missed more phishing samples.

## Additional analysis

### URL length

The mean URL length was 52.19 characters for phishing samples and 29.35 characters for legitimate samples. Welch's two-group t-test returned a t-statistic of 22.05 and a p-value below 0.001. A very long legitimate URL was present, so the means should be interpreted alongside the medians: 42 characters for phishing and 24 for legitimate samples.

### False negatives

The Random Forest missed 40 phishing samples on the primary test split. Compared with all phishing samples in that test set, the missed samples had a lower median URL length (25.5 versus 43 characters) and higher median hyperlink counts (36.5 versus 5 total hyperlinks; 7.5 versus 2 external links). These are descriptive patterns among the missed samples; they do not prove why the model made those predictions.

### Feature importance

Permutation importance ranked `count_www`, `shortest_word_length`, and `url_length` among the highest features. Other high-ranked features included hyperlink counts. Importance describes how the model used features on this test set; it does not show that a feature causes phishing.

### Host-based evaluation

A separate stratified group split kept each normalized hostname entirely in either training or test data. No hostname was shared between the two sets.

| Metric | Host-based Random Forest |
|:---|---:|
| Accuracy | 0.9658 |
| Phishing precision | 0.9619 |
| Phishing recall | 0.9653 |
| Phishing F1 | 0.9636 |
| ROC-AUC | 0.9943 |
| False negatives | 50 |
| False positives | 55 |

This is a hostname-based evaluation. Different subdomains may still be treated as separate hosts, and its test samples differ from those in the random split.

## Figures

![Selected feature correlation heatmap](reports/figures/selected_correlation_heatmap.png)

![Random Forest permutation importance](reports/figures/feature_importance.png)

Additional charts and results are in the notebook.

## Run the project

1. Download the three Excel files from the [CompPhish Mendeley record](https://data.mendeley.com/datasets/fmbs4kp9wz/3).
2. Place them in the local `data` folder. The files are ignored by Git and are not included in this repository.
3. Create and activate a Python virtual environment.
4. Install the packages listed in `requirements.txt`.
5. Start Jupyter Notebook, open `notebooks/phishing_detection.ipynb`, and run the cells from top to bottom.

## Limitations

Phishing and legitimate samples came from different collection sources, which may introduce source-specific patterns. The dataset reflects a defined collection period and may not represent future attacks or all regions and languages. The random split may include related hosts in both sets. The hostname-based split reduces exact-host overlap but does not ensure every subdomain under a registered domain stays together. Performance on CompPhish does not guarantee performance on other datasets or future websites.

## Sources

- Goenka, R. (2026). *CompPhish*, Mendeley Data, Version 3. [https://doi.org/10.17632/fmbs4kp9wz.3](https://doi.org/10.17632/fmbs4kp9wz.3). Licence: CC BY 4.0.
- Goenka, R., Chawla, M., & Tiwari, N. (2026). “CompPhish: A comprehensive dataset for diversified phishing detection.” *Data in Brief*, 69, 113219. [https://doi.org/10.1016/j.dib.2026.113219](https://doi.org/10.1016/j.dib.2026.113219).
