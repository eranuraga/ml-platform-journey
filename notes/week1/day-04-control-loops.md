# 📒 Day 4 Notes - Python Control Flow & Loops

## 🎯 Day Objective

Understand Python control flow and loops and become comfortable using conditions, Boolean logic, iteration, `range()`, `enumerate()`, `while`, `break`, `continue`, and nested loops.

The focus was to combine the data structures learned during previous days with decision-making and iteration to process SRE-oriented data such as services, incidents, replicas, CPU utilization, error rates, and service health.

---

# 📚 Concepts Learned

## Conditional Control Flow

Conditional statements allow a program to make decisions based on data.

The basic structure is:

```python
if condition:
    # execute something
elif another_condition:
    # execute something else
else:
    # default action
```

Example:

```python
cpu = 87

if cpu > 90:
    print("CRITICAL")
elif cpu > 80:
    print("WARNING")
else:
    print("HEALTHY")
```

For:

```text
CPU = 87
```

the result is:

```text
WARNING
```

Important learning:

> Conditions are evaluated from top to bottom, and Python executes the first matching branch.

---

## Order of Conditions Matters

Given:

```python
if cpu > 90:
    print("CRITICAL")
elif cpu > 80:
    print("WARNING")
```

a CPU value of `95` satisfies both:

```text
95 > 90 → True
95 > 80 → True
```

However, Python executes the first matching condition and does not continue evaluating the remaining `elif` branches.

Therefore:

```text
95 → CRITICAL
```

This means conditions should normally be arranged from the most specific or highest-priority condition to the more general conditions.

---

## Comparison Operators

Practiced comparison operators:

```text
==    Equal to
!=    Not equal to
>     Greater than
<     Less than
>=    Greater than or equal to
<=    Less than or equal to
```

These expressions evaluate to Boolean values:

```text
True
False
```

and can therefore control program execution.

---

## Boolean Logic

Learned how multiple conditions can be combined using:

```text
and
or
not
```

Example:

```python
cpu = 92
error_rate = 0.02

if cpu > 90 and error_rate > 0.05:
    print("CRITICAL")
elif cpu > 90 or error_rate > 0.05:
    print("WARNING")
else:
    print("HEALTHY")
```

For:

```text
CPU > 90             → True
error_rate > 0.05    → False
```

therefore:

```text
True AND False → False
True OR False  → True
```

The service is classified as:

```text
WARNING
```

Important distinction:

```text
AND
 ↓
Every condition must be True

OR
 ↓
At least one condition must be True
```

---

# `for` Loops

A `for` loop is used to iterate through elements of a collection.

Example:

```python
services = [
    "payment-api",
    "orders-api",
    "inventory-api",
    "billing-api"
]

for service in services:
    print(f"Checking {service}")
```

The loop processes one element at a time:

```text
services
   ↓
payment-api
   ↓
orders-api
   ↓
inventory-api
   ↓
billing-api
```

The loop variable `service` references the current element during each iteration.

---

## Using `enumerate()`

When both the element and its position are required, `enumerate()` can be used.

Example:

```python
for number, service in enumerate(
    services,
    start=1
):
    print(f"{number}. {service}")
```

This produces:

```text
1. payment-api
2. orders-api
3. inventory-api
4. billing-api
```

This also reinforces Tuple unpacking:

```python
number, service
```

The values produced by `enumerate()` are unpacked into two variables.

---

# `range()`

`range()` can generate a sequence of integers.

General structure:

```python
range(start, stop, step)
```

The `stop` value is excluded.

For example:

```python
replicas = 5

for replica in range(1, replicas + 1):
    print(f"Starting replica {replica}")
```

produces:

```text
Starting replica 1
Starting replica 2
Starting replica 3
Starting replica 4
Starting replica 5
```

Using:

```python
replicas + 1
```

was necessary because the stop value is excluded.

---

# `while` Loops

A `while` loop continues executing while its condition remains `True`.

Example:

```python
attempt = 1

while attempt < 4:
    print(
        f"Health check attempt {attempt}"
    )
    attempt += 1
```

This produces:

```text
Health check attempt 1
Health check attempt 2
Health check attempt 3
```

Important learning:

> A `while` loop must normally change something related to its condition, otherwise it can become an infinite loop.

In this example:

```python
attempt += 1
```

eventually causes:

```python
attempt < 4
```

to become `False`.

---

## `for` vs `while`

The main mental model:

```text
for
 ↓
Process elements from a collection
or known sequence


while
 ↓
Continue while a condition
remains True
```

For example:

```text
Process every service
        ↓
      for


Retry until successful
        ↓
      while
```

---

# `break`

`break` immediately exits the current loop.

Practiced this by finding the first P1 incident:

```python
incidents = [
    {
        "service": "orders-api",
        "severity": "P3"
    },
    {
        "service": "billing-api",
        "severity": "P2"
    },
    {
        "service": "inventory-api",
        "severity": "P1"
    },
    {
        "service": "payment-api",
        "severity": "P1"
    }
]
```

