# 📒 DVC Day 4 — Reproducible ML Pipelines

## 🎯 Objective

Learn how DVC **defines, tracks, and selectively reproduces** machine-learning workflow stages. Our practice pipeline validates service metrics and trains a simple incident classifier.

**One sentence to remember:** `dvc.yaml` defines the workflow, `dvc.lock` records its resolved state, and `dvc repro` reproduces stages when their tracked state requires it.

---

## 1. Why DVC Pipelines? ⭐⭐⭐⭐⭐

Running `python validate.py` and `python train.py` manually does not automatically track which inputs each script consumed, what it produced, or which steps must run again after changes.

```text
Raw CSV
   ↓
Validate → reports/validation.json
   ↓
Train → models/incident_model.joblib
```

**DVC Pipelines** declare these relationships, so the workflow can be repeated and unnecessary executions can be skipped.

## 2. Three Important Files ⭐⭐⭐⭐⭐

| File | Purpose | Git tracked? |
|---|---|---|
| `dvc.yaml` | Defines stages, commands, dependencies, and outputs | Yes |
| `dvc.lock` | Records stage commands and dependency/output hashes after reproduction | Yes |
| `*.dvc` | Tracks individual DVC-managed data files or directories | Yes |
| `.dvc/cache/` | Stores actual content-addressed data | No |

**Mental model:**

```text
dvc.yaml  → What should run?
dvc.lock  → What state was recorded?
dvc repro → What must run or be restored now?
```

## 3. Stages, Commands, Dependencies, Outputs ⭐⭐⭐⭐⭐

```yaml
stages:
  preprocess:
    cmd: python preprocess.py
    deps:
      - preprocess.py
      - data/raw.csv
    outs:
      - data/processed.csv
```

- **Stage:** `preprocess`
- **Command (`cmd`):** `python preprocess.py`
- **Dependencies (`deps`):** script and raw dataset
- **Output (`outs`):** processed dataset

When `data/raw.csv` changes, DVC detects that a declared dependency changed and normally reproduces the stage.

> Declare *all* meaningful file inputs. Undeclared dependencies can cause stale outputs because DVC cannot detect their changes.

## 4. Our Day 4 Practice Project

```text
python-for-ai/
└── dvc_pipeline/
    ├── data/service_metrics.csv
    ├── scripts/validate.py
    ├── scripts/train.py
    ├── reports/validation.json
    ├── models/incident_model.joblib
    ├── dvc.yaml
    └── dvc.lock
```

The practice dataset contains `service`, `cpu`, `memory`, `latency`, and `incident`. It is **far too small for meaningful ML accuracy assessment**; this exercise tests pipeline mechanics.

## 5. Validation Stage

`scripts/validate.py` checks the dataset and writes a report:

```python
from pathlib import Path
import json
import pandas as pd

source = Path("data/service_metrics.csv")
output = Path("reports/validation.json")
required = ["service", "cpu", "memory", "latency", "incident"]

df = pd.read_csv(source)
missing = [col for col in required if col not in df.columns]
if missing:
    raise ValueError(f"Missing columns: {missing}")
if df.empty or df[required].isna().any().any():
    raise ValueError("Empty data or missing values")
if not df["incident"].isin([0, 1]).all():
    raise ValueError("Invalid incident labels")

output.parent.mkdir(parents=True, exist_ok=True)
output.write_text(json.dumps({"status": "passed", "rows": len(df)}, indent=2))
```

The report is an output of `validate` and a dependency of `train`.

## 6. Training Stage

For this orchestration lab, a simple deterministic `DummyClassifier` produces a model artifact:

```python
from pathlib import Path
import json
import joblib
import pandas as pd
from sklearn.dummy import DummyClassifier

assert json.loads(Path("reports/validation.json").read_text())["status"] == "passed"
df = pd.read_csv("data/service_metrics.csv")
X = df[["cpu", "memory", "latency"]]
y = df["incident"]
model = DummyClassifier(strategy="most_frequent")
model.fit(X, y)
Path("models").mkdir(exist_ok=True)
joblib.dump(model, "models/incident_model.joblib")
```

**Why `DummyClassifier`?** We wanted a repeatable output without introducing model-quality distractions.

