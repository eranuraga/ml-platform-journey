# 📒 DVC Revision — Day 1 & Day 2

## 🎯 Objective

Learn **data versioning** using Git + DVC and restore historical versions of our SRE practice dataset. These notes are for a **10–15 minute revision**, following our NumPy notebook style.

```text
Git     → code and .dvc metadata history
DVC     → dataset content and pipeline dependencies
MLflow  → training runs, metrics and models
```

---

# DAY 1 — DVC Fundamentals

## 1. Why DVC? ⭐⭐⭐⭐⭐

Our SRE Incident Predictor might train on a 1,000-row CSV today and a 1,500-row CSV tomorrow. If we want to reproduce the first model, we need the **exact earlier dataset**.

**DVC tracks dataset versions alongside Git.** Restoration works only when the corresponding data is available in a local DVC cache or remote storage.

## 2. Git vs DVC vs MLflow ⭐⭐⭐⭐⭐

| Tool | Main responsibility | Example |
|---|---|---|
| **Git** | Source code and small metadata history | `train.py`, `.dvc` files |
| **DVC** | Data content versions and pipeline dependencies | `service_incidents.csv` |
| **MLflow** | Experiments, metrics, model lifecycle | Recall, registered models |

**Important:** Git can track small CSV files too. DVC is especially useful for large or frequently changing data. The tools **complement**, not replace, each other.

## 3. How `dvc add` Works ⭐⭐⭐⭐⭐

```bash
dvc add data/service_incidents.csv
```

```text
service_incidents.csv
        ↓ dvc add
   ┌────┴────────────┐
   ▼                 ▼
.csv.dvc metadata   DVC cache
(content hash)     (actual bytes)
   ↓
  Git
```

- **`.dvc` file:** Records the tracked path and content hash.
- **DVC cache:** Stores content-addressed data objects locally.
- **Git commit:** Versions the `.dvc` pointer.
- **DVC remote:** Optional shared data storage; covered on Day 3.

A simplified pointer looks like:

```yaml
outs:
  - md5: illustrative-hash
    size: 12345
    path: service_incidents.csv
```

**Mental model:** Git remembers *which version*; DVC manages *the bytes*.

## 4. Install and Initialize DVC

```bash
cd ~/mlops/ml-platform-journey
python -m pip install dvc
dvc version
dvc init
git status
```

`dvc init` creates DVC configuration such as `.dvc/` and `.dvcignore`. Commit shareable configuration, **not** local cache or credentials.

## 5. Track Our First Dataset ⭐⭐⭐⭐⭐

```bash
ls -lh data/service_incidents.csv
```

If the CSV is **already Git-tracked**, first stop tracking its bytes without deleting the local file:

```bash
git ls-files --error-unmatch data/service_incidents.csv
git rm --cached data/service_incidents.csv
```

Run `git rm --cached` **only if** the first command confirms the file is tracked.

```bash
dvc add data/service_incidents.csv
cat data/service_incidents.csv.dvc
git status

git add .dvc .dvcignore
git add data/service_incidents.csv.dvc data/.gitignore
git commit -m "Initialize DVC and track incident dataset"
```

**Result:** Git records the pointer and ignore rules; DVC caches the actual CSV content.

## 6. Day 1 Commands

| Command | Purpose |
|---|---|
| `dvc init` | Initialize DVC |
| `dvc add FILE` | Track or update dataset content |
| `dvc status` | Check whether tracked data changed |
| `cat FILE.dvc` | Inspect hash/path metadata |
| `git commit` | Record the pointer version |

> **Day 1 takeaway:** Git versions the `.dvc` metadata; DVC manages the dataset content.

---

# DAY 2 — Dataset Versioning & Time Travel

## 7. Goal ⭐⭐⭐⭐⭐

```text
Dataset V1 (4 rows) → Git Commit A
        ↓ edit + dvc add
Dataset V2 (6 rows) → Git Commit B
        ↓ git restore pointer + dvc checkout
Dataset V1 (4 rows)
        ↓ restore pointer + checkout
Dataset V2 (6 rows)
```

We used a **separate practice CSV**, so our original training dataset remained untouched.

## 8. Create Dataset V1

```bash
mkdir -p data/dvc_practice
cat > data/dvc_practice/service_metrics.csv <<'EOF_CSV'
service,cpu,memory,latency,incident
payment-api,82,71,120,0
orders-api,55,60,340,0
inventory-api,91,84,450,1
billing-api,67,52,180,0
EOF_CSV

dvc add data/dvc_practice/service_metrics.csv
git add data/dvc_practice/service_metrics.csv.dvc data/dvc_practice/.gitignore
git commit -m "DVC practice dataset V1"
V1_COMMIT=$(git rev-parse HEAD)
```

**V1:** Four data rows. Your actual commit was **`4ff373e`**.

## 9. Modify the Dataset and Create V2 ⭐⭐⭐⭐⭐

```bash
cat >> data/dvc_practice/service_metrics.csv <<'EOF_CSV'
auth-api,88,79,510,1
search-api,73,66,290,0
EOF_CSV

dvc status
```

You observed:

```text
data/dvc_practice/service_metrics.csv.dvc:
    changed outs:
        modified: data/dvc_practice/service_metrics.csv
```

Then:

```bash
dvc add data/dvc_practice/service_metrics.csv
git add data/dvc_practice/service_metrics.csv.dvc
git commit -m "DVC practice dataset V2"
V2_COMMIT=$(git rev-parse HEAD)
git log --oneline -5
```

**V2:** Six data rows. Your actual commit was **`f0bf388`**.

```text
f0bf388 DVC practice dataset V2
4ff373e DVC practice dataset V1
22feca7 Initialize DVC and track incident dataset
```

**Important:** A changed hash does **not** create a Git commit automatically. You explicitly stage and commit the updated pointer.

## 10. `git restore` vs `dvc checkout` ⭐⭐⭐⭐⭐

| Command | Purpose |
|---|---|
| `git restore --source=COMMIT -- FILE.dvc` | Restore historical **pointer metadata** |
| `dvc checkout FILE.dvc` | Restore **actual dataset bytes** matching that pointer |

```text
git restore → select hash in .dvc file
       ↓
dvc checkout → materialize matching data in workspace
```

## 11. Restore Dataset V1 ⭐⭐⭐⭐⭐

```bash
git restore --source=4ff373e -- \
  data/dvc_practice/service_metrics.csv.dvc

dvc checkout data/dvc_practice/service_metrics.csv.dvc

tail -n +2 data/dvc_practice/service_metrics.csv | wc -l
```

**Expected:** `4` data rows.

DVC printed `M data/dvc_practice/service_metrics.csv`. Here **`M` means the working file was modified during checkout**, not an error.

## 12. Return to Dataset V2 ⭐⭐⭐⭐⭐

```bash
git restore --source=HEAD -- \
  data/dvc_practice/service_metrics.csv.dvc

dvc checkout data/dvc_practice/service_metrics.csv.dvc

tail -n +2 data/dvc_practice/service_metrics.csv | wc -l
dvc status
git status --short
```

**Expected:** Six data rows and:

```text
Data and pipelines are up to date.
```

Your terminal confirmed the DVC status. The row counts after restoration weren't included in the transcript, so explicitly verifying them is useful.

## 13. Why Old Versions Can Be Restored

We need **both** the Git pointer and the actual content:

```text
Git V1 commit → Hash A ─┐
                        ├→ Restore V1
DVC cache/remote → Data A ┘
```

If the data is absent from **both** cache and remote storage, Git metadata alone cannot reconstruct it.

## 14. Common Mistakes ⭐⭐⭐⭐⭐

- **Thinking Git stores the DVC-tracked CSV:** Git normally stores the `.dvc` pointer, not the data bytes.
- **Forgetting `dvc add` after editing:** The `.dvc` hash does not automatically update.
- **Using only `git restore`:** The pointer changes, but `dvc checkout` is needed to restore the working data.
- **Assuming `git clone` downloads data:** A clone gets pointers; the data requires cache or remote access.
- **Treating checkout's `M` as an error:** It often means DVC updated the working file.
- **Confusing `git status` and `dvc status`:** They inspect different state.

## 15. Quick Revision Cheat Sheet

```bash
# Version dataset after editing
dvc status
dvc add data/dvc_practice/service_metrics.csv
git add data/dvc_practice/service_metrics.csv.dvc
git commit -m "Update dataset version"

# Inspect Git history
git log --oneline -5

# Restore V1 pointer + data
git restore --source=4ff373e -- data/dvc_practice/service_metrics.csv.dvc
dvc checkout data/dvc_practice/service_metrics.csv.dvc

# Return to V2 pointer + data
git restore --source=HEAD -- data/dvc_practice/service_metrics.csv.dvc
dvc checkout data/dvc_practice/service_metrics.csv.dvc

# Verify
dvc status
git status --short
```

**Note:** The commit IDs above are from our actual exercise, not universal IDs.

## 16. Interview / Self-Revision Questions

1. What problem does DVC solve?
2. How do Git, DVC and MLflow differ?
3. What is stored in a `.dvc` file?
4. What is a content hash?
5. Why can the same filename have multiple dataset versions?
6. Why rerun `dvc add` after changing data?
7. What does `dvc checkout` do?
8. How does `git restore` differ from `dvc checkout`?
9. Can Git history alone recover missing dataset bytes?
10. Why will we need `dvc push` and `dvc pull` on Day 3?

## 17. Definition of Done

### Day 1

- [x] Understood Git vs DVC vs MLflow.
- [x] Initialized DVC in the existing repository.
- [x] Tracked a dataset with `dvc add`.
- [x] Inspected DVC metadata.

### Day 2

- [x] Created and committed V1 (4 rows).
- [x] Modified and committed V2 (6 rows).
- [x] Observed `dvc status` detecting changes.
- [x] Restored V1 pointer and ran `dvc checkout`.
- [x] Returned to V2 and confirmed DVC is up to date.
- [ ] Optional: Verify row counts after both restores.

## The Two Sentences to Remember

> **Day 1:** Git versions the DVC pointer; DVC manages the dataset content.
>
> **Day 2:** `git restore` selects a historical pointer, and `dvc checkout` restores its matching bytes when available.

---

### Next — DVC Day 3: Remote Storage

We'll learn `dvc remote add`, `dvc push` and `dvc pull` so our dataset versions can be shared and recovered beyond the local Mac.