Processing:

```python
for incident in incidents:
    if incident["severity"] == "P1":
        print(
            f"P1 found: "
            f"{incident['service']}"
        )
        break
```

produces:

```text
P1 found: inventory-api
```

The second P1 incident is never processed because `break` terminates the loop after the first match.

Mental model:

```text
Loop
 ↓
Condition met?
 ↓
break
 ↓
EXIT ENTIRE LOOP
```

---

# `continue`

`continue` behaves differently from `break`.

It skips the remaining code for the **current iteration** and moves to the next iteration.

Example:

```python
services = [
    {
        "name": "payment-api",
        "enabled": True
    },
    {
        "name": "orders-api",
        "enabled": False
    },
    {
        "name": "inventory-api",
        "enabled": True
    }
]
```

Using:

```python
for service in services:
    if service["enabled"] == False:
        continue

    print(
        f"Checking {service['name']}"
    )
```

produces:

```text
Checking payment-api
Checking inventory-api
```

`orders-api` is skipped.

Mental model:

```text
continue
    ↓
Skip CURRENT iteration
    ↓
Continue with next item


break
    ↓
Stop ENTIRE loop
```

---

## Understanding `break` vs `continue`

Practiced using:

```python
services = [
    "a",
    "b",
    "c",
    "d"
]

for service in services:
    if service == "b":
        continue

    if service == "d":
        break

    print(service)
```

The output is:

```text
a
c
```

Reasoning:

```text
a
 ↓
No continue
No break
 ↓
PRINT


b
 ↓
continue
 ↓
SKIP


c
 ↓
No continue
No break
 ↓
PRINT


d
 ↓
break
 ↓
STOP LOOP
```

This helped clearly establish the difference between the two statements.

---

# Nested Loops

A loop can exist inside another loop.

Example:

```python
services = {
    "payment-api": [
        "pod-1",
        "pod-2"
    ],
    "orders-api": [
        "pod-1",
        "pod-2",
        "pod-3"
    ],
    "inventory-api": [
        "pod-1"
    ]
}
```

Processing:

```python
for service, pods in services.items():
    print(service)

    for pod in pods:
        print(pod)
```

The outer loop processes each service.

The inner loop processes each pod belonging to that service.

Mental model:

```text
Service
   ↓
List of Pods
   ↓
Each Pod
```

This structure is particularly relevant to infrastructure data where resources naturally contain other resources.

For example:

```text
Cluster
  ↓
Nodes
  ↓
Pods
```

or:

```text
Service
  ↓
Instances
```

---

# Combining Lists, Dictionaries, Loops & Conditions

The exercises combined concepts learned throughout the previous days.

Example:

```python
services = [
    {
        "name": "payment-api",
        "cpu": 92,
        "status": "UP"
    },
    {
        "name": "orders-api",
        "cpu": 55,
        "status": "UP"
    },
    {
        "name": "inventory-api",
        "cpu": 88,
        "status": "DOWN"
    }
]
```

Each Dictionary represents a service.

The List represents the collection of services.

A loop processes every service.

Conditions classify the service.

Conceptually:

```text
List
 ↓
Dictionary
 ↓
for loop
 ↓
Read attributes
 ↓
if / elif / else
 ↓
Classification
```

This is becoming a common data-processing pattern.

---

# SRE Health Checker

The main challenge was building a small service health classifier.

The input data contained:

```text
Service Name
CPU
Error Rate
Status
```

The rules were:

```text
Status == DOWN
        ↓
       DOWN

Otherwise:

CPU > 90
AND
Error Rate > 5%
        ↓
     CRITICAL

Otherwise:

CPU > 80
OR
Error Rate > 5%
        ↓
     WARNING

Otherwise
        ↓
     HEALTHY
```

The implementation processed:

```python
services = [
    {
        "name": "payment-api",
        "cpu": 92,
        "error_rate": 0.07,
        "status": "UP"
    },
    {
        "name": "orders-api",
        "cpu": 55,
        "error_rate": 0.01,
        "status": "UP"
    },
    {
        "name": "inventory-api",
        "cpu": 88,
        "error_rate": 0.03,
        "status": "DOWN"
    },
    {
        "name": "billing-api",
        "cpu": 72,
        "error_rate": 0.06,
        "status": "UP"
    }
]
```

and classified each service.

Expected result:

```text
payment-api → CRITICAL
orders-api → HEALTHY
inventory-api → DOWN
billing-api → WARNING
```

---

# Conditional Priority

An important learning from the health checker was that the order of conditions represents priority.

For example:

```python
if service["status"] == "DOWN":
```

was checked before CPU or error-rate conditions.

Therefore:

```text
inventory-api

status = DOWN
cpu = 88
```

is classified as:

```text
DOWN
```

rather than:

```text
WARNING
```

even though CPU is greater than `80`.

This demonstrates that:

> Correct conditions alone are not enough; their evaluation order must also represent the intended operational rules.

