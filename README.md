# Cancer Risk Prediction using Machine Learning

A machine-learning application that predicts a patient's **cancer risk level** as **Low, Medium, or High** from a set of demographic, lifestyle, environmental, and health-related risk factors.

The project includes a training/experimentation notebook and a **Streamlit web application** for both single-patient and batch CSV predictions.

> **Important:** This project is intended for educational and research purposes only. It is **not a medical diagnostic tool** and predictions should not be used as a substitute for professional medical advice, diagnosis, or treatment.

---

## Features

- Multiclass cancer-risk prediction:
  - `Low`
  - `Medium`
  - `High`
- Class-weighted **XGBoost** model for handling imbalanced risk classes.
- Streamlit web interface.
- Two prediction modes:
  - **Manual input** for a single patient.
  - **CSV upload** for batch predictions.
- Displays class probabilities for predictions.
- Allows batch prediction results to be downloaded as a CSV file.
- Uses saved model artifacts so the trained model does not need to be retrained when the application starts.
- Training notebook includes experiments with:
  - Random Forest
  - Logistic Regression
  - SMOTE
  - Optuna hyperparameter optimization
  - XGBoost
  - Class-weighted XGBoost

---

## Project Structure

```text
Cancer_Risk_prediction - Final/
│
├── app.py
├── main.py
├── Cancer_Risk_Prediction_(ML).ipynb
├── cancer-risk-factors.csv
│
├── final_xgb_class_weighted.pkl
├── label_encoder.pkl
├── feature_names.pkl
│
├── requirements.txt
├── pyproject.toml
├── uv.lock
├── .python-version
└── README.md
```

### Important files

| File | Description |
|---|---|
| `app.py` | Streamlit application for cancer-risk prediction |
| `Cancer_Risk_Prediction_(ML).ipynb` | Model development, experimentation, evaluation, and tuning |
| `cancer-risk-factors.csv` | Dataset used for model development |
| `final_xgb_class_weighted.pkl` | Saved XGBoost classification model used by the application |
| `label_encoder.pkl` | Saved encoder used to convert model classes back to `High`, `Low`, and `Medium` |
| `feature_names.pkl` | Saved list defining the exact input feature order expected by the model |
| `requirements.txt` | Python dependencies |
| `pyproject.toml` | Project metadata and dependency configuration |

---

## Dataset

The included dataset contains **2,000 records and 21 columns**.

The original dataset contains patient identifiers, cancer type, risk factors, and the target variable `Risk_Level`.

### Target

```text
Risk_Level
├── Low
├── Medium
└── High
```

The dataset contains:

- 1,574 `Medium` risk records
- 324 `Low` risk records
- 102 `High` risk records

This class imbalance is one reason class weighting and other imbalance-handling approaches were explored during model development.

### Model input features

The deployed model uses the following **17 numeric features**:

```text
Age
Gender
Smoking
Alcohol_Use
Obesity
Family_History
Diet_Red_Meat
Diet_Salted_Processed
Fruit_Veg_Intake
Physical_Activity
Air_Pollution
Occupational_Hazards
BRCA_Mutation
H_Pylori_Infection
Calcium_Intake
BMI
Physical_Activity_Level
```

`Patient_ID`, `Cancer_Type`, `Overall_Risk_Score`, and `Risk_Level` are not used as inputs to the deployed model.

> For CSV prediction, the uploaded file must contain the required feature columns. The application reorders them according to `feature_names.pkl`. Missing columns are currently filled with `0`, so input data should be validated carefully before use.

---

## Machine Learning Workflow

The notebook explores several approaches before producing the saved model used by the Streamlit application.

### 1. Data preparation

The target column is:

```text
Risk_Level
```

The following columns were excluded from the model input during the final modeling workflow:

```text
Patient_ID
Cancer_Type
Overall_Risk_Score
```

The target labels were encoded using `LabelEncoder`.

The dataset was split using a stratified train/test split:

```text
Training: 80%
Testing: 20%
Random state: 42
```

### 2. Class imbalance

The target classes are imbalanced, particularly the `High` risk class.

The notebook therefore experimented with:

- SMOTE
- Random Forest
- XGBoost
- Class weighting

The final deployed approach uses **class-weighted XGBoost**.

### 3. Hyperparameter optimization

Optuna was used to search for XGBoost hyperparameters while optimizing multiclass **macro F1** during the final class-weighted XGBoost development stage.

The search included parameters such as:

- `n_estimators`
- `max_depth`
- `learning_rate`
- `subsample`
- `colsample_bytree`
- `gamma`
- `reg_alpha`
- `reg_lambda`

The saved model in the repository is the trained `XGBClassifier` artifact:

```text
final_xgb_class_weighted.pkl
```

---

## Streamlit Application

The application is implemented in `app.py`.

Run it with:

```bash
streamlit run app.py
```

The application provides two modes.

### 1. Manual input

Select:

```text
Manual input (single)
```

Enter the patient's feature values through the sidebar and click:

```text
Predict
```

The application displays:

- Predicted risk level
- Probability for each risk class
- Probability of `High` risk
- A warning when the predicted `High` probability is at least `0.50`

### 2. Batch CSV prediction

