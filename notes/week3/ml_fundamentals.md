# 📒 Week 3 - Day 1 Notes — Machine Learning Fundamentals

## 🎯 Day Objective

Understand the fundamental concepts behind Machine Learning before training the first model.

The focus was not on ML algorithms or mathematics yet.

Instead, the objective was to understand:

```text
Business Problem
      ↓
Dataset
      ↓
Features + Target
      ↓
Training / Validation / Test
      ↓
Model
      ↓
Prediction
      ↓
Evaluation
      ↓
Deployment
      ↓
Monitoring
```

This session established the vocabulary and mental models required to understand the ML lifecycle from an MLOps engineering perspective.

---

# 📚 What is Machine Learning?

Traditional programming usually works like:

```text
Rules
 +
Data
 ↓
Program
 ↓
Result
```

For example, an SRE health classifier could contain explicitly written rules:

```python
if cpu > 90 and error_rate > 0.05:
    incident = True
```

The engineer defines the relationship between the input and output.

Machine Learning changes this approach.

Instead of manually defining every rule:

```text
Historical Data
      +
Known Answers
      ↓
ML Algorithm
      ↓
Learn Patterns
      ↓
Model
      ↓
Predictions
```

For example:

```text
CPU
Memory
Error Rate
Latency
Restart Count
      ↓
Historical Examples
      ↓
ML Algorithm
      ↓
Learn relationship
      ↓
Predict Incident
```

Important learning:

> Traditional programming explicitly defines the rules, while Machine Learning attempts to learn useful relationships from historical data.

---

# Sample / Observation

A **sample**, also called an **observation**, represents one record in the dataset.

Example:

```text
service       cpu   memory   error_rate   incident
payment-api    92      84       0.07          1
```

This entire row represents one observation.

Conceptually:

```text
Dataset
   ↓
Rows
   ↓
Samples / Observations
```

If a dataset contains:

```text
10,000 rows
```

then it generally contains:

```text
10,000 observations
```

at the current level of understanding.

---

# Features ⭐⭐⭐⭐⭐

Features are the input variables that the model is allowed to use to make predictions.

For an incident-prediction problem:

```text
CPU
Memory
Error Rate
Request Rate
Latency
Restart Count
```

can be features.

Mental model:

```text
Features
   ↓
Information available to model
   ↓
Model uses information
   ↓
Prediction
```

The important question is:

> **What information should the model be allowed to use when making its prediction?**

---

# Target / Label ⭐⭐⭐⭐⭐

The **target**, also called the **label**, is what the model is trying to predict.

For example:

```text
incident_next_hour
```

may contain:

```text
0 → No incident
1 → Incident
```

Conceptually:

```text
FEATURES                       TARGET

CPU                            Incident?
Memory
Error Rate        ─────────→   0 / 1
Latency
Restart Count
```

Important distinction:

> Features are the inputs. The target is the answer the model is learning to predict.

---

# Important Mistake — Target is NOT a Feature

Initially identified:

```text
CPU
Memory
Error Rate
Incident
```

as features.

This was incorrect because:

```text
incident
```

is the target itself.

The corrected relationship is:

```text
Features:
CPU
Memory
Error Rate

Target:
Incident
```

Mental model:

```text
        INPUTS                    ANSWER
           │                         │
           ▼                         ▼
CPU + Memory + Error Rate ─────→ Incident
         X                          y
```

Important learning:

> The target must not accidentally be included in the feature set because that would expose the answer to the model.

---

# Identifier Columns

The example dataset also contained:

```text
service
```

such as:

```text
payment-api
orders-api
inventory-api
```

For the first model, this column was deliberately excluded from the features.

This does NOT mean:

> Strings can never be ML features.

Categorical data can later be transformed or encoded into representations suitable for ML.

The better question is:

> **Does this field contain useful predictive information, and if so, how should it be represented?**

For the first model, the service name is being treated as an identifier rather than a feature.

---

# `X` and `y` ⭐⭐⭐⭐⭐

A very common ML convention is:

```text
X → Features
y → Target
```

For the service incident dataset:

```python
X = df[
    [
        "cpu",
        "memory",
        "error_rate",
        "request_rate",
        "latency",
        "restart_count"
    ]
]
```

