# MLflow Day 3 — Model Registry, Versions, Aliases & Model Lifecycle

## Objective

Understand how MLflow Model Registry helps manage trained models after experimentation.

The goal is to move from tracking multiple trained models to managing **registered model versions, aliases, promotion, rollback, and model lineage**.

Our learning progression:

```text
Day 1
Experiment Tracking
      ↓
Day 2
MLflow Model Packaging
      ↓
Day 3
Model Registry
      ↓
Model Lifecycle Management
```

**Important:** Registering or promoting a model does not automatically deploy it to production.

---

## 1. Why Do We Need a Model Registry?

On Day 1, we created experiments and tracked multiple runs.

```text
Experiment: sre-incident-predictor
      │
      ├── Run 1
      │    ├── Parameters
      │    ├── Metrics
      │    └── Model
      │
      ├── Run 2
      │    ├── Parameters
      │    ├── Metrics
      │    └── Model
      │
      └── Run 3
           ├── Parameters
           ├── Metrics
           └── Model
```

Imagine training 100 models.

Some models perform poorly. Some perform well. A few pass additional validation.

We need to answer:

- Which models are approved for further evaluation?
- Which model versions are available?
- Which model is currently preferred?
- How do we promote a new model?
- How do we roll back to an earlier version?
- How do we trace a registered model to its training run?

This is where **MLflow Model Registry** becomes useful.

### Mental Model

```text
Experiment Tracking
       ↓
What did we try?

Model Registry
       ↓
Which models are we managing?
```

Not every experiment run needs to become a registered model.

---

## 2. Experiment Tracking vs Model Registry

These are different responsibilities.

| Experiment Tracking | Model Registry |
|---|---|
| Tracks training attempts | Manages registered models |
| Organizes experiments and runs | Organizes model versions |
| Stores parameters and metrics | Maintains model lifecycle metadata |
| Records artifacts | References logged model artifacts |
| Helps compare experiments | Helps manage selected models |
| Answers "What did we try?" | Answers "Which model version should we use?" |

### Architecture

```text
                   MLflow
                     │
          ┌──────────┴──────────┐
          ▼                     ▼
   Experiment Tracking      Model Registry
          │                     │
          ▼                     ▼
     Experiments          Registered Models
          │                     │
          ▼                     ▼
        Runs                Versions
          │                     │
          ▼                     ▼
   Params / Metrics           Aliases
          │                     │
          └──────────┬──────────┘
                     ▼
                Model Lineage
```

**Important:** The Registry does not replace experiment tracking.

It builds on the models produced by tracked experiments.

---

## 3. Registered Model vs Model Version

These two concepts must be clear.

## Registered Model

A registered model is a logical identity under which model versions are managed.

Example:

```text
sre-incident-predictor
```

Think of it as the name of the ML capability we're managing.

## Model Version

A specific registered model artifact under that logical identity.

Example:

```text
Registered Model:
sre-incident-predictor
       │
       ├── Version 1
       ├── Version 2
       ├── Version 3
       └── Version 4
```

Each model version can reference a particular logged model.

### Mental Model

```text
Registered Model
       ↓
Logical Model Identity

Model Version
       ↓
Specific Registered Model Artifact
```

Similar to software releases:

```text
payment-api
   ├── v1
   ├── v2
   └── v3
```

But here we're managing ML model versions.

---

## 4. Registering Our First Model

We already logged a model on Day 2.

Example:

```python
model_info = mlflow.sklearn.log_model(
    sk_model=model,
    name="incident_model",
    signature=signature,
    input_example=input_example
)
```

The returned model information contains a Model URI.

```python
print(model_info.model_uri)
```

We can register this logged model.

```python
import mlflow

registered_model = mlflow.register_model(
    model_uri=model_info.model_uri,
    name="sre-incident-predictor"
)
```

Inspect:

```python
print(registered_model.name)
print(registered_model.version)
```

For a newly created registered model, the first version is normally:

```text
sre-incident-predictor
       │
       └── Version 1
```

### What happened?

```text
Training Run
      ↓
Logged Model
      ↓
Model URI
      ↓
Register Model
      ↓
Registered Model Version 1
```

**Important:** Registration creates a model version entry. It does not automatically deploy the model.

---

## 5. Registering Another Model Version

Suppose another training run produces a different model.

```text
Run A
   ↓
Model A

Run B
   ↓
Model B
```

Register both under the same name:

```python
mlflow.register_model(
    model_uri=model_a_uri,
    name="sre-incident-predictor"
)

mlflow.register_model(
    model_uri=model_b_uri,
    name="sre-incident-predictor"
)
```

Now:

```text
sre-incident-predictor
       │
       ├── Version 1 → Model A
       │
       └── Version 2 → Model B
```

### Important Distinction

We did not create two different registered models.

