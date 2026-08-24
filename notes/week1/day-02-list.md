# 📒 Day 2 Notes (16-Aug-2026)

## 🎯 Day Objective

Understand Python Lists and become comfortable storing, accessing, modifying, sorting, copying, iterating over, filtering, and processing collections of data.

The focus was to understand Lists through SRE-oriented examples such as services, incidents, CPU metrics, and response times.

---

# 📚 Concepts Learned

## Lists

A List is an **ordered and mutable collection** that can store multiple values.

Example:

```python
services = [
    "payment-api",
    "orders-api",
    "inventory-api",
    "user-api"
]
```

---

## Indexing & Negative Indexing

Individual List elements can be accessed using their index.

```python
services[0]
```

Python indexes start from `0`.

Negative indexing allows accessing elements starting from the end:

```python
services[-1]
```

Important distinction:

* Indexing returns the object stored at that position.
* Slicing returns another List.

---

## List Slicing

A portion of a List can be extracted using slicing.

```python
services[1:3]
```

The starting index is included while the ending index is excluded.

Slicing can also be used to create another List:

```python
backup_services = services[:]
```

---

## Adding Elements

### `append()`

Adds one object to the end of the List.

```python
services.append("billing-api")
```

If another List is appended, the complete List becomes one nested element.

### `extend()`

Adds each element from another iterable individually.

```python
services.extend(new_services)
```

### `insert()`

Adds an element at a specific index.

```python
services.insert(1, "auth-api")
```

Mental model:

```text
append() → Add ONE object

extend() → Add EACH object

insert() → Add object at a specific position
```

---

## Removing Elements

Learned three different approaches.

### `remove()`

Removes an element using its **value**.

```python
services.remove("inventory-api")
```

### `pop()`

Removes an element using its **index** and returns the removed value.

```python
removed_service = services.pop()
```

Without an index, `pop()` removes the last element.

### `del`

`del` is a Python statement rather than a List method.

```python
del services[2]
```

Mental model:

```text
Know VALUE
    ↓
remove()

Know INDEX
    ↓
del

Know INDEX + need removed value
    ↓
pop()
```

---

## Sorting Lists

`sort()` modifies the existing List:

```python
latencies.sort()
```

Descending:

```python
latencies.sort(reverse=True)
```

`sorted()` creates a new sorted List without modifying the original:

```python
sorted_latencies = sorted(latencies)
```

Important concept learned:

> Some Python operations mutate existing objects while others return new objects.

---

## Copying Lists

This does **not** create an independent copy:

```python
backup_services = primary_services
```

Both variables reference the same underlying List object.

Therefore, modifying the List through one variable will also be visible through the other variable.

A separate List can be created using:

```python
backup_services = primary_services[:]
```

or:

```python
backup_services = primary_services.copy()
```

Important learning:

> Assignment does not necessarily mean copying an object.

---

## Looping Through Lists

Lists can be processed using `for` loops:

```python
for service in services:
    print(service)
```

When both the position and value are required, `enumerate()` can be used:

```python
for index, service in enumerate(services, start=1):
    print(index, service)
```

---

## Filtering Lists

Practiced creating a new List based on a condition.

```python
slow_requests = []

for latency in latencies:
    if latency > 300:
        slow_requests.append(latency)
```

This introduced an important data-processing pattern:

```text
Collection
    ↓
Iterate
    ↓
Evaluate condition
    ↓
Select matching values
    ↓
New collection
```

---

## Nested Lists

Lists can contain other Lists.

Example:

```python
service_metrics = [
    ["payment-api", 82, 71],
    ["orders-api", 45, 60],
    ["inventory-api", 91, 84]
]
```

Each inner List can represent a record containing:

```text
[service_name, cpu, memory]
```

Nested Lists can themselves be iterated over and processed.

---

## Accumulator Pattern

While calculating latency statistics, learned the accumulator pattern:

```python
total_latency = 0

for latency in response_times:
    total_latency += latency
```

Mental model:

```text
Initialize
    ↓
Iterate
    ↓
Update accumulated value
    ↓
Final result
```

Used this pattern to calculate:

* Total latency
* Average latency
* Number of slow requests
* Percentage of slow requests

---

## Finding Maximum & Minimum Values

Implemented maximum and minimum calculations manually instead of using Python's built-in `max()` and `min()` functions.

Example:

```python
max_response_time = response_times[0]

for latency in response_times:
    if latency > max_response_time:
        max_response_time = latency
```

This helped understand the algorithmic logic behind built-in functions.

---

# 💡 Engineering Learnings

* Lists are useful for working with ordered collections of operational data.
* Understand whether an operation modifies an existing object or returns a new object.
* Multiple variables can reference the same Python object.
* Choose descriptive variable names that explain what the data represents.
* Avoid using names of Python built-in functions as variables.
* Filtering, iteration, and accumulation are fundamental data-processing patterns.
* Built-in Python functions become easier to understand when their underlying logic has been implemented manually.
* Code readability matters just as much as producing the correct output.

---

# ⚠️ Mistakes I Made

Initially assumed:

```python
backup_services = primary_services
```

created another independent List.

Instead, both variables referenced the same List object.

A separate List can be created using:

```python
backup_services = primary_services.copy()
```

---

While testing `append()` and `extend()`, I used the same already-modified List for both operations.

For a proper comparison, the original List should be reset before testing each operation independently.

---

Used:

```python
sum = 0
```

for calculating total latency.

This shadows Python's built-in `sum()` function.

Better:

```python
total_latency = 0
```

Also learned that descriptive loop variables such as:

```python
for latency in response_times:
```

are preferable to:

```python
for i in response_times:
```

when the variable represents an actual value rather than an index.

---

# 🚀 Production Takeaways

As an SRE / Platform Engineer, Lists can represent operational data such as:

```text
Kubernetes API
      ↓
List of Pods

Monitoring System
      ↓
List of Metrics

Incident System
      ↓
List of Incidents

Application
      ↓
List of Requests
```

Being comfortable with Lists allows me to:

```text
Receive Data
    ↓
Iterate
    ↓
Filter
    ↓
Analyze
    ↓
Transform
    ↓
Produce Result
```

These same fundamentals will later be used when working with APIs, Pandas, ML pipelines, Kubernetes automation, observability data, and the SRE Copilot.

---

# 🏆 End of Day Reflection

Today's biggest takeaway wasn't memorizing List methods such as `append()`, `remove()`, `pop()`, or `sort()`.

It was understanding how Python lets me **store, reference, mutate, iterate over, filter, and analyze collections of data**.

The exercises involving service health, CPU utilization, incidents, and response times also helped connect basic Python Lists directly to real SRE problems.

From today onwards, I'll consciously think:

> "What data am I storing, how will I access it, and should my operation modify the original collection or create a new one?"

This will become increasingly important as I move from basic Python into APIs, ML data processing, MLOps, and AI Platform Engineering.

**Lists: Theory ✅ | Hands-on ✅**