and:

```python
y = df["incident_next_hour"]
```

Conceptually:

```text
                    DATASET
                       │
             ┌─────────┴─────────┐
             ▼                   ▼
          FEATURES             TARGET
             │                   │
             ▼                   ▼
             X                   y
```

---

# Understanding `X`

`X` contains the information the model will use for learning and prediction.

For example:

```text
CPU   Memory   Error Rate   Request Rate   Latency   Restarts

92      84        0.07          1200          450        3
55      60        0.01           800          120        0
88      79        0.06           950          380        2
```

Each row represents one observation.

Each column represents one feature.

Mental model:

```text
X

ROWS
 ↓
Samples


COLUMNS
 ↓
Features
```

This connects directly with the NumPy and Pandas concepts learned earlier.

---

# Understanding `y`

`y` contains the correct target value associated with every sample.

For example:

```text
1
0
1
0
...
```

where:

```text
1 → Incident next hour
0 → No incident next hour
```

Every row in `X` must correspond to a target in `y`.

Conceptually:

```text
X row 0 ─────────→ y row 0
X row 1 ─────────→ y row 1
X row 2 ─────────→ y row 2
...
```

If:

```text
X.shape = (30, 6)
```

this means:

```text
30 samples
6 features
```

while:

```text
y.shape = (30,)
```

means:

```text
30 target values
```

Therefore:

```text
30 observations in X
        ↓
30 corresponding answers in y
```

---

# Pandas Connection

Because multiple columns were selected for `X`:

```python
X = df[
    [
        "cpu",
        "memory",
        "error_rate",
        "request_rate",
        "latency",
        "restart_count"
    ]
]
```

`X` is a:

```text
Pandas DataFrame
```

Because one column was selected for `y`:

```python
y = df["incident_next_hour"]
```

`y` is a:

```text
Pandas Series
```

This reinforced the Pandas concept:

```text
df["column"]
      ↓
Series


df[["column1", "column2"]]
      ↓
DataFrame
```

---

# Supervised Learning

The first ML problems in this journey use **supervised learning**.

Supervised learning means the training data contains known answers.

Conceptually:

```text
Features                     Known Target
   │                              │
   ▼                              ▼
CPU                            Incident = 1
Memory
Error Rate
Latency
   │                              │
   └──────────────┬───────────────┘
                  ↓
              Training
```

The model learns relationships between the inputs and known outputs.

---

# Classification vs Regression ⭐⭐⭐⭐⭐

Two major supervised-learning problem types were introduced.

## Classification

Classification predicts a **category or class**.

Examples:

```text
Will service fail?
      ↓
YES / NO


Is email spam?
      ↓
SPAM / NOT SPAM


Is transaction fraudulent?
      ↓
FRAUD / NOT FRAUD
```

For the service incident problem:

```text
0 → No incident
1 → Incident
```

Therefore this is:

```text
Binary Classification
```

---

# Regression

Regression predicts a numerical value.

Examples:

```text
Next-hour CPU
      ↓
82.7%


Request latency
      ↓
238 ms


Future resource demand
      ↓
numeric value
```

The simplest mental model:

```text
What kind of answer
am I predicting?
        ↓

Category?
   ↓
Classification


Number?
   ↓
Regression
```

---

# Classification vs Regression Exercise

Correctly identified:

| Problem                                     | ML Problem     |
| ------------------------------------------- | -------------- |
| Predict whether a service will fail         | Classification |
| Predict next hour's CPU utilization         | Regression     |
| Predict whether an email is spam            | Classification |
| Predict request latency                     | Regression     |
| Predict whether a transaction is fraudulent | Classification |

A useful language pattern:

```text
"Whether..."
     ↓
Often Classification


"How much / what value..."
     ↓
Often Regression
```

The actual target type should always be checked rather than relying only on wording.

---

# Training Data ⭐⭐⭐⭐⭐

Training data is used by the ML algorithm to learn relationships between features and the target.

Conceptually:

```text
X_train
   +
y_train
   ↓
ML Algorithm
   ↓
Learn Parameters
   ↓
Trained Model
```