Select:

```text
Upload CSV (batch)
```

Upload a CSV containing the required model features.

The application:

1. Reads the CSV.
2. Selects the required features.
3. Reorders them according to `feature_names.pkl`.
4. Converts values to numeric values.
5. Generates predictions.
6. Adds class probabilities.
7. Displays the results.
8. Provides a CSV download.

Example output columns:

```text
Age
Gender
...
Physical_Activity_Level
Predicted_Risk_Level
prob_High
prob_Low
prob_Medium
```

---

## Installation

### Prerequisites

The project configuration specifies:

```text
Python >= 3.13
```

Creating a virtual environment is recommended.

### Create virtual environment

```bash
python -m venv .venv
```

### Activate it

#### Windows

```bash
.venv\Scripts\activate
```

#### Linux/macOS

```bash
source .venv/bin/activate
```

### Install dependencies

```bash
pip install streamlit pandas numpy joblib scikit-learn xgboost
```

If you want to reproduce the experimentation notebook, also install:

```bash
pip install imbalanced-learn optuna jupyter
```

> **Note:** The repository's `requirements.txt` currently contains `xgboot`, which appears to be a typo for `xgboost`. Installing `xgboost` explicitly as shown above avoids that issue.

---

## Run the Application

From the project directory:

```bash
streamlit run app.py
```

Streamlit will start a local server and provide a URL similar to:

```text
http://localhost:8501
```

Open the URL in your browser.

---

## CSV Input Format

For batch prediction, the CSV should contain the model's 17 feature columns.

A minimal header is:

```csv
Age,Gender,Smoking,Alcohol_Use,Obesity,Family_History,Diet_Red_Meat,Diet_Salted_Processed,Fruit_Veg_Intake,Physical_Activity,Air_Pollution,Occupational_Hazards,BRCA_Mutation,H_Pylori_Infection,Calcium_Intake,BMI,Physical_Activity_Level
```

Do not include `Risk_Level` when using the application for prediction.

The application can ignore extra columns, but the safest approach is to provide exactly the required model features.

---

## Model Artifacts

The application loads three serialized artifacts:

```python
model = joblib.load("final_xgb_class_weighted.pkl")
le = joblib.load("label_encoder.pkl")
feature_names = joblib.load("feature_names.pkl")
```

These files must remain in the same directory as `app.py` unless the loading paths in the application are changed.

### Why `feature_names.pkl` is important

The model expects features in a specific order. `feature_names.pkl` stores that order and is used by the application to ensure the input dataframe matches the model's expected feature layout.

---

## Example Workflow

```text
Patient / CSV Input
        │
        ▼
Input Validation & Feature Selection
        │
        ▼
Feature Ordering
        │
        ▼
Saved XGBoost Model
        │
        ▼
Risk Prediction
        │
        ├── Low
        ├── Medium
        └── High
        │
        ▼
Class Probabilities
        │
        ▼
Display / CSV Download
```

---

## Model Development Experiments

The notebook contains multiple stages of experimentation rather than only the final model.

### Models explored

- Random Forest Classifier
- Logistic Regression
- XGBoost Classifier
- Class-weighted XGBoost

### Imbalance techniques explored

- SMOTE
- Inverse-frequency class weighting

### Evaluation

The notebook uses metrics including:

- Precision
- Recall
- F1-score
- Macro F1-score
- Confusion matrix

Macro F1 is particularly useful for this multiclass problem because it gives each risk class equal importance rather than allowing the majority `Medium` class to dominate the evaluation.

---

## Limitations

This project has several important limitations:

1. **The model is not clinically validated.**
2. The dataset contains a relatively small number of `High` risk examples compared with `Medium` risk examples.
3. The quality of predictions depends heavily on the quality and distribution of the training data.
4. The application currently converts invalid or missing numeric input to `0`, which may not represent a meaningful patient value.
5. The probability values produced by the classifier should not be interpreted as medically calibrated probabilities without appropriate validation and calibration.
6. The project is a machine-learning demonstration and should not be used to make clinical decisions.

---

## Future Improvements

Potential improvements include:

- Add proper input validation and clinically meaningful value ranges.
- Replace automatic missing-value filling with explicit validation or user prompts.
- Add feature descriptions and units to the Streamlit interface.
- Add model explainability using SHAP.
- Add probability calibration and calibration curves.
- Evaluate the model using repeated stratified cross-validation.
- Report per-class performance, especially recall for the `High` class.
- Add automated tests for preprocessing and prediction.
- Package the model and preprocessing pipeline together to prevent training/inference inconsistencies.
- Containerize the application using Docker.
- Deploy the Streamlit application to a suitable hosting platform.
- Add model/version metadata and reproducible training configuration.

---

## Disclaimer

This project is provided for **educational and research purposes only**.

A prediction of `Low`, `Medium`, or `High` risk from this application does not constitute a medical diagnosis, prognosis, or treatment recommendation. Any real-world medical assessment should be performed by qualified healthcare professionals using clinically validated methods and appropriate patient information.

---

## License

No license is currently specified for this repository. If this project is intended for public distribution, add an appropriate license file such as `MIT`, `Apache-2.0`, or another license suitable for the project.
