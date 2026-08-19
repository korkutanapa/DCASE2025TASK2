# DCASE 2025 Task 2 – TDA-Based Anomalous Sound Detection

This repository contains the code and extracted feature files used for a **Topological Data Analysis (TDA)-based anomalous sound detection framework** developed for **DCASE 2025 Task 2: First-Shot Unsupervised Anomalous Sound Detection for Machine Condition Monitoring**.

## Overview

The proposed framework represents machine sounds using topological descriptors extracted from Mel spectrograms.

The general processing pipeline is:

```text
WAV recording
    ↓
Mel spectrogram
    ↓
Cubical complex
    ↓
Persistent homology (H0 and H1)
    ↓
TDA feature extraction
    ↓
Feature selection
    ↓
kNN-based anomaly scoring
    ↓
DCASE official evaluation
```

A broad collection of persistence-based descriptors is extracted, including:

* birth and death statistics
* persistence lifetime statistics
* Betti curve descriptors
* persistence landscapes
* persistence silhouettes
* persistence images
* Carlsson coordinates
* tail and dominant persistence statistics
* weighted birth/death/midlife descriptors
* H0–H1 interaction features

---

## Repository Structure

### 1. TDA Feature Extraction

```text
ORJ_DCASE_TDA_FEATURE_CREATOR.ipynb
```

This notebook converts machine-sound recordings into TDA feature vectors.

Main steps:

1. Load WAV recordings.
2. Compute Mel spectrograms.
3. Interpret each spectrogram as a two-dimensional scalar field.
4. Construct a cubical complex.
5. Compute persistent homology for (H_0) and (H_1).
6. Extract statistical, geometric, functional, and image-based TDA descriptors.
7. Save the resulting feature matrices as Excel files.

---

### 2. Development-Set Feature Selection

```text
ORJ_DCASE_FEATURE_SELECTION_FOR_DEVELOPMENT_DATASET.ipynb
```

This notebook evaluates the TDA feature space on the labeled DCASE development machines.

The development data are used to identify informative TDA descriptors and construct the transferable feature pool used in the subsequent unseen-machine experiments.

Feature subsets are evaluated using kNN-based anomaly scores and the DCASE performance measures:

* source-domain AUC
* target-domain AUC
* partial AUC at (p=0.1)

The main ranking criterion is based on the harmonic mean of these measures.

The development-stage search is used as an **oracle analysis**, since development test labels are available during this stage.

---

### 3. Feature Selection for Unseen Machines

```text
ORJ_DCASE_FEATURE_SELECTON_FOR_UNSEEN_DATA.ipynb
```

This notebook performs machine-specific feature selection for the previously unseen DCASE evaluation machine types.

The search starts from the TDA feature pool identified using the development machines.

For each unseen machine type, the available normal training recordings and unlabeled evaluation recordings are analyzed to identify a compact machine-specific subset of TDA descriptors.

The resulting selected feature subsets are reported and can subsequently be used for anomaly scoring.

Evaluation machine types include:

```text
AutoTrash
BandSealer
CoffeeGrinder
HomeCamera
Polisher
ScrewFeeder
ToyPet
ToyRCCar
```

---

### 4. Anomaly Scoring and Official Evaluation

```text
ORJ_DCASE_EVALUATOR_AND_OFFICIAL_CODE.ipynb
```

This notebook uses the selected machine-specific TDA feature subsets to calculate anomaly scores.

Normal training recordings are used as the reference set, and test recordings are scored according to their k-nearest-neighbor distance in the selected TDA feature space.

The notebook then:

1. computes anomaly scores,
2. generates DCASE-compatible output files, and
3. evaluates the results using the official DCASE 2025 Task 2 evaluation procedure.

Typical output files are:

```text
anomaly_score_{MachineType}_section_00_test.csv
decision_result_{MachineType}_section_00_test.csv
```

---

## Extracted Feature Files

The repository also contains the extracted TDA feature matrices so that the feature-selection and evaluation experiments can be reproduced without repeating the complete persistent-homology computation.

Evaluation-machine feature files follow names such as:

```text
cubical_mel_tda_features_AutoTrash.xlsx
cubical_mel_tda_features_AutoTrash_thr.xlsx
cubical_mel_tda_features_BandSealer.xlsx
cubical_mel_tda_features_BandSealer_thr.xlsx
...
```

Development-set feature files follow names such as:

```text
cubical_mel_tda_features_dev_ToyCar_train.xlsx
cubical_mel_tda_features_dev_ToyCar_test.xlsx
cubical_mel_tda_features_dev_ToyTrain_train.xlsx
cubical_mel_tda_features_dev_ToyTrain_test.xlsx
...
```

---

## Recommended Execution Order

The notebooks should be run in the following order:

```text
1. ORJ_DCASE_TDA_FEATURE_CREATOR.ipynb

2. ORJ_DCASE_FEATURE_SELECTION_FOR_DEVELOPMENT_DATASET.ipynb

3. ORJ_DCASE_FEATURE_SELECTON_FOR_UNSEEN_DATA.ipynb

4. ORJ_DCASE_EVALUATOR_AND_OFFICIAL_CODE.ipynb
```

If the extracted Excel feature files are already available, the computationally expensive TDA feature-extraction stage can be skipped.

---

## Dataset

This project is designed for the **DCASE 2025 Task 2 first-shot anomalous sound detection dataset**.

The raw audio dataset is not distributed in this repository and should be obtained from the official DCASE Challenge source.

---

## Installation

Clone the repository:

```bash
git clone https://github.com/korkutanapa/DCASE2025TASK2.git
cd DCASE2025TASK2
```

The notebooks are designed to run in Python/Jupyter environments such as **Google Colab**.

Typical dependencies include:

```bash
pip install numpy pandas scipy scikit-learn librosa matplotlib openpyxl
```

Additional persistent-homology and TDA libraries are installed or imported by the corresponding feature-generation notebook.

---

## Method Summary

The main objective of this study is to investigate whether **topological descriptors extracted from Mel spectrograms can provide transferable information for anomalous machine-sound detection**.

The overall methodology can be summarized as:

```text
Development machines
        ↓
TDA feature extraction
        ↓
Identification of informative TDA feature pool
        ↓
Transfer to unseen machine types
        ↓
Machine-specific feature selection
        ↓
kNN anomaly detection
        ↓
Official DCASE evaluation
```

The anomaly detector itself is deliberately simple, allowing the contribution of the TDA representation and feature-selection strategy to be studied directly.

---

## Citation

If you use this repository or the proposed methodology, please cite the related work:

**Enhancing First-Shot Anomalous Sound Detection in Noisy Industrial Environments**

---

## Author

**Korkut Anapa**
Institute of Applied Mathematics
Middle East Technical University
Ankara, Türkiye
