# Predicting-patient-outcome-using-XGBoost

Date: 18.06.2021


## Introduction & Aim

Pneumonia is a common reason for admission to critical-care and is also among the most common secondary infections acquired within those settings. Moreover, patients with pneumonia in critical-care settings are at high risk of many complications.
MIMIC-III is a freely available database of deidentified health data for patients admitted to critical-care units of the Beth Israel Deaconess Medical Center between 2001 and 2012. Data of 25 laboratory tests and vital signs for 930 patients with pneumonia recorded over 48 hours were extracted from MIMIC-III. An end-to-end machine learning pipeline using eXtreme Gradient Boosting (XGBoost) classification was developed and implemented to predict 30-day in-hospital mortality from aggregates of vital signs and laboratory test results for patients with pneumonia.

## Description of Dataset 

A total of 930 patients who were diagnosed with pneumonia were included in the study. The outcome to be predicted was 30-day in-hospital mortality (died vs. survived) and was not proportionally distributed amongst the 930 patients in the dataset. A total of 212 (23.5%) of patients died in-hospital within 30 days while the remaining 691 (76.5%) survived. The dataset also included data on patients’ age and vital signs from the first 48 hours of their ICU including heart rate (HR), systolic blood pressure (SBP), diastolic blood pressure (DBP), mean blood pressure, temperature (oC), respiratory rate (RR), oxyhaemoglobin saturation (SPO2), fraction of inspired oxygen (FIO2), peripheral O2 saturation, and partial pressure of carbon dioxide in venous blood (PVCO2). Moreover, lab values, such as albumin, arterial (blood) PH level, liver and kidney function, blood glucose, creatinine, haemoglobin, lymphocytes, lactate dehydrogenase, neutrophils, and platelets, urea, and white blood cell count (WBC) were also included in the dataset. All aforementioned demographic data, lab values, and vital signs were included as features in the machine learning algorithm. 

## Data Processing and Exploration

Data were first inspected for outliers and errors using boxplots and descriptive statistics. Values were confirmed to be out of range based on critical-care literature and were subsequently replaced by the mean value for that feature. Out of range values were identified for temperature (< 13 oC), age (negative values and values > 104), and diastolic blood pressure (DBP) (> 200). Missing values were not imputed as XGBoost is considered robust against missing values. 
Since the data comprised of a time-series of patient measurements over 48 hours, they were transformed into an appropriate matrix that can be fed into the XGBoost algorithm by aggregating the 48-hour data for each of the variables into one measurement, ∆m, corresponding to the absolute difference between the first and last recorded values for that feature m for each patient. 
The generated dataset was then explored using summary statistics and boxplots. Moreover, t-tests comparing levels of measurement between those who died and those who survived were then carried out and suggested that values of [absolute change in] heart rate (HR), PaCO2, PaO2, platelets, spontaneous respiration rate, urea, white blood cell count (WBC), age, and albumin were significantly different across the groups (p<0.05). 

## XGBoost Implementation, Hyperparameter Tuning, & Model Evaluation

The 903 cases in the dataset were randomly split into training (67%) and test (33%) samples. First, hyperparameter tuning using cross validated randomised search grids was carried out using the training data. Parameter estimates from the final/best model, i.e., the one with the optimal hyperparameters fitted to all the training data, were used to predict unseen cases from the test data and evaluate the final model. 
The following hyperparameters were tuned in the model training process: positive scaling scale_pos_weight), the number of estimators (trees) to be used, the depth (i.e., number of levels) of a tree, the minimum number of instances needed in each node (min_child_weight), and the learning rate. Positive scaling was used to address the class imbalance in outcome by scaling the gradient of the positive class – i.e., to train a class-weighted XGBoost. The remaining set of hyperparamaters that were tuned were selected based on literature regarding important parameters to tune in an XGBoost classifier. The random search grid was carried out using 10-fold cross validation as is typically done in common practice and for computational efficiency. 
Since this was a binary classification (survived vs. died), the learning objective was defined as “logistic”, and owing to the class imbalance in outcomes, the area under the precision recall curve was selected as an evaluation metric for training. This is because the default metric, accuracy, is likely to be biased in the presence of class imbalance. 
The final model has good precision (0.91) in predicting death. That is, when it predicts death, the model is correct 91% of the time. However, the model had slightly better precision when predicting survival (0.94). Performance on recall was mixed. Note that as indicated in the sci-kit learn documentation, in binary classification, recall of the positive class is sensitivity and recall of the negative class is specificity. In the case of predicting survival (the negative class), the model had a recall rate (i.e., specificity) of 0.97 whereas in predicting death (the positive class), the model performed slightly worse with a recall rate (i.e., sensitivity) of 0.79. That is, the model correctly identifies 97% of those who survive but only 79% of those who die. Thus, the rate of false negatives when predicting death is 21% and the rate of false positives 3%. The F1-score is the weighted average of precision and recall. As can be expected by the results of precision and recall, the F1-score is better in the case of predicting survival (0.95) as compared to death (0.85). The discrepancy between performance on the negative and positive classes can also be seen in the difference between the macro and weighted averages of the evaluation metrics, whereby the weighted averages suggest better overall performance as compared to the macro averages. Moreover, the model had high accuracy (93%). However, in the case of an imbalanced dataset such as the one at hand, accuracy can be biased and measures of recall, precision, and F-1 can be more informative in evaluating the model. 
Furthermore, the area under the ROC curve was also calculated as a measure of performance. The AUC represents the probability that the model ranks a randomly selected positive datapoint more highly than a randomly selected negative datapoint. The model performed well with an AUC of 0.95. However, as in the case of accuracy, in the presence of imbalanced data the AUC can be misleading. 


## Conclusion

Overall, the XGBoost model has good performance; however, the model does perform worse when predicting the positive outcome (death). This is important to note because in medical statistics, it is often the case that reducing the false negative rate is more important than reducing the false positive rate. That is, it is typically considered reasonable to utilise additional resources for a patient who will not experience the disease/outcome rather than leave a patient who will untreated. Furthermore, there are a few improvements that can be implemented in the developed pipeline. For instance, using nested-cross validation for model selection and evaluation. This was not carried out for computational efficiency. Moreover, other key measures could be utilised in model evaluation including calibration, discrimination, and even clinical usefulness (see Steyerberg & Vergouwe, 2014). 


## Journal References:
Steyerberg, E. W., & Vergouwe, Y. (2014). Towards better clinical prediction models: seven steps for development and an ABCD for validation. European heart journal, 35(29), 1925–1931. https://doi.org/10.1093/eurheartj/ehu207
Other Blog/Forum Posts:
https://machinelearningmastery.com/feature-importance-and-feature-selection-with-xgboost-in-python/
https://stats.stackexchange.com/questions/156923/should-i-make-decisions-based-on-micro-averaged-or-macro-averaged-evaluation-mea
https://machinelearningmastery.com/extreme-gradient-boosting-ensemble-in-python/
https://towardsdatascience.com/cross-validation-and-hyperparameter-tuning-how-to-optimise-your-machine-learning-model-13f005af9d7d
https://medium.com/analytics-vidhya/what-is-a-pipeline-in-machine-learning-how-to-create-one-bda91d0ceaca
https://machinelearningmastery.com/hyperparameter-optimization-with-random-search-and-grid-search/
https://machinelearningmastery.com/roc-curves-and-precision-recall-curves-for-classification-in-python/
