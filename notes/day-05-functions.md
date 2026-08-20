# 📒 Day 5 Notes - Python Functions

## 🎯 Day Objective

Understand Python Functions and learn how to turn repeated or related logic into reusable, readable, and testable units of code.

The focus was to understand how functions accept inputs, process them, return outputs, and allow larger Python programs to be broken into smaller, reusable pieces.

The exercises were based on SRE-oriented examples such as service health checks, error rates, alerts, request analysis, and incident counting.

---

# 📚 Concepts Learned

## Functions

A Function is a reusable block of code designed to perform a specific task.

A function is defined using the `def` keyword:

```python id="5q0fks"
def greet():
    print("Hello")
```

The function does not execute when it is defined.

It executes when it is called:

```python id="3g2hyb"
greet()
```

Important distinction:

```text id="i6zy2v"
def function():
      ↓
Define the function


function()
      ↓
Call the function
```

---

## Function Parameters & Arguments

Functions can accept input values using parameters.

Example:

```python id="9uk8z4"
def check_service(service, environment):
    print(
        f"Checking {service} "
        f"in {environment}"
    )
```

Here:

```text id="k9q0ec"
service
environment
```

are parameters.

When calling:

```python id="e4fsv5"
check_service(
    "payment-api",
    "production"
)
```

the supplied values are arguments.

Important distinction:

> A parameter is the variable defined by the function, while an argument is the actual value supplied when calling the function.

---

## `print()` vs `return` ⭐

One of the most important concepts learned today was the difference between displaying a result and returning a result.

Using `print()`:

```python id="8gsz3x"
def calculate_error_rate(errors, requests):
    print(errors / requests)
```

The function displays the value but does not provide it to the caller for further processing.

Using `return`:

```python id="00h9e9"
def calculate_error_rate(errors, requests):
    return errors / requests
```

allows the caller to store and use the returned value:

```python id="m63wlw"
rate = calculate_error_rate(
    5,
    100
)

print(rate)
```

The returned value can then be used in other logic:

```python id="phqkpm"
if rate > 0.03:
    print("WARNING")
```

Mental model:

```text id="5fo2ui"
print()
   ↓
Display result


return
   ↓
Give result back to caller
```

Important learning:

> Reusable functions should generally return data rather than deciding how that data should be displayed.

---

## `return` Ends Function Execution

When Python reaches a `return` statement, the function immediately ends.

Example:

```python id="bf8d2e"
def test():
    print("Before return")

    return "DONE"

    print("After return")
```

The code after `return` is not executed.

Mental model:

```text id="d8fu5z"
Function starts
      ↓
Execute code
      ↓
return
      ↓
Function ends
```

This allows functions to exit early when appropriate.

---

# Positional Arguments

Arguments can be passed based on their position.

Example:

```python id="3ek81r"
def calculate_availability(
    successful_requests,
    total_requests
):
    return (
        successful_requests
        / total_requests
    ) * 100
```

Calling:

```python id="x4d8u4"
calculate_availability(
    990,
    1000
)
```

maps:

```text id="7w2m9z"
990
 ↓
successful_requests

1000
 ↓
total_requests
```

With positional arguments, the order determines which parameter receives each value.

---

# Keyword Arguments

Arguments can also be passed using parameter names.

Example:

```python id="cql8oi"
calculate_availability(
    total_requests=1000,
    successful_requests=990
)
```

The order is no longer important because the parameter names identify where each value belongs.

Mental model:

```text id="2d8d0j"
Positional argument
        ↓
Position determines meaning


Keyword argument
        ↓
Parameter name determines meaning
```

---

# Default Arguments

A function parameter can have a default value.

Example:

```python id="ogmdxr"
def create_alert(
    service,
    severity="P3"
):
    print(
        f"Service is {service} "
        f"with severity {severity}"
    )
```

Calling:

```python id="2a1e1x"
create_alert("payment-api")
```

uses the default:

```text id="jb4lzw"
P3
```

While:

```python id="e4qtu6"
create_alert(
    "payment-api",
    "P1"
)
```

overrides the default.

Important learning:

> Default arguments provide sensible behavior when the caller does not supply a value.

---

# Multiple Return Values

A function can return multiple values.

Example:

```python id="5g7j2v"
def analyze_requests(
    total_requests,
    failed_requests
):
    successful_requests = (
        total_requests - failed_requests
    )

    error_rate = (
        failed_requests / total_requests
    )

    return successful_requests, error_rate
```

The caller can unpack the returned values:

```python id="h4s0pj"
successful, error_rate = analyze_requests(
    1000,
    25
)
```