We created **two versions under the same registered model identity**.

Version numbers are assigned by the Registry. Don't assume a particular version number if versions already exist.

---

## 6. Why Model Versions Alone Are Not Enough

Imagine our inference application loads:

```text
Version 1
```

After a new release:

```text
Version 2
```

Then:

```text
Version 5
```

If the application directly references version numbers, we may need to update deployment configuration whenever the preferred model changes.

Instead, we can introduce meaningful references.

This brings us to **Model Aliases**.

---

## 7. Model Aliases

A model alias is a named reference to a registered model version.

Example:

```text
Registered Model:
sre-incident-predictor
       │
       ├── Version 1 ← champion
       │
       └── Version 2 ← candidate
```

Instead of asking:

```text
Give me Version 1
```

we can ask:

```text
Give me the champion model
```

### Mental Model

```text
Model Version
      ↓
Specific Artifact

Model Alias
      ↓
Meaningful Pointer to a Version
```

Aliases are mutable.

This means the same alias can be reassigned to a different version.

---

## 8. Candidate vs Champion

These are model lifecycle concepts represented using aliases.

## Candidate

A model being considered for promotion.

```text
Version 2
    ↑
 candidate
```

## Champion

The model currently selected as the preferred model for a particular workflow.

```text
Version 1
    ↑
 champion
```

### Architecture

```text
Registered Model
      │
      ├── Version 1
      │      ↑
      │   champion
      │
      └── Version 2
             ↑
          candidate
```

**Important:** `candidate` and `champion` are naming conventions, not automatic quality states.

MLflow does not automatically determine which model deserves these aliases.

Also, assigning `champion` does not guarantee that the model is deployed or currently serving traffic.

---

## 9. Setting Model Aliases

MLflow provides the `MlflowClient` API.

```python
from mlflow import MlflowClient

client = MlflowClient()
```

Set Version 1 as champion:

```python
client.set_registered_model_alias(
    name="sre-incident-predictor",
    alias="champion",
    version="1"
)
```

Set Version 2 as candidate:

```python
client.set_registered_model_alias(
    name="sre-incident-predictor",
    alias="candidate",
    version="2"
)
```

Result:

```text
sre-incident-predictor
       │
       ├── Version 1 ← champion
       │
       └── Version 2 ← candidate
```

### Verify an Alias

```python
champion = client.get_model_version_by_alias(
    name="sre-incident-predictor",
    alias="champion"
)

print(champion.version)
```

This returns the version currently associated with the alias.

---

## 10. Loading a Model Using a Registry URI

On Day 2, we loaded models using their logged Model URI.

Now we can load models through the Registry.

### Load a Specific Version

```python
model = mlflow.pyfunc.load_model(
    "models:/sre-incident-predictor/1"
)
```

This loads Version 1.

### Load Using an Alias

```python
model = mlflow.pyfunc.load_model(
    "models:/sre-incident-predictor@champion"
)
```

Then:

```python
predictions = model.predict(
    X_test
)
```

### Mental Model

```text
Inference Application
         ↓
models:/sre-incident-predictor@champion
         ↓
Model Registry
         ↓
Resolve champion Alias
         ↓
Model Version
         ↓
Model Artifact
         ↓
Load Model
         ↓
Prediction
```

This separates the application's model reference from a hardcoded version number.

---

## 11. Model Promotion

Suppose:

```text
Version 1 → champion
Version 2 → candidate
```

After evaluation, Version 2 passes our quality checks.

We want to promote Version 2.

Before:

```text
champion  → Version 1
candidate → Version 2
```

After:

```text
champion  → Version 2
candidate → Version 2
```

An alias can be reassigned.

```python
client.set_registered_model_alias(
    name="sre-incident-predictor",
    alias="champion",
    version="2"
)
```

Now:

```text
Version 1

Version 2 ← champion
          ← candidate
```

The `candidate` alias can be removed or reassigned separately when appropriate.

### Mental Model

```text
Candidate Model
       ↓
Evaluation
       ↓
Quality Gates
       ↓
Approval
       ↓
Update Champion Alias
```

### Important Production Consideration

Updating the alias changes the Registry reference.

It does not automatically reload models already running in memory.

A deployment or model-reload mechanism must apply the change to serving instances.

---

## 12. Model Rollback

Suppose Version 2 was promoted.

```text
champion → Version 2
```

After deployment, we observe:

```text
Prediction Quality ↓
False Alerts ↑
Latency ↑
```

We may need to restore the previous approved model.

Rollback:

```python
client.set_registered_model_alias(
    name="sre-incident-predictor",
    alias="champion",
    version="1"
)
```

Now:

```text
champion → Version 1
```

### Mental Model

```text
Version 1
    ↓
Known-Good Model

Version 2
    ↓
Problematic Release

Rollback
    ↓
champion → Version 1
```