Soon this will appear in code as:

```python
model.fit(
    X_train,
    y_train
)
```

Mental model:

```text
fit()
 ↓
Learn from training examples
```

---

# Test Data ⭐⭐⭐⭐⭐

Test data contains observations that were not used to train the model.

Conceptually:

```text
Dataset
   │
   ├─────────────┐
   ▼             ▼
Training       Test
Data           Data
   │             │
   ▼             │
Train Model      │
   │             │
   └─────────────┘
         ↓
Evaluate model
on unseen data
```

The goal is to answer:

> **Does the model work on data it did not see during training?**

---

# Why We Don't Train and Test on the Same Data

If the model is evaluated using the same data it learned from, the result may look artificially good.

The model may simply have learned patterns specific to the training dataset.

The real goal is:

```text
Historical Training Data
          ↓
Learn useful patterns
          ↓
Previously Unseen Data
          ↓
Still perform well
```

This leads to the concept of generalization.

---

# Generalization ⭐⭐⭐⭐⭐

Generalization means:

> A model performs well on new, unseen observations rather than only on its training data.

Conceptually:

```text
Training Examples
      ↓
Learn Patterns
      ↓
Unseen Data
      ↓
Useful Predictions
```

Generalization is one of the main objectives of ML model development.

---

# Validation Data

In addition to training and test datasets, a validation dataset may be used.

Conceptually:

```text
Dataset
   │
   ├── Training
   ├── Validation
   └── Test
```

## Training

Used to learn model parameters.

## Validation

Used during model development for decisions such as:

```text
Model selection
Hyperparameter tuning
Threshold selection
```

## Test

Used for final evaluation.

Mental model:

```text
Train
 ↓
Learn


Validation
 ↓
Choose / Tune


Test
 ↓
Final Evaluation
```

---

# Parameters vs Hyperparameters ⭐⭐⭐⭐

An important distinction for future MLOps work.

## Parameters

Parameters are learned by the model during training.

Examples can include:

```text
Weights
Coefficients
```

Conceptually:

```text
Training Data
      ↓
Training Algorithm
      ↓
Learn Parameters
```

---

## Hyperparameters

Hyperparameters configure the training process or model behavior.

Examples:

```text
Number of Trees
Maximum Tree Depth
Learning Rate
Regularization Strength
```

Conceptually:

```text
Hyperparameters
       ↓
Configure Training
       ↓
Training
       ↓
Model Parameters Learned
```

This distinction will become important when using **MLflow**.

MLflow will eventually help track:

```text
Experiment
   │
   ├── Hyperparameters
   ├── Metrics
   ├── Model
   └── Artifacts
```

---

# Training vs Inference ⭐⭐⭐⭐⭐

These are two fundamentally different phases.

## Training

```text
Historical Data
      ↓
Model learns
      ↓
Model Artifact
```

Eventually:

```python
model.fit(
    X_train,
    y_train
)
```

---

## Inference

Inference means using an already-trained model to make predictions.

Eventually:

```python
predictions = model.predict(
    X_test
)
```

Mental model:

```text
TRAINING

Data
 ↓
Learn
 ↓
Model


INFERENCE

New Data
 ↓
Existing Model
 ↓
Prediction
```

This distinction becomes extremely important in production MLOps architecture.

---

# Overfitting ⭐⭐⭐⭐⭐

Overfitting occurs when a model learns the training data too specifically and fails to generalize well.

Example:

```text
Training Accuracy = 99%
Test Accuracy     = 72%
```

This may indicate:

```text
Training Data
     ↓
Model learned too specifically
     ↓
Excellent training performance
     ↓
Poor unseen-data performance
```

Mental model:

```text
Memorized training patterns
          ↓
      Overfitting
```

---

# Underfitting

Underfitting occurs when the model does not learn enough useful structure from the data.

Example:

```text
Training Accuracy = 61%
Test Accuracy     = 59%
```

Conceptually:

```text
Model insufficiently captures
useful patterns
       ↓
Underfitting
```

The overall objective is:

```text
Underfitting
     ↓
Good Generalization
     ↓
Overfitting
```

The model should learn enough structure without becoming overly specific to its training data.

---

