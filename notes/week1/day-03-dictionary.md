# 📒 Day 3 Notes - Python Dictionaries (17-Aug-2026)

## 🎯 Day Objective

Understand Python Dictionaries and become comfortable creating, accessing, modifying, deleting, iterating over, and analyzing structured key-value data.

The focus was to understand Dictionaries through SRE-oriented examples such as services, metrics, incidents, and service health information.

---

# 📚 Concepts Learned

## Dictionaries

A Dictionary stores data as **key-value pairs**.

Example:

```python
service = {
    "name": "payment-api",
    "environment": "production",
    "status": "UP",
    "cpu": 82,
    "memory": 71
}
```

A Dictionary allows data to be accessed using meaningful keys rather than numeric indexes.

For example:

```python
service["cpu"]
```

is more descriptive than:

```python
service[3]
```

when working with structured service information.

---

## Creating Dictionaries

A Dictionary can contain multiple key-value pairs and values can be of different types.

```python
service = {
    "name": "payment-api",
    "replicas": 4,
    "healthy": True,
    "owner": None
}
```

An empty Dictionary can be created using:

```python
service = {}
```

The type can be verified using:

```python
print(type(service))
```

The number of key-value pairs can be checked using:

```python
print(len(service))
```

Important learning:

> `len(dictionary)` returns the number of key-value pairs in the Dictionary.

---

## Dictionary Keys Must Be Unique

When the same key is defined more than once:

```python
service = {
    "name": "payment-api",
    "cpu": 82,
    "cpu": 95
}
```

the later value replaces the earlier value.

Therefore, a Dictionary cannot maintain multiple separate values under the exact same key.

---

## Accessing Dictionary Values

Values can be accessed using their keys:

```python
print(service["name"])
print(service["cpu"])
print(service["status"])
```

The key acts as the identifier for retrieving the associated value.

---

## Handling Missing Keys

Trying to access a key that doesn't exist:

```python
print(service["region"])
```

raises:

```text
KeyError
```

This happens because the requested key does not exist in the Dictionary.

---

## Using `get()`

Instead of directly accessing a potentially missing key:

```python
service["region"]
```

the `get()` method can be used:

```python
service.get("region")
```

This returns:

```text
None
```

when the key does not exist.

A default value can also be provided:

```python
service.get("region", "unknown")
```

which returns:

```text
unknown
```

Mental model:

```text
dictionary["missing"]
        ↓
     KeyError

dictionary.get("missing")
        ↓
       None

dictionary.get("missing", default)
        ↓
     default
```

This is particularly useful when processing external or optional data.

---

## Adding Dictionary Entries

A new key-value pair can be added using:

```python
service["region"] = "ap-south-1"
```

If the key does not already exist, a new entry is created.

---

## Updating Dictionary Values

The same syntax can update an existing value:

```python
service["cpu"] = 90
service["replicas"] = 6
```

The behavior depends on whether the key already exists:

```text
Key does not exist
        ↓
       ADD

Key already exists
        ↓
      UPDATE
```

---

## Using `update()`

Multiple values can be updated at once using `update()`:

```python
service.update({
    "cpu": 90,
    "memory": 80,
    "status": "DEGRADED"
})
```

This is useful when several key-value pairs need to be changed together.

---

## Removing Dictionary Entries

A Dictionary entry can be removed using `del`:

```python
del service["region"]
```

A key can also be removed using `pop()`:

```python
removed_status = service.pop("status")
```

Unlike `del`, `pop()` returns the removed value.

This is similar to the distinction learned while working with Lists.

---

## Iterating Through Dictionaries

Dictionary keys can be iterated using:

```python
for key in service.keys():
    print(key)
```

Dictionary values can be iterated using:

```python
for value in service.values():
    print(value)
```

The most useful approach when both key and value are required is:

```python
for key, value in service.items():
    print(key, value)
```

Tuple unpacking is being used here:

```text
(key, value)
     ↓
key variable + value variable
```

This connected today's Dictionary learning with Tuple unpacking from the previous session.

---

## Dictionary Membership

Using:

```python
"cpu" in service
```

checks whether `"cpu"` exists as a **key** in the Dictionary.

For example:

```python
print("cpu" in service)
```

returns:

```text
True
```

But:

```python
print(82 in service)
```

checks the keys, not the values, so it returns:

```text
False
```

To check values:

```python
print(82 in service.values())
```

This distinction is important when processing Dictionary data.

---

## Nested Dictionaries

Dictionaries can contain other Dictionaries.

Example:

```python
service = {
    "name": "payment-api",
    "environment": "production",
    "metrics": {
        "cpu": 82,
        "memory": 71,
        "error_rate": 0.03
    }
}
```

Nested values can be accessed using multiple keys:

```python
service["metrics"]["cpu"]
```

The value can also be modified:

```python
service["metrics"]["cpu"] = 95
```

and new nested values can be added:

```python
service["metrics"]["latency"] = 240
```

Mental model:

```text
service
│
├── name
├── environment
│
└── metrics
      │
      ├── cpu
      ├── memory
      ├── error_rate
      └── latency
```

---

## Lists of Dictionaries

A very important structure for real-world data is a List containing Dictionaries.

Example:

```python
services = [
    {
        "name": "payment-api",
        "cpu": 82,
        "status": "UP"
    },
    {
        "name": "orders-api",
        "cpu": 45,
        "status": "DOWN"
    }
]
```

Compared with:

```python
services = [
    ["payment-api", 82, "UP"],
    ["orders-api", 45, "DOWN"]
]
```

the Dictionary-based structure is more descriptive.

Instead of:

```python
service[1]
```

we can use:

```python
service["cpu"]
```

This makes the code communicate intent more clearly.

