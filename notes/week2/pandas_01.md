# 📒 Week 2 - Day 2 Notes — Pandas Fundamentals

## 🎯 Day Objective

Understand the fundamentals of Pandas and become comfortable working with labeled, tabular data using **Series and DataFrames**.

The focus was deliberately kept narrow:

```text
DataFrame
    ↓
Inspect
    ↓
Select
    ↓
Filter
    ↓
Sort
    ↓
Transform
    ↓
Summarize
    ↓
Export
```

The goal was not to memorize dozens of Pandas methods.

Instead, the goal was to develop the correct mental model for working with tabular data before moving into data cleaning, EDA, and ML preprocessing.

---

# 📚 Concepts Learned

## Why Pandas?

During the previous NumPy session, numerical data looked like:

```python
metrics = np.array([
    [82, 71],
    [55, 60],
    [91, 84]
])
```

To access CPU values:

```python
metrics[:, 0]
```

This requires remembering:

```text
Column 0 → CPU
Column 1 → Memory
```

Pandas allows the same data to have meaningful labels:

```python
df = pd.DataFrame({
    "service": [
        "payment-api",
        "orders-api",
        "inventory-api"
    ],
    "cpu": [
        82,
        55,
        91
    ]
})
```

Now CPU can be accessed using:

```python
df["cpu"]
```

Important mental model:

```text
NumPy
   ↓
Numerical arrays
   ↓
Positions matter


Pandas
   ↓
Tabular data
   ↓
Labels have meaning
```

---

# Pandas Series

A Pandas `Series` represents one-dimensional labeled data.

Example:

```python
cpu = pd.Series([
    82,
    55,
    91
])
```

The type can be checked using:

```python
print(type(cpu))
```

Conceptually:

```text
Series
   ↓
One-dimensional collection
   ↓
Values + Index
```

A Series can contain a single column of a larger DataFrame.

For example:

```python
services["cpu"]
```

returns a Series.

Important learning:

> A Series can be thought of as one labeled column of data.

---

# Pandas DataFrame

A `DataFrame` represents two-dimensional tabular data.

Example:

```python
services = pd.DataFrame({
    "service": [
        "payment-api",
        "orders-api"
    ],
    "cpu": [
        82,
        55
    ],
    "status": [
        "UP",
        "DOWN"
    ]
})
```

Conceptually:

```text
        service        cpu    status

0       payment-api     82      UP
1       orders-api      55      DOWN
```

The main mental model:

```text
DataFrame
   ↓
Table
   ↓
Rows + Columns + Values
```

This is different from treating the data purely as a numerical matrix.

---

# Creating DataFrames

Practiced creating DataFrames in two ways.

## Dictionary of Lists

```python
services = pd.DataFrame({
    "service": [
        "payment-api",
        "orders-api"
    ],
    "cpu": [
        82,
        55
    ],
    "status": [
        "UP",
        "DOWN"
    ]
})
```

Conceptually:

```text
Column Name
    ↓
List of column values
```

---

## List of Dictionaries

The same table can be represented using:

```python
services = pd.DataFrame([
    {
        "service": "payment-api",
        "cpu": 82,
        "status": "UP"
    },
    {
        "service": "orders-api",
        "cpu": 55,
        "status": "DOWN"
    }
])
```

Conceptually:

```text
Dictionary
    ↓
One record / row


List of Dictionaries
    ↓
Collection of records
```

This connects directly with the Python structures learned during Week 1.

Important learning:

> Pandas builds on Python concepts I already know rather than replacing them.

---

# Inspecting a DataFrame ⭐

Before analyzing an unfamiliar dataset, the first step should be understanding what the data looks like.

Practiced:

```python
services.head()
```

to inspect the first rows.

```python
services.tail()
```

to inspect the final rows.

```python
services.shape
```

to determine:

```text
(number of rows, number of columns)
```

For example:

```text
(4, 4)
```

means:

