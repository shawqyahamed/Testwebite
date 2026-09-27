# COMP70049 – Assignment 1: Machine Learning for Cyber Security

**Student ID:** CB018446
**Module:** COMP70049

This submission contains four Sections. Each one applies one classic machine learning model and one deep learning model to a cyber security problem, then compares them with accuracy, precision, recall, F1-score, confusion matrices, ROC and precision-recall curves, and training plots.

| Section | Problem | Dataset | Classic ML model | Deep learning model |
|---|---|---|---|---|
| 01 | Phishing / spam email detection | Enron Spam (Kaggle) | Logistic Regression (TF-IDF) | LSTM |
| 02 | Network intrusion detection (multi-class) | NSL-KDD (Kaggle) | Random Forest | 1D CNN |
| 03 | Network anomaly detection (unsupervised) | UNSW-NB15 (Kaggle) | Isolation Forest | Autoencoder |
| 04 | Ransomware detection | CIC-AndMal2017 (UNB CIC) | Gradient Boosting | Bidirectional LSTM |

---

## 1. Files in this submission

```
CB018446_Section_01.ipynb   # Notebook – Spam/phishing email detection
CB018446_Section_02.ipynb   # Notebook – Intrusion detection (NSL-KDD)
CB018446_Section_03.ipynb   # Notebook – Anomaly detection (UNSW-NB15)
CB018446_Section_04.ipynb   # Notebook – Ransomware detection (CIC-AndMal2017)
cb018446_section_01.py      # Python script export of Section 01
cb018446_section_02.py      # Python script export of Section 02
cb018446_section_03.py      # Python script export of Section 03
cb018446_section_04.py      # Python script export of Section 04
README.md                   # This file
```

The `.ipynb` notebooks are the primary deliverable and contain saved outputs. The `.py` files are direct exports of the same notebooks from Google Colab.

---

## 2. Requirements

### Recommended environment
- **Google Colab** (all notebooks were developed and run there; Python 3.13 runtime).
- A **GPU runtime** is optional but speeds up the deep learning models (Runtime → Change runtime type → T4 GPU).

### Python libraries
Each notebook installs its own dependencies in the first cell:

```bash
pip install numpy pandas scikit-learn matplotlib tensorflow
```

Sections 01–03 also use the **Kaggle CLI** (`kaggle`), which is pre-installed on Colab. Section 03 reads Parquet files, which needs `pyarrow` (pre-installed on Colab; run `pip install pyarrow` if running locally).

### Kaggle API token (Sections 01–03)
Sections 01, 02 and 03 download their datasets automatically from Kaggle. You need your own Kaggle API token:

1. Sign in at https://www.kaggle.com → **Settings** → **API** → **Create New Token**.
2. In the second code cell of each notebook, replace the token value with your own:
   ```python
   os.environ["KAGGLE_API_TOKEN"] = "<YOUR_KAGGLE_API_TOKEN>"
   ```

---

## 3. How to run (Google Colab)

1. Go to https://colab.research.google.com and choose **File → Upload notebook**.
2. Upload the notebook for the section you want to run.
3. (Optional) Switch to a GPU runtime.
4. Complete the dataset step for that section (see Section 4 below).
5. Click **Runtime → Run all**.
6. Results (metrics, classification reports) print in the cell outputs, and all graphs display inline.

Run each notebook in its own fresh runtime. The sections are independent of each other and do not share files or variables.

### Running the `.py` scripts instead
The `.py` exports contain Colab shell commands (lines starting with `!`) and, in Section 04, `google.colab.files.upload()`. They are intended to be pasted into or run inside Colab. To run them locally, replace the `!` lines with their terminal equivalents (for example, run `kaggle datasets download ...` in a terminal first) and point the file paths at your local copy of the data.

---

## 4. Dataset setup per section

### Section 01 – Spam / phishing email detection
- **Dataset:** Enron Spam Data — Kaggle `marcelwiechmann/enron-spam-data`
- **Setup:** Automatic. The notebook downloads and unzips the data to `~/Downloads/enron_spam_data.csv`.
- **What it does:** Combines subject and body, cleans the text (lowercase, removes URLs, email addresses, punctuation and numbers), splits 70/30 (stratified), then trains:
  - Logistic Regression on TF-IDF features (10,000 features, unigrams + bigrams)
  - LSTM on tokenised, padded sequences (vocab 10,000, length 300, 5 epochs)
