# Heartbeat Anomaly Detection
This notebook analyzes an abnormal heartbeat symptom - named Premature Ventricular Contractions (PVC) - using raw EKG signals. This is an area of interest as consistent PVC detections may indicate potential heart arrhythmias. EKG data is extracted from [MIT-BIH Arrhythmia Database](https://physionet.org/physiobank/database/mitdb/), obtained from 47 subjects.

PVCs can be identified using the following features:
1. Premature beat
2. Absent P wave
3. Wide QRS complexes
4. Compensatory pause following the contraction

Below is an example obtained from the MIT-BIH dataset, to observe the differences between a PVC and normal heartbeat.

![pvc_example](https://user-images.githubusercontent.com/42630569/52937160-351d0400-3313-11e9-9a5f-740797694c7a.png)

## Approach

EKG data is split into **10-second windows** and analyzed individually to determine if each of them contains **at least 1 PVC**. Since EKG signals are noisy in nature, smoothing is performed to increase the ease of analysis. In addition to that, normalizing signals is necessary since patients' heartbeat are unique and have different baselines. Different models are then tested and compared using ROC AUC, precision and recall.

## Smoothing of EKG signals

3 different smoothing methods were used in this notebook - Fourier transform, convolution of a scaled window, and Savitzky-Golay filter. By observation, Savitzky-Golay filter is selected as it does not alter the window frame or significantly change the magnitude of R peaks.

![smooth](https://user-images.githubusercontent.com/42630569/52937747-d8224d80-3314-11e9-9638-c07ab88e68f4.png)

## Normalizing EKG signals

Patients have unique EKG signals and their norms may different. Hence it is necessary to normalized EKG signals in order to increase prediction accuracy.

![norm](https://user-images.githubusercontent.com/42630569/52937911-2899ab00-3315-11e9-903e-f37769a94aa3.png)

## Features extraction

In this notebook, we will test 2 different modeling approaches. First, we will feed raw EKG signals into a 1-D CNN model. Next, we will compare the results with models (e.g. random forest, logistic regression). To proceed, we will need to extract features and this was done using annotation data provided by MIT-BIH, as well as the [biosppy ECG package](https://biosppy.readthedocs.io/en/stable/biosppy.signals.html#biosppy-signals-ecg).

To run biosppy, run the following code:
```
from biosppy.signals.ecg import ecg
```

The features extracted include **age**, **gender**, **signals** (min, max, std), **heartbeat** (min, max, mean, std), **R-peaks** (min, max, mean, std) and **distance between R-R peaks** (min, max, mean, std).


## Results of different models

The dataset is train and tested using the following models with their results for the test set are tabulated below. GridSearch was run to identify the hyperparameters that maximizes ROC AUC score. Specific numbers like TP, TN, FP, FN or the optimal hyperparameteres for each model are included in the confusion matrix in the notebook. 

| Model | ROC AUC | Accuracy | Specificity | Sensitivity | Recall | Precision |
| --- | --- | --- | --- | --- | --- | --- |
| Logistics Regression | 0.760 | 0.801 | 0.873 | 0.646 | 0.706 | 0.646 |
| Random Forest | 0.935 | 0.940 | 0.948 | 0.922 | 0.893 | 0.922 |
| GBC | 0.956 | 0.961 | 0.970 | 0.942 | 0.937 | 0.942 |
| SVC | 0.883 | 0.894 | 0.914 | 0.852 | 0.824 | 0.852 |
| CNN | 0.787 | 0.723 | 0.810 | 0.539 | 0.571 | 0.539 |