# Data Leakage 🚨 ⭐⭐⭐⭐⭐

Data leakage occurs when information that should not legitimately be available during training influences the model.

This can make evaluation results appear unrealistically good.

Example:

Suppose the problem is:

> Predict whether a service will experience an incident tomorrow.

Using:

```text
tomorrow_incident_count
```

as a feature would expose future information.

The model effectively receives part of the answer.

---

# Preprocessing Leakage

Another important example:

```text
Entire Dataset
      ↓
Calculate normalization statistics
      ↓
Train/Test Split
```

The test data influenced the preprocessing.

A safer conceptual approach:

```text
Dataset
   ↓
Split
   ↓
Training Data
   ↓
Learn preprocessing
   ↓
Apply learned preprocessing
   ↓
Train + Test
```

This principle will later be implemented using Scikit-learn pipelines.

---

# Baseline Model

A model should be compared against a simple baseline.

Suppose:

```text
95% of samples = No Incident
5% of samples  = Incident
```

A trivial system that always predicts:

```text
NO INCIDENT
```

already gets:

```text
95% accuracy
```

Therefore, an ML model achieving:

```text
94% accuracy
```

would not automatically be useful.

Important learning:

> A high metric does not mean much without context and an appropriate baseline.

This will become important when learning evaluation metrics.

---

# ML Lifecycle ⭐⭐⭐⭐⭐

The complete high-level ML lifecycle introduced today:

```text
Business Problem
       ↓
Collect Data
       ↓
Clean / Validate Data
       ↓
Define Features + Target
       ↓
Train / Validation / Test
       ↓
Train Model
       ↓
Evaluate
       ↓
Tune
       ↓
Create Model Artifact
       ↓
Deploy
       ↓
Inference
       ↓
Monitor
       ↓
Retrain
```

This lifecycle is the foundation for understanding why MLOps exists.

---

# Traditional Software vs ML Systems

Traditional software lifecycle:

```text
Code
 ↓
Build
 ↓
Test
 ↓
Deploy
 ↓
Monitor
```

ML introduces additional moving pieces:

```text
Data
 ↓
Training
 ↓
Model
 ↓
Evaluation
 ↓
Deployment
 ↓
Inference
 ↓
Monitoring
 ↓
Data / Model Changes
 ↓
Retraining
```

Important MLOps learning:

> Production ML systems involve both software lifecycle management and data/model lifecycle management.

---

# SRE Incident Prediction Problem

Defined the first ML problem for the journey:

> **Predict whether a Kubernetes/service workload is likely to experience an incident in the next hour.**

Dataset fields:

```text
service
cpu
memory
error_rate
request_rate
latency
restart_count
incident_next_hour
```

Potential features:

```text
cpu
memory
error_rate
request_rate
latency
restart_count
```

Target:

```text
incident_next_hour
```

Problem type:

```text
Binary Classification
```

This connects:

```text
SRE
 +
Observability
 +
Machine Learning
 +
MLOps
```

and will continue to be used during the upcoming Scikit-learn exercises.

---

# Practical Exercise — Creating `X` and `y`

Loaded:

```python
import pandas as pd

df = pd.read_csv(
    "data/service_incidents.csv"
)
```

Created the feature matrix:

```python
X = df[
    [
        "cpu",
        "memory",
        "error_rate",
        "request_rate",
        "latency",
        "restart_count"
    ]
]
```

Created the target:

```python
y = df[
    "incident_next_hour"
]
```

Then inspected:

```python
print(X.shape)
print(y.shape)

print(X.head())
print(y.head())
```

This represented the first practical conversion from:

```text
Raw Dataset
     ↓
Features + Target
     ↓
X + y
```

which is the standard input structure for supervised ML training.

---

# Connection to Scikit-learn

The next step will be:

```text
X + y
  ↓
train_test_split()
  ↓
┌──────────────────────┐
│                      │
▼                      ▼
X_train              X_test
y_train              y_test
│                      │
▼                      │
model.fit()             │
│                      │
▼                      │
Trained Model           │
│                      │
└──────────┬────────────┘
           ▼
     model.predict()
           ↓
      Predictions
           ↓
Compare with y_test
           ↓
       Evaluation
```