```text
4 rows
4 columns
```

---

# `size` vs `shape`

An important distinction:

```python
services.shape
```

describes the table structure.

For example:

```text
(4, 4)
```

while:

```python
services.size
```

returns the total number of cells:

```text
4 × 4 = 16
```

Mental model:

```text
shape
   ↓
Rows × Columns


size
   ↓
Total number of values/cells
```

---

# Inspecting Columns

Column names can be inspected using:

```python
services.columns
```

This helps answer:

> What information does this dataset contain?

For an ML dataset, understanding the available columns is one of the first steps before deciding which data may eventually become features or labels.

---

# Inspecting Data Types

Data types can be inspected using:

```python
services.dtypes
```

This helps determine whether columns contain:

```text
Integers
Floats
Strings / objects
Booleans
etc.
```

Data types will become increasingly important during data cleaning and ML preprocessing.

---

# `info()`

Used:

```python
services.info()
```

to inspect the overall DataFrame structure.

`info()` provides useful information such as:

* Number of rows
* Column names
* Non-null counts
* Data types

Mental model:

```text
info()
   ↓
Structural health check of the dataset
```

This will become one of the first commands used when inspecting unfamiliar ML data.

---

# `describe()`

Used:

```python
services.describe()
```

to generate summary statistics for numerical columns.

For example:

```text
CPU
Memory
```

may produce statistics such as:

```text
count
mean
std
min
25%
50%
75%
max
```

At the current stage, the important idea is:

> `describe()` provides a quick numerical summary of the dataset.

The statistical concepts will be explored more deeply when required for ML.

---

# Column Selection ⭐⭐⭐⭐⭐

Selecting a single column:

```python
services["cpu"]
```

returns a:

```text
Series
```

Selecting multiple columns:

```python
services[
    [
        "cpu",
        "service",
        "status"
    ]
]
```

returns a:

```text
DataFrame
```

Important distinction:

```text
df["cpu"]
    ↓
Series


df[["cpu"]]
    ↓
DataFrame
```

The outer brackets perform DataFrame selection.

The inner List specifies which columns should be selected.

---

# Moving Away from Positional Thinking

During NumPy, data was often selected using positions:

```python
metrics[:, 0]
```

With Pandas:

```python
services["cpu"]
```

expresses the same intent more clearly.

Comparison:

```text
NumPy

metrics[:, 0]

"I want column 0."


Pandas

services["cpu"]

"I want CPU."
```

Important learning:

> Pandas allows code to express the meaning of tabular data rather than relying entirely on positional indexes.

---

# Boolean Filtering ⭐⭐⭐⭐⭐

Boolean filtering was one of the most important concepts from today's session.

Given:

```python
services = pd.DataFrame({
    "service": [
        "payment-api",
        "orders-api",
        "inventory-api",
        "billing-api"
    ],
    "cpu": [
        82,
        55,
        91,
        67
    ],
    "memory": [
        71,
        60,
        84,
        52
    ],
    "status": [
        "UP",
        "DOWN",
        "UP",
        "UP"
    ]
})
```

Services with CPU greater than 80:

```python
services[
    services["cpu"] > 80
]
```

The condition:

```python
services["cpu"] > 80
```

creates a Boolean Series.

Conceptually:

```text
payment-api      True
orders-api       False
inventory-api    True
billing-api      False
```

Pandas then keeps the rows where the condition is `True`.

Mental model:

```text
DataFrame
    ↓
Column
    ↓
Condition
    ↓
Boolean Mask
    ↓
Matching Rows
```

This directly connects with NumPy Boolean masking.

---

# Multiple Filtering Conditions

Conditions can be combined.

## AND

```python
services[
    (services["cpu"] > 80)
    &
    (services["status"] == "UP")
]
```

Meaning:

```text
CPU > 80
AND
status == UP
```

---

## OR

```python
services[
    (services["cpu"] > 80)
    |
    (services["memory"] > 80)
]
```

