\# CompPhish Phishing Website Detection



A machine learning project comparing URL-derived and HTML-derived features for phishing detection.



\## Project question



How well do URL-derived, HTML-derived, and combined features distinguish phishing websites from legitimate websites in CompPhish?



\## Dataset



This project uses CompPhish Version 3. The feature spreadsheet contains 15,358 samples and 70 features, plus the URL, label, and serial number. Label `0` means legitimate and label `1` means phishing.



The data audit found no missing values or fully duplicated rows. Seven repeated URLs had the same label; the analysis kept one row per URL, leaving 15,351 samples. The constant `brand\_subdomain` feature was excluded from modelling. The experiments used 46 varying URL-derived features, 23 HTML-derived features, and 69 combined features.



Download the feature spreadsheet, mapping file, and data dictionary from the \[CompPhish Mendeley page](https://data.mendeley.com/datasets/fmbs4kp9wz/3) and place them in `data/`. The dataset files are not included in this repository.



\## Method



The main experiment used an 80/20 stratified train/test split with `random\_state=42`. The same split was used for all five classifiers. Features were scaled in a pipeline for Logistic Regression, KNN, and the linear SVM. Decision Tree and Random Forest were trained without scaling.



Metrics include accuracy, phishing precision, phishing recall, phishing F1, ROC-AUC, and confusion matrices. Phishing is treated as the positive class.



\## Classifier comparison



| Model | Accuracy | Phishing precision | Phishing recall | Phishing F1 | ROC-AUC |

|---|---:|---:|---:|---:|---:|

| Logistic Regression | 0.9547 | 0.9489 | 0.9549 | 0.9519 | 0.9901 |

| Decision Tree | 0.9515 | 0.9486 | 0.9479 | 0.9482 | 0.9513 |

| KNN | 0.9583 | 0.9613 | 0.9493 | 0.9553 | 0.9847 |

| Linear SVM | 0.9570 | 0.9479 | 0.9611 | 0.9545 | 0.9899 |

| Random Forest | \*\*0.9736\*\* | \*\*0.9715\*\* | \*\*0.9722\*\* | \*\*0.9719\*\* | \*\*0.9955\*\* |



Random Forest performed best on the stratified random test split. It missed 40 phishing samples and incorrectly flagged 41 legitimate samples.



\## Feature-group comparison



| Feature set | Accuracy | Phishing precision | Phishing recall | Phishing F1 | ROC-AUC | False negatives |

|---|---:|---:|---:|---:|---:|---:|

| URL-only | 0.9525 | 0.9481 | 0.9507 | 0.9494 | 0.9880 | 71 |

| HTML-only | 0.9319 | 0.9509 | 0.9014 | 0.9255 | 0.9823 | 142 |

| Combined | \*\*0.9736\*\* | \*\*0.9715\*\* | \*\*0.9722\*\* | \*\*0.9719\*\* | \*\*0.9955\*\* | \*\*40\*\* |



On this split, combined features performed best. These results describe CompPhish and this split; they do not guarantee performance on future websites.



\## Additional analysis



\- A Welch two-group t-test compared mean URL length: phishing mean 52.19 characters; legitimate mean 29.35; t-statistic 22.05; p-value below 0.001.

\- Permutation importance ranked `count\_www`, `shortest\_word\_length`, and `url\_length` among the highest features.

\- False-negative samples had shorter median URLs and higher median hyperlink counts than phishing samples overall in the test set.

\- A hostname-based split kept exact normalized hostnames separate. Random Forest achieved 0.9658 accuracy, 0.9636 phishing F1, and 0.9943 ROC-AUC. Different subdomains may still be treated as separate hosts.



\## Figures



!\[Selected feature correlation heatmap](reports/figures/selected\_correlation\_heatmap.png)



!\[Random Forest permutation importance](reports/figures/feature\_importance.png)



Additional charts and results are in the notebook.



\## Run the project



1\. Download the three required Excel files from Mendeley and place them in `data/`.

2\. From the project folder, install the required packages:



&#x20;  ```powershell

&#x20;  python -m venv .venv

&#x20;  .\\.venv\\Scripts\\python.exe -m pip install -r requirements.txt

