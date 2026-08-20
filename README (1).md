# DCASE 2025 Task 2 – TDA-Based First-Shot Anomalous Sound Detection

This repository contains the notebooks and extracted feature files used to study a **Topological Data Analysis (TDA)** representation for **DCASE 2025 Task 2: First-Shot Unsupervised Anomalous Sound Detection for Machine Condition Monitoring**.

The project is designed to investigate whether topological descriptors extracted from Mel spectrograms can provide useful and transferable information for anomalous machine-sound detection while keeping the final anomaly detector deliberately simple.

> **Important methodological distinction**
>
> - The **development-set stage** is a supervised oracle/capability analysis: development test labels are intentionally used to identify informative TDA descriptors.
> - The **unseen/evaluation-machine feature-selection stage** is strict train-only: it uses only the 990 source-normal and 10 target-normal training recordings for each unseen machine. Evaluation/test recordings and their labels are not used for feature selection.

---

## Processing Pipeline

```text
WAV recording
    ↓
waveform normalization
    ↓
Mel spectrogram
    ↓
normalized cubical filtration
    ↓
Persistent Homology (H0 and H1)
    ↓
TDA descriptor extraction
    ↓
Development-set oracle feature analysis
    ↓
frozen development-derived feature pool
    ↓
strict train-only machine-specific feature selection
    ↓
kNN anomaly scoring
    ↓
DCASE-compatible output files
    ↓
official DCASE 2025 Task 2 evaluation
```

---

## Repository Contents

The current repository contains four main notebooks.

### 1. `ORJ_DCASE_TDA_FEATURE_CREATOR_CLEAN.ipynb`

Creates TDA feature vectors directly from WAV recordings.

Main operations:

1. recursively finds WAV files,
2. loads each recording as mono while preserving the original sampling rate,
3. standardizes the waveform,
4. computes a Mel spectrogram,
5. converts the Mel power spectrogram to dB,
6. constructs a normalized cubical filtration,
7. computes persistent homology for `H0` and `H1`,
8. extracts a broad collection of TDA descriptors, and
9. saves the resulting feature matrix as an Excel file.

Current Mel-spectrogram settings in the notebook are:

```text
N_FFT      = 2048
HOP_LENGTH = 512
N_MELS     = 128
SR         = original recording sampling rate
```

The notebook uses GUDHI's `CubicalComplex` implementation for persistent homology.

The extracted descriptor families include, among others:

- birth statistics,
- death statistics,
- persistence-lifetime statistics,
- persistence entropy,
- Betti-curve descriptors,
- persistence landscapes,
- persistence silhouettes,
- persistence-image descriptors,
- Carlsson-coordinate features,
- dominant/tail persistence statistics,
- weighted birth/death/midlife descriptors, and
- cross-homology / `H0`–`H1` interaction descriptors.

---

### 2. `ORJ_DCASE_FEATURE_SELECTION_FOR_DEVELOPMENT_DATASET_CLEAN.ipynb`

Performs the **development-set oracle analysis**.

This notebook uses the labeled DCASE development test data intentionally. Its purpose is not to simulate unseen-machine training, but to determine whether the TDA representation contains anomaly-relevant information and to identify a transferable descriptor pool.

The current machine-specific search evaluates:

```text
k ∈ {3, 5, 10, 20, 30}
subset size = 1 ... 20
```

The default ranking objective is

```text
HM(AUC_source, AUC_target, pAUC@0.1)
```

with the following tie-breakers:

1. `pAUC@0.1`,
2. `min(AUC_source, AUC_target)`,
3. overall AUC,
4. fewer selected features, and
5. smaller `k`.

The search combines:

- exhaustive single-feature evaluation,
- exhaustive pair evaluation,
- k-preserving beam search for higher-dimensional subsets,
- a train-normal redundancy filter, and
- optional same-size swap refinement.

Important outputs include:

```text
/content/joint_k_machine_specific_tda_oracle/
    joint_k_machine_specific_tda_oracle.xlsx
    best_oracle_feature_pool.json
```

The notebook also contains a global-subset/global-`k` oracle analysis that can search for one common subset used across all development machines.

### Why labels are allowed here

This stage is a **development oracle/capability study**. Development labels are used to learn which TDA descriptors are informative. They are not used to select features from the unseen evaluation recordings.

---

### 3. `ORJ_DCASE_FEATURE_SELECTION_FOR_UNSEEN_DATA_TRAIN_ONLY.ipynb`

Performs **strict train-only machine-specific feature selection** for the eight unseen DCASE evaluation machine types:

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