## 7. Our `dvc.yaml` ⭐⭐⭐⭐⭐

```yaml
stages:
  validate:
    cmd: python scripts/validate.py
    deps:
      - scripts/validate.py
      - data/service_metrics.csv
    outs:
      - reports/validation.json

  train:
    cmd: python scripts/train.py
    deps:
      - scripts/train.py
      - data/service_metrics.csv
      - reports/validation.json
    outs:
      - models/incident_model.joblib
```

Because `train` depends on `reports/validation.json`, DVC can infer the stage relationship. Both stages also directly depend on the CSV.

## 8. Reproducing the Pipeline ⭐⭐⭐⭐⭐

From `dvc_pipeline/`:

```bash
dvc repro
cat dvc.lock
dvc status
```

First run: DVC executes required stages and records their state in `dvc.lock`.

Second run without changes:

```bash
dvc repro
```

Unchanged stages are normally **skipped**.

**Important:** `dvc.lock` is a reproducibility record, not a record of every historical execution. Git versions its successive states.

## 9. What Triggers Reproduction? ⭐⭐⭐⭐⭐

| Change | Expected behavior | Why |
|---|---|---|
| No relevant changes | Both skipped | Inputs/outputs match recorded state |
| `data/service_metrics.csv` changes | Validate and train rerun | Both directly depend on CSV |
| `scripts/train.py` changes | Train reruns | Only train depends on that script |
| `scripts/validate.py` changes | Validate reruns; train may rerun | Depends on whether downstream input changes |
| Model output missing | Restore from cache or reproduce as needed | Tracked output no longer present |

**Nuance:** If `validate.py` changes but its resulting `validation.json` has **identical content**, the downstream train stage can remain unchanged and be skipped, assuming its other inputs are unchanged.

DVC tracks **content and declared state**, not whether a changed line of code was semantically important. Even a changed `print()` statement can trigger reproduction of a stage that depends on that script.

## 10. Useful Commands

```bash
# See what has changed
dvc status

# Execute stages that require reproduction
dvc repro

# Inspect stage definitions and recorded hashes
cat dvc.yaml
cat dvc.lock

# Restore available cached outputs into workspace
dvc checkout

# Inspect Git changes before committing
git status --short
```

Commit the pipeline definition, lockfile, and scripts to Git. DVC-managed output files should not be added to Git as ordinary large files.

## 11. Common Mistakes

1. **Assuming `dvc repro` always runs everything:** unchanged stages are usually skipped.
2. **Omitting script dependencies:** DVC cannot detect changes to undeclared inputs.
3. **Confusing `dvc.yaml` and `dvc.lock`:** definition versus recorded state.
4. **Assuming an upstream rerun forces every downstream rerun:** unchanged output content may let downstream stages skip.
5. **Treating the six-row classifier as production ML:** it only demonstrates pipeline mechanics.
6. **Assuming DVC alone captures Python package versions:** use environment/dependency management separately.

## 12. Quick Revision Cheat Sheet

```text
cmd  → command to execute
deps → tracked inputs
outs → tracked outputs

dvc.yaml → pipeline definition
dvc.lock → hashes and recorded state
dvc repro → selective reproduction
dvc status → changed/missing state
dvc checkout → restore tracked outputs from cache
```

## 13. Interview Revision Questions

1. What is the difference between `dvc.yaml` and `dvc.lock`?
2. How does DVC infer dependencies between stages?
3. Why does a changed dataset rerun both stages in our example?
4. Why can a changed validation script leave training skipped?
5. What happens if a tracked model output disappears?
6. What are the limitations of DVC reproducibility if dependencies are undeclared?

## 14. Day 4 Definition of Done

- [x] Identified `cmd`, `deps`, and `outs`.
- [x] Understood `dvc.yaml`, `dvc.lock`, and `dvc repro`.
- [x] Analyzed no-change, data-change, and code-change scenarios.
- [x] Explained cached-output restoration versus rerunning.
- [x] Completed D4-1, D4-2, and the five-scenario challenge.

**DVC Day 4: COMPLETE — Reproducible Pipelines.**

**Next:** DVC Day 5 — Connect pipeline execution to MLflow experiment and model lineage.
