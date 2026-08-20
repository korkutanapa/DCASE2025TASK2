# DCASE 2025 Task 2 – TDA-Based Anomalous Sound Detection

This repository contains the code and extracted feature files used for a Topological Data Analysis (TDA)-based approach to **DCASE 2025 Task 2: First-Shot Unsupervised Anomalous Sound Detection for Machine Condition Monitoring**.

## Overview

The framework converts machine sounds into topological descriptors derived from Mel spectrograms:

```text
WAV
 ↓
Mel spectrogram
 ↓
Cubical complex
 ↓
Persistent homology (H0 and H1)
 ↓
266 TDA descriptors
 ↓
Feature selection
 ↓
kNN anomaly scoring
```

The study follows two feature-selection stages:

1. **Development machines:** labeled development data are used to identify anomaly-informative TDA descriptors, producing a transferable pool of **68 features**.
2. **Unseen machines:** a machine-specific subset is selected from this pool using **only the available normal training recordings** (990 source-normal + 10 target-normal). Test recordings and test labels are not used during feature selection.

After the subset is fixed, anomaly scoring is performed by a separate kNN evaluator using the selected feature space.

## Main Notebooks

### 1. TDA Feature Extraction

`ORJ_DCASE_TDA_FEATURE_CREATOR_CLEAN.ipynb`

Generates the TDA feature vectors from WAV recordings using Mel spectrograms, cubical persistent homology, and H0/H1 persistence descriptors.

### 2. Development-Set Feature Search

`ORJ_DCASE_FEATURE_SELECTION_FOR_DEVELOPMENT_DATASET_CLEAN.ipynb`

Searches the labeled DCASE development machines for informative TDA feature subsets and kNN settings.

The selected machine-specific subsets form a transferable pool of **68 unique TDA descriptors**.

### 3. Train-Only Feature Selection for Unseen Machines

`ORJ_DCASE_FEATURE_SELECTION_FOR_UNSEEN_DATA_TRAIN_ONLY.ipynb`

Selects a machine-specific feature subset for each unseen evaluation machine using only its normal training recordings.

The source-normal and target-normal domains are analyzed separately during this selection stage. The resulting subset is fixed before any test recording is evaluated.

### 4. Anomaly Scoring and Evaluation

`ORJ_DCASE_EVALUATOR_AND_OFFICIAL_CODE.ipynb`

Uses the selected machine-specific feature subsets for final anomaly scoring.

The normal source and target training recordings are combined into a normal reference bank, and each test recording is scored using the mean Euclidean distance to its **10 nearest normal neighbors**.

## Extracted Feature Files

The repository includes precomputed TDA feature matrices, allowing the feature-selection and evaluation stages to be reproduced without recomputing persistent homology.

Typical evaluation files are:

```text
cubical_mel_tda_features_AutoTrash_thr.xlsx
cubical_mel_tda_features_AutoTrash.xlsx
cubical_mel_tda_features_BandSealer_thr.xlsx
cubical_mel_tda_features_BandSealer.xlsx
...
```

The `_thr.xlsx` files contain the normal training recordings, while the corresponding `.xlsx` files contain the evaluation recordings.

Development files follow the format:

```text
cubical_mel_tda_features_dev_ToyCar_train.xlsx
cubical_mel_tda_features_dev_ToyCar_test.xlsx
...
```

## Recommended Execution Order

```text
1. ORJ_DCASE_TDA_FEATURE_CREATOR_CLEAN.ipynb
2. ORJ_DCASE_FEATURE_SELECTION_FOR_DEVELOPMENT_DATASET_CLEAN.ipynb
3. ORJ_DCASE_FEATURE_SELECTION_FOR_UNSEEN_DATA_TRAIN_ONLY.ipynb
4. ORJ_DCASE_EVALUATOR_AND_OFFICIAL_CODE.ipynb
```

If the extracted Excel feature files are already available, the TDA feature-extraction stage can be skipped.

## Dataset

The raw audio dataset is not included in this repository. It should be obtained from the official **DCASE 2025 Task 2** dataset.

## Environment

The notebooks are designed for Python/Jupyter environments such as Google Colab. Main dependencies include:

```bash
pip install numpy pandas scipy scikit-learn librosa matplotlib openpyxl
```

Additional TDA dependencies are imported or installed in the feature-generation notebook.

## Related Work

**Enhancing First-Shot Anomalous Sound Detection in Noisy Industrial Environments**

Korkut Anapa, İsmail Güzel, and Ceylan Yozgatlıgil.