The notebook starts from a frozen development-derived pool of **68 TDA descriptors**.

For each machine, feature selection uses only:

```text
990 source-normal training recordings
 10 target-normal training recordings
-------------------------------------
1000 normal training recordings total
```

The evaluation/test feature file is **not loaded during subset selection**.

### Train-only preprocessing

For the subset-selection search, feature-wise min-max parameters are estimated from the pooled 1000 normal training recordings. Source-normal and target-normal samples are then retained as separate reference banks.

### Train-only proxy scoring

Candidate feature subsets are evaluated using two normal reference geometries:

```text
source-normal kNN: kS = 10
target-normal kNN: kT = 3
```

For normal samples, leave-one-out distances are used when scoring against their own reference bank. Distances are divided by `sqrt(number_of_features)` for dimensional comparability.

The domain-specific distances are combined through

```text
score = min(d_source, d_target)
```

for the normal-only reliability calculations.

### Normal-only feature-selection objective

The label-free subset fitness combines:

- source-normal compactness,
- target-normal compactness,
- source-tail stability,
- target-tail stability,
- target-normal acceptance under source-normal geometry,
- source/target stability balance, and
- a redundancy penalty.

The current positive-component weights are:

```text
source compactness       0.20
target compactness       0.20
source tail stability    0.15
target tail stability    0.15
target acceptance        0.15
domain stability balance 0.15
```

with a redundancy penalty applied separately.

### Search strategy

```text
D = 1      exhaustive search over 68 features
D = 2      exhaustive search over C(68,2) = 2278 pairs
D = 3..20  beam search + random candidate injections
```

Candidate quality is also calibrated within each dimensionality using a robust z-score so that subsets with different numbers of features can be compared more fairly.

The notebook reports the highest-ranked subsets and produces an evaluator-ready:

```python
FEATURES_BY_MACHINE = {...}
```

dictionary.

Typical outputs include:

```text
selected_top10_by_machine.csv
best_by_dimension.csv
feature_effectiveness_ranking.csv
```

---

### 4. `ORJ_DCASE_EVALUATOR_AND_OFFICIAL_CODE.ipynb`

Uses fixed machine-specific feature subsets to calculate anomaly scores and run the official DCASE 2025 Task 2 evaluator.

The current notebook contains a hard-coded:

```python
FEATURES_BY_MACHINE = {...}
```

dictionary.

If the train-only feature-selection notebook is rerun and produces different rank-1 subsets, update this dictionary before evaluating the new selection.

### Final anomaly detector

The final detector is intentionally simple.

For every machine:

1. the selected feature columns are loaded,
2. preprocessing parameters are fitted **only on source-normal training samples**,
3. each feature is centered by its source-normal median,
4. the scale is chosen from source-normal IQR, MAD, or standard deviation as a numerical fallback,
5. the 990 source-normal and 10 target-normal samples are pooled into one normal reference bank,
6. Euclidean kNN is fitted with `k = 10`, and
7. each test recording receives the mean distance to its 10 nearest normal samples.

Therefore,

```text
larger anomaly score = more anomalous
```

The optional binary decision threshold is the `0.99` quantile of leave-one-out normal-bank scores.

The notebook generates DCASE-compatible files such as:

```text
anomaly_score_{MachineType}_section_00_test.csv
decision_result_{MachineType}_section_00_test.csv
```

It then clones and runs the official evaluator from:

https://github.com/nttcslab/dcase2025_task2_evaluator

and exports the official score tables.

> **WARNING — Colab cleanup cell**
>
> The first executable cell of the current evaluator notebook deletes all files and directories under `/content`.
> If you already uploaded feature files or other data into `/content`, either skip/remove that cleanup cell or execute it **before** uploading the required files.

---

## Precomputed Feature Files

The repository includes precomputed Excel feature matrices, which makes it possible to reproduce feature-selection and evaluation experiments without recomputing persistent homology from the raw WAV recordings.

### Evaluation-machine files

For each unseen machine, the current naming convention is:

```text
cubical_mel_tda_features_{Machine}.xlsx
cubical_mel_tda_features_{Machine}_thr.xlsx
```

In the current workflow:

- `*_thr.xlsx` is treated as the training-feature file,
- the corresponding file without `_thr` is treated as the evaluation/test-feature file.

Examples:

```text
cubical_mel_tda_features_AutoTrash_thr.xlsx
cubical_mel_tda_features_AutoTrash.xlsx
cubical_mel_tda_features_CoffeeGrinder_thr.xlsx
cubical_mel_tda_features_CoffeeGrinder.xlsx
```