This structure is especially common when working with:

* JSON
* REST APIs
* Kubernetes APIs
* Cloud APIs
* Monitoring systems
* ML metadata

---

## Dictionary Counting Pattern

One of the most important hands-on patterns learned today was using a Dictionary to count occurrences.

Given:

```python
incidents = [
    {"service": "payment-api", "severity": "P1"},
    {"service": "orders-api", "severity": "P2"},
    {"service": "payment-api", "severity": "P1"},
    {"service": "inventory-api", "severity": "P3"},
    {"service": "orders-api", "severity": "P1"},
    {"service": "payment-api", "severity": "P2"}
]
```

a Dictionary can be built to count incidents by service:

```python
count = {}

for incident in incidents:
    service = incident["service"]

    if service in count:
        count[service] += 1
    else:
        count[service] = 1
```

Result:

```python
{
    "payment-api": 3,
    "orders-api": 2,
    "inventory-api": 1
}
```

The same pattern can be reused for counting incidents by severity:

```python
count_severity = {}

for incident in incidents:
    severity = incident["severity"]

    if severity in count_severity:
        count_severity[severity] += 1
    else:
        count_severity[severity] = 1
```

Result:

```python
{
    "P1": 3,
    "P2": 2,
    "P3": 1
}
```

Important learning:

> The algorithm remains the same; only the field being counted changes.

This is the beginning of recognizing reusable programming patterns instead of solving every problem from scratch.

---

## Building a Small Interactive Dictionary Program

Also practiced a simple translation program using a Dictionary:

```python
sample_dict = {
    "mouth": "Mund",
    "finger": "Finger",
    "leg": "Bein",
    "hand": "Hand",
    "face": "Gesicht",
    "nose": "Nase"
}
```

The program uses:

* `input()`
* `while`
* `if / elif / else`
* Dictionary membership
* Dictionary lookup
* `break`

Example:

```python
while True:
    word = input("Enter a word in English or EXIT: ")

    if word == "EXIT":
        print("Bye!")
        break
    elif word in sample_dict:
        print("Translation:", sample_dict[word])
    else:
        print("No match!")
```

This helped connect Dictionary operations with control flow and interactive programs.

---

# 💡 Engineering Learnings

* Dictionaries are useful when data has meaningful names associated with values.
* Keys provide a more descriptive way to access structured data than numeric indexes.
* Dictionary keys must be unique.
* `dictionary["key"]` raises `KeyError` when the key doesn't exist.
* `.get()` provides a safer way to access optional keys.
* `dictionary.get(key, default)` allows a default value when data is missing.
* The same syntax can be used to add or update Dictionary values.
* `.items()` provides key/value pairs and naturally connects with Tuple unpacking.
* Dictionary membership using `in` checks keys by default.
* Nested Dictionaries allow more complex structured data to be modeled.
* Lists of Dictionaries are a natural representation of records returned by APIs.
* Dictionary counting is a reusable algorithmic pattern.
* The same counting pattern can be applied to different fields such as service, severity, region, status, or environment.
* Good data structures make code easier to understand and maintain.

---

# ⚠️ Mistakes I Made

Initially created an empty List for UP service names:

```python
up_service_name = []
```

but forgot to append the service name when the service status was `UP`.

The missing operation was:

```python
up_service_name.append(service["name"])
```

This reinforced the importance of checking that every variable being reported is actually populated during processing.

---

Used variable names such as:

```python
values
```

inside loops where each iteration contained only a single value.

A more precise name is:

```python
value
```

because the variable represents one value rather than a collection.

---

Initially left the highest CPU service calculation incomplete.

The pattern needed was similar to the maximum-latency algorithm learned while working with Lists:

```text
Start with first service
        ↓
Compare CPU
        ↓
Is current CPU higher?
        ↓
Update highest service
        ↓
Continue
```

This reinforces that the same algorithmic patterns can be applied to different data structures.

---

# 🚀 Production Takeaways

In production systems, data commonly arrives as structured key-value information.

Examples:

```text
REST API
   ↓
JSON
   ↓
List of Dictionaries

Kubernetes API
   ↓
Resources
   ↓
Nested Dictionaries

Monitoring System
   ↓
Metrics
   ↓
Structured Data

ML Pipeline
   ↓
Experiment Metadata
   ↓
Dictionaries
```

Being comfortable with Dictionaries allows me to:

```text
Receive structured data
        ↓
Access meaningful fields
        ↓
Validate
        ↓
Modify / Transform
        ↓
Aggregate
        ↓
Generate useful information
```

This will become increasingly important when working with APIs, Kubernetes automation, ML metadata, MLOps pipelines, and the SRE Copilot.

---

# 🏆 End of Day Reflection

Today's biggest takeaway was understanding that **Dictionaries are not just another Python collection—they are a way of modeling structured information using meaningful names.**

The transition from:

```python
["payment-api", 82, "UP"]
```

to:

```python
{
    "name": "payment-api",
    "cpu": 82,
    "status": "UP"
}
```

made it clear why Dictionaries are so useful for real-world engineering data.

I also learned how:

```text
Dictionary
    ↓
Nested data
    ↓
List of Dictionaries
    ↓
Counting / Aggregation
```

creates a foundation for processing the kind of structured data I will encounter in APIs, Kubernetes, observability systems, ML pipelines, and eventually the SRE Copilot.

The Dictionary counting exercise was particularly useful because it showed that once I understand an algorithmic pattern, I can reuse it for different problems by changing the field being analyzed.

From today onwards, when I receive structured data, I'll consciously ask:

> **"What are the entities, what are their attributes, and what data structure represents that relationship most clearly?"**

**Dictionaries: Theory ✅ | Hands-on ✅**