Meaning:

```text
CPU > 80
OR
Memory > 80
```

---

## Not Equal

```python
services[
    services["status"] != "UP"
]
```

finds services whose status is not `UP`.

Important learning:

```text
& → AND
| → OR
```

The individual conditions should be surrounded by parentheses.

This is another concept carried directly from NumPy.

---

# Sorting Data

Rows can be sorted using:

```python
services.sort_values(
    "cpu",
    ascending=False
)
```

This sorts services from highest CPU to lowest CPU.

Mental model:

```text
ascending=True
       ↓
Low → High


ascending=False
       ↓
High → Low
```

By default, `sort_values()` returns a sorted result rather than necessarily changing the original DataFrame.

This connects with the earlier Python distinction between operations that return new data and operations that mutate existing data.

---

# Creating Derived Columns ⭐⭐⭐⭐⭐

Pandas allows new columns to be created from existing columns using vectorized operations.

For example:

```python
services["cpu_fraction"] = (
    services["cpu"] / 100
)
```

This converts:

```text
82
55
91
67
```

into:

```text
0.82
0.55
0.91
0.67
```

No explicit loop is required.

---

## Memory Fraction

Similarly:

```python
services["memory_fraction"] = (
    services["memory"] / 100
)
```

---

## Boolean Derived Columns

Created:

```python
services["high_cpu"] = (
    services["cpu"] > 80
)
```

This produces Boolean values:

```text
True
False
True
False
```

Also created:

```python
services["healthy"] = (
    services["status"] == "UP"
)
```

Mental model:

```text
Existing Column
      ↓
Vectorized Expression
      ↓
Derived Column
```

This is an important bridge toward ML feature engineering.

---

# NumPy Vectorization → Pandas Vectorization

Yesterday:

```python
cpu / 100
```

was performed on a NumPy array.

Today:

```python
services["cpu"] / 100
```

was performed on a Pandas Series.

The same array-oriented idea is being reused.

Conceptually:

```text
NumPy
   ↓
Vectorized numerical operations


Pandas
   ↓
Vectorized operations
on labeled columns
```

This demonstrates how NumPy concepts support Pandas.

---

# Basic Aggregation

Practiced numerical summaries such as:

```python
services["cpu"].mean()
```

```python
services["cpu"].max()
```

```python
services["cpu"].min()
```

```python
services["memory"].mean()
```

These answer questions such as:

```text
What is average CPU?

What is highest CPU?

What is lowest CPU?

What is average memory?
```

The key idea is:

```text
Select column
      ↓
Apply aggregation
      ↓
Get summary
```

---

# `value_counts()`

Practiced:

```python
services[
    "status"
].value_counts()
```

This counts how many times each unique status appears.

For example:

```text
UP      3
DOWN    1
```

This connects directly with the Dictionary counting algorithm implemented manually during Week 1.

Previously:

```text
Empty Dictionary
      ↓
Loop
      ↓
Check existing key
      ↓
Increment
```

Now Pandas provides:

```python
value_counts()
```

Important learning:

> Higher-level libraries often provide concise operations for algorithms that I already understand from Python fundamentals.

---

# `count()` vs `value_counts()`

An important distinction:

```python
df["status"].count()
```

returns the number of non-null values in the column.

It does **not** count each status category separately.

For category frequencies:

```python
df["status"].value_counts()
```

should be used.

Mental model:

```text
count()
   ↓
How many non-missing values?


value_counts()
   ↓
How many of each distinct value?
```

---

# Boolean Mean Trick

A Boolean condition such as:

```python
services["cpu"] > 80
```

produces:

```text
True
False
True
False
```

In numerical calculations:

```text
True  → 1
False → 0
```

Therefore:

```python
(
    services["cpu"] > 80
).mean()
```

calculates the fraction of rows where the condition is `True`.

For example:

```text
2 True values
4 total values

2 / 4
  ↓
0.5
```