This connects directly with Tuple unpacking learned earlier.

Conceptually:

```text id="1ds7pu"
Function
   ↓
( successful_requests, error_rate )
   ↓
Tuple
   ↓
Unpacking
```

---

# Variable Scope

Variables created inside a function are normally local to that function.

Example:

```python id="x6bx0h"
def calculate():
    result = 10
    print(result)

calculate()
```

`result` can be accessed inside the function.

Trying:

```python id="0j2q4x"
print(result)
```

outside the function raises a `NameError`.

Mental model:

```text id="b08f3f"
Outside function
       │
       │
   ┌───▼──────────┐
   │ Function     │
   │              │
   │ local vars   │
   │              │
   └──────────────┘
```

Important learning:

> Local variables should normally remain local to the function that owns them.

This encourages functions to communicate through parameters and return values rather than shared global state.

---

# Type Hints

Functions can specify expected parameter and return types using type hints.

Example:

```python id="o8ku5e"
def calculate_error_rate(
    errors: int,
    requests: int
) -> float:
    return errors / requests
```

This communicates:

```text id="m6x1tg"
errors
  ↓
expected int

requests
  ↓
expected int

return
  ↓
expected float
```

An important observation was that Python does not automatically enforce these annotations at runtime.

For example:

```python id="3fgrf1"
calculate_error_rate(
    2.43,
    5.69
)
```

can still execute.

Important learning:

> Type hints communicate intent and help developers and tools, but they are not by themselves runtime validation.

Later, runtime validation will be introduced using tools such as Pydantic and FastAPI.

---

# Docstrings

Functions can document their purpose using a docstring.

Example:

```python id="q4b8t2"
def calculate_error_rate(
    errors: int,
    requests: int
) -> float:
    """Calculate the fraction of failed requests."""
    return errors / requests
```

Docstrings should explain the purpose of the function clearly without becoming unnecessarily verbose.

---

# Pure vs Impure Functions

A function is easier to reason about when its output depends only on its inputs and it does not modify external state.

Example:

```python id="3yt1g9"
def calculate_error_rate(
    errors,
    requests
):
    return errors / requests
```

The same inputs produce the same output.

Conceptually:

```text id="c7e4s4"
INPUT
  ↓
FUNCTION
  ↓
OUTPUT
```

This makes such functions easier to:

* Test
* Reuse
* Debug
* Understand

Functions that modify global state or external systems are more difficult to reason about.

Important learning:

> Prefer functions that communicate through inputs and return values when practical.

---

# `*args`

Learned the basic concept of `*args`.

Example:

```python id="7s7qyg"
def total_errors(*errors):
    print(errors)
```

Calling:

```python id="p3u0q7"
total_errors(
    10,
    20,
    30
)
```

causes `errors` to behave like a Tuple:

```text id="7w7g4s"
(10, 20, 30)
```

Mental model:

```text id="ar51cf"
Unknown number of positional arguments
              ↓
            *args
              ↓
            Tuple
```

Only the basic concept was covered today.

---

# `**kwargs`

Learned the basic concept of `**kwargs`.

Example:

```python id="rv0p5x"
def service_info(**details):
    print(details)
```

Calling:

```python id="q2b8v5"
service_info(
    name="payment-api",
    cpu=82,
    status="UP"
)
```

causes `details` to behave like a Dictionary.

Mental model:

```text id="2qx0hu"
Unknown number of keyword arguments
              ↓
           **kwargs
              ↓
          Dictionary
```

Only the basic concept was covered today.

---

# Refactoring the SRE Health Classifier

A major exercise was converting logic from the previous day's script into a reusable function.

Instead of directly writing classification logic inside a loop:

```text id="v4n2tz"
Service
  ↓
if / elif / else
  ↓
Print result
```

the logic was extracted into:

```python id="ab8nj7"
def classify_service(
    cpu,
    error_rate,
    status
):
    if status == "DOWN":
        return "DOWN"
    elif cpu > 90 and error_rate > 0.05:
        return "CRITICAL"
    elif cpu > 90 or error_rate > 0.05:
        return "WARNING"
    else:
        return "HEALTHY"
```

The caller can then decide what to do with the result:

```python id="q1u4b0"
health = classify_service(
    92,
    0.07,
    "UP"
)

print(health)
```

This created a clean separation:

```text id="n3t6vi"
Input data
    ↓
Classification function
    ↓
Health result
    ↓
Caller decides what to do
```

This was the first major step toward reusable application design.

---

# Refactoring the Incident Counter

