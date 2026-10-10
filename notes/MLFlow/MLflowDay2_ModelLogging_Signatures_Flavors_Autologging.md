# MLflow Day 2 — Model Logging, Signatures, Flavors & Autologging

## Objective

Understand how MLflow packages trained machine learning models so they can be stored, loaded, and used consistently across different environments.

The goal is to move from:

```text
Trained Scikit-learn Model
          ↓
     model.joblib
```

to:

```text
Trained Model
      ↓
MLflow Model
      │
      ├── Model Artifact
      ├── Flavors
      ├── Signature
      ├── Input Example
      ├── Dependencies
      └── Metadata
      ↓
Load Model
      ↓
Predict
```

**Important:** MLflow model logging does not automatically deploy a model into production.

---

## 1. Revisiting Day 1 — Experiment Tracking

On Day 1, we learned:

```text
MLflow
   │
   └── Experiment
          │
          ├── Run 1
          │    ├── Parameters
          │    ├── Metrics
          │    ├── Tags
          │    └── Artifacts
          │
          └── Run 2
               ├── Parameters
               ├── Metrics
               └── Artifacts
```

We also logged a Scikit-learn model:

```python
mlflow.sklearn.log_model(
    sk_model=model,
    name="incident_model"
)
```

But we didn't explore what MLflow actually stores when we log a model.

That's our Day 2 topic.

**Mental Model:**

```text
Day 1
    ↓
Track the training experiment

Day 2
    ↓
Package and load the trained model
```

---

## 2. What Is an MLflow Model?

An MLflow Model is a standardized package for representing a trained machine learning model.

It can include:

```text
MLflow Model
│
├── Trained Model
│
├── MLmodel Metadata
│
├── Model Flavors
│
├── Input/Output Signature
│
├── Input Example
│
└── Environment Dependencies
```

### Why is this useful?

Previously, we saved models using:

```python
import joblib

joblib.dump(
    model,
    "incident_model.joblib"
)
```

This saves the trained Python object.

But another engineer might ask:

- Which framework produced this model?
- What inputs does it expect?
- What output does it produce?
- Which Python packages are required?
- How should the model be loaded?
- What does a valid prediction request look like?

An MLflow Model helps answer these questions through its packaging and metadata.

**Mental Model:**

```text
Joblib
    ↓
Serialize a Python object

MLflow Model
    ↓
Package a model with
metadata and loading interfaces
```

---

## 3. MLflow Model Flavors

A **flavor** describes a supported interface for loading or using an MLflow Model.

Suppose we trained a Scikit-learn Logistic Regression model.

MLflow can represent it using two important flavors:

```text
                  MLflow Model
                       │
            ┌──────────┴──────────┐
            ▼                     ▼
      sklearn flavor         pyfunc flavor
            │                     │
            ▼                     ▼
     Native Scikit-learn      Generic MLflow
         interface             Python interface
```

## Scikit-learn Flavor

Used when we want to load the model as a native Scikit-learn estimator or pipeline.

```python
mlflow.sklearn.load_model(
    model_uri
)
```

The returned object supports the native Scikit-learn interface.

For example:

```python
model.predict(X_test)
```

Depending on the estimator, we may also access methods such as:

```python
model.predict_proba(X_test)
```

## Pyfunc Flavor

`pyfunc` stands for **Python Function**.

It provides a common prediction interface for supported MLflow models.

```python
mlflow.pyfunc.load_model(
    model_uri
)
```

Then:

```python
model.predict(X_test)
```

### Why does pyfunc matter?

Imagine an ML platform supports:

```text
Logistic Regression
Random Forest
XGBoost
PyTorch
Custom Python Models
```

Without a common interface, model-serving infrastructure might require framework-specific handling.

With pyfunc:

```text
Different Model Frameworks
           ↓
     MLflow pyfunc
           ↓
    Common predict()
           ↓
    Inference System
```

**Mental Model:**

```text
sklearn flavor
    ↓
Native Scikit-learn interface

pyfunc flavor
    ↓
Common MLflow prediction interface
```