which means:

```text
50%
```

This is a useful data-analysis pattern.

---

# Reading CSV Files ⭐⭐⭐⭐⭐

Instead of manually reading and parsing a file, Pandas can load tabular CSV data using:

```python
df = pd.read_csv(
    "data/services.csv"
)
```

Conceptually:

```text
services.csv
     ↓
pd.read_csv()
     ↓
DataFrame
```

This is important because ML datasets commonly enter analysis workflows from files or other tabular data sources.

---

# Writing CSV Files

A DataFrame can be written using:

```python
df.to_csv(
    "output/services-enriched.csv",
    index=False
)
```

The:

```python
index=False
```

option prevents Pandas from writing its DataFrame index as an additional CSV column.

Mental model:

```text
DataFrame
     ↓
to_csv()
     ↓
CSV File
```

---

# Day 2 Main Challenge — Service Dataset

Loaded:

```text
data/services.csv
```

into a DataFrame:

```python
df = pd.read_csv(
    "data/services.csv"
)
```

Then performed:

```text
Load
 ↓
Inspect
 ↓
Filter CPU > 80
 ↓
Sort by CPU
 ↓
Calculate average CPU
 ↓
Inspect status counts
 ↓
Create cpu_fraction
 ↓
Export enriched dataset
```

Finally:

```python
df.to_csv(
    "output/services-enriched.csv",
    index=False
)
```

This represented the first small end-to-end Pandas data-processing workflow.

---

# Pandas vs NumPy

One of the most important conceptual distinctions from today:

## NumPy

Best mental model:

```text
Numerical Array
      ↓
Positions
      ↓
Vectorized Numerical Computation
```

Example:

```python
metrics[:, 0]
```

---

## Pandas

Best mental model:

```text
Tabular Dataset
      ↓
Named Columns
      ↓
Records
      ↓
Data Analysis
```

Example:

```python
df["cpu"]
```

Comparison:

```text
NumPy

metrics[:, 0]

       ↓

"Give me column 0."


Pandas

df["cpu"]

       ↓

"Give me CPU."
```

This was the most important mental-model improvement from the revised Pandas session.

---

# Pandas Selection Mental Model

Instead of trying to memorize many selection APIs, today's simplified approach was:

```text
What do I want?
      ↓

A column?
      ↓
df["column"]


Rows matching a condition?
      ↓
df[condition]


Several columns?
      ↓
df[["column1", "column2"]]
```

`loc` and `iloc` were intentionally not made a major focus today.

They will be introduced naturally when a real data-processing problem requires them.

This avoids memorizing APIs without understanding when they are useful.

---

# 💡 Engineering Learnings

* A Pandas Series represents one-dimensional labeled data.
* A DataFrame represents two-dimensional tabular data.
* DataFrames can be created from existing Python structures such as Dictionaries and Lists.
* Pandas allows columns to be referenced by meaningful names instead of numerical positions.
* `shape` represents rows and columns, while `size` represents the total number of cells.
* Data should be inspected before being transformed.
* Boolean filtering is conceptually similar to NumPy Boolean masking.
* Multiple conditions can be combined using `&` and `|`.
* Vectorized operations can create derived columns without explicit loops.
* Aggregations can summarize individual numerical columns.
* `value_counts()` provides frequency counts for categorical data.
* `count()` and `value_counts()` answer different questions.
* Boolean means can calculate the fraction of records matching a condition.
* `read_csv()` converts external tabular data into a DataFrame.
* `to_csv()` writes processed DataFrames back to external files.
* `index=False` prevents the Pandas index from becoming an unwanted CSV column.
* Pandas should be learned by asking questions about data rather than memorizing methods.

---

# ⚠️ Things That Initially Needed Clarification

The initial Pandas session introduced too many APIs at once.

Methods such as:

```text
loc
iloc
groupby
fillna
dropna
value_counts
sort_values
```

