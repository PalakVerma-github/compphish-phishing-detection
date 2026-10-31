\# Viva Preparation Guide: CompPhish Phishing Website Detection



\## 1. Project summary



This project uses supervised machine learning to classify websites as legitimate or phishing using features provided in the CompPhish dataset.



The main question is:



> How well can URL-derived features, HTML-derived features, and both feature groups together distinguish phishing websites from legitimate websites?



This project is an offline analysis of an existing dataset. It does not visit live websites or provide real-time browser protection.



\## 2. Dataset and labels



The project uses CompPhish Version 3.



\- The original feature spreadsheet contains 15,358 rows and 73 columns.

\- Each row includes a URL, a label, a serial number, and website features.

\- Label `0` means legitimate.

\- Label `1` means phishing.

\- The data dictionary describes 70 named features: 47 URL-derived and 23 HTML-derived.



The initial audit found no missing values and no fully duplicated rows. Seven URLs appeared twice, each time with the same label. The analysis kept one row per URL, leaving 15,351 unique URLs:



\- Legitimate: 8,154

\- Phishing: 7,197



The `brand\_subdomain` feature was constant in this version of the data, so it could not help distinguish the classes.



\## 3. Data preparation and leakage checks



Before modeling, the notebook audited the data, checked the labels and feature types, and removed repeated URLs.



Two features, `url\_more\_than\_mean` and `domain\_more\_than\_mean`, were excluded after their dictionary descriptions raised a concern: their thresholds refer to mean lengths of phishing URLs or domains. If such thresholds were calculated using the full dataset, they could reveal information from the labels or test data. Excluding them makes the modeling process more cautious.



The model input also excludes the URL text itself, the label, the serial number, and the hostname. The hostname is retained separately only for a grouped evaluation; it is not given to the classifiers as a feature.



After these exclusions, the combined model input contains 67 numeric features:



\- 44 URL-derived features

\- 23 HTML-derived features



\## 4. Exploratory data analysis



The exploratory analysis included:



\- Counts of legitimate and phishing websites

\- Summary statistics and plots for URL length and HTML form counts

\- Correlation heatmaps for all features and for selected features

\- A Welch two-sample t-test comparing average URL length between the two classes



In this dataset, the average URL length was about 29.35 characters for legitimate sites and 52.19 for phishing sites. The test returned a p-value of approximately `6.80 × 10⁻¹⁰⁶`. This is evidence of a difference in average URL length in this dataset; it does not prove that URL length causes a website to be phishing.



\## 5. Classifiers compared



The notebook compared these five classifiers:



1\. Logistic Regression

2\. Decision Tree

3\. K-Nearest Neighbors (KNN)

4\. Linear Support Vector Machine (SVM)

5\. Random Forest



The first four provide comparisons across the classification methods studied in the project. Random Forest is an additional tree-based ensemble model.



Scaling was used for models affected by feature magnitude: Logistic Regression, KNN, and Linear SVM. The scaler was fitted using the training data and then applied to the test data.



\## 6. Evaluation metrics



The project treats phishing, label `1`, as the positive class.



\- \*\*Accuracy:\*\* fraction of all predictions that are correct.

\- \*\*Precision:\*\* among sites predicted as phishing, the fraction that are phishing.

\- \*\*Recall:\*\* among actual phishing sites, the fraction detected.

\- \*\*F1-score:\*\* a combined measure of phishing precision and recall.

\- \*\*ROC-AUC:\*\* how well a model ranks phishing sites above legitimate sites across possible decision thresholds.

\- \*\*Confusion matrix:\*\* counts of correct and incorrect predictions for each class.

\- \*\*Log loss:\*\* evaluates the quality of probability predictions; lower is better.



Recall matters because a phishing site predicted as legitimate is a missed phishing case. Accuracy alone is not enough to judge a phishing detector.



\## 7. Reported model results



These are the results you shared for the stratified test-set comparison. Confirm them against the outputs in the final saved notebook before presenting them.



| Model | Accuracy | Phishing precision | Phishing recall | Phishing F1 | ROC-AUC |

|---|---:|---:|---:|---:|---:|

| Logistic Regression | 0.9525 | 0.9468 | 0.9521 | 0.9494 | 0.9896 |

| Decision Tree | 0.9580 | 0.9536 | 0.9569 | 0.9553 | 0.9579 |

| KNN | 0.9577 | 0.9600 | 0.9493 | 0.9546 | 0.9837 |

| Linear SVM | 0.9538 | 0.9482 | 0.9535 | 0.9508 | 0.9893 |

| Random Forest | 0.9730 | 0.9709 | 0.9715 | 0.9712 | 0.9959 |