Important: Not every framework-specific operation is available through the generic pyfunc interface.

---

## 4. Model Signature

One of the most important Day 2 concepts.

A **model signature** describes the expected input and output schema of a model.

Our SRE Incident Predictor expects features such as:

```text
cpu
memory
error_rate
request_rate
latency
restart_count
```

The model produces a prediction:

```text
0 → No Incident
1 → Incident
```

Conceptually:

```text
                MODEL SIGNATURE

INPUT
─────────────────────────────
cpu              numeric
memory           numeric
error_rate       numeric
request_rate     numeric
latency          numeric
restart_count    numeric

              ↓ MODEL ↓

OUTPUT
─────────────────────────────
incident prediction
0 or 1
```

### Why is a signature important?

Imagine an inference client sends:

```json
{
  "cpu": 92,
  "memory": 84
}
```

But the model expects six features.

Or:

```json
{
  "cpu": "HIGH"
}
```

instead of a numerical CPU value.

A signature documents the expected input/output structure and enables schema validation in supported MLflow inference interfaces.

**Mental Model:**

```text
SIGNATURE
    ↓
Model Input/Output Contract
```

---

## 5. Inferring a Model Signature

MLflow provides:

```python
from mlflow.models import infer_signature
```

After training:

```python
predictions = model.predict(
    X_test
)
```

Infer the signature:

```python
signature = infer_signature(
    X_test,
    predictions
)
```

### What happens?

```text
X_test
   ↓
Input feature names and types

predictions
   ↓
Output structure and type

        ↓

infer_signature()

        ↓

Model Signature
```

Inspect it:

```python
print(signature)
```

For a DataFrame containing six numerical features, MLflow can infer their names and data types.

**Important:** A signature inferred from sample data reflects that sample's structure. We must ensure the example represents the actual model input contract.

---

## 6. Model Signature vs Data Validation

These are related but different concepts.

### Data Validation

Checks whether data satisfies our application or training requirements.

Examples:

```text
CPU must be between 0 and 100

Error rate must be between 0 and 1

Required columns must exist

Target labels must be valid
```

### Model Signature

Describes the input/output schema expected by the logged model.

Examples:

```text
cpu → numeric

memory → numeric

error_rate → numeric
```

A model signature does not automatically enforce every business rule.

For example, a numerical signature alone doesn't guarantee:

```text
0 <= cpu <= 100
```

**Mental Model:**

```text
Data Validation
      ↓
Is this data acceptable?

Model Signature
      ↓
Does this data match the
model's expected schema?
```

---

## 7. Input Example

A signature describes the expected structure.

An **input example** shows an actual valid input.

For our SRE model:

```text
cpu              92
memory           84
error_rate       0.07
request_rate     1200
latency          450
restart_count    3
```

Create an example:

```python
input_example = X_test.iloc[[0]]
```

### Why double brackets?

Remember Pandas:

```python
X_test.iloc[0]
```

returns a Series.

But:

```python
X_test.iloc[[0]]
```

returns a one-row DataFrame.

This preserves the tabular structure expected by our model.

### Signature vs Input Example

```text
SIGNATURE
    ↓
What structure and types are expected?


INPUT EXAMPLE
    ↓
What does a valid input look like?
```

**Mental Model:**

```text
Signature = Contract

Input Example = Sample Request
```

---

## 8. Logging a Model with Signature and Input Example

On Day 1:

```python
mlflow.sklearn.log_model(
    sk_model=model,
    name="incident_model"
)
```

On Day 2:

```python
mlflow.sklearn.log_model(
    sk_model=model,
    name="incident_model",
    signature=signature,
    input_example=input_example
)
```

Now the logged model contains more useful metadata.

Conceptually:

```text
incident_model
      │
      ├── Trained Model
      ├── Flavors
      ├── Signature
      ├── Input Example
      ├── Dependencies
      └── Metadata
```

This is much more useful for downstream consumers than an unexplained serialized model file.

---

## 9. Model Dependencies

A trained Scikit-learn model depends on its software environment.

For example:

```text
Python
NumPy
Scikit-learn
Pandas
Cloudpickle
```

