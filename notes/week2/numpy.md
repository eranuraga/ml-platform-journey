# 📒 Week 2 - Day 1 Notes — NumPy Fundamentals

## 🎯 Day Objective

Understand the fundamentals of NumPy and learn why numerical arrays are more suitable than regular Python Lists for numerical and ML-oriented data processing.

The focus was to move from manually processing collections using Python loops toward **array-oriented and vectorized numerical operations**.

The exercises used SRE-oriented data such as:

* CPU utilization
* Memory utilization
* Request latency
* Error rates
* Service metrics

This provided the first bridge from general Python programming toward **data processing and ML engineering**.

---

# 📚 Concepts Learned

## What is NumPy?

NumPy is a Python library designed for efficient numerical computing.

The central NumPy object is:

```text
ndarray
```

which means:

> **N-dimensional array**

Example:

```python
import numpy as np

cpu = np.array([
    82,
    55,
    91,
    67
])
```

The type can be checked using:

```python
print(type(cpu))
```

which returns a NumPy:

```text
ndarray
```

---

## Python List vs NumPy Array

One of the first experiments demonstrated an important difference between Python Lists and NumPy arrays.

Python List:

```python
numbers = [1, 2, 3, 4]

print(numbers * 2)
```

produces:

```text
[1, 2, 3, 4, 1, 2, 3, 4]
```

For a List:

```text
* 2
 ↓
Repeat the collection twice
```

With NumPy:

```python
numbers = np.array([
    1,
    2,
    3,
    4
])

print(numbers * 2)
```

produces:

```text
[2 4 6 8]
```

NumPy performs the operation on every element.

Mental model:

```text
Python List
     ↓
General-purpose collection


NumPy Array
     ↓
Numerical array
     ↓
Element-wise numerical operations
```

Important learning:

> NumPy provides an array-oriented model for numerical computation rather than normal Python collection behavior.

---

# Array Shape, Size & Dimensions

Three important NumPy properties were introduced:

```python
array.shape
array.size
array.ndim
```

Given:

```python
cpu = np.array([
    82,
    55,
    91,
    67
])
```

the result is conceptually:

```text
shape → (4,)
size  → 4
ndim  → 1
```

Mental model:

```text
shape
  ↓
How is the data arranged?


size
  ↓
How many total elements exist?


ndim
  ↓
How many dimensions exist?
```

These concepts become increasingly important when working with ML datasets.

---

# One-Dimensional Arrays

Example:

```python
cpu = np.array([
    82,
    55,
    91,
    67
])
```

This is a one-dimensional array.

```python
cpu.ndim
```

returns:

```text
1
```

and:

```python
cpu.shape
```

returns:

```text
(4,)
```

The trailing comma represents a one-dimensional shape containing four elements.

---

# Two-Dimensional Arrays

NumPy arrays can contain multiple dimensions.

Example:

```python
metrics = np.array([
    [82, 71],
    [55, 60],
    [91, 84]
])
```

Conceptually:

```text
               CPU     Memory

payment-api     82       71
orders-api      55       60
inventory-api   91       84
```

The array has:

```text
shape → (3, 2)
ndim  → 2
size  → 6
```

Meaning:

```text
3 rows
2 columns
6 total values
```

This is the beginning of thinking about numerical data as matrices rather than individual Python values.

---

# NumPy Indexing

Individual values can be accessed using:

```python
array[row, column]
```

Given:

```python
metrics = np.array([
    [82, 71],
    [55, 60],
    [91, 84]
])
```

CPU of `payment-api`:

```python
metrics[0, 0]
```

Memory of `payment-api`:

```python
metrics[0, 1]
```

CPU of `inventory-api`:

```python
metrics[2, 0]
```

Important learning:

```text
metrics[row, column]
```

provides a clear mental model for accessing two-dimensional numerical data.

---

# NumPy Slicing

Entire rows or columns can be selected using slicing.

Entire second row:

```python
metrics[1]
```

Entire first column:

```python
metrics[:, 0]
```

Entire second column:

```python
metrics[:, 1]
```

First two rows:

```python
metrics[0:2, :]
```

Mental model:

```text
:
 ↓
All values along that dimension
```

Therefore:

```python
metrics[:, 0]
```

means:

> All rows from column 0.

This becomes very useful when columns represent numerical features.

---

# Vectorized Operations ⭐

One of the most important NumPy concepts learned was **vectorization**.

Given:

```python
latencies = np.array([
    120,
    340,
    89,
    450,
    230
])
```

operations can be applied directly to the complete array.

Addition:

```python
latencies + 10
```

Multiplication:

```python
latencies * 2
```

Division:

```python
latencies / 1000
```

Comparison:

```python
latencies > 300
```