In this comparison, Random Forest had the highest accuracy, phishing recall, phishing F1-score, and ROC-AUC among the five models. These results describe performance on this dataset and split; they do not guarantee the same performance on future websites.



\## 8. URL-only, HTML-only, and combined features



The feature-group experiment compared the Random Forest using URL-derived features, HTML-derived features, and both groups together.



| Feature set | Accuracy | Phishing precision | Phishing recall | Phishing F1 | ROC-AUC | Missed phishing sites |

|---|---:|---:|---:|---:|---:|---:|

| URL-only | 0.9525 | 0.9456 | 0.9535 | 0.9495 | 0.9878 | 67 |

| HTML-only | 0.9319 | 0.9509 | 0.9014 | 0.9255 | 0.9823 | 142 |

| Combined | 0.9730 | 0.9709 | 0.9715 | 0.9712 | 0.9959 | 41 |



On this test split, the combined feature set performed best overall and missed fewer phishing sites than either feature group alone. This suggests that both URL and HTML information contributed to the result in this experiment.



\## 9. Hostname-grouped evaluation



A random train/test split can place websites from the same hostname in both sets. This may make evaluation less representative of performance on entirely unseen hosts.



To examine this, the notebook also used a stratified group split based on hostname. It reported zero hostnames shared between the training and test sets. In that grouped evaluation, the Random Forest result you shared was:



\- Accuracy: 0.9687

\- Phishing precision: 0.9647

\- Phishing recall: 0.9687

\- Phishing F1-score: 0.9667

\- Confusion matrix: `\[\[1580, 51], \[45, 1394]]`



This is a separate evaluation from the ordinary stratified comparison above. When discussing it, clearly say that it is the hostname-grouped Random Forest result; do not present it as if every model in the comparison used that same split unless the notebook confirms that.



\## 10. Feature importance and missed phishing sites



Permutation importance estimates how much a model’s score changes when one feature is shuffled. In the reported analysis, features near the top included `shortest\_word\_length`, `count\_www`, `no\_of\_hyperlinks`, `no\_of\_internal\_links`, `domain\_length`, and `url\_length`.



The missed-phishing analysis examined feature averages for phishing sites the Random Forest predicted as legitimate. In one reported test set, there were 41 missed phishing samples. The averages can help describe those errors, but they do not explain every individual prediction or prove why a sample was misclassified.



\## 11. Limitations



\- The models use precomputed features from one dataset version; results may differ on newer or independently collected websites.

\- The analysis does not test a live detection system or measure how quickly it works.

\- Dataset labels and feature definitions affect what the models can learn.

\- A strong score on a test split is not proof of perfect real-world performance.

\- Feature importance describes this fitted model and evaluation data; it is not proof that a feature causes phishing.



\## 12. Likely viva questions



\### What is supervised learning?



Supervised learning trains a model using examples that already have known labels. Here, the labels are legitimate (`0`) and phishing (`1`).



\### Why did you use a stratified split?



Stratification keeps the class proportions roughly similar in the training and test sets.



\### Why scale some models?



Scaling puts numeric features on comparable ranges. This is useful for models such as KNN, Logistic Regression, and SVM, whose results can be affected by feature magnitude.



\### Why is phishing recall important?



Low phishing recall means more actual phishing websites are incorrectly predicted as legitimate. Those are false negatives.



\### Why report precision as well as recall?



High recall can come with more legitimate sites incorrectly flagged as phishing. Precision shows how often a phishing prediction is correct.



\### Why use a confusion matrix?



It shows the counts of correct and incorrect predictions for both classes, including false negatives and false positives.



\### Why compare URL-only and HTML-only features?



This helps assess how the two feature groups perform separately and whether combining them improves the measured results.



\### Why did you perform a hostname-grouped evaluation?



It checks performance when hostnames do not overlap between training and test sets, giving a more demanding test of generalization to unseen hosts.



\### Which model performed best?



Random Forest had the strongest results in the reported stratified comparison. The conclusion is limited to the tested dataset, features, and split.



\### Does this mean the model is ready to block websites?



No. This project evaluates classifiers on a prepared dataset. A live system would need further testing on current, independently collected websites and careful handling of incorrect predictions.



\## 13. Short project explanation



“My project compares machine-learning classifiers for identifying phishing websites in the CompPhish dataset. I audited and cleaned the data, explored differences between the classes, and compared URL-derived features, HTML-derived features, and their combination. I evaluated the models using phishing-focused metrics such as recall, precision, F1-score, ROC-AUC, and confusion matrices. Random Forest performed best in the reported test-set comparison. I also checked a hostname-grouped split to examine results when hostnames were kept separate. These findings apply to the dataset and evaluation setups used; they do not guarantee performance on live websites.”