Imagine training a model with one version of Scikit-learn and loading it later using an incompatible version.

The model may fail to load or behave unexpectedly.

MLflow can record inferred dependency information alongside the model.

Typical model package files may include:

```text
MLmodel
conda.yaml
python_env.yaml
requirements.txt
model.pkl
input_example.json
```

Exact files depend on the MLflow version, model flavor and logging configuration.

### Why dependencies matter

```text
Trained Model
      +
Framework
      +
Package Versions
      +
Runtime Environment
      ↓
Reliable Model Loading
```

Important:

> Recording dependencies improves reproducibility, but doesn't guarantee that every environment will behave identically.

---

## 10. Model URI

After logging a model, we need a way to reference it.

MLflow uses **Model URIs**.

Capture the result:

```python
model_info = mlflow.sklearn.log_model(
    sk_model=model,
    name="incident_model",
    signature=signature,
    input_example=input_example
)
```

Then:

```python
print(model_info.model_uri)
```

The URI identifies the logged model.

### Mental Model

```text
Training Run
      ↓
Logged Model
      ↓
Model URI
      ↓
Load Model
```

A model URI is a reference to the model, not the model object itself.

Examples of MLflow URI schemes include run-based references, logged-model references and registered-model references.

We will learn registered-model URIs in Day 3.

---

## 11. Loading a Model Using the Scikit-learn Flavor

After logging:

```python
model_info = mlflow.sklearn.log_model(
    sk_model=model,
    name="incident_model",
    signature=signature,
    input_example=input_example
)
```

Load it:

```python
loaded_model = mlflow.sklearn.load_model(
    model_info.model_uri
)
```

Predict:

```python
loaded_predictions = loaded_model.predict(
    X_test
)
```

Compare predictions:

```python
print(predictions)

print(loaded_predictions)
```

We expect the restored model to produce the same predictions for the same valid input in a compatible environment.

This is similar to:

```python
joblib.dump()
joblib.load()
```

but with MLflow's model packaging and metadata.

---

## 12. Loading the Same Model Using Pyfunc

We can also load the model through the generic MLflow interface.

```python
pyfunc_model = mlflow.pyfunc.load_model(
    model_info.model_uri
)
```

Then:

```python
pyfunc_predictions = pyfunc_model.predict(
    X_test
)
```

We have now loaded the same model using two interfaces.

```text
                   MLflow Model
                         │
              ┌──────────┴──────────┐
              ▼                     ▼
      sklearn.load_model      pyfunc.load_model
              │                     │
              ▼                     ▼
       Native Estimator        Generic Wrapper
              │                     │
              └──────────┬──────────┘
                         ▼
                      predict()
```

### Key Difference

| Scikit-learn Flavor | Pyfunc Flavor |
|---|---|
| Native Scikit-learn object | Generic MLflow model wrapper |
| Framework-specific methods available | Common prediction interface |
| Useful for Python ML development | Useful for standardized inference systems |

**Mental Model:**

```text
Same Model
    ↓
Different Supported Interfaces
```

---

## 13. Manual Logging

On Day 1, we manually logged everything.

```python
mlflow.log_params({
    "max_iter": 1000,
    "random_state": 42
})
```

```python
mlflow.log_metrics({
    "accuracy": accuracy,
    "precision": precision,
    "recall": recall,
    "f1": f1
})
```

```python
mlflow.sklearn.log_model(
    sk_model=model,
    name="incident_model"
)
```

This is **manual logging**.

### Advantages

- Full control over what gets logged.
- Custom business metrics.
- Custom evaluation artifacts.
- Meaningful tags.
- Explicit experiment documentation.

### Disadvantages

- More code.
- Easy to forget important parameters.
- Repetitive across many training scripts.

---

## 14. Autologging

MLflow provides automatic logging for supported ML frameworks.

For Scikit-learn:

```python
mlflow.sklearn.autolog()
```

Example:

```python
import mlflow
import mlflow.sklearn

from sklearn.linear_model import LogisticRegression

mlflow.set_experiment(
    "sre-incident-predictor"
)

mlflow.sklearn.autolog()

with mlflow.start_run():

    model = LogisticRegression(
        max_iter=1000,
        random_state=42
    )

    model.fit(
        X_train,
        y_train
    )
```

MLflow can automatically capture information such as:

```text
Estimator Parameters
Training Metrics
Model Artifacts
Model Metadata
Signatures
Dataset Information
```

The exact information depends on the framework, MLflow version and autologging configuration.

### Mental Model

```text
Manual Logging
      ↓
Engineer explicitly logs information


Autologging
      ↓
MLflow automatically captures
supported training information
```

---

## 15. Manual Logging vs Autologging

| Manual Logging | Autologging |
|---|---|
| Explicit control | Automatic capture |
| More code | Less code |
| Custom metrics | Standard framework metrics |
| Business context | Technical training metadata |
| Custom artifacts | Supported automatic artifacts |

### Should We Always Use Autologging?

No.

Autologging is useful, but it doesn't automatically understand every business or operational requirement.

For example, our incident predictor cares about:

```text
False Positives
      ↓
False Alerts

False Negatives
      ↓
Missed Incidents
```

Scikit-learn doesn't inherently know the operational cost of missing an incident.

Therefore, manual logging is still valuable.

---

## 16. Combining Autologging and Manual Logging

This is an important production pattern.

```text
Autologging
     ↓
Standard Framework Metadata

          +

Manual Logging
     ↓
Custom Operational Metrics

          ↓

Complete Experiment Context
```

Example:

```python
from sklearn.metrics import confusion_matrix

tn, fp, fn, tp = confusion_matrix(
    y_test,
    predictions,
    labels=[0, 1]
).ravel()
```

Log operational metrics:

```python
mlflow.log_metrics({
    "false_positives": int(fp),
    "false_negatives": int(fn)
})
```

Now MLflow can show:

```text
accuracy = 0.83
recall = 0.50
false_positives = 0
false_negatives = 1
```

This is more meaningful for our SRE use case.

### Important

When combining autologging and manual logging, avoid accidentally logging the same model twice unless that is intentional.

---

## 17. Model Artifact vs MLflow Model

An artifact can be any file produced by a run.

Examples:

```text
evaluation.json
confusion_matrix.png
feature_report.csv
```

An MLflow Model is a specialized model package.

```text
MLflow Model
    │
    ├── Model Files
    ├── Flavors
    ├── Signature
    ├── Dependencies
    └── Metadata
```

**Important Distinction:**

```text
Artifact
    ↓
General Output File

MLflow Model
    ↓
Structured Model Package
```

An MLflow Model uses artifacts for its stored files, but includes model-specific structure and metadata.

---

## 18. Training-Serving Consistency

Remember our Scikit-learn preprocessing lessons.

Our training workflow might use:

```text
Raw Data
    ↓
SimpleImputer
    ↓
StandardScaler
    ↓
OneHotEncoder
    ↓
LogisticRegression
```

If we log only the Logistic Regression estimator, production still needs the exact preprocessing logic.

A better approach is to log the **entire fitted Scikit-learn Pipeline**.

Example:

```python
mlflow.sklearn.log_model(
    sk_model=pipeline,
    name="incident_pipeline",
    signature=signature,
    input_example=input_example
)
```

Here, `pipeline` should be the fitted preprocessing-plus-model pipeline.

The signature and input example should describe the raw features accepted by that pipeline.

### Why this matters

```text
TRAINING
    ↓
Preprocessing + Model
    ↓
Logged Pipeline

INFERENCE
    ↓
Load Pipeline
    ↓
Raw Features
    ↓
Same Preprocessing
    ↓
Prediction
```

**Mental Model:**

> Package preprocessing and the trained model together whenever the model depends on that preprocessing.

This helps reduce training-serving skew.

---

## 19. Common Mistakes

### Mistake 1 — Signature vs Input Example

Incorrect:

```text
Signature = Example Input
```

Correct:

```text
Signature
    ↓
Input/Output Schema

Input Example
    ↓
Representative Valid Input
```

### Mistake 2 — Thinking Signature Replaces Data Validation

