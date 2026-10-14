# 📒 DVC Day 3 — Remote Storage, Push, Pull & Dataset Sharing

## 🎯 Objective

Understand **why DVC remote storage exists**, how to configure it, and how another engineer can retrieve the correct dataset using **Git + DVC**.

By the end of Day 3, we should be able to explain and demonstrate:

- Local DVC cache versus DVC remote storage.
- `dvc remote add`, `dvc push`, `dvc fetch`, `dvc checkout`, and `dvc pull`.
- Shared configuration versus machine-specific configuration.
- How a fresh Git clone restores DVC-tracked datasets.
- How to protect both current and historical dataset versions.

---

## 1. Why Do We Need a DVC Remote? ⭐⭐⭐⭐⭐

On Day 2, we created two versions of our practice dataset:

| Dataset version | Rows | Git commit |
|---|---:|---|
| V1 | 4 | `4ff373e` |
| V2 | 6 | `f0bf388` |

Git stores the `.dvc` metadata file, which identifies the dataset contents. **Git does not store the dataset bytes when the CSV is DVC-tracked and Git-ignored.**

```text
Git repository
  └── service_metrics.csv.dvc  → content hash

DVC local cache
  ├── content for V1
  └── content for V2
```

**Problem:** If we clone the Git repository on another machine, the `.dvc` file arrives, but the dataset content does not. If the original local cache is lost, we also need another copy of the data.

**Solution:** Store the DVC data objects in a **DVC remote** (for example, a filesystem directory, S3 bucket, or suitable object storage).

> **Mental model:** Git shares *data references*. A DVC remote shares *data contents*.

---

## 2. DVC Cache vs DVC Remote ⭐⭐⭐⭐⭐

| DVC local cache | DVC remote |
|---|---|
| Usually on the engineer's machine | Separate storage location, potentially shared |
| Holds content-addressed data objects | Holds uploaded content-addressed data objects |
| Used by `dvc checkout` | Used by `dvc push`, `dvc fetch`, `dvc pull` |
| Makes local restoration efficient | Enables retrieval by other clones/machines |
| Not automatically a backup | May be part of a backup/recovery design |

```text
                  dvc push
Local DVC Cache ───────────────→ DVC Remote
                ←───────────────
                  dvc fetch

Local DVC Cache ──dvc checkout──→ Working Dataset
```

**Important:** A remote directory on the same Mac is useful for learning but does **not** protect against loss of the entire Mac.

---

## 3. Configuring Our First DVC Remote ⭐⭐⭐⭐⭐

We used a **local filesystem remote** to learn the mechanics before introducing cloud credentials.

Our actual repository root during the exercise was:

```text
/Users/anurakuk/Desktop/python-for-ai
```

The final remote was outside that repository:

```text
/Users/anurakuk/dvc-remote-storage
```

### Step 1 — Navigate to the Git repository root

```bash
cd ~/Desktop/python-for-ai
pwd
git status --short
```

If working in a nested directory, locate the root with:

```bash
git rev-parse --show-toplevel
```

### Step 2 — Create the storage directory

```bash
mkdir -p ~/dvc-remote-storage
```

### Step 3 — Add a default remote

For a **local, machine-specific lab**, keep the remote configuration private to this clone:

```bash
dvc remote add --local -d storage /Users/anurakuk/dvc-remote-storage
```

If a remote named `storage` is already defined, modify its local URL instead:

```bash
dvc remote modify --local storage url /Users/anurakuk/dvc-remote-storage
```

Verify:

```bash
dvc remote list
```

Example:

```text
storage /Users/anurakuk/dvc-remote-storage (default)
```

### Command breakdown

| Command | Purpose |
|---|---|
| `dvc remote add` | Define a DVC storage location |
| `storage` | Logical name of the remote |
| `-d` | Make this the default remote |
| `--local` | Save configuration only for this working copy |
| `dvc remote list` | Display configured remotes |

