# Multimodal Autism Classification from fMRI and Anatomy

End‑to‑end notebooks for classifying Autism Spectrum Condition (ASC) vs. Control from neuroimaging. The repo includes a **multimodal deep‑learning pipeline** (resting‑state fMRI + anatomical MRI features) and a **gradient‑boosted baseline** on ROI‑level timeseries features.

> Notebooks:
>
> • `Multimodal_Autism_FMRI_Classification.ipynb` — full pipeline (data loading → preprocessing → feature engineering → model → evaluation)
>
> • `Resting_State_FMRI_XGBoost_Model.ipynb` — classical ML baseline on extracted fMRI features

---

## Overview

* **Task**: Binary classification (ASC vs. Control) from resting‑state fMRI; optional T1/anatomical features.
* **Modalities**: rs‑fMRI (required), T1/anatomical MRI (optional, if available).
* **Pipelines**:

  * **Multimodal DL**: CNN/DenseNet‑style feature extractor for volumes/time or ROI series → fusion → classifier.
  * **XGBoost Baseline**: ROI‑level summary features (e.g., functional connectivity/statistics) → gradient boosting.
* **Tooling**: `nilearn`, `nibabel`, `scikit‑learn`, `xgboost`, and PyTorch (or Keras) for the DL model; fully runnable in the provided notebooks.

---

## Data

This project expects **resting‑state fMRI NIfTI files** (optionally T1 anatomical NIfTI). You can run on:

* **Public datasets** such as ABIDE I/II (found here: https://fcon_1000.projects.nitrc.org/indi/abide/abide_II.html), or
* **Custom dataset** organized in subject folders.

**Typical folder layout**

```
Data/
  sub-0001/
    func/rs_fmri.nii.gz
    anat/T1w.nii.gz        # optional
  sub-0002/
    func/rs_fmri.nii.gz
    anat/T1w.nii.gz        # optional
  ...
labels.csv                 # columns: subject_id,label (0=Control, 1=ASC)
```

> 📝 The notebooks include cells to adapt paths, filename patterns, and label CSV formats the data.

---

## Preprocessing & Feature Engineering

* **Loading**: `nibabel`/`nilearn` to load NIfTI volumes.
* **Standardization**: temporal de‑trending/standardization (rs‑fMRI).
* **Spatial handling**: resampling to a common template (e.g., MNI 2mm) for consistent shape.
* **Masking / ROI extraction**: Harvard–Oxford atlas (or your preferred atlas) via `nilearn.input_data` masker.
* **Feature sets**:

  * **Voxel/patch sequences** (for DL).
  * **ROI‑level summaries** (means, variance, temporal features).
  * **Connectivity matrices** (optional; Fisher‑z correlations among ROI time series).

All steps are implemented in the notebooks with clear, editable parameters.

---

## Models

### 1) Multimodal Deep Learning (`Multimodal_Autism_FMRI_Classification.ipynb`)

* **Backbone**: CNN/DenseNet‑style blocks to encode fMRI (and optional anatomical) volumes or ROI sequences.
* **Fusion**: Concatenation / attention‑style fusion of modality embeddings.
* **Head**: Fully‑connected layers → sigmoid output for binary classification.
* **Training**: BCE/CE loss, early stopping on validation AUC/accuracy; configurable batch size/epochs.

### 2) XGBoost Baseline (`Resting_State_FMRI_XGBoost_Model.ipynb`)

* **Input**: ROI‑level feature table (statistics and/or connectivity features).
* **Model**: `xgboost.XGBClassifier` with hyperparameters you can tune via grid/Optuna.
* **Why baseline**: Fast iterate‑and‑compare to ensure the DL model meaningfully improves over classical ML.

---

## Results


**Split**: 80/20 stratified subject split (no subject leakage)

| Data       | Acc  | Loss |
| ---------- | ---- | ---- |
| Test       | 0.70 | 0.14 |

* **Reproducibility**: fixed random seeds and deterministic data splits where possible.

---

## Installation

### 1) Clone

```bash
git clone https://github.com/ET-777/Austim_FMRI_Classification.git
cd Austim_FMRI_Classification
```

### 2) Environment

Use Python 3.10+ and install requirements (adjust as needed):

```bash
pip install numpy pandas scikit-learn xgboost nibabel nilearn matplotlib torch torchvision torchaudio
```

> If you prefer TensorFlow/Keras, swap the DL dependencies accordingly.

> 📦 GPU needed.

---

## Usage

### A) Multimodal DL notebook

1. Open `Multimodal_Autism_FMRI_Classification.ipynb` in Jupyter/Colab.
2. Set your `Data/` path and label CSV path.
3. Run preprocessing cells (masker setup, resampling, ROI extraction).
4. Configure model hyperparameters (epochs, batch size, learning rate).
5. Train → evaluate → save metrics/plots.

### B) XGBoost baseline notebook

1. Open `Resting_State_FMRI_XGBoost_Model.ipynb`.
2. Point to your precomputed ROI feature table (or let the notebook compute it).
3. Train XGBoost, examine feature importance, and evaluate.

---

## Reproducibility

* Deterministic train/val/test subject split with fixed seed.
* Version your environment.
* Save trained weights and results with a run ID.

---

## Repository Structure (suggested)

```
Austim_FMRI_Classification/
  Data/                      # your data lives outside version control
  notebooks/
    Multimodal_Autism_FMRI_Classification.ipynb
    Resting_State_FMRI_XGBoost_Model.ipynb
  artifacts/                 # saved models, metrics, plots
  README.md
  requirements.txt
```

---

## Notes & Tips

* **Atlas choice** matters; try Harvard–Oxford, AAL, or Schaefer atlases.
* **Confounds**: consider motion parameters, global signal, and nuisance regression.
* **Subject‑level CV**: ensure splits are by subject, not by volume, to avoid leakage.
* **Class imbalance**: use stratified splits and/or class weights.

---

## Citation / Acknowledgments

ABIDE I/II
https://fcon_1000.projects.nitrc.org/indi/abide/abide_II.html

---

## License

This project is released under the MIT License.