### Important

Reassigning the Registry alias alone does not guarantee that production has rolled back.

If a serving application already loaded Version 2, it may continue using that model until the deployment is updated or the model is reloaded.

A complete rollback requires verifying which model version is actually serving.

---

## 13. Promotion vs Deployment vs Rollback

These are related but different operations.

| Operation | Meaning |
|---|---|
| Registration | Add a model version to the Registry |
| Promotion | Select a model version for a lifecycle role |
| Deployment | Make a model available for inference |
| Rollback | Restore a previously approved model or deployment |

### Architecture

```text
Training
   ↓
Evaluation
   ↓
Register Candidate
   ↓
Quality Gates
   ↓
Promote Champion
   ↓
Deployment Pipeline
   ↓
Production Inference
   ↓
Monitoring
   ↓
Rollback if Required
```

**Important Lesson:**

> Registry lifecycle management and actual production deployment are separate responsibilities.

---

## 14. Model Quality Gates

MLflow Registry does not automatically determine whether a model is good enough for production.

We must define evaluation criteria.

For our SRE Incident Predictor, imagine:

```text
Candidate Model
      ↓
Accuracy Check
      ↓
Recall Check
      ↓
False Negative Check
      ↓
Latency Check
      ↓
Approval
      ↓
Champion
```

Example criteria:

```text
Recall >= 0.90

False Negative Rate <= 0.10

Inference Latency within SLO
```

These are illustrative requirements, not universal thresholds.

### Why is this important?

Suppose:

| Metric | Version 1 | Version 2 |
|---|---:|---:|
| Accuracy | 95% | 90% |
| Precision | 98% | 82% |
| Recall | 50% | 95% |
| F1 | 66% | 88% |

Version 1 has higher accuracy.

But Version 2 catches more actual incidents.

For a high-severity incident predictor, Version 2 might be more suitable, depending on the cost of false alerts and missed incidents.

**The model with the highest accuracy does not automatically become champion.**

---

## 15. Model Lineage

Lineage means understanding where a model came from.

Suppose production uses:

```text
sre-incident-predictor@champion
```

We should be able to trace:

```text
Champion Alias
      ↓
Registered Model Version
      ↓
Logged Model
      ↓
Training Run
      ↓
Parameters
Metrics
Artifacts
```

With additional tracking, we can extend this to:

```text
Dataset Version
      ↓
Git Commit
      ↓
Training Configuration
      ↓
MLflow Run
      ↓
Logged Model
      ↓
Registered Version
      ↓
Champion
      ↓
Deployment
```

### Why Lineage Matters

Imagine a model starts producing incorrect predictions.

An engineer asks:

- Which training run produced it?
- What data was used?
- Which code version trained it?
- What were the evaluation metrics?
- Who approved it?
- When was it promoted?

Lineage helps us investigate these questions.

**Important:** Complete dataset and code lineage must be captured explicitly or through suitable integrations. Registration alone does not guarantee it.

---

## 16. Model Registry vs Artifact Store

Do not confuse these.

### Model Registry

Manages metadata such as:

```text
Registered Model
Model Versions
Aliases
Descriptions
Tags
Model References
```

### Artifact Store

Stores actual model files.

```text
model.pkl
MLmodel
requirements.txt
evaluation.json
```

### Architecture

```text
                 Model Registry
                       │
                       ▼
                Registered Model
                       │
                       ▼
                   Version 2
                       │
                       ▼
                  Artifact URI
                       │
                       ▼
                 Artifact Store
                       │
                       ▼
                 Actual Model Files
```

**Mental Model:**

```text
Registry
    ↓
Which model and version?

Artifact Store
    ↓
Where are the actual files?
```

This distinction becomes particularly important in Day 4 when we study PostgreSQL and object storage.

---

## 17. End-to-End Registry Lifecycle

Our complete Day 3 workflow:

```text
                     DATA
                       ↓
                   VALIDATE
                       ↓
                  PREPROCESS
                       ↓
                    TRAIN
                       ↓
                   EVALUATE
                       ↓
                  MLFLOW RUN
                       │
             ┌─────────┼─────────┐
             ▼         ▼         ▼
          Params     Metrics   Artifacts
                       │
                       ▼
                  Logged Model
                       │
                       ▼
                  Model Registry
                       │
                       ▼
                 Registered Model
                       │
                ┌──────┴──────┐
                ▼             ▼
             Version 1     Version 2
                │             │
                ▼             ▼
             champion      candidate
                              │
                              ▼
                         Quality Gates
                              │
                              ▼
                           Promote
                              │
                              ▼
                         champion → V2
                              │
                              ▼
                          Deployment
                              │
                              ▼
                           Inference
                              │
                              ▼
                           Monitor
                              │
                              ▼
                       Rollback if Needed
```

---