No explicit Python loop is required.

Mental model:

```text
Array
  ↓
Operation
  ↓
Apply operation to elements
  ↓
New Array
```

This is fundamentally different from manually iterating through Python Lists.

---

# Boolean Arrays

A comparison against a NumPy array produces an array of Boolean values.

Example:

```python
latencies > 300
```

can produce:

```text
[False True False True False]
```

Each Boolean corresponds to the element at the same position.

Conceptually:

```text
120 > 300 → False
340 > 300 → True
89  > 300 → False
450 > 300 → True
230 > 300 → False
```

This Boolean array can then be used for filtering.

---

# Boolean Masking ⭐⭐⭐⭐⭐

Boolean masking allows NumPy arrays to be filtered without manually writing loops.

Example:

```python
latencies[
    latencies > 300
]
```

returns only values where the condition is `True`.

For:

```python
latencies = np.array([
    120,
    340,
    89,
    450,
    230
])
```

the result is:

```text
[340 450]
```

Mental model:

```text
Array
  ↓
Condition
  ↓
Boolean Mask
  ↓
Apply Mask
  ↓
Filtered Array
```

This is one of the most important concepts from today's session because the same idea will appear again while working with Pandas.

---

# Combining Boolean Conditions

Multiple NumPy conditions can be combined.

For example:

```python
latencies = np.array([
    120,
    450,
    80,
    600,
    210,
    90,
    510
])
```

To find values between `100` and `300`:

```python
mask = (
    (latencies >= 100)
    &
    (latencies <= 300)
)

filtered_latencies = latencies[mask]
```

Result:

```text
[120 210]
```

Important NumPy distinction:

For element-wise Boolean operations:

```text
&  → AND
|  → OR
~  → NOT
```

rather than Python's normal scalar Boolean operators:

```text
and
or
not
```

---

# Aggregation

NumPy provides built-in functions for summarizing numerical arrays.

Practiced:

```python
np.sum()
np.mean()
np.min()
np.max()
np.std()
```

Given:

```python
latencies = np.array([
    120,
    450,
    80,
    600,
    210,
    90,
    510
])
```

these can calculate:

```text
Total latency
Average latency
Minimum latency
Maximum latency
Standard deviation
```

This connected directly with the manual algorithms written during Week 1.

Previously:

```text
Initialize accumulator
      ↓
Loop
      ↓
Add values
      ↓
Calculate result
```

Now:

```python
np.sum(latencies)
np.mean(latencies)
```

can perform those numerical aggregations directly.

Important learning:

> Understanding the manual algorithm first made NumPy's aggregation functions easier to understand rather than treating them as magic.

---

# Standard Deviation

Practiced:

```python
np.std(latencies)
```

Standard deviation provides information about how spread out the numerical values are around their average.

At the current stage, the focus is simply to understand:

```text
Low standard deviation
        ↓
Values relatively close together


High standard deviation
        ↓
Values more spread out
```

The deeper statistical interpretation will be introduced when required for ML.

---

# Understanding `axis` ⭐⭐⭐⭐⭐

The `axis` argument controls the direction along which an aggregation operates.

Given:

```python
metrics = np.array([
    [82, 71],
    [55, 60],
    [91, 84]
])
```

Without an axis:

```python
np.mean(metrics)
```

NumPy calculates the mean across all elements.

---

## `axis=0`

```python
np.mean(
    metrics,
    axis=0
)
```

operates down the rows and produces one result per column.

Conceptually:

```text
        CPU     Memory
         ↓         ↓
        mean      mean
```

Result represents:

```text
Average CPU
Average Memory
```

Mental model:

```text
axis=0
   ↓
Reduce DOWN the rows
   ↓
One result per column
```

---

## `axis=1`

```python
np.mean(
    metrics,
    axis=1
)
```

operates across the columns and produces one result per row.

Conceptually:

```text
Service 1 → mean(CPU, Memory)
Service 2 → mean(CPU, Memory)
Service 3 → mean(CPU, Memory)
```

Mental model:

```text
axis=1
   ↓
Reduce ACROSS columns
   ↓
One result per row
```

Important learning:

> It is safer to understand the direction of reduction than simply memorizing that `axis=0` means columns and `axis=1` means rows.

---

# Array Creation Utilities

Practiced basic NumPy utilities for creating arrays.

## `np.zeros()`

```python
np.zeros(5)
```

creates an array initialized with zeros.

---

## `np.ones()`

```python
np.ones(5)
```

creates an array initialized with ones.

---

## `np.arange()`

```python
np.arange(
    0,
    10,
    2
)
```

creates evenly spaced values using:

```text
start
stop
step
```

similar to Python's `range()`.

---

## `np.linspace()`