began competing for mental space while NumPy indexing and slicing were still fresh.

The revised approach focused instead on the core mental model:

```text
DataFrame
    ↓
Inspect
    ↓
Select
    ↓
Filter
    ↓
Sort
    ↓
Transform
    ↓
Summarize
```

This made Pandas much clearer.

---

Initially, NumPy-style positional thinking naturally carried over into Pandas.

For example:

```python
metrics[:, 0]
```

was already familiar.

The important transition was understanding:

```python
df["cpu"]
```

as a more meaningful representation of the same intent.

The lesson became:

```text
NumPy
   ↓
Think primarily in positions


Pandas
   ↓
Think primarily in labels and columns
```

---

An important distinction was also identified between:

```python
df["status"].count()
```

and:

```python
df["status"].value_counts()
```

The first counts non-null records.

The second counts each unique category.

---

When exporting a DataFrame using:

```python
df.to_csv(...)
```

the Pandas index would normally also be written.

Using:

```python
index=False
```

prevents the index from becoming an unnecessary column in the output dataset.

---

# 🚀 Production / MLOps Takeaways

Pandas is important to the MLOps journey because ML systems rarely begin with a perfectly prepared numerical matrix.

Real data typically arrives as:

```text
CSV
Database
API
Data Warehouse
Object Storage
Feature Store
Logs
Telemetry
```

and contains meaningful fields.

Conceptually:

```text
Raw Tabular Data
       ↓
Pandas DataFrame
       ↓
Inspect
       ↓
Filter / Transform
       ↓
Clean
       ↓
Features / Labels
       ↓
NumPy / ML Library
       ↓
Model
```

Pandas therefore operates at an important boundary:

```text
Real-world data
       ↓
Pandas
       ↓
ML-ready numerical data
```

The goal is not to become a Pandas specialist.

The goal is to become comfortable enough with tabular data that I can understand:

* What data entered an ML pipeline
* Which records were selected
* Which columns were transformed
* Whether the data appears valid
* What dataset eventually reached the model

This will become important for:

* Training pipelines
* Feature engineering
* Data validation
* Dataset debugging
* Model troubleshooting
* Drift investigation
* ML pipeline observability

---

# 🏆 End of Day Reflection

Today's biggest improvement was not learning more Pandas methods.

It was developing a clearer **mental model for Pandas**.

Yesterday with NumPy:

```text
Numerical Array
      ↓
Position
      ↓
Vectorized Computation
```

Today with Pandas:

```text
Tabular Dataset
      ↓
Named Columns
      ↓
Filtering / Transformation
      ↓
Analysis
```

The most useful realization was that NumPy and Pandas are not competing concepts.

They operate at different levels:

```text
NumPy
   ↓
Numerical computation


Pandas
   ↓
Structured tabular data
```

Boolean filtering and vectorization also showed that yesterday's NumPy knowledge is already transferring into Pandas.

The simplified workflow that I want to retain is:

```text
LOAD
 ↓
INSPECT
 ↓
SELECT
 ↓
FILTER
 ↓
SORT
 ↓
TRANSFORM
 ↓
SUMMARIZE
 ↓
EXPORT
```

From today onwards, rather than asking:

> **"Which Pandas method am I supposed to remember?"**

I'll first ask:

> **"What question am I trying to answer about this dataset?"**

Then I'll choose the operation that answers that question.

This should make Pandas easier to learn naturally as it continues appearing throughout the ML and MLOps journey.

**Pandas Mental Model: ✅**

**Series & DataFrame Fundamentals: Theory ✅ | Hands-on ✅**

**Selection & Boolean Filtering: Theory ✅ | Hands-on ✅**

**Vectorized Transformations: Fundamentals ✅ | Hands-on ✅**

**CSV Processing: Fundamentals ✅ | Hands-on ✅**

**Next: Week 2 — Day 3: Pandas Data Cleaning + EDA ⭐⭐⭐⭐⭐**