- **Outputs:** class distribution chart, confusion matrices, ROC curves, LSTM accuracy plot, model comparison table and bar chart.

### Section 02 – Network intrusion detection
- **Dataset:** NSL-KDD — Kaggle `kiranmahesh/nslkdd`
- **Setup:** Automatic. Files are downloaded to `~/Downloads/` (`kdd_train.csv`, `kdd_test.csv`).
- **What it does:** Assigns the 41 NSL-KDD column names, maps raw attack labels into five classes (Normal, DoS, Probe, R2L, U2R), one-hot encodes `protocol_type`, `service` and `flag`, scales numeric features, then trains:
  - Random Forest (200 trees, max depth 25, balanced class weights)
  - 1D CNN on 32 PCA components (2 convolution blocks, 20 epochs)
- **Outputs:** category distribution, confusion matrices, per-class precision-recall curves, top-15 feature importances, CNN accuracy plot, comparison table and chart.

### Section 03 – Network anomaly detection
- **Dataset:** UNSW-NB15 — Kaggle `dhoogla/unswnb15`
- **Setup:** Automatic. Files are downloaded to `~/Downloads/` (`UNSW_NB15_training-set.parquet`, `UNSW_NB15_testing-set.parquet`).
- **What it does:** Cleans the data, removes duplicates, selects the top 15 numeric features by mutual information, one-hot encodes `proto`, `service` and `state`, and trains both models on **normal traffic only** (unsupervised):
  - Isolation Forest (200 trees, contamination 0.1)
  - Autoencoder (64-32-16-32-64, 30 epochs); anomaly threshold = 95th percentile of reconstruction error on normal training data
- **Outputs:** label distribution, confusion matrices, ROC and precision-recall curves, autoencoder loss plot, comparison table and chart.

### Section 04 – Ransomware detection
- **Dataset:** CIC-AndMal2017 — https://www.unb.ca/cic/datasets/andmal2017.html
- **Setup:** Manual upload (this dataset is not on Kaggle).
  1. Download from the UNB CIC site: `Ransomware-CSVs.zip`, `Benign-CSVs.zip`, and their matching `Ransomware-CSVs.md5` and `Benign-CSVs.md5` files.
  2. When the upload cell runs, a **Choose Files** button appears. Select **all four files** at once.
  3. The notebook computes MD5 hashes of the zip files and prints them next to the expected values from the `.md5` files. Check they match before continuing.
  4. The notebook then unzips both archives and finds all CSV files automatically.
- **What it does:** Labels ransomware flows 1 and benign flows 0, engineers features from raw IP and timestamp columns (private/public source and destination IP, hour of day), drops identifier columns, removes rows with missing values and duplicates, selects the top 30 features by mutual information, log-transforms skewed features, scales, splits 70/30 (stratified), then trains:
  - Gradient Boosting (200 estimators, max depth 4, learning rate 0.1)
  - Bidirectional LSTM (each selected feature treated as a timestep, 20 epochs)
- **Outputs:** class distribution, confusion matrices, ROC and precision-recall curves, top-15 feature importances, LSTM accuracy plot, comparison table and chart.
- **Note:** The CIC-AndMal2017 archives are large. Uploading through the browser can take a while; mounting Google Drive and copying the files from there is a faster alternative.

---

## 5. Reproducibility

- A fixed `random_state=42` is used for data splits, scikit-learn models and PCA.
- Deep learning results can vary slightly between runs because TensorFlow training is not fully deterministic, especially on GPU.
- Scalers and encoders are fitted on the training data only and then applied to the test data, to avoid data leakage.

---

## 6. Troubleshooting

| Problem | Fix |
|---|---|
| `401 Unauthorized` or `403` from Kaggle | Check the Kaggle token in cell 2, and make sure you have accepted the dataset's terms on its Kaggle page. |
| `FileNotFoundError` for a dataset file | Re-run the download cell and check the `ls` output. Paths assume Colab's home directory (`/root`). |
| Section 04 finds no CSV files | Make sure the zip files were uploaded with their original names (`Ransomware-CSVs.zip`, `Benign-CSVs.zip`). |
| Section 04 MD5 values do not match | The download is corrupted or incomplete. Download the zip again. |
| Colab runs out of memory | Restart the runtime and run only one section per session. Section 04 is the most memory-intensive. |
| Deep learning training is slow | Switch to a GPU runtime. |
