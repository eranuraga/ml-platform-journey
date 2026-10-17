# 📒 DVC Day 5 — DVC + MLflow Integration and End-to-End ML Lineage

## 🎯 Objective

Connect **Git code identity**, **DVC dataset/pipeline state**, and **MLflow experiment/model identity** into one traceable ML workflow.

**One sentence to remember:** Git identifies committed code, DVC manages data and pipeline state, and MLflow records the training run and resulting model.

---

## 1. What Is ML Lineage? ⭐⭐⭐⭐⭐

Lineage answers: **Which code, data, configuration, and training run produced this model?**

```text
Git Commit
    ↓
Dataset Content Fingerprint / DVC Dependency
    ↓
DVC Pipeline (validate → train)
    ↓
MLflow Run (parameters, metrics, tags)
    ↓
Logged Model
    ↓
Registered Model Version
```

**Lineage ≠ guaranteed reproducibility.** Lineage describes provenance; reproducing the result additionally needs available dataset objects, committed code, compatible dependencies, seeds, and appropriate execution conditions.

## 2. DVC vs MLflow ⭐⭐⭐⭐⭐

| DVC | MLflow |
|---|---|
| Tracks dataset and pipeline dependencies | Tracks runs, metrics, parameters, and artifacts |
| `dvc repro` determines which stages execute | `mlflow.start_run()` records an execution |
| `dvc.yaml` + `dvc.lock` describe/record pipeline state | Runs and Model Registry describe experiment/model lineage |
| Can skip unchanged training stages | Does not automatically trigger DVC pipeline reproduction |

**Key connection:** DVC executes `train.py`; MLflow calls *inside* `train.py` record the experiment.

If `dvc repro` skips the training stage, **no new MLflow run is created by that script**.

## 3. Our Day 5 Pipeline

```text
dvc_pipeline/
├── data/service_metrics.csv
├── scripts/validate.py
├── scripts/train.py       ← enhanced with MLflow
├── reports/validation.json
├── models/incident_model.joblib
├── dvc.yaml
└── dvc.lock
```

The `validate` and `train` stages remain as defined on Day 4. Training additionally logs to an MLflow tracking server.

## 4. Capture the Code Identity

```python
import subprocess

git_commit = subprocess.check_output(
    ["git", "rev-parse", "HEAD"], text=True
).strip()
```

`git rev-parse HEAD` identifies the latest commit, **not uncommitted working-tree changes**.

Check before claiming exact code lineage:

```bash
git rev-parse HEAD
git status --short
```

For stronger reproducibility, commit the code and pipeline definition before training, or explicitly record any dirty working-tree state.

## 5. Capture the Exact Dataset Bytes ⭐⭐⭐⭐⭐

The pipeline uses the CSV as a declared DVC dependency. `dvc.lock` may still hold the *previous* dependency hash while `train.py` is executing, so our exercise fingerprinted the actual input file directly:

```python
import hashlib
from pathlib import Path

def sha256_file(path):
    digest = hashlib.sha256()
    with open(path, "rb") as f:
        for chunk in iter(lambda: f.read(1024 * 1024), b""):
            digest.update(chunk)
    return digest.hexdigest()

dataset_sha256 = sha256_file(Path("data/service_metrics.csv"))
```

**Important:** This SHA-256 is the file's content fingerprint. It is **not necessarily the same hash algorithm/value DVC records** in `dvc.lock` or a `.dvc` pointer.

Verify on macOS:

```bash
shasum -a 256 data/service_metrics.csv
```

## 6. Integrate MLflow Into Training ⭐⭐⭐⭐⭐

The essential integration pattern is:

```python
import mlflow
import mlflow.sklearn
from mlflow.models import infer_signature
from sklearn.dummy import DummyClassifier

mlflow.set_tracking_uri("http://127.0.0.1:5000")
mlflow.set_experiment("sre-incident-dvc-lineage")

# X and y are loaded from the validated service_metrics.csv.
# git_commit and dataset_sha256 are computed as shown above.
with mlflow.start_run(run_name="dvc-lineage-training") as run:
    model = DummyClassifier(strategy="most_frequent")
    model.fit(X, y)
    predictions = model.predict(X)
    training_accuracy = float((predictions == y).mean())

    mlflow.log_params({
        "model_type": "DummyClassifier",
        "strategy": "most_frequent",
        "training_rows": len(X),
    })
    mlflow.log_metric("training_accuracy", training_accuracy)
    mlflow.set_tags({
        "git_commit": git_commit,
        "dataset_sha256": dataset_sha256,
        "pipeline": "dvc",
        "dataset_path": "data/service_metrics.csv",
    })

    model_info = mlflow.sklearn.log_model(
        sk_model=model,
        name="incident_model",
        signature=infer_signature(X, predictions),
        input_example=X.iloc[[0]],
    )
    print(run.info.run_id, model_info.model_uri)
```

Our actual script also saved `models/incident_model.joblib` with `joblib.dump()`, satisfying the DVC training stage's declared output.

**Important:** `training_accuracy` on the same seven rows used for fitting is *not* a production model-quality metric.

## 7. Reproduce the Integrated Workflow

From `dvc_pipeline/`:

```bash
# Start your MLflow server separately, then:
dvc repro
```

```text
dvc repro
    ↓
Check changed dependencies
    ↓
Execute train stage if required
    ↓
Log MLflow run + lineage tags
    ↓
Write local model output
    ↓
Update dvc.lock
```

The MLflow run is an **external side effect**. If logging fails after partial work, retries may create duplicate or partial runs. Production pipelines should handle retries and consistency deliberately.

## 8. Our Actual Day 5 Results ⭐⭐⭐⭐⭐

| Field | Recorded value |
|---|---|
| Git commit | `f0bf388a784a469666b7e2bbe356e6b1431c99a9` |
| Dataset SHA-256 | `d2400069d98c5c030291282e792cdd840441b32fc55f4ed134f6506273c96fdd` |
| MLflow Run ID | `cef17481c1804d13a256865b3a8cd3f0` |
| Run status | `FINISHED` |
| Model type | `DummyClassifier` |
| Training rows | `7` |
| Training accuracy | `0.7142857142857143` (71.43%) |
| Logged Model URI | `models:/m-3a47d968ac404cb1ab199d25247f3d5e` |
| Registered model | `sre-incident-dvc-lineage` |
| Registered version | `1` |
| Test prediction | `[0]` |

These values were obtained from the successful exercise. The Git commit and dataset hash were **logged as tags**; a fully reproducible claim still requires verifying the dataset bytes and that the training code was committed/clean.

## 9. Verify the MLflow Run

```python
import mlflow

mlflow.set_tracking_uri("http://127.0.0.1:5000")
run = mlflow.get_run("cef17481c1804d13a256865b3a8cd3f0")

print(run.info.status)
print(run.data.tags.get("git_commit"))
print(run.data.tags.get("dataset_sha256"))
print(run.data.params)
print(run.data.metrics)
```

Observed: `FINISHED`, expected tags, parameters, and `training_accuracy`.

## 10. Load the Logged Model and Predict ⭐⭐⭐⭐⭐

```python
import mlflow
import pandas as pd

mlflow.set_tracking_uri("http://127.0.0.1:5000")
model = mlflow.pyfunc.load_model(
    "models:/m-3a47d968ac404cb1ab199d25247f3d5e"
)
sample = pd.DataFrame([{"cpu": 82, "memory": 71, "latency": 120}])
print(model.predict(sample))
```

Observed output:

```text
<class 'mlflow.pyfunc.PyFuncModel'>
Prediction: [0]
```

The artifact downloaded successfully, and the loaded model accepted the expected input schema.

## 11. Register the Model ⭐⭐⭐⭐⭐