A signature does not automatically validate every business rule.

```text
cpu = 500
```

may be numerical but still invalid for CPU percentage.

### Mistake 3 — Assuming Pyfunc and Native Flavors Are Identical

Both may support prediction, but the native flavor can expose additional framework-specific methods.

### Mistake 4 — Assuming Dependencies Guarantee Reproducibility

Dependencies help, but complete reproducibility may also require:

```text
Dataset Version
Code Version
Training Configuration
Random Seeds
Environment Details
```

### Mistake 5 — Logging Only the Estimator

If training uses preprocessing, logging only the estimator may leave production without the required transformations.

### Mistake 6 — Assuming Autologging Captures Everything

Custom business metrics and operational meaning often require manual logging.

---

## 20. MLOps / SRE Connection

Day 1:

```text
Train
  ↓
Evaluate
  ↓
Track Run
  ↓
Compare Results
```

Day 2:

```text
Train
  ↓
Evaluate
  ↓
Log Model
  │
  ├── Signature
  ├── Input Example
  ├── Flavors
  ├── Dependencies
  └── Metadata
        ↓
     Model URI
        ↓
     Load Model
        ↓
     Predict
```

This creates a more portable and understandable model artifact.

For an ML platform engineer, this matters because different teams may train models using different frameworks.

A common packaging and inference interface makes platform integration easier.

---

## 21. Quick Revision Cheat Sheet

```python
import mlflow
import mlflow.sklearn

from mlflow.models import infer_signature

# Assume model is already fitted
predictions = model.predict(X_test)

# Infer signature
signature = infer_signature(
    X_test,
    predictions
)

# Create input example
input_example = X_test.iloc[[0]]

# Log model inside an active run
model_info = mlflow.sklearn.log_model(
    sk_model=model,
    name="incident_model",
    signature=signature,
    input_example=input_example
)

# Inspect model URI
print(model_info.model_uri)

# Load native sklearn model
sklearn_model = mlflow.sklearn.load_model(
    model_info.model_uri
)

# Predict
sklearn_predictions = sklearn_model.predict(
    X_test
)

# Load generic pyfunc model
pyfunc_model = mlflow.pyfunc.load_model(
    model_info.model_uri
)

# Predict
pyfunc_predictions = pyfunc_model.predict(
    X_test
)

# Enable autologging before fitting
mlflow.sklearn.autolog()
```

**Note:** This cheat sheet demonstrates the individual APIs. In a complete training script, logging should happen within the intended active run, and autologging should be configured before training.

---

## 22. Interview Revision Questions

1. What is an MLflow Model?
2. How is an MLflow Model different from a `.joblib` file?
3. What is a model flavor?
4. What is the difference between Scikit-learn and pyfunc flavors?
5. What is a model signature?
6. Why do we need an input example?
7. What is the difference between a signature and data validation?
8. Why are model dependencies important?
9. What is a Model URI?
10. How do we load a logged model?
11. What is autologging?
12. When should we use manual logging instead of autologging?
13. Why should preprocessing and the trained estimator be packaged together?
14. Does logging a model automatically deploy it?

---

## 23. Day 2 Definition of Done

- [x] Understood MLflow Model packaging.
- [x] Understood model flavors.
- [x] Explored Scikit-learn and pyfunc interfaces.
- [x] Created a model signature.
- [x] Created an input example.
- [x] Logged a model with signature and input example.
- [x] Understood dependency packaging.
- [x] Inspected a Model URI.
- [x] Loaded a logged model.
- [x] Performed predictions using a loaded model.
- [x] Explored autologging.
- [x] Compared manual logging and autologging.
- [x] Understood custom operational metric logging.

## MLflow Day 2: COMPLETE ✅

### The One Sentence to Remember

**MLflow Tracking records how a model was trained, while an MLflow Model packages the trained model with the interfaces, schema, dependencies and metadata needed to understand and use it consistently.**

---

### Next: MLflow Day 3

Model Registry, Registered Models, Model Versions, Aliases, Candidate/Champion, Promotion, Rollback and Model Lineage.
