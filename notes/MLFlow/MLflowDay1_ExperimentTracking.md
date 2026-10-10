# MLflow Day 1 — Experiment Tracking

## Objective

Understand why MLflow is needed, how experiment tracking works, and how to track and compare machine learning training runs.

The goal is to move from manually training and saving models to maintaining a **traceable and organized ML experimentation workflow**.

---

## 1. Why Do We Need MLflow?

Before MLflow, our SRE Incident Predictor followed this workflow:

```text
Data
  ↓
Validation
  ↓
Feature Preparation
  ↓
Training
  ↓
Evaluation
  ↓
Print Metrics
  ↓
Save model.joblib
```

This works when training a model once.

But imagine training the same model 50 times with different parameters.

We might end up with:

```text
models/
├── model.joblib
├── model_v2.joblib
├── model_final.joblib
└── model_final_latest.joblib
```

Problems:

- Which parameters were used?
- Which dataset produced the model?
- What were its accuracy, precision, recall and F1?
- Which training run produced which artifact?
- Which model performed better?
- Can we reproduce the training experiment?

**MLflow helps us organize and track these experiments.**

Important:

> MLflow provides tracking infrastructure. Complete reproducibility also requires capturing code versions, dataset versions, dependencies and other relevant information.

---

## 2. What Is MLflow?

MLflow is an open-source platform for managing machine learning and AI workflows.

Some of its important capabilities include:

```text
MLflow
   │
   ├── Experiment Tracking
   │
   ├── Model Packaging
   │
   ├── Model Registry
   │
   └── Model Evaluation / Deployment Integration
```

On Day 1, we focus only on:

**MLflow Experiment Tracking**

It helps us record:

```text
Training Run
     │
     ├── Parameters
     ├── Metrics
     ├── Tags
     ├── Artifacts
     └── Logged Models
```

---

## 3. MLflow Experiment vs Run

This is the most important Day 1 concept.

```text
MLflow
   │
   └── Experiment
          │
          ├── Run 1
          │    ├── Parameters
          │    ├── Metrics
          │    └── Artifacts
          │
          ├── Run 2
          │    ├── Parameters
          │    ├── Metrics
          │    └── Artifacts
          │
          └── Run 3
               ├── Parameters
               ├── Metrics
               └── Artifacts
```

### Experiment

A logical collection of related training runs.

Example:

```python
mlflow.set_experiment(
    "sre-incident-predictor"
)
```

This selects or creates an experiment.

### Run

A tracked execution of our training workflow.

```python
with mlflow.start_run():
    ...
```

Each run has a unique Run ID.

**Mental Model:**

```text
Experiment
    ↓
Collection of related attempts

Run
    ↓
One tracked attempt
```

---

## 4. Parameters vs Metrics vs Tags vs Artifacts

These four concepts must be clear.

| Component | Meaning | Example |
|---|---|---|
| Parameters | What did I configure? | `max_iter=1000` |
| Metrics | What did I measure? | `recall=0.75` |
| Tags | What additional context exists? | `purpose=baseline` |
| Artifacts | What files were produced? | `evaluation.json` |

## Parameters

Parameters represent configuration values used during training.

Example:

```python
mlflow.log_param(
    "max_iter",
    1000
)
```

Multiple parameters:

```python
mlflow.log_params({
    "model_type": "LogisticRegression",
    "max_iter": 1000,
    "random_state": 42
})
```

**Mental Model:**

```text
PARAMETERS
    ↓
What did I configure?
```

## Metrics

Metrics represent numerical measurements.

Example:

```python
mlflow.log_metric(
    "accuracy",
    0.83
)
```

Multiple metrics:

```python
mlflow.log_metrics({
    "accuracy": 0.83,
    "precision": 1.0,
    "recall": 0.50,
    "f1": 0.67
})
```

**Mental Model:**

```text
METRICS
    ↓
What happened?
```

## Tags

Tags provide descriptive metadata.

```python
mlflow.set_tag(
    "purpose",
    "baseline-classifier"
)
```

Examples:

```text
project     = sre-incident-predictor
purpose     = baseline-classifier
team        = ml-platform
```

**Mental Model:**

```text
TAGS
    ↓
What context should I attach?
```

## Artifacts

Artifacts are files generated or retained during a run.

Examples:

```text
evaluation.json
confusion_matrix.png
classification_report.html
feature_report.csv
```

Log an artifact:

```python
mlflow.log_artifact(
    "evaluation.json"
)
```

**Mental Model:**

```text
ARTIFACTS
    ↓
What files did this run produce?
```

---

## 5. Installing MLflow

Install MLflow inside the Python virtual environment:

```bash
pip install mlflow
```

