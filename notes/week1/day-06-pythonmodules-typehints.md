# 📒 Day 6 Notes - Python Modules & Type Hints

## 🎯 Day Objective

Understand how Python applications can be organized across multiple files using **Modules and Packages**, and learn how **Type Hints** communicate the expected inputs and outputs of functions.

The focus was to move from writing functions inside one large Python script toward organizing related functionality into separate modules with clear responsibilities.

The exercises used the SRE functions created during previous days, such as service health classification, incident counting, and error-rate calculation.

---

# 📚 Concepts Learned

## Python Modules

A Python file containing Python code is a **module**.

For example:

```text
health.py
```

is a module.

It can contain functions such as:

```python
def classify_service(service):
    ...
```

Similarly:

```text
incidents.py
```

can contain:

```python
def count_incidents(incidents, field):
    ...
```

and:

```text
metrics.py
```

can contain:

```python
def calculate_error_rate(errors, requests):
    ...
```

Important learning:

> Modules allow related functionality to be grouped together instead of keeping the entire application inside one large Python file.

---

## Organizing Functions into Modules

Previously, several functions could exist inside one file:

```text
main.py

classify_service()
count_incidents()
calculate_error_rate()
```

These functions perform different responsibilities.

They can instead be organized as:

```text
app/
│
├── health.py
│     └── classify_service()
│
├── incidents.py
│     └── count_incidents()
│
└── metrics.py
      └── calculate_error_rate()
```

This provides clearer separation of responsibilities.

Mental model:

```text
Health-related logic
        ↓
     health.py

Incident-related logic
        ↓
   incidents.py

Metric-related logic
        ↓
    metrics.py
```

Important learning:

> Modules should generally be organized around related responsibilities rather than simply splitting code because a file has become large.

---

# Importing Modules

A module can be imported using:

```python
import health
```

Functions inside the module can then be accessed using:

```python
health.classify_service(service)
```

Mental model:

```text
health
   ↓
Module

classify_service
   ↓
Function inside module
```

The dot notation accesses something contained inside the imported module.

---

# Importing Specific Functions

Instead of importing the complete module:

```python
import health
```

a specific function can be imported:

```python
from health import classify_service
```

The function can then be called directly:

```python
classify_service(service)
```

Important distinction:

```text
import health
      ↓
health.classify_service()


from health import classify_service
      ↓
classify_service()
```

Both approaches are valid.

The choice depends on which form makes the code easier to understand.

---

# Import Aliases

Modules can be imported using aliases.

Example:

```python
import health as h
```

The module can then be accessed using:

```python
h.classify_service(service)
```

This concept will become especially common when working with data libraries.

For example:

```python
import numpy as np
import pandas as pd
```

Important learning:

> An alias provides another name through which the imported module can be referenced.

Aliases should be used when they improve readability rather than simply making names shorter.

---

# Python Packages

A directory containing related Python modules can be organized as a **package**.

For example:

```text
app/
├── __init__.py
├── health.py
├── incidents.py
└── metrics.py
```

At our current level:

```text
app
 ↓
Package

health.py
 ↓
Module

incidents.py
 ↓
Module

metrics.py
 ↓
Module
```

This allows imports such as:

```python
from app.health import classify_service
```

Read from left to right:

```text
app
 ↓
Package

health
 ↓
Module

classify_service
 ↓
Function
```

---

# `__init__.py`

At our current level, `__init__.py` identifies and initializes a directory as a regular Python package.

Example:

```text
app/
├── __init__.py
├── health.py
├── incidents.py
└── metrics.py
```

The `__init__.py` file can remain completely empty for now.

Its presence communicates that:

```text
app/
 ↓
Python Package
```

and the Python files inside can be modules belonging to that package.

For example:

```text
app.health
app.incidents
app.metrics
```

Important learning:

> `__init__.py` is related to the package itself, while individual `.py` files represent modules.

Modern Python also supports some packages without `__init__.py`, but that is not required for the current learning stage.