### Development-machine files

Development files follow the pattern:

```text
cubical_mel_tda_features_dev_{Machine}_train.xlsx
cubical_mel_tda_features_dev_{Machine}_test.xlsx
```

The repository currently includes development features for:

```text
ToyCar
ToyTrain
bearing
fan
gearbox
slider
valve
```

---

## Recommended Execution Order

### Reproduce the complete pipeline from WAV files

```text
1. ORJ_DCASE_TDA_FEATURE_CREATOR_CLEAN.ipynb
2. ORJ_DCASE_FEATURE_SELECTION_FOR_DEVELOPMENT_DATASET_CLEAN.ipynb
3. ORJ_DCASE_FEATURE_SELECTION_FOR_UNSEEN_DATA_TRAIN_ONLY.ipynb
4. Update FEATURES_BY_MACHINE in the evaluator if necessary
5. ORJ_DCASE_EVALUATOR_AND_OFFICIAL_CODE.ipynb
```

### Reproduce experiments from the feature files already stored in this repository

The expensive TDA feature-generation stage can be skipped:

```text
1. ORJ_DCASE_FEATURE_SELECTION_FOR_DEVELOPMENT_DATASET_CLEAN.ipynb
2. ORJ_DCASE_FEATURE_SELECTION_FOR_UNSEEN_DATA_TRAIN_ONLY.ipynb
3. Update FEATURES_BY_MACHINE in the evaluator if necessary
4. ORJ_DCASE_EVALUATOR_AND_OFFICIAL_CODE.ipynb
```

If you only want to evaluate the feature subsets already hard-coded in the evaluator notebook, run the evaluator notebook with the required `*_thr.xlsx` and test `.xlsx` files available in the configured base directory.

---

## Dataset

This work uses data from **DCASE 2025 Task 2**.

Official task page:

https://dcase.community/challenge2025/task-first-shot-unsupervised-anomalous-sound-detection-for-machine-condition-monitoring

Official datasets:

- Development dataset: https://zenodo.org/records/15097779
- Additional training dataset: https://zenodo.org/records/15392814
- Evaluation dataset: https://zenodo.org/records/15519362

The raw WAV datasets are not redistributed in this repository.

For each DCASE 2025 evaluation machine, the additional training set provides 990 source-domain normal recordings and 10 target-domain normal recordings. The evaluation set contains 200 unlabeled test recordings per machine.

---

## Installation

The notebooks are intended primarily for Python/Jupyter/Google Colab environments.

Clone the repository:

```bash
git clone https://github.com/korkutanapa/DCASE2025TASK2.git
cd DCASE2025TASK2
```

A practical local installation is:

```bash
pip install numpy pandas scipy scikit-learn librosa matplotlib openpyxl tqdm gudhi
```

The notebooks may also install missing packages automatically when executed in Google Colab.

---

## Methodological Notes

### Development oracle versus unseen-machine selection

The two feature-selection stages serve different purposes and should not be conflated.

**Development machines**

```text
labeled development test data
        ↓
supervised oracle search
        ↓
identify anomaly-informative TDA descriptors
        ↓
freeze transferable feature pool
```

**Unseen evaluation machines**

```text
990 source-normal + 10 target-normal only
        ↓
normal-only subset-selection criterion
        ↓
machine-specific feature subset
        ↓
fixed kNN anomaly detector
        ↓
evaluation test recordings
```

The evaluation test recordings are used only after feature selection, for anomaly scoring and official evaluation.

### Why use a simple kNN detector?

The anomaly detector is intentionally lightweight. This makes it easier to study the contribution of the TDA representation and the feature-selection strategy without confounding the analysis with a large learned classifier or embedding model.

---

## Official Evaluation

The official DCASE 2025 Task 2 evaluator is available at:

https://github.com/nttcslab/dcase2025_task2_evaluator

The official evaluation includes source-domain AUC, target-domain AUC, partial AUC, and the challenge aggregation procedure used to compute the official score.

---

## Citation

If you use this repository, please cite the corresponding study when its final bibliographic information is available:

**Enhancing First-Shot Anomalous Sound Detection in Noisy Industrial Environments**

Please also cite the official DCASE 2025 Task 2 dataset/task publications as required by the challenge organizers.

---

## Author

**Korkut Anapa**  
Institute of Applied Mathematics  
Middle East Technical University  
Ankara, Türkiye

---

## Scope of the Repository

This repository is a research codebase. Paths, Colab-specific settings, hard-coded feature dictionaries, and experiment parameters reflect the current experimental workflow and may need to be adapted for another environment.