**Important:** Configuring a remote does **not** upload any dataset. We must run `dvc push`.

---

## 4. `.dvc/config` vs `.dvc/config.local` ⭐⭐⭐⭐⭐

This was a key lesson from our troubleshooting.

| File | Git tracked? | Appropriate contents |
|---|---|---|
| `.dvc/config` | Normally yes | Shared remote name and URL (for example, team S3 bucket) |
| `.dvc/config.local` | Normally Git-ignored | Machine-specific URLs, credentials, local overrides |

For a team-shared remote, the repository might contain:

```ini
[core]
    remote = storage
['remote "storage"']
    url = s3://example-team-datasets/dvc
```

Then another engineer who clones the repository can discover where to fetch the data, **provided they also have network access and appropriate credentials**.

For our local Mac lab, `/Users/anurakuk/dvc-remote-storage` is **not portable**. We configure it in `.dvc/config.local` for each clone.

> **Mental model:** Shared configuration goes to Git; private or machine-specific configuration stays local.

**Never commit secrets or access keys to `.dvc/config`.**

---

## 5. `dvc push` — Upload Data Objects ⭐⭐⭐⭐⭐

Our tracked practice dataset is:

```text
MLFlow/data/dvc_practice/service_metrics.csv
```

Its metadata file is:

```text
MLFlow/data/dvc_practice/service_metrics.csv.dvc
```

Push the version referenced by the **current** `.dvc` file:

```bash
dvc push MLFlow/data/dvc_practice/service_metrics.csv.dvc
```

Conceptually:

```text
Git-tracked .dvc file → expected hash
                                ↓
                          DVC local cache
                                ↓
                            dvc push
                                ↓
                            DVC remote
```

**Important distinctions:**

- `dvc push` uploads dataset objects to the DVC remote.
- `git push` publishes Git commits containing code, `.dvc` metadata and any committed shared DVC configuration.
- The DVC remote stores content-addressed objects, **not necessarily files under their original CSV filenames**.
- A normal `dvc push` does not guarantee that **all historical versions** in the cache are uploaded.

### Historical versions: V1 and V2

We wanted the remote to retain both V1 and V2. Our installed DVC command did **not** accept `dvc push --rev`:

```text
ERROR: unrecognized arguments: --rev
```

So we used a straightforward workflow: select the historical `.dvc` pointer with Git, then push the referenced object.

**Precondition:** Work from a clean repository, and ensure both versions are still present in the local DVC cache. Preserve unrelated edits before using `git restore`.

```bash
# Current V2 pointer: push its object
dvc push MLFlow/data/dvc_practice/service_metrics.csv.dvc

# Select V1 metadata (no need to check out the CSV)
git restore --source=4ff373e -- \
  MLFlow/data/dvc_practice/service_metrics.csv.dvc

# Push the object referenced by V1
dvc push MLFlow/data/dvc_practice/service_metrics.csv.dvc

# Restore current committed V2 metadata
git restore --source=HEAD -- \
  MLFlow/data/dvc_practice/service_metrics.csv.dvc

# Verify
dvc status
git status --short
```

**Why no `dvc checkout` during the pushes?** We only need to change which hash the `.dvc` file references. `dvc push` reads the referenced object from the local cache. We do not need to rewrite the working CSV.

---

## 6. `dvc fetch` vs `dvc checkout` vs `dvc pull` ⭐⭐⭐⭐⭐

| Command | Direction | What it does |
|---|---|---|
| `dvc push` | Cache → Remote | Uploads required data objects |
| `dvc fetch` | Remote → Cache | Downloads data objects without checking them out |
| `dvc checkout` | Cache → Workspace | Restores tracked files from cached objects |
| `dvc pull` | Remote → Cache → Workspace | Fetches required objects and checks out files |

```text
dvc fetch
    ↓
Download into local cache

dvc checkout
    ↓
Restore working dataset from cache

dvc pull
    ↓
Fetch + checkout
```