## 18. Common Mistakes

### Mistake 1 — Experiment vs Registered Model

Incorrect:

```text
Experiment = Registered Model
```

Correct:

```text
Experiment
    ↓
Collection of Training Runs

Registered Model
    ↓
Logical Identity for Model Versions
```

### Mistake 2 — Run vs Model Version

A run represents a tracked execution.

A model version represents a registered model artifact.

They are not the same thing.

### Mistake 3 — Champion Means Automatically Deployed

Incorrect:

```text
champion alias assigned
        ↓
Automatically serving production
```

Correct:

```text
champion alias assigned
        ↓
Deployment / Reload Required
        ↓
Production Serving
```

### Mistake 4 — Highest Accuracy Always Wins

Model selection depends on operational requirements.

For incident prediction, recall and false negatives may be especially important.

### Mistake 5 — Deleting Older Versions During Promotion

Older approved versions can be valuable for rollback, audit and comparison.

Do not delete them merely because a new version was promoted.

### Mistake 6 — Assuming Registry Guarantees Lineage

The Registry provides useful references to model versions and their sources.

Complete lineage still requires capturing relevant dataset, code and environment information.

---

## 19. MLOps / SRE Connection

Model Registry introduces familiar software release-management concepts into ML.

```text
Software Engineering        MLOps

Application Release    →    Model Version

Release Candidate      →    Candidate Model

Approved Release       →    Champion Model

Deployment             →    Model Serving

Rollback               →    Previous Approved Model
```

For an MLOps/SRE engineer, the important responsibilities include:

- Model version traceability.
- Controlled promotion.
- Rollback readiness.
- Access control.
- Deployment verification.
- Model artifact availability.
- Operational monitoring.

### Production Lesson

> A model is not production-ready merely because it has been registered. It must pass validation, be deployed safely, and be monitored after deployment.

---

## 20. Quick Revision Cheat Sheet

```python
import mlflow

from mlflow import MlflowClient

# MLflow client
client = MlflowClient()

# Register a previously logged model
registered = mlflow.register_model(
    model_uri=model_info.model_uri,
    name="sre-incident-predictor"
)

# Inspect assigned version
print(registered.version)

# Set champion alias
client.set_registered_model_alias(
    name="sre-incident-predictor",
    alias="champion",
    version="1"
)

# Set candidate alias
client.set_registered_model_alias(
    name="sre-incident-predictor",
    alias="candidate",
    version="2"
)

# Resolve champion
champion = client.get_model_version_by_alias(
    name="sre-incident-predictor",
    alias="champion"
)

print(champion.version)

# Load champion model
model = mlflow.pyfunc.load_model(
    "models:/sre-incident-predictor@champion"
)

# Predict
predictions = model.predict(X_test)

# Promote Version 2
client.set_registered_model_alias(
    name="sre-incident-predictor",
    alias="champion",
    version="2"
)

# Rollback to Version 1
client.set_registered_model_alias(
    name="sre-incident-predictor",
    alias="champion",
    version="1"
)
```

**Note:** This example assumes Versions 1 and 2 already exist. A real promotion or rollback process must also update or reload the serving deployment when necessary.

---

## 21. Interview Revision Questions

1. Why do we need MLflow Model Registry?
2. What is the difference between Experiment Tracking and Model Registry?
3. What is a Registered Model?
4. What is a Model Version?
5. What is a Model Alias?
6. What is the difference between `candidate` and `champion`?
7. How do we register a logged model?
8. How do we load a model using a Registry URI?
9. What happens when we change the `champion` alias?
10. Does changing an alias automatically deploy a new model?
11. How do we promote a candidate model?
12. How do we roll back to an earlier version?
13. What is model lineage?
14. How is Model Registry different from Artifact Storage?
15. Why shouldn't we automatically promote the model with the highest accuracy?

---

## 22. Day 3 Definition of Done

- [x] Understood why Model Registry is needed.
- [x] Distinguished Experiment Tracking from Model Registry.
- [x] Understood Registered Models.
- [x] Understood Model Versions.
- [x] Registered logged models.
- [x] Created multiple model versions.
- [x] Understood Model Aliases.
- [x] Used `candidate` and `champion`.
- [x] Loaded a model using a Registry URI.
- [x] Understood model promotion.
- [x] Understood model rollback.
- [x] Connected model versions to training lineage.
- [x] Understood Registry vs Artifact Store.

## MLflow Day 3: COMPLETE ✅

### The One Sentence to Remember

**MLflow Model Registry manages the identity, versions, aliases and lifecycle metadata of selected ML models, while deployment systems are responsible for making those models available in production.**

---

### Next: MLflow Day 4

Tracking Server Architecture, PostgreSQL Backend Store, Artifact Storage, Remote Tracking, Metadata vs Artifacts, and Failure Domains.