---

# Categorizing Results

Instead of only printing service health, the results were also stored:

```python
down_service = []
critical_service = []
warning_service = []
healthy_service = []
```

During processing:

```python
if service["status"] == "DOWN":
    down_service.append(
        service["name"]
    )
```

Similar logic was used for the other categories.

This allowed additional calculations after processing:

```text
Total Services
Healthy Count
Warning Count
Critical Count
Down Count
```

Important learning:

> Processing data and retaining the result enables further aggregation and reporting.

---

# Counters vs Collections

Previously I used integer counters:

```python
healthy_count = 0
```

Today I also stored actual service names:

```python
healthy_service = []
```

Then:

```python
len(healthy_service)
```

can provide the count.

The List additionally preserves information about **which services** belong to that category.

This illustrates an engineering decision:

```text
Need only count?
      ↓
Integer counter


Need count + identities?
      ↓
Collection
```

---

# 💡 Engineering Learnings

* Control flow allows programs to make decisions based on data.
* Conditions are evaluated from top to bottom.
* The order of `if / elif / else` conditions can affect the result.
* `and` requires all participating conditions to be True.
* `or` requires at least one participating condition to be True.
* `for` loops are useful for processing collections.
* `while` loops are useful when processing should continue while a condition remains True.
* `range()` generates integer sequences and excludes its stop value.
* `enumerate()` provides both position and value while iterating.
* `break` exits the entire current loop.
* `continue` skips only the current iteration.
* Nested loops allow hierarchical data to be processed.
* Lists, Dictionaries, conditions, and loops naturally work together for structured data processing.
* Correct business or operational rules require both correct conditions and correct condition ordering.
* Retaining classification results allows later aggregation and reporting.
* Code should represent the actual operational rule rather than merely producing the expected output.

---

# ⚠️ Mistakes I Made

While using f-strings with Dictionary access, initially used double quotes both for the outer f-string and Dictionary key:

```python
print(
    f"P1 found: {incident["service"]}"
)
```

This creates a quoting conflict.

Using different quote types solves the problem:

```python
print(
    f"P1 found: {incident['service']}"
)
```

This reinforced the importance of understanding string delimiters when embedding expressions inside f-strings.

---

In the final health-check summary, initially printed:

```python
Healthy services count
```

twice.

The final line should have reported:

```python
Down services count
```

This was a small output bug rather than a logic error, but reinforced the need to verify final results rather than assuming the reporting code is correct.

---

While simulating a health-check retry with a `while` loop, the program printed success after the loop rather than making success itself control the loop.

This works for a simple simulation, but real retry logic should conceptually behave like:

```text
Perform Check
      ↓
Successful?
   /      \
 YES       NO
  ↓         ↓
Stop      Retry
```

This will become more relevant when building functions and real application logic.

---

# 🚀 Production Takeaways

Control flow and loops form the basic execution logic behind many SRE automation tasks.

For example:

```text
Kubernetes API
      ↓
List of Pods
      ↓
Loop
      ↓
Check Pod Status
      ↓
Healthy?
   /      \
 YES       NO
  ↓         ↓
Continue   Alert
```

Another example:

```text
Incident Feed
      ↓
Loop Through Incidents
      ↓
Severity == P1?
      ↓
Take Priority Action
```

Retry logic can conceptually use:

```text
Request
   ↓
Success?
 /      \
YES      NO
 ↓        ↓
Done     Retry
           ↓
       Max attempts?
```

These concepts will later appear in:

* API processing
* Kubernetes automation
* Cloud automation
* ML pipelines
* Model validation
* Health checks
* Retry mechanisms
* SRE Copilot tool execution
* Incident processing

---

# 🏆 End of Day Reflection

Today's biggest takeaway wasn't simply learning how to write `for` and `while` loops.

It was understanding how **data structures + iteration + decision-making combine to create actual program behavior**.

The previous days taught me how to represent data using:

```text
Lists
Tuples
Dictionaries
Sets
```

Today I started making decisions and performing actions using that data:

```text
Data
 ↓
Iterate
 ↓
Evaluate
 ↓
Decide
 ↓
Classify
 ↓
Store Result
 ↓
Aggregate
```

The SRE Health Checker was particularly useful because it combined concepts from several previous days into one small program.

I also learned an important engineering lesson:

> **Producing the correct output is not enough. The conditions and execution flow should correctly represent the intended operational rule.**

The distinction between `break` and `continue` also became clear:

```text
continue
   ↓
Skip current iteration

break
   ↓
Stop entire loop
```

From today onwards, when writing automation logic I'll consciously ask:

> **"What data am I processing, what decision am I making, when should processing continue, and when should it stop?"**

This will become increasingly important as I move from small Python scripts toward functions, APIs, automation, ML pipelines, and production AI Platform Engineering.

**Control Flow: Theory ✅ | Hands-on ✅**

**Loops: Theory ✅ | Hands-on ✅**

**Next: Functions ⭐**