```python
np.linspace(
    0,
    1,
    5
)
```

creates a specified number of evenly spaced values between two boundaries.

Only the basic usage of these utilities was required.

---

# Reshaping Arrays

NumPy can change how existing data is arranged using:

```python
reshape()
```

Example:

```python
data = np.arange(24)
```

Initially:

```text
shape → (24,)
```

The same data can be reshaped:

```python
data.reshape(
    6,
    4
)
```

producing:

```text
shape → (6, 4)
```

or:

```python
data.reshape(
    4,
    6
)
```

producing:

```text
shape → (4, 6)
```

Important learning:

> `reshape()` changes how the data is arranged but does not change the number of elements.

Mental model:

```text
24 elements

6 × 4 = 24  ✅
4 × 6 = 24  ✅
```

A reshape must therefore be compatible with the total number of elements.

---

# Element-Wise vs Matrix Operations

NumPy distinguishes between element-wise operations and matrix operations.

Given:

```python
A = np.array([
    [1, 2],
    [3, 4]
])

B = np.array([
    [5, 6],
    [7, 8]
])
```

Element-wise multiplication:

```python
A * B
```

multiplies corresponding elements.

Matrix multiplication:

```python
A @ B
```

performs matrix multiplication.

Only the distinction was introduced today.

Deeper linear algebra will be learned when it becomes relevant to ML rather than studying it independently.

---

# SRE Metrics Challenge

Practiced NumPy using:

```python
metrics = np.array([
    [82, 71, 0.02],
    [55, 60, 0.01],
    [91, 84, 0.07],
    [67, 52, 0.03],
    [88, 79, 0.06]
])
```

Columns represented:

```text
CPU
Memory
Error Rate
```

Used NumPy to calculate:

* Array shape
* Average CPU
* Average memory
* Highest CPU
* Highest error rate
* CPU values greater than 80
* Error rates greater than 0.05
* Average values by metric
* Normalized CPU values
* Reshaped numerical data

This combined:

```text
2D Arrays
   +
Column Slicing
   +
Boolean Masking
   +
Aggregation
   +
Vectorization
   +
Reshaping
```

---

# Selecting Columns from a 2D Array

Given:

```text
Column 0 → CPU
Column 1 → Memory
Column 2 → Error Rate
```

CPU values can be selected using:

```python
metrics[:, 0]
```

Memory:

```python
metrics[:, 1]
```

Error rate:

```python
metrics[:, 2]
```

This is an important foundation for understanding ML features.

Later:

```text
Rows
 ↓
Samples / observations

Columns
 ↓
Features
```

will become a central ML concept.

---

# Filtering 2D Numerical Data

To identify CPU values greater than `80`:

```python
metrics[
    metrics[:, 0] > 80,
    0
]
```

Conceptually:

```text
Select CPU column
       ↓
CPU > 80
       ↓
Boolean mask
       ↓
Apply mask
       ↓
Matching CPU values
```

The same pattern was used for error rates greater than `0.05`.

---

# Normalization

Practiced a very simple normalization operation:

```python
metrics[:, 0] / 100
```

For CPU percentages such as:

```text
82
55
91
67
88
```

this produces values such as:

```text
0.82
0.55
0.91
0.67
0.88
```

This introduced the general idea of transforming numerical features into another scale.

More formal feature scaling and normalization will be covered later when working with ML preprocessing.

---

# Data Representation Matters

One important observation from the final challenge was that the NumPy array contained:

```text
CPU
Memory
Error Rate
```

but did not contain service names.

Therefore:

```python
metrics[
    metrics[:, 0] > 80,
    0
]
```

can return matching CPU values but cannot directly tell us:

```text
payment-api
inventory-api
...
```

unless labels are stored separately.

This demonstrated an important distinction:

```text
NumPy
   ↓
Excellent numerical representation


But

Real-world tabular data often also needs:
   ↓
Column names
Row labels
Mixed data types
```

This provides the bridge toward **Pandas DataFrames**.

---

# Meaningful Aggregation

Initially used:

```python
np.mean(metrics)
```

for "average metrics across services."

This calculates one average across:

```text
CPU
+
Memory
+
Error Rate
```

Although NumPy allows this mathematically, these values represent different measurements and scales.

A more meaningful calculation is:

```python
np.mean(
    metrics,
    axis=0
)
```

which produces:

```text
Average CPU
Average Memory
Average Error Rate
```

Important engineering learning:

> Just because an operation is mathematically possible does not mean the resulting metric has useful meaning.

The meaning and units of the data must be considered before aggregating it.

---

# 💡 Engineering Learnings