```python
import mlflow
from mlflow import MlflowClient

mlflow.set_tracking_uri("http://127.0.0.1:5000")
registered = mlflow.register_model(
    model_uri="models:/m-3a47d968ac404cb1ab199d25247f3d5e",
    name="sre-incident-dvc-lineage",
)
print(registered.name, registered.version)

client = MlflowClient()
client.set_registered_model_alias(
    name="sre-incident-dvc-lineage",
    alias="candidate",
    version=registered.version,
)
candidate = client.get_model_version_by_alias(
    name="sre-incident-dvc-lineage",
    alias="candidate",
)
print(candidate.name, candidate.version, candidate.source)
```

Observed registration: **Version 1**, with a successful alias lookup in our exercise.

```text
Git + Dataset Fingerprint
          ↓
DVC Pipeline
          ↓
MLflow Run
          ↓
Logged Model
          ↓
Registered Model: sre-incident-dvc-lineage
          ↓
Version 1 → candidate
```

**Do not treat this demonstration model as a production champion.** It was trained on only seven rows using `DummyClassifier`.

## 12. Lineage vs Reproducibility ⭐⭐⭐⭐⭐

To reproduce a historical model reliably, we need:

- **Committed code:** Git commit and clean/recorded working-tree state.
- **Data:** exact content identity and retrievable DVC objects.
- **Pipeline:** `dvc.yaml`, `dvc.lock`, declared dependencies and outputs.
- **Training settings:** parameters, seeds, preprocessing and splits.
- **Environment:** compatible Python/library versions and relevant runtime configuration.
- **Experiment:** MLflow Run ID, metrics, logged model and artifact access.

A Git SHA and dataset SHA alone **do not guarantee** bit-for-bit identical results.

## 13. Common Mistakes

1. **DVC and MLflow do the same job:** DVC reproduces dependencies; MLflow tracks experiment results.
2. **A Git commit always identifies executed code:** uncommitted changes are not captured by `HEAD`.
3. **The SHA-256 file fingerprint is the DVC MD5 hash:** these are distinct identifiers.
4. **`dvc repro` always creates an MLflow run:** unchanged training stages are skipped.
5. **A registered model is automatically deployed:** registry and serving are separate.
6. **Training accuracy on seven rows measures production quality:** it does not.
7. **Logging metadata guarantees data availability:** old DVC objects and MLflow artifacts must remain retrievable.

## 14. Quick Revision Cheat Sheet

```bash
# DVC pipeline
dvc status
dvc repro
cat dvc.yaml
cat dvc.lock

# Git code identity and clean-tree check
git rev-parse HEAD
git status --short

# Dataset content fingerprint (macOS)
shasum -a 256 data/service_metrics.csv
```

```python
# MLflow lineage tags (inside an active run)
mlflow.set_tag("git_commit", git_commit)
mlflow.set_tag("dataset_sha256", dataset_sha256)

# Resolve a logged model
model = mlflow.pyfunc.load_model(model_uri)

# Register it
registered = mlflow.register_model(model_uri, "sre-incident-dvc-lineage")
```

## 15. Interview Revision Questions

1. How do DVC pipelines and MLflow complement each other?
2. Why log dataset identity and Git commit in MLflow?
3. Why is reading an old `dvc.lock` during training potentially misleading?
4. Why is a dirty Git working tree a lineage risk?
5. What happens to MLflow logging when DVC skips training?
6. How do you trace a registered model version back to a run and dataset?
7. Why does successful lineage logging not guarantee reproducibility?
8. What could happen if MLflow is unavailable during a DVC training stage?

## 16. Day 5 Definition of Done

- [x] Connected MLflow tracking to DVC training.
- [x] Captured Git commit and dataset SHA-256 tags.
- [x] Created a finished MLflow run.
- [x] Logged parameters, training metric, and model.
- [x] Loaded the model and predicted successfully.
- [x] Registered the logged model as Version 1.
- [x] Resolved the candidate alias.
- [x] Understood the limits of the lineage record.

**DVC Day 5: COMPLETE — End-to-End ML Lineage.**

**Next:** FastAPI + Docker — expose a registered model through an HTTP inference service.