Verify installation:

```bash
mlflow --version
```

Import:

```python
import mlflow
```

---

## 6. Creating Our First MLflow Experiment

We used our existing SRE Incident Predictor.

```python
import mlflow

mlflow.set_experiment(
    "sre-incident-predictor"
)
```

Now start a run:

```python
with mlflow.start_run():
    print("Training started")
```

The `with` statement manages the run context.

Conceptually:

```text
start_run()
     ↓
Active Run
     ↓
Log Parameters
     ↓
Log Metrics
     ↓
Log Artifacts
     ↓
Run Ends
```

The run is normally closed automatically when the context exits.

---

## 7. Tracking Our Scikit-learn Model

Our original model:

```python
from sklearn.linear_model import LogisticRegression

model = LogisticRegression(
    max_iter=1000,
    random_state=42
)

model.fit(
    X_train,
    y_train
)
```

After training:

```python
predictions = model.predict(
    X_test
)
```

Now MLflow helps us track this workflow.

Example:

```python
import mlflow
import mlflow.sklearn

from sklearn.linear_model import LogisticRegression

from sklearn.metrics import (
    accuracy_score,
    precision_score,
    recall_score,
    f1_score
)

# Assumes X_train, X_test, y_train, y_test
# have already been prepared.

mlflow.set_experiment(
    "sre-incident-predictor"
)

with mlflow.start_run():

    model = LogisticRegression(
        max_iter=1000,
        random_state=42
    )

    mlflow.log_params({
        "model_type": "LogisticRegression",
        "max_iter": 1000,
        "random_state": 42
    })

    model.fit(X_train, y_train)

    predictions = model.predict(X_test)

    accuracy = accuracy_score(
        y_test, predictions
    )

    precision = precision_score(
        y_test, predictions,
        zero_division=0
    )

    recall = recall_score(
        y_test, predictions,
        zero_division=0
    )

    f1 = f1_score(
        y_test, predictions,
        zero_division=0
    )

    mlflow.log_metrics({
        "accuracy": accuracy,
        "precision": precision,
        "recall": recall,
        "f1": f1
    })

    mlflow.set_tag(
        "purpose",
        "baseline-classifier"
    )

    mlflow.sklearn.log_model(
        sk_model=model,
        name="incident_model"
    )
```

**Important:** MLflow does not replace Scikit-learn.

```text
Scikit-learn
     ↓
Train and Evaluate

MLflow
     ↓
Track and Manage Results
```

---

## 8. Logging an Evaluation Artifact

Create an evaluation report:

```python
import json

evaluation = {
    "accuracy": float(accuracy),
    "precision": float(precision),
    "recall": float(recall),
    "f1": float(f1)
}

with open("evaluation.json", "w") as file:
    json.dump(
        evaluation,
        file,
        indent=2
    )
```

Log it inside the active MLflow run:

```python
mlflow.log_artifact(
    "evaluation.json"
)
```

Now our run contains:

```text
Run
 ├── Parameters
 ├── Metrics
 ├── Tags
 ├── evaluation.json
 └── incident_model
```

---

## 9. MLflow UI

Start the local MLflow server:

```bash
mlflow server \
  --host 127.0.0.1 \
  --port 5000
```

Open:

```text
http://127.0.0.1:5000
```

To send training runs to this server:

```python
mlflow.set_tracking_uri(
    "http://127.0.0.1:5000"
)
```

Configure the tracking URI before selecting the experiment.

In the UI:

```text
MLflow UI
    ↓
Experiments
    ↓
sre-incident-predictor
    ↓
Runs
    ↓
Parameters / Metrics / Artifacts
```

We can inspect individual runs and compare multiple runs.

---

## 10. Comparing Multiple Runs

Suppose we have:

| Metric | Run A | Run B |
|---|---:|---:|
| Accuracy | 95% | 90% |
| Precision | 98% | 82% |
| Recall | 50% | 95% |
| F1 | 66% | 88% |

Which run is better?

There is no universal answer.

For our incident predictor, missing incidents may be more expensive than generating additional alerts.

Run B may therefore deserve consideration because it catches more actual incidents.

However, we should also inspect false positives, dataset quality and evaluation reliability.

**Important Lesson:**

> The highest accuracy does not automatically mean the best production model.

MLflow makes these comparisons easier by storing metrics from different runs together.

---

## 11. Parallel Coordinates Plot

During our Day 1 exercise, we compared two runs in the MLflow UI using the Parallel Coordinates Plot.

Conceptually:

```text
Run A ── Parameters ── Accuracy ── Recall ── F1

Run B ── Parameters ── Accuracy ── Recall ── F1
```

This visualization helps identify relationships between model configurations and measured outcomes.

