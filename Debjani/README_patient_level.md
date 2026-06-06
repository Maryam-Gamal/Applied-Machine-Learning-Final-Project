# Patient-level COVID severity prediction

This folder contains my smaller machine learning part of the group project:

**Predicting severity of COVID-19 infection based on single-cell sequencing data**

I used basic patient and clinical features to test whether a simple machine learning model can predict COVID severity. I treated this as an exploratory comparison with the genetic analysis.

## Main file

Run this notebook:

`covid_severity_patient_features.ipynb`

The notebook uses two data files:

`data/clinical_basic_features.csv`

This is the small Broad cohort used for the main comparison with the group project.

`data/Dataset_080620_COVID19_Mexico.csv`

This is a larger Mexico patient dataset used only as an external benchmark.

## Why I used this approach

The Broad dataset has only 112 rows. After removing controls, there are 86 COVID-positive patients.

This is too small for a strong clinical prediction model, so I kept the ML methods simple:

- Dummy baseline
- Logistic regression
- Random forest
- 5-fold cross-validation
- Basic regression on WHO score
- Basic clustering check

I did not use the model as clinical proof. I used it to ask:

Can basic patient features give any prediction signal for COVID severity?

## Step-by-step summary of the notebook

### 1. Load the Broad cohort

The file `clinical_basic_features.csv` has 112 participants and 18 columns.

The WHO score distribution in the raw file is:

| WHO score | Patients |
|---|---:|
| 0 | 27 |
| 1 | 4 |
| 2 | 1 |
| 3 | 15 |
| 4 | 22 |
| 5 | 19 |
| 6 | 9 |
| 7 | 15 |

Controls are removed because the question is severity among COVID-positive patients, not COVID vs control.

After removing controls:

- COVID-positive patients: 86
- Lower severity, WHO < 5: 43
- Higher severity, WHO >= 5: 43

This made the binary target balanced.

### 2. Clean the features

I cleaned string columns and converted numeric columns to numbers.

For example:

- Age
- BMI
- Number of vaccine doses
- WBC
- Platelets
- D-dimer
- Ferritin

For the model, I used:

- 12 numeric features
- 3 categorical features

Important: I removed `Variant_Group` from the model features.

I did this because variant group is more like a time/wave label, and it could make the model learn dataset structure instead of patient-level severity.

### 3. Classification model

The main prediction target was:

- 0 = lower severity, WHO score < 5
- 1 = higher severity, WHO score >= 5

I used 5-fold stratified cross-validation because the dataset is small.

Main Broad cohort results:

| Model | Balanced accuracy | ROC-AUC | F1 |
|---|---:|---:|---:|
| Dummy baseline | 0.500 | 0.500 | 0.256 |
| Logistic regression | 0.653 | 0.664 | 0.667 |
| Random forest | 0.676 | 0.757 | 0.650 |

The random forest had the best ROC-AUC. Logistic regression had a similar F1 score and is easier to explain.

My interpretation:

The model finds some signal, but the result is not strong enough to claim a reliable clinical model. It is enough for a small exploratory ML comparison.

### 4. Feature coefficient check

I fitted a logistic regression model to look at which features had larger coefficients.

Top features by absolute coefficient:

| Feature | Coefficient |
|---|---:|
| WBC | 1.058 |
| Number of vaccine doses | -0.795 |
| Age | 0.773 |
| CKD | -0.467 |
| Race/Ethnicity: White, Hispanic | 0.436 |
| Diabetes | 0.381 |
| BMI | 0.358 |
| Ferritin | 0.264 |
| D-dimer | 0.225 |
| HTN | 0.196 |

I would not present these as causal findings. They are only model patterns from a small dataset.

### 5. Regression check

I also tried predicting the original WHO score directly.

Results:

| Model | MAE | RMSE | R2 |
|---|---:|---:|---:|
| Mean baseline | 1.342 | 1.665 | -0.023 |
| Ridge regression | 1.219 | 1.515 | 0.153 |
| Random forest regressor | 1.097 | 1.385 | 0.292 |

The random forest regressor improved over the mean baseline, but the R2 is still low. This supports the same conclusion: there is some signal, but not enough for a strong prediction claim.

### 6. Clustering check

I used K-means clustering as an unsupervised check.

Results:

| k | Silhouette | ARI vs severity |
|---:|---:|---:|
| 2 | 0.145 | 0.056 |
| 3 | 0.124 | 0.041 |
| 4 | 0.136 | 0.060 |

The ARI values are very low. This means the patient features do not naturally form clusters that match the severity labels very well.

### 7. Data augmentation / SMOTE check

I kept SMOTE only as an optional sensitivity check.

I did not use synthetic patients as the main evidence.

Results:

| Method | Balanced accuracy | ROC-AUC | F1 |
|---|---:|---:|---:|
| Class weighting | 0.653 | 0.664 | 0.667 |
| SMOTE inside CV | 0.653 | 0.669 | 0.667 |

SMOTE did not meaningfully improve the result.

If asked, my explanation is:

I used SMOTE only inside cross validation training folds, not on the full dataset before splitting. I will claim that SMOTE creates real new patients.

### 8. Mexico benchmark

The Mexico dataset is much larger, but it is a different dataset with a different outcome.I could not find any other large data set with similar features.

After filtering confirmed COVID patients:

- Confirmed COVID patients: 146,681
- Not severe: 121,544
- Severe: 25,137

The severe outcome is based on death, ICU, or intubation.

Mexico benchmark results:

| Model | Balanced accuracy | ROC-AUC | F1 |
|---|---:|---:|---:|
| Dummy baseline | 0.500 | 0.500 | 0.000 |
| Logistic regression | 0.804 | 0.878 | 0.586 |

This showed that basic clinical features can predict severity better in a much larger real-patient dataset. However, it should not be mixed directly with the Broad result because the features and severity definition are not exactly the same.

## For presentation

My part tested whether simple patient-level features can predict COVID severity.

The Broad cohort result was modest:

- Best Broad ROC-AUC: 0.757 with random forest
- Best Broad balanced accuracy: 0.676 with random forest

The Mexico benchmark was stronger:

- Logistic regression ROC-AUC: 0.878
- Balanced accuracy: 0.804

This supports the idea that clinical features can contain severity signal, but the small Broad cohort is not enough for a strong standalone prediction model.

The gene-level part is still important because it is better for biological interpretation and possible drug-discovery ideas.


## Main limitations

The main limitation is sample size. The Broad cohort has only 86 COVID-positive patients after controls are removed.

Other limitations:

- The model is exploratory, not clinical.
- Cross-validation helps, but it does not replace external validation.
- Feature importance and coefficients are not causal proof.
- The Mexico dataset is useful for comparison but has a different outcome definition.
- Race/ethnicity and vaccination status should be interpreted carefully because they can reflect social and time-period effects.