> **One sentence:** `dvc pull` combines the retrieval and workspace-restoration steps.

---

## 7. Simulating a Second Engineer — `git clone` + `dvc pull` 🔥

This was the **main Day 3 exercise**.

### Step 1 — Clone the repository

```bash
cd ~/Desktop
git clone ~/Desktop/python-for-ai python-for-ai-clone
cd python-for-ai-clone
```

A fresh Git clone contains the committed `.dvc` files and code, but not the DVC-tracked CSV contents.

```bash
ls MLFlow/data/dvc_practice/
```

Expected before pulling: `service_metrics.csv.dvc` exists; `service_metrics.csv` is absent.

### Step 2 — Configure the remote in the clone

Because our lab remote is a local Mac path, configure it separately:

```bash
dvc remote add --local -d storage /Users/anurakuk/dvc-remote-storage
```

If `storage` already exists:

```bash
dvc remote modify --local storage url /Users/anurakuk/dvc-remote-storage
```

Verify:

```bash
dvc remote list
```

### Step 3 — Retrieve the dataset

```bash
dvc pull
```

Verify the content:

```bash
cat MLFlow/data/dvc_practice/service_metrics.csv
```

Expected dataset V2:

```csv
service,cpu,memory,latency,incident
payment-api,82,71,120,0
orders-api,55,60,340,0
inventory-api,91,84,450,1
billing-api,67,52,180,0
auth-api,88,79,510,1
search-api,73,66,290,0
```

Count **data rows**, excluding the header:

```bash
tail -n +2 MLFlow/data/dvc_practice/service_metrics.csv | wc -l
```

Expected:

```text
6
```

Check:

```bash
dvc status
```

Expected when workspace and metadata match:

```text
Data and pipelines are up to date.
```

### What did we prove?

```text
ENGINEER A                        ENGINEER B
    │                                 │
    ├── git push ──→ Git repository ──→ git clone
    │                                 │
    └── dvc push ──→ DVC remote ──────→ dvc pull
                                      │
                                      ▼
                                 Dataset V2
```

**The second clone does not need the original repository's DVC cache** when the required objects are present in the remote.

---

## 8. Optional: Restore V1 in the Second Clone

Once both historical objects are available remotely:

```bash
# Select V1 pointer
git restore --source=4ff373e -- \
  MLFlow/data/dvc_practice/service_metrics.csv.dvc

# Retrieve V1
dvc pull MLFlow/data/dvc_practice/service_metrics.csv.dvc

# Expect 4 data rows
tail -n +2 MLFlow/data/dvc_practice/service_metrics.csv | wc -l

# Return to V2 pointer
git restore --source=HEAD -- \
  MLFlow/data/dvc_practice/service_metrics.csv.dvc

# Retrieve V2
dvc pull MLFlow/data/dvc_practice/service_metrics.csv.dvc

# Expect 6 data rows
tail -n +2 MLFlow/data/dvc_practice/service_metrics.csv | wc -l
```

**Mental model:** Git selects the version; DVC retrieves the bytes matching that version.

---

## 9. Local Filesystem Remote vs Cloud Object Storage

Our lab used:

```text
/Users/anurakuk/dvc-remote-storage
```

In a team environment, we might use S3 or an appropriately configured OCI Object Storage endpoint.

For an S3 remote:

```bash
python -m pip install 'dvc[s3]'
dvc remote add -d storage s3://example-team-datasets/dvc
```

Then the core workflow is still:

```bash
dvc push
dvc pull
```

The main differences are **credentials, network access, IAM permissions, storage durability and cost**.

### SRE / MLOps considerations

- Keep credentials outside Git.
- Use least-privilege access to remote storage.
- Protect important historical objects from accidental deletion.
- Plan retention and backup policies.
- Monitor storage availability and download failures.
- Do not assume that a DVC remote is an immutable backup.

---

## 10. Common Mistakes ⭐⭐⭐⭐⭐

### Mistake 1 — Confusing `git push` and `dvc push`