It becomes particularly useful when comparing many training runs.

---

## 12. Run ID vs Run Name

Every tracked MLflow run has a unique Run ID.

```text
Run ID
   ↓
Unique identifier for a run
```

MLflow can also generate readable run names.

```text
Run Name
   ↓
Human-friendly label
```

The Run ID is important for referencing a specific run and tracing its outputs.

---

## 13. MLflow Tracking vs Model Registry

These are different responsibilities.

### Experiment Tracking

```text
What did we try?
     ↓
Experiment
     ↓
Runs
     ↓
Parameters / Metrics / Artifacts
```

### Model Registry

```text
Which models are managed?
     ↓
Registered Model
     ↓
Versions
     ↓
Aliases
     ↓
Champion / Candidate
```

**Tracking records experimentation. Registry manages selected models and their lifecycle metadata.**

Model Registry is covered in Day 3.

---

## 14. Common Mistakes

### Mistake 1 — Experiment vs Run

Incorrect:

```text
Experiment = one training attempt
```

Correct:

```text
Experiment
    ├── Run 1
    ├── Run 2
    └── Run 3
```

### Mistake 2 — Parameters vs Metrics

```text
max_iter = 1000
```

Parameter.

```text
recall = 0.85
```

Metric.

### Mistake 3 — Metric vs Artifact

```text
accuracy = 0.91
```

Metric.

```text
evaluation.json
```

Artifact.

### Mistake 4 — Logged Model Means Deployed Model

Logging a model does not automatically deploy it.

```text
Train
  ↓
Log Model
  ↓
Tracked Model
```

Deployment is a separate process.

### Mistake 5 — Assuming MLflow Guarantees Reproducibility

MLflow tracks information that is logged or automatically captured.

For stronger reproducibility, we also need:

```text
Code Version
Dataset Version
Dependencies
Training Configuration
Random Seeds
Environment Information
```

We will extend this with DVC and other MLOps practices.

---

## 15. MLOps / SRE Connection

Before MLflow:

```text
Train
  ↓
Evaluate
  ↓
Print Results
  ↓
Save Model
```

After introducing MLflow:

```text
Train
  ↓
Evaluate
  ↓
MLflow Run
  │
  ├── Parameters
  ├── Metrics
  ├── Tags
  ├── Artifacts
  └── Logged Model
         ↓
      MLflow UI
         ↓
    Compare Runs
         ↓
  Investigate Results
```

This creates the foundation for experiment traceability and model lifecycle management.

---

## 16. Quick Revision Cheat Sheet

```python
import mlflow
import mlflow.sklearn

# Tracking server
mlflow.set_tracking_uri(
    "http://127.0.0.1:5000"
)

# Experiment
mlflow.set_experiment(
    "sre-incident-predictor"
)

# Run
with mlflow.start_run():

    # Parameters
    mlflow.log_param(
        "max_iter",
        1000
    )

    # Metrics
    mlflow.log_metric(
        "accuracy",
        0.91
    )

    mlflow.log_metric(
        "recall",
        0.85
    )

    # Tags
    mlflow.set_tag(
        "purpose",
        "baseline"
    )

    # Artifact
    mlflow.log_artifact(
        "evaluation.json"
    )

    # Trained model
    mlflow.sklearn.log_model(
        sk_model=model,
        name="incident_model"
    )
```

---

## 17. Interview Revision Questions

1. Why do we need MLflow Experiment Tracking?
2. What is the difference between an Experiment and a Run?
3. What are parameters, metrics, tags and artifacts?
4. What is a Run ID?
5. How does MLflow help compare training runs?
6. Does MLflow automatically select the best model?
7. Does logging a model mean it is deployed?
8. What is the difference between Experiment Tracking and Model Registry?
9. What information would you track to reproduce a model?
10. Why is MLflow useful in a team with many ML engineers?

---

## 18. Day 1 Definition of Done

- [x] Understood why experiment tracking is needed.
- [x] Installed MLflow.
- [x] Created an experiment.
- [x] Created multiple runs.
- [x] Logged parameters.
- [x] Logged metrics.
- [x] Added tags.
- [x] Logged artifacts.
- [x] Logged a Scikit-learn model.
- [x] Opened the MLflow UI.
- [x] Compared two runs.
- [x] Understood Run IDs and run names.

## MLflow Day 1: COMPLETE ✅

### The One Sentence to Remember

**MLflow Experiment Tracking helps us record, organize, compare and trace machine learning training runs instead of relying on scattered files and terminal output.**

---

### Next: MLflow Day 2

Model Logging, Flavors, Signatures, Input Examples, Dependencies, Model URIs, Model Loading and Autologging.