This is where the next session will begin.

---

# 💡 Engineering Learnings

* ML begins with a clearly defined prediction problem.
* Rows represent samples/observations.
* Features are the information available to the model.
* The target is what the model is trying to predict.
* `X` conventionally represents features.
* `y` conventionally represents the target.
* Classification predicts categories.
* Regression predicts numerical values.
* Training data is used to learn.
* Test data evaluates performance on unseen observations.
* Validation data helps with model-development decisions.
* Generalization matters more than memorizing training data.
* Parameters are learned by the model.
* Hyperparameters configure the model/training process.
* Training and inference are separate lifecycle phases.
* Overfitting means the model is too specific to training data.
* Underfitting means the model has not learned enough useful structure.
* Data leakage can produce misleading evaluation results.
* Preprocessing must be designed carefully to avoid leakage.
* Models should be compared against meaningful baselines.
* MLOps manages much more than simply deploying a model.

---

# ⚠️ Mistakes / Clarifications

Initially included:

```text
incident
```

among the features.

This was incorrect because `incident` is the target.

Correct relationship:

```text
X

CPU
Memory
Error Rate


y

Incident
```

Important lesson:

> The target must not be accidentally exposed as an input feature.

---

Initially, the `service` column was excluded because it contains Strings.

The more precise understanding is:

> The service name is excluded from this first model because it is being treated as an identifier, not simply because it is a String.

Categorical data can potentially become a feature after appropriate encoding.

---

# 🚀 Production / MLOps Takeaways

From an SRE perspective, the ML lifecycle can be mapped to familiar operational concepts.

```text
SOFTWARE / SRE              ML / MLOps

Service Deployment          Model Deployment

Application Version         Model Version

CPU / Memory                Feature / Data Health

Request Latency             Inference Latency

Application Errors          Prediction / Serving Errors

Logs                        Training / Inference Logs

Rollback                    Model Rollback

Service Monitoring          Model Monitoring

Configuration Drift         Data / Feature Drift

Application Release         Model Promotion

Incident Response           ML/Data Incident Response
```

The goal of this journey is therefore not to become a Data Scientist.

The goal is to understand ML deeply enough to:

```text
Build
Train
Evaluate
Package
Deploy
Observe
Troubleshoot
Scale
Govern
Automate
```

ML systems reliably.

This is where the existing SRE background will eventually become a significant advantage.

---

# 🏆 End of Day Reflection

Today was the first session where the journey moved from:

```text
Python Engineering
      ↓
Data Processing
```

into:

```text
Machine Learning
```

The most important mental model was:

```text
                DATASET
                   │
         ┌─────────┴─────────┐
         ▼                   ▼
      FEATURES             TARGET
         │                   │
         ▼                   ▼
         X                   y
         │                   │
         └─────────┬─────────┘
                   ▼
               TRAINING
                   ↓
                 MODEL
                   ↓
               INFERENCE
                   ↓
              PREDICTION
```

The second major takeaway was that training a model is only one part of the overall lifecycle:

```text
Data
 ↓
Training
 ↓
Evaluation
 ↓
Artifact
 ↓
Deployment
 ↓
Inference
 ↓
Monitoring
 ↓
Retraining
```

That complete lifecycle is what makes MLOps necessary.

From today onwards, whenever I encounter an ML use case, I'll first ask:

> **What exactly are we predicting?**

> **What are the features available at prediction time?**

> **What is the target?**

> **Is this classification or regression?**

> **How will we evaluate whether the model generalizes to unseen data?**

> **Could any of the data introduce leakage?**

Those questions matter more than immediately choosing an ML algorithm.

**Machine Learning Fundamentals: Theory ✅**

**Features / Target / X / y: Theory ✅ | Hands-on ✅**

**Classification vs Regression: Theory ✅**

**Train / Validation / Test: Fundamentals ✅**

**Generalization / Overfitting / Underfitting: Fundamentals ✅**

**Data Leakage: Fundamentals ✅**

**ML Lifecycle: Fundamentals ✅**

**Next: Week 3 — Day 2: Scikit-learn + Train/Test Split + First ML Model 🚀**