* NumPy arrays are designed for numerical computation.
* `ndarray` is the central NumPy data structure.
* `shape` describes how an array is arranged.
* `size` describes the total number of elements.
* `ndim` describes the number of dimensions.
* NumPy supports multidimensional numerical data naturally.
* Vectorized operations allow calculations without explicit Python loops.
* Comparisons against arrays produce Boolean arrays.
* Boolean masks allow efficient numerical filtering.
* Multiple array conditions can be combined using `&`, `|`, and `~`.
* Aggregation functions summarize numerical data.
* `axis` determines the direction of aggregation.
* `reshape()` changes structure without changing the total number of elements.
* Element-wise multiplication and matrix multiplication are different operations.
* Numerical data can be normalized using vectorized operations.
* Data representation determines what questions can easily be answered.
* Mathematical validity does not automatically imply that an aggregation is meaningful.
* NumPy is a foundation for the numerical operations used by many ML libraries.

---

# ⚠️ Mistakes I Made

While solving the filtering exercise for requests below `100 ms`, initially wrote:

```python
latencies[
    latencies < 300
]
```

instead of:

```python
latencies[
    latencies < 100
]
```

The NumPy syntax was correct, but the implemented condition did not match the actual requirement.

This reinforced an important engineering lesson:

> Correct syntax does not guarantee correct logic.

---

While calculating:

> Average metrics across services

initially used:

```python
np.mean(metrics)
```

This averaged CPU, Memory, and Error Rate into one number.

Although NumPy can perform the calculation, combining measurements with different meanings and scales into a single average is not particularly useful.

The more meaningful operation was:

```python
np.mean(
    metrics,
    axis=0
)
```

which calculates one average for each metric.

---

While filtering:

```python
metrics[
    metrics[:, 0] > 80,
    0
]
```

the result returned CPU values rather than service names.

This was not a NumPy error.

The service names were simply not part of the numerical array.

This highlighted that:

> The structure chosen for storing data determines what information is available during processing.

---

# 🚀 Production / MLOps Takeaways

NumPy is not something I need to study as an isolated mathematical library.

For the MLOps journey, its importance is understanding the **numerical representation of ML data**.

Conceptually:

```text
Raw Data
   ↓
Numerical Representation
   ↓
NumPy Arrays
   ↓
Feature Processing
   ↓
ML Model
```

ML datasets are often conceptually represented as:

```text
ROWS
 ↓
Samples / observations


COLUMNS
 ↓
Features
```

For example:

```text
          CPU   Memory   Error Rate
Service1   82     71        0.02
Service2   55     60        0.01
Service3   91     84        0.07
```

can eventually become:

```text
X
 ↓
Feature Matrix
 ↓
ML Model
```

NumPy concepts such as:

```text
shape
slicing
vectorization
Boolean masking
aggregation
axis
reshape
```

will therefore appear underneath:

* Pandas
* Scikit-learn
* Feature preprocessing
* Model training
* Model inference
* Tensor libraries
* ML pipelines

The goal is not to become a NumPy specialist.

The goal is:

> **Understand numerical arrays well enough that numerical data flowing through an ML system is not a black box.**

---

# 🏆 End of Day Reflection

Today's biggest takeaway was understanding the transition from **general-purpose Python collections to numerical arrays**.

During Week 1, numerical processing often looked like:

```text
List
 ↓
for loop
 ↓
condition
 ↓
append
 ↓
result
```

NumPy introduced another model:

```text
Array
 ↓
Vectorized operation
 ↓
Boolean mask
 ↓
Aggregation
 ↓
Result
```

The biggest concepts that clicked were:

```text
Vectorization
Boolean Masking
2D Slicing
Axis
Reshape
```

Boolean masking was especially useful because it showed how entire numerical datasets can be filtered using conditions without explicitly writing Python loops.

Understanding `axis` also introduced an important way of thinking about multidimensional data:

```text
axis=0
 ↓
Reduce down rows
 ↓
Result per column


axis=1
 ↓
Reduce across columns
 ↓
Result per row
```

Another important learning was that **understanding the data is as important as knowing the NumPy operation**.

NumPy may allow an operation, but I still need to ask:

> **"Does this calculation make sense for the data and units I am working with?"**

From today onwards, when looking at numerical ML data, I'll consciously ask:

> **"What is the shape, what do the rows represent, what do the columns represent, and which dimension am I operating on?"**

This is the first major step from Python engineering toward **ML data engineering**.

**NumPy Fundamentals: Theory ✅ | Hands-on ✅**

**Vectorization & Boolean Masking: Theory ✅ | Hands-on ✅**

**Multidimensional Arrays & Axis: Fundamentals ✅ | Hands-on ✅**

**Next: Pandas — DataFrames, Real Tabular Data & Data Processing ⭐⭐⭐⭐⭐**
