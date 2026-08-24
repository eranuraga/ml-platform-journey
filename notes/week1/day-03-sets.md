# 📒 Day 3 Notes - Python Sets (17-Aug-2026)

## 🎯 Day Objective

Understand Python Sets and become comfortable working with unique values, membership checks, deduplication, and set operations such as union, intersection, and difference.

The focus was to understand how Sets can be useful for SRE-oriented problems such as identifying unique services, correlating recently deployed services with alerts, and comparing groups of operational data.

---

# 📚 Concepts Learned

## Sets

A Set is a collection of **unique values**.

Example:

```python id="9u0c6v"
services = {
    "payment-api",
    "orders-api",
    "inventory-api"
}
```

Unlike Lists, Sets do not allow duplicate values.

For example:

```python id="w9o6q7"
services = {
    "payment-api",
    "orders-api",
    "payment-api"
}
```

results in a Set containing only:

```text id="a1x4yt"
payment-api
orders-api
```

Important learning:

> Sets are useful when uniqueness matters and when I need to perform membership or comparison operations.

---

## Creating an Empty Set

An empty Set cannot be created using:

```python id="a0n4r6"
services = {}
```

because:

```python id="c4g3s7"
{}
```

creates an empty Dictionary.

An empty Set must be created using:

```python id="7v1p0a"
services = set()
```

This is an important Python-specific distinction.

---

## Adding Values

Values can be added to a Set using:

```python id="w4p6t2"
services.add("billing-api")
```

Adding the same value more than once does not create duplicates:

```python id="7f2qv1"
services.add("billing-api")
services.add("billing-api")
```

The Set still contains only one `"billing-api"` entry.

This demonstrates the unique-value property of Sets.

---

## Removing Values

A value can be removed using:

```python id="f9v3w8"
services.remove("billing-api")
```

`remove()` expects the value to exist.

A safer alternative is:

```python id="e1c5h2"
services.discard("billing-api")
```

The difference is:

```text id="a5kq7b"
remove()
    ↓
Value must exist
    ↓
Otherwise raises KeyError


discard()
    ↓
Value may or may not exist
    ↓
No error if missing
```

This distinction is useful when processing data where the existence of an element cannot always be guaranteed.

---

## Set Membership

Sets are particularly useful for checking whether a value exists.

Example:

```python id="k6m9s2"
critical_services = {
    "payment-api",
    "inventory-api"
}
```

Membership can be checked using:

```python id="4g7d1m"
print("payment-api" in critical_services)
```

which returns:

```text id="5e2j8c"
True
```

while:

```python id="9s3n7k"
print("orders-api" in critical_services)
```

returns:

```text id="r2v6y5"
False
```

This makes Sets useful for questions such as:

> Is this service part of the critical-service group?

> Has this service already been processed?

> Is this service present in the affected-service set?

---

## Deduplicating a List Using a Set

A common use case is converting a List into a Set to remove duplicates.

Given:

```python id="2d7n4q"
affected_services = [
    "payment-api",
    "orders-api",
    "payment-api",
    "inventory-api",
    "orders-api",
    "payment-api"
]
```

A Set can be created:

```python id="5p8x1r"
affected_set = set(affected_services)
```

This produces a collection containing only unique services.

The Set can be converted back to a List:

```python id="m6v9z2"
affected_list = list(affected_set)
```

Important learning:

> Sets are useful for deduplication, but converting a List to a Set does not preserve the original ordering in the same way a List does.

Therefore, a Set should not be chosen when the original order is important.

---

# Set Union

Union combines all unique values from two Sets.

Given:

```python id="c8k2q4"
alerting_services = {
    "payment-api",
    "orders-api",
    "inventory-api"
}

recently_deployed = {
    "payment-api",
    "billing-api",
    "inventory-api"
}
```

Union can be performed using:

```python id="n7v4m2"
alerting_services | recently_deployed
```

or:

```python id="p3h8s6"
alerting_services.union(recently_deployed)
```

The result contains every unique service appearing in either Set.

Mental model:

```text id="w2f5j9"
A ∪ B

Everything in A
+
Everything in B
```

For the example:

```text id="c6z1r8"
payment-api
orders-api
inventory-api
billing-api
```

---

# Set Intersection

Intersection returns values that are present in **both Sets**.

Using:

```python id="g9x4q1"
alerting_services.intersection(recently_deployed)
```

or:

```python id="u5m7n3"
alerting_services & recently_deployed
```

the result is:

```text id="p1r8c6"
payment-api
inventory-api
```

because those services are:

* Alerting
* AND recently deployed

This creates a simple incident-correlation rule:

```text id="w8j2v5"
Alerting Services
       ∩
Recently Deployed
       ↓
Potential deployment-related incidents
```

This is an important conceptual connection to the future SRE Copilot.

---

# Set Difference

Difference returns values that exist in the first Set but not in the second.

Example:

```python id="m4z7p2"
alerting_services - recently_deployed
```

means:

> Services that are alerting but were not recently deployed.

The reverse:

```python id="k8q5n1"
recently_deployed - alerting_services
```

means:

> Services that were recently deployed but are not currently alerting.

Important learning:

> Set difference is directional.

Therefore:

```text id="f7c2m9"
A - B
```

is not the same as:

```text id="d3h6r8"
B - A
```

The meaning depends on which Set is on the left side.

---

# Set Intersection with Critical Services

Sets can also be used to identify overlap between operational categories.

Example:

```python id="v9s4k2"
critical_services = {
    "payment-api",
    "inventory-api"
}
```

Then:

```python id="q3m7x5"
critical_services.intersection(recently_deployed)
```

identifies services that are both:

* Critical
* Recently deployed

Similarly:

```python id="j6r1w8"
critical_services.intersection(alerting_services)
```

identifies services that are:

* Critical
* Alerting

This allows multiple operational conditions to be correlated using Set operations.

---

# Subset & Superset

Learned how to check whether all elements of one Set exist inside another.

Example:

```python id="b8k3n6"
critical = {
    "payment-api",
    "inventory-api"
}

all_services = {
    "payment-api",
    "inventory-api",
    "orders-api",
    "billing-api"
}
```

Check:

```python id="x7m2q4"
critical.issubset(all_services)
```

and:

```python id="r5v9k1"
all_services.issuperset(critical)
```

Both express the same relationship from opposite directions.

Mental model:

```text id="q8f4m2"
critical ⊆ all_services

therefore:

all_services ⊇ critical
```

This is useful for reasoning about whether one operational group is completely contained within another.

---

# Combining Lists, Dictionaries and Sets

One of the most useful exercises was combining the collection types learned so far.

Given:

```python id="t3w7m9"
services = [
    {
        "name": "payment-api",
        "status": "UP",
        "cpu": 92
    },
    {
        "name": "orders-api",
        "status": "UP",
        "cpu": 55
    },
    {
        "name": "inventory-api",
        "status": "DOWN",
        "cpu": 88
    },
    {
        "name": "billing-api",
        "status": "UP",
        "cpu": 40
    }
]
```

and:

```python id="v6p2r8"
recently_deployed = {
    "payment-api",
    "inventory-api"
}
```

the service data can be processed using Lists and Dictionaries, while Set operations can be used for correlation.

For example:

```text id="h5q9m1"
Service Data
     ↓
Find High CPU Services
     ↓
Convert to Set
     ↓
Intersect with Recently Deployed
     ↓
Potential Deployment-Related Issue
```

This demonstrates how different Python collection types can work together to solve a practical engineering problem.

---

# SRE Incident Correlation

The final exercise combined:

* List of Dictionaries
* Dictionary lookups
* Conditions
* Lists
* Sets
* Set intersection

The goal was to identify services that were both operationally unhealthy and recently deployed.

Conceptually:

```text id="n4t8v2"
Service Data
    │
    ├── Status = DOWN
    │
    ├── CPU > 80
    │
    └── Recently Deployed
            │
            ▼
      Correlation
            │
            ▼
Potential Incident
```

For example:

```text id="x3m7p1"
DOWN:
inventory-api

HIGH CPU:
payment-api
inventory-api

RECENTLY DEPLOYED:
payment-api
inventory-api
```

The overlap identifies:

```text id="z6q2h8"
inventory-api
```

as a potential deployment-related incident.

Important learning:

> The Set operation is only part of the solution. The important engineering skill is translating operational conditions into explicit data-processing rules.

---

# 💡 Engineering Learnings

* Sets are useful when uniqueness matters.
* Sets are particularly useful for membership checks and comparing groups of data.
* Converting a List to a Set is a simple way to remove duplicates.
* Sets do not provide List-style positional indexing.
* `remove()` raises an error when the value does not exist, while `discard()` safely ignores a missing value.
* Union combines unique values from multiple Sets.
* Intersection identifies values shared by multiple Sets.
* Difference identifies values unique to the first Set.
* Set difference is directional.
* Subsets and supersets describe containment relationships between Sets.
* Sets can work together with Lists and Dictionaries to build useful operational data-processing logic.
* Choosing the appropriate collection type is part of modeling the problem correctly.

---

# ⚠️ Mistakes I Made

Initially used:

```python id="b5x7m3"
services = {}
```

when thinking about an empty Set.

Learned that `{}` creates an empty Dictionary.

The correct way to create an empty Set is:

```python id="q8v2n5"
services = set()
```

---

Initially used a Set operation without storing or printing the result:

```python id="j4m8r1"
alerting_services.union(recently_deployed)
```

The operation returns a new Set, but the result is discarded if it is not assigned or printed.

For example:

```python id="p6x3k9"
all_services = alerting_services.union(
    recently_deployed
)
```

or:

```python id="n2w7v4"
print(
    alerting_services.union(
        recently_deployed
    )
)
```

---

During the final incident-correlation exercise, initially used the DOWN services directly as the final incident result.

The result happened to be correct for the given data, but the actual rule should combine all relevant conditions.

The intended logic is:

```text id="r7m3q5"
DOWN
  ∩
HIGH CPU
  ∩
RECENTLY DEPLOYED
```

This reinforced an important lesson:

> Getting the expected answer is not enough; the implemented logic should represent the actual business or operational rule.

---

# 🚀 Production Takeaways

Sets can be extremely useful in SRE and Platform Engineering when comparing groups of operational entities.

Examples:

```text id="y4q8m2"
Alerting Services
       ∩
Recently Deployed Services
       ↓
Potential Deployment-Related Incidents
```

Other examples include:

```text id="t6p3k9"
Affected Services
       ∩
Critical Services
       ↓
High-Priority Incidents
```

and:

```text id="w2n7r4"
Recently Changed Services
       -
Currently Alerting Services
       ↓
Services Changed Without Current Alerts
```

This type of set-based correlation is a simple foundation for the kind of operational reasoning an SRE Copilot could eventually perform using data from:

* Prometheus
* Kubernetes
* Deployment systems
* Incident management systems
* Logs
* Cloud APIs

The Set operations themselves are simple, but combining multiple data sources and defining the correct correlation rules is where the real engineering value appears.

---

# 🏆 End of Day Reflection

Today's biggest takeaway was understanding that Sets are not simply Lists without duplicates.

They solve a different class of problems.

Sets are particularly useful when I need to answer questions such as:

> "Is this value present?"

> "Which values exist in both groups?"

> "Which values are unique to one group?"

> "Which services are affected by multiple conditions?"

The incident-correlation exercise was especially useful because it connected the Python fundamentals from the last few days:

```text id="c9f4m1"
Lists
  +
Dictionaries
  +
Conditions
  +
Sets
  ↓
Operational Correlation
```

This also gave me a first small glimpse of how the future SRE Copilot could reason over different sources of operational data.

From today onwards, when working with collections I'll consciously ask:

> **"Do I need ordering, named attributes, uniqueness, or comparison between groups?"**

That question will help me choose between Lists, Tuples, Dictionaries, and Sets more intentionally.

**Sets: Theory ✅ | Hands-on ✅**