---

# `__name__`

Python automatically creates a special variable for every module:

```python
__name__
```

Its value depends on how the Python file is being used.

If a file is executed directly:

```bash
python health.py
```

then inside `health.py`:

```python
__name__
```

has the value:

```text
__main__
```

If the same file is imported:

```python
from app.health import classify_service
```

then its `__name__` represents its module name:

```text
app.health
```

Mental model:

```text
                health.py
                    │
             How was it loaded?
               /           \
              /             \
      Executed directly     Imported
             │                 │
             ▼                 ▼
       __name__ =          __name__ =
       "__main__"          "app.health"
```

Important learning:

> `__name__` helps a module understand whether it is being executed directly or imported by another module.

---

# `if __name__ == "__main__":`

The `__name__` variable allows Python code to distinguish between direct execution and importing.

Example:

```python
def classify_service(service):
    ...
```

Test code might be added:

```python
if __name__ == "__main__":
    service = {
        "name": "payment-api",
        "cpu": 92,
        "error_rate": 0.07,
        "status": "UP"
    }

    print(classify_service(service))
```

When running:

```bash
python health.py
```

Python sees:

```text
__name__ == "__main__"
```

and executes the block.

However, when another module does:

```python
from app.health import classify_service
```

inside `health.py`:

```text
__name__ = "app.health"
```

Therefore:

```text
"app.health" == "__main__"
```

is `False`, and the direct-execution code does not run.

Mental model:

```text
Run file directly
       ↓
__name__ = "__main__"
       ↓
Run main-only code


Import file
       ↓
__name__ = package.module
       ↓
Skip main-only code
```

Important learning:

> `if __name__ == "__main__":` allows code to run only when the file is executed directly, while preventing that code from executing when the file is imported.

---

# Why the `__main__` Guard Matters

Without a main guard:

```python
def classify_service(service):
    ...

print("Testing health module")
```

importing:

```python
from app.health import classify_service
```

would execute:

```python
print("Testing health module")
```

during the import.

That may be undesirable.

Using:

```python
if __name__ == "__main__":
    print("Testing health module")
```

separates:

```text
Reusable module code
        ↓
Functions / Classes

from

Direct execution code
        ↓
Testing / Demo / Application startup
```

---

# Application Structure

The functions created during previous days can now be organized as:

```text
sre_tools/
│
├── app/
│   ├── __init__.py
│   ├── health.py
│   ├── incidents.py
│   └── metrics.py
│
└── main.py
```

`health.py` contains:

```text
classify_service()
```

`incidents.py` contains:

```text
count_incidents()
```

`metrics.py` contains:

```text
calculate_error_rate()
```

and `main.py` can import them:

```python
from app.health import classify_service
from app.incidents import count_incidents
from app.metrics import calculate_error_rate
```

The architecture becomes:

```text
                    main.py
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
      health.py   incidents.py   metrics.py
          │            │            │
          ▼            ▼            ▼
      classify      count       calculate
      service      incidents    error rate
```

This is the first step toward structuring Python as an application rather than a collection of independent scripts.

---

# Type Hints

Type Hints communicate the expected types of function parameters and return values.

Example:

```python
def calculate_error_rate(
    errors: int,
    requests: int
) -> float:
    return errors / requests
```

This communicates:

```text
errors
  ↓
int

requests
  ↓
int

return
  ↓
float
```

Important learning:

> Type Hints describe the intended contract of a function.

---

# Primitive Type Hints

Basic Python types can be used directly:

```python
def service_info(
    name: str,
    cpu: int,
    error_rate: float,
    enabled: bool
) -> str:
    ...
```

Common types:

```text
str
int
float
bool
None
```

---

# Return Type Hints

The type after:

```python
->
```

describes the expected return type.

Example:

```python
def calculate_error_rate(
    errors: int,
    requests: int
) -> float:
```

means the function is expected to return a `float`.

If a function does not return a meaningful value:

```python
def print_service(
    service: str
) -> None:
    print(service)
```

`-> None` communicates that the function does not return useful data.

---

# Lists in Type Hints

A List containing strings can be represented as:

```python
list[str]
```

Example:

```python
def print_services(
    services: list[str]
) -> None:
    ...
```

Read:

```text
list[str]
   ↓
List containing strings
```

---

# Dictionaries in Type Hints

A Dictionary containing String keys and Integer values can be represented as:

```python
dict[str, int]
```

Example:

```python
counts = {
    "P1": 3,
    "P2": 2,
    "P3": 1
}
```

Its type can conceptually be represented as:

```text
dict[str, int]
     │     │
     │     └── values are integers
     │
     └──────── keys are strings
```

---

# Nested Collection Type Hints

The incident data used previously looks like:

```python
incidents = [
    {
        "service": "payment-api",
        "severity": "P1"
    },
    {
        "service": "orders-api",
        "severity": "P2"
    }
]
```

This can be represented as:

```python
list[dict[str, str]]
```

Read from outside inward:

```text
list
 ↓
contains Dictionaries
 ↓
Dictionary keys are strings
 ↓
Dictionary values are strings
```

The incident counter can therefore be annotated:

```python
def count_incidents(
    incidents: list[dict[str, str]],
    field: str
) -> dict[str, int]:
    ...
```

This communicates:

```text
INPUT

incidents
   ↓
List of Dictionaries
   ↓
String keys / String values


field
   ↓
String


OUTPUT

Dictionary
   ↓
String keys / Integer counts
```

---

# Optional Values

Sometimes a function may return a value or `None`.

Example:

```python
def get_service_owner(
    service: dict
) -> str | None:
    return service.get("owner")
```

If:

```python
service = {
    "name": "payment-api",
    "owner": "platform-team"
}
```

the result is:

```text
platform-team
```

If:

```python
service = {
    "name": "payment-api"
}
```

the result is:

```text
None
```

Therefore:

```python
str | None
```

means:

```text
Return either:

str
OR
None
```

Older Python code may use:

```python
from typing import Optional

def get_service_owner(...) -> Optional[str]:
```

Both communicate the same basic idea.

---

# Type Hints Are Not Runtime Validation

An important concept from the previous day was reinforced.

Given:

```python
def calculate_error_rate(
    errors: int,
    requests: int
) -> float:
    return errors / requests
```

Python can still allow:

```python
calculate_error_rate(
    2.43,
    5.69
)
```

even though the function hints specify:

```text
int
```

Important distinction:

```text
Type Hints
    ↓
Communicate expected types
    ↓
Help developers
    ↓
Help IDEs / static analysis


NOT


Automatic runtime validation
```

Later in the roadmap:

```text
Type Hints
    +
Pydantic
    +
FastAPI
    ↓
Runtime validated API inputs
```

This will make the distinction more important.

---

# Complex Types

A service Dictionary may contain different types of values:

```python
service = {
    "name": "payment-api",
    "cpu": 92,
    "error_rate": 0.07,
    "status": "UP"
}
```

The values include:

```text
str
int
float
```

Rather than creating complicated Type Hints immediately, these structures will later be represented more clearly using concepts such as:

```text
TypedDict
Dataclasses
Pydantic Models
Classes
```

Important learning:

> Type Hints should improve clarity. They should not be made unnecessarily complicated simply to type every possible structure.

---

# Modules + Type Hints Together

Modules organize **where functionality belongs**.

Type Hints communicate **what that functionality expects and returns**.

Together:

```text
Application
     │
     ├── health.py
     │      ↓
     │   classify_service()
     │      ↓
     │   clear input/output contract
     │
     ├── incidents.py
     │      ↓
     │   count_incidents()
     │      ↓
     │   clear input/output contract
     │
     └── metrics.py
            ↓
        calculate_error_rate()
            ↓
        clear input/output contract
```

Mental model:

```text
Modules
   ↓
Organize responsibilities


Type Hints
   ↓
Communicate contracts
```

---

# 💡 Engineering Learnings

* A `.py` file can act as a Python module.
* Modules allow related functionality to be grouped together.
* Packages organize related modules.
* `__init__.py` identifies/initializes a directory as a regular Python package at our current level.
* `import module` keeps access through the module namespace.
* `from module import function` imports a specific name directly.
* Import aliases provide alternative names for imported modules.
* `__name__` is automatically created by Python for each module.
* A directly executed file receives `__name__ == "__main__"`.
* An imported module receives its module/package name as `__name__`.
* The `if __name__ == "__main__":` guard separates direct-execution behavior from reusable imported behavior.
* Type Hints communicate function contracts.
* `list[str]` describes a List of strings.
* `dict[str, int]` describes String-to-Integer mappings.
* Nested collection types can describe structured data.
* `str | None` communicates an optional result.
* `-> None` communicates that a function does not return meaningful data.
* Type Hints improve readability but do not automatically validate values at runtime.
* Code should be split by responsibility rather than arbitrarily divided into files.

---

# ⚠️ Things That Initially Needed Clarification

Initially, the relationship between:

```text
__init__.py
__name__
__main__
```

was not completely clear.

After experimenting with the code, the mental model became:

```text
__init__.py
     ↓
"This directory is a Python package."


__name__
     ↓
"How is this module currently being loaded?"


if __name__ == "__main__":
     ↓
"Run this block only when this file
 is executed directly."
```

Running a module directly demonstrated:

```text
__name__ = "__main__"
```

while importing it demonstrated:

```text
__name__ = "package.module"
```

Experimenting with both behaviors made the concept much clearer than simply memorizing the syntax.

---

Initially, Type Hints could appear to mean:

> Python will reject values of another type.

Testing showed that this is not the case.

Type Hints communicate expected types but do not automatically enforce them at runtime.

---

# 🚀 Production Takeaways

As applications grow, keeping everything inside one Python file becomes difficult to maintain.

A production-style application can instead evolve toward:

```text
Application
│
├── API
├── Configuration
├── Health Logic
├── Incident Logic
├── Metrics Logic
├── Validation
└── Logging
```

with each responsibility organized into appropriate modules.

For example:

```text
main.py
   │
   ├── app.health
   ├── app.incidents
   └── app.metrics
```

This same principle will later apply when building:

* FastAPI applications
* ML training pipelines
* Model-serving services
* MLOps workflows
* Kubernetes automation
* LLM applications
* SRE Copilot tools

Type Hints will become increasingly useful because these systems will contain many functions and components communicating with one another.

Clear contracts make it easier to understand:

```text
What goes in?
     ↓
What does this component expect?
     ↓
What comes out?
```

---

# 🏆 End of Session Reflection

Today's biggest takeaway was understanding how Python moves from:

```text
Functions
    ↓
Modules
    ↓
Packages
    ↓
Application
```

Modules are not simply a way to create more files.

They allow functionality to be organized around clear responsibilities.

Understanding `__name__` and the `__main__` guard also explained how the same Python file can behave differently depending on whether it is executed directly or imported as reusable code.

Type Hints added another important engineering concept:

> **A function should communicate its expected inputs and outputs clearly.**

The combined mental model is:

```text
Modules
   ↓
Where does this responsibility belong?


Type Hints
   ↓
What does this responsibility expect and return?
```

From today onwards, when a Python application starts growing, I'll consciously ask:

> **"Which responsibility does this code belong to, and what contract should it expose to the rest of the application?"**

This will become increasingly important as the journey moves toward APIs, ML pipelines, model serving, MLOps, and AI Platform Engineering.

**Modules: Theory ✅ | Hands-on ✅**

**Packages: Fundamentals ✅ | Hands-on ✅**

**Type Hints: Fundamentals ✅ | Hands-on ✅**

**Next: Exceptions & Error Handling ⭐**