Previously, similar counting logic was written separately for services and severities.

The repeated algorithm was identified and converted into a reusable function:

```python id="6h4mp5"
def count_incidents(
    incidents,
    field
):
    count = {}

    for incident in incidents:
        field_value = incident[field]

        if field_value in count:
            count[field_value] += 1
        else:
            count[field_value] = 1

    return count
```

The function can now be reused:

```python id="j4sk02"
service_counts = count_incidents(
    incidents,
    "service"
)

severity_counts = count_incidents(
    incidents,
    "severity"
)
```

This produces different results using the same underlying algorithm.

Important learning:

> If two pieces of code perform the same logic but differ only in one input, that difference can often become a function parameter.

This was the most important software-engineering lesson from today's refactoring exercise.

---

# 💡 Engineering Learnings

* Functions allow large programs to be broken into smaller reusable units.
* Parameters allow the same function to work with different inputs.
* Arguments are the actual values supplied when calling a function.
* `return` makes a function's result reusable by the caller.
* `print()` should generally be left to the caller when the function is intended to provide reusable data.
* Positional arguments depend on order, while keyword arguments depend on parameter names.
* Default arguments provide sensible values when optional inputs are omitted.
* Multiple return values naturally connect with Tuple unpacking.
* Local variables belong to the function scope in which they are created.
* Type hints communicate expected input and output types but do not automatically enforce them at runtime.
* Pure functions are easier to test and reason about.
* `*args` collects variable positional arguments into a Tuple.
* `**kwargs` collects variable keyword arguments into a Dictionary.
* Repeated logic should be identified and extracted into reusable functions.
* Good function design separates **what a function calculates** from **what the caller does with the result**.
* Functions should ideally have a clear responsibility rather than becoming large blocks of unrelated logic.

---

# ⚠️ Mistakes I Made

Initially used nested double quotes inside an f-string:

```python id="i34iqd"
print(
    f"{service["name"]} is in {health} state"
)
```

This creates a quoting conflict.

The corrected version is:

```python id="z7qp2n"
print(
    f"{service['name']} is in {health} state"
)
```

or the Dictionary value can be assigned to a variable before formatting the message.

---

In the `analyze_requests()` function, initially used an unnecessary trailing comma:

```python id="l4v8yq"
return successful_requests, error_rate,
```

This still produces a Tuple, but the cleaner form is:

```python id="0x4eau"
return successful_requests, error_rate
```

---

The initial `count_incidents()` implementation printed the result inside the function:

```python id="t9b2q0"
print(count)
```

The function was improved to:

```python id="v71q8m"
return count
```

so the caller can decide whether to:

* Print the result
* Log it
* Store it
* Send it through an API
* Test it

This reinforced the difference between **calculation** and **presentation**.

---

# 🚀 Production Takeaways

Functions are one of the most important building blocks for production Python systems.

An SRE application can be structured as:

```text id="rv5ed3"
Input
  ↓
Validation
  ↓
Processing Functions
  ↓
Business / Operational Logic
  ↓
Result
  ↓
Logging / API / Storage
```

For example:

```text id="xj2sn7"
Incident
   ↓
validate_incident()
   ↓
classify_severity()
   ↓
calculate_impact()
   ↓
generate_summary()
```

Instead of keeping everything in one large script, each function can have a clear responsibility.

This will later become important when building:

* FastAPI services
* ML pipelines
* Model serving code
* MLOps workflows
* Kubernetes automation
* LLMOps pipelines
* SRE Copilot tools

The SRE Health Classifier was especially useful because it showed how operational rules can be extracted into reusable functions.

---

# 🏆 End of Day Reflection

Today's biggest takeaway was understanding that **functions are the bridge between writing scripts and writing software**.

Before Functions, most of the Python code was written as a sequence of instructions:

```text id="j6b7wp"
Read data
  ↓
Loop
  ↓
Check condition
  ↓
Process result
```

Functions introduced a reusable abstraction:

```text id="s5q8wq"
Input
  ↓
Function
  ↓
Output
```

The most important shift was learning to look at repeated logic and ask:

> **"What part of this behavior stays the same, and what part should become an input?"**

The `count_incidents()` exercise demonstrated this particularly well. Instead of writing separate counting logic for services and severities, the changing field became a parameter.

This is an important step toward thinking like a software engineer rather than simply writing scripts.

From today onwards, when writing Python I'll consciously ask:

> **"Does this piece of logic have a clear responsibility, and should it be a reusable function?"**

**Functions: Theory ✅ | Hands-on ✅**