```text
git push → code + .dvc metadata + shared config

dvc push → dataset content objects
```

**Both are required** when publishing a new dataset version for colleagues.

### Mistake 2 — Assuming remote configuration uploads data

`dvc remote add` only configures a destination. Use `dvc push` to upload objects.

### Mistake 3 — Committing machine-specific paths

A path such as `/Users/anurakuk/dvc-remote-storage` is not useful on another engineer's computer. Keep it in `.dvc/config.local`, or use a team-accessible remote in `.dvc/config`.

### Mistake 4 — Assuming `git clone` downloads datasets

Git retrieves the `.dvc` pointer, not the dataset content. Run `dvc pull` with access to the correct remote.

### Mistake 5 — Assuming current `dvc push` uploads all history

Historical content objects may need to be pushed separately. Keep required versions available and verify recovery.

### Mistake 6 — Treating a same-machine remote as disaster recovery

A second folder on one Mac demonstrates the workflow but does not protect against loss of that Mac.

### Mistake 7 — Treating a hash as the dataset itself

The `.dvc` file records a content identifier and metadata; the actual bytes must be available in a cache or remote.

---

## 11. Quick Revision Cheat Sheet

```bash
# Confirm repository root
git rev-parse --show-toplevel

# Create a local lab remote outside the repository
mkdir -p ~/dvc-remote-storage

# Configure a machine-local default remote
dvc remote add --local -d storage ~/dvc-remote-storage

# List remotes
dvc remote list

# Track or refresh dataset metadata
dvc add MLFlow/data/dvc_practice/service_metrics.csv

# Upload current tracked dataset content
dvc push MLFlow/data/dvc_practice/service_metrics.csv.dvc

# Retrieve content into local cache only
dvc fetch

# Restore working files from local cache
dvc checkout

# Fetch and restore working files
dvc pull

# Check workspace status
dvc status

# Inspect Git changes
git status --short
```

**Note:** If `storage` already exists, use `dvc remote modify --local storage url <path>` instead of adding it again.

---

## 12. Interview / Weekend Revision Questions

1. Why do we need DVC remote storage?
2. What is the difference between a DVC cache and a DVC remote?
3. What does `dvc remote add -d` mean?
4. Does configuring a remote automatically upload datasets?
5. What is the difference between `.dvc/config` and `.dvc/config.local`?
6. Why should object-store credentials never be committed to Git?
7. What does `dvc push` upload?
8. What is the difference between `git push` and `dvc push`?
9. Why does `git clone` not automatically retrieve DVC-tracked CSV contents?
10. What is the difference between `dvc fetch`, `dvc checkout` and `dvc pull`?
11. How does a second engineer retrieve the dataset version referenced by a `.dvc` file?
12. Why might a historical dataset be unavailable even though its Git commit exists?
13. Does a local filesystem remote protect against losing the whole machine?
14. How does this workflow change when the remote is S3 or OCI Object Storage?

---

## 13. Day 3 Definition of Done

- [x] Understood why DVC remote storage is needed.
- [x] Distinguished local cache from remote storage.
- [x] Configured a local filesystem remote.
- [x] Understood `-d` and `--local`.
- [x] Understood `.dvc/config` versus `.dvc/config.local`.
- [x] Practiced the `dvc push` workflow.
- [x] Understood how historical dataset versions are preserved remotely.
- [x] Simulated a second Git clone.
- [x] Understood how `dvc pull` retrieves the dataset.
- [x] Connected the lab workflow to S3 / OCI Object Storage.

## DVC Day 3: COMPLETE ✅

### The One Sentence to Remember

**Git publishes the dataset references, `dvc push` publishes the dataset objects, and `dvc pull` retrieves the data corresponding to the checked-out `.dvc` metadata.**

---

### Next: DVC Day 4 — Reproducible Pipelines

Learn `dvc.yaml`, `dvc.lock`, pipeline stages, dependencies, outputs, and `dvc repro`.
