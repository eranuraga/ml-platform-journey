# 📒 Day 6 Notes - Python Exceptions & Error Handling

## 🎯 Day Objective

Understand Python Exceptions and learn how to detect, handle, raise, propagate, and communicate errors in a controlled way.

The focus was to move from simply seeing Python errors as failures to understanding them as **signals that can be handled deliberately at the appropriate layer of an application**.

The exercises used SRE-oriented examples such as error-rate calculation, service configuration, replica processing, incident validation, and custom incident exceptions.

---

# 📚 Concepts Learned

## What is an Exception?

An Exception occurs when Python encounters a situation where normal execution cannot continue.

Examples encountered earlier in the journey include:

```text id="f9b4a0"
ZeroDivisionError
    ↓
10 / 0
```

```text id="3tv6y0"
KeyError
    ↓
dictionary["missing_key"]
```

```text id="5q3y0u"
TypeError
    ↓
"10" + 20
```

```text id="80mzqn"
NameError
    ↓
Using a variable that does not exist
```

Important learning:

> An Exception represents a failure condition that interrupts normal program execution.

---

# Reading a Traceback

A Python traceback provides information about where and why an exception occurred.

The important information can generally be read from the bottom upward:

```text id="f7ca8e"
Exception Type
      ↓
Exception Message
      ↓
Line that failed
      ↓
Call path that led to the failure
```

For example:

```python id="7t8fr5"
def calculate_error_rate(errors, requests):
    return errors / requests

rate = calculate_error_rate(10, 0)
```

results in:

```text id="m0q9o2"
ZeroDivisionError: division by zero
```

The traceback is therefore not just an error message.

It is a debugging tool that helps identify:

* What failed
* Where it failed
* How execution reached the failing code

This is similar to reading stack traces while troubleshooting production systems.

---

# `try` / `except`

Python provides `try` and `except` for handling known failure conditions.

Example:

```python id="n4a8y8"
try:
    rate = errors / requests
except ZeroDivisionError:
    print("Requests cannot be zero")
```

Mental model:

```text id="w0ng3c"
try
 ↓
Execute risky operation
 ↓
Exception?
   / \
 NO   YES
 ↓      ↓
Continue except
```

The `try` block contains code that may raise an exception.

The `except` block defines what should happen when a matching exception occurs.

---

# Catching Specific Exceptions

Exceptions should normally be caught specifically when the application knows how to handle them.

Example:

```python id="dw9k2k"
try:
    region = service["region"]
except KeyError:
    region = "unknown"
```

This is preferable to blindly using:

```python id="x1x3ov"
except:
    ...
```

or:

```python id="yq9r3s"
except Exception:
    ...
```

everywhere.

Important learning:

> Catch the exception you actually understand and know how to handle.

---

# Capturing the Exception Object

An exception can be captured using `as`.

Example:

```python id="ix6qjf"
try:
    region = service["region"]
except KeyError as exc:
    print(exc)
```

The variable `exc` contains the exception object.

This is useful when logging or reporting the actual failure.

For example:

```python id="h4d2ry"
except KeyError as exc:
    logger.error(
        "Missing required field: %s",
        exc
    )
```

This becomes especially useful in production observability.

---

# `try` / `except` with Multiple Exceptions

Different problems may require different handling.

Example:

```python id="m0xqk8"
def calculate_per_replica_load(
    total_load,
    replicas
):
    try:
        return total_load / replicas

    except TypeError:
        print("Values must be numeric")

    except ZeroDivisionError:
        print("Replicas value cannot be zero")
```

Two different failure modes are handled separately:

```text id="d1y4wx"
TypeError
   ↓
Input types are invalid


ZeroDivisionError
   ↓
Replicas is zero
```

Important learning:

> Different failure modes may require different responses.

---

# `else`

A `try` statement can contain an `else` block.

The `else` block runs only when the `try` block completes successfully.

Example:

```python id="r4r3qk"
try:
    replica_count = int(value)

except ValueError:
    print("Invalid replica count")

else:
    print(
        f"Replica count: {replica_count}"
    )
```

Mental model:

```text id="j7y4nl"
try
 ↓
Success?
 /    \
NO    YES
↓      ↓
except else
```

This is useful when the successful path should be kept separate from the exception-handling logic.

---

# `finally`

The `finally` block runs regardless of whether an exception occurred.

Example:

```python id="7i8d6s"
try:
    replica_count = int(value)

except ValueError:
    print("Invalid replica count")

else:
    print(
        f"Replica count: {replica_count}"
    )

finally:
    print("Processing completed")
```

Both successful and failed execution eventually reach:

```text id="abxv1g"
Processing completed
```

Mental model:

```text id="yyqg4l"
try
 ↓
success or failure
 ↓
except / else
 ↓
finally
```

`finally` is particularly useful when cleanup must happen regardless of success or failure.

Typical examples include:

* Closing resources
* Releasing locks
* Cleaning temporary state
* Finalizing operations

---

# `raise` ⭐

An application can deliberately raise an Exception when it detects invalid input.

Example:

```python id="93o7l5"
def validate_severity(severity):

    if severity not in {
        "P1",
        "P2",
        "P3"
    }:
        raise ValueError(
            f"Invalid severity: {severity}"
        )

    return True
```

Calling:

```python id="u4g39f"
validate_severity("P9")
```

raises:

```text id="j4n7b2"
ValueError: Invalid severity: P9
```

Important learning:

> `raise` allows a lower-level function to explicitly communicate that it cannot safely continue with the supplied input.

---

# Why Raising an Exception Can Be Better Than Returning a Default

Consider error-rate calculation.

This implementation:

```python id="a7t4p3"
def calculate_error_rate(
    errors,
    requests
):
    try:
        return errors / requests

    except ZeroDivisionError:
        return 0.0
```

technically works.

But:

```text id="q5k8s1"
5 errors
0 requests
```

does not logically mean:

```text id="f7g9r2"
0% error rate
```

The input itself is invalid for the calculation.

A better approach may be:

```python id="k4p1x3"
def calculate_error_rate(
    errors,
    requests
):
    if requests <= 0:
        raise ValueError(
            "requests must be greater than zero"
        )

    return errors / requests
```

Important production lesson:

> Handling an exception does not mean silently replacing an invalid result with a convenient value.

---

# Exception Propagation ⭐⭐⭐⭐⭐

An Exception does not have to be handled where it first occurs.

If a function raises an Exception and does not handle it, Python allows the Exception to propagate upward through the call stack.

Example:

```text id="x8g5c9"
validate_severity()
        ↓
detect invalid input
        ↓
raise InvalidIncidentError
        ↓
process_incident()
        ↓
doesn't handle it
        ↓
caller
```

This is **exception propagation**.

The important principle is:

> The lower-level function can identify the problem, while a higher-level application can decide what to do about it.

---

# Lower-Level vs Higher-Level Error Handling

This became one of the most important mental models from today's session.

```text id="v4q8w2"
LOWER-LEVEL FUNCTION
        ↓
detect problem
        ↓
raise meaningful exception
        ↓
propagate
        ↓
HIGHER-LEVEL APPLICATION
        ↓
decide what to do
```

For example:

```text id="v5p9h1"
validate_incident()
        ↓
Invalid incident
        ↓
raise InvalidIncidentError
        ↓
main.py
        ↓
catch exception
        ↓
log / skip / alert / fail
```

This creates a clean separation of responsibilities.

---

# Where Should an Exception Be Handled?

A useful rule learned today:

> **Catch an exception where the application can make a meaningful decision about how to handle it.**

A lower-level function may know:

> "This incident is invalid."

But it may not know:

> "Should the application skip it, retry it, log it, or terminate?"

That decision may belong to a higher-level component.

Mental model:

```text id="r3v7k4"
LOW-LEVEL
detects failure
    ↓
raises
    ↓
HIGH-LEVEL
decides response
```

---

# Custom Exceptions

Python allows applications to define domain-specific exceptions.

Example:

```python id="3z5k0g"
class InvalidIncidentError(Exception):
    pass
```

Now an incident validation function can raise something much more meaningful:

```python id="mc0m8v"
raise InvalidIncidentError(
    "Invalid incident data"
)
```

Compared with:

```text id="8v2j6d"
ValueError
```

a custom exception communicates the domain-specific meaning:

```text id="h9n3m4"
InvalidIncidentError
        ↓
This failure specifically concerns
incident validation
```

This becomes useful in larger applications where multiple types of validation or operational failures exist.

---

# Exception Validation Example

Created:

```python id="d5t8q1"
class InvalidIncidentError(Exception):
    pass
```

Then used it in:

```python id="4u1p7s"
def validate_incident(incident):
    ...
```

Validation rules included:

* Required fields must exist.
* Unexpected fields should be rejected.
* Severity must be one of `P1`, `P2`, `P3`.

When validation fails:

```python id="m2v5k8"
raise InvalidIncidentError(...)
```

The exception is allowed to propagate to the higher-level application.

---

# Exception Handling Across Modules

The concepts of Modules and Exceptions were combined.

Structure:

```text id="r4a7u2"
app/
├── __init__.py
├── exceptions.py
└── incidents.py

main.py
```

`exceptions.py`:

```python id="6k0f3e"
class InvalidIncidentError(Exception):
    pass
```

`incidents.py`:

```python id="p8z2m5"
def validate_incident(incident):
    ...
    raise InvalidIncidentError(...)
```

`main.py`:

```python id="v4x1n9"
from app.incidents import validate_incident
from app.exceptions import InvalidIncidentError

for incident in incidents:

    try:
        validate_incident(incident)

    except InvalidIncidentError as exc:
        print(
            f"Invalid incident: {exc}"
        )

    else:
        print("Incident valid")
```

This demonstrated the complete flow:

```text id="q8c3m7"
main.py
   ↓
validate_incident()
   ↓
invalid data detected
   ↓
raise InvalidIncidentError
   ↓
propagate
   ↓
main.py catches
   ↓
decides what to do
```

This was the most important practical exercise of the session.

---

# Exception Chaining

Python can preserve the original exception when raising a new one.

Example:

```python id="d2q8x0"
try:
    value = int(replica_count)

except ValueError as exc:
    raise InvalidIncidentError(
        "Invalid replica count"
    ) from exc
```

The:

```python id="6x5r8f"
from exc
```

preserves the original cause.

Mental model:

```text id="n4w7p2"
Original ValueError
        ↓
caused
        ↓
InvalidIncidentError
```

This helps preserve debugging context while exposing a more meaningful application-level exception.

Only the concept was introduced; deeper exception chaining patterns can be learned later when needed.

---

# Avoiding Dangerous Exception Handling

Avoid:

```python id="x7m1q8"
try:
    something()
except:
    pass
```

This silently hides failures.

For example:

```text id="v2k9r3"
Model failed to load
       ↓
except:
    pass
       ↓
Application continues
       ↓
Potentially broken system
```

This is dangerous in production.

Also avoid catching broad exceptions unnecessarily:

```python id="p8d4q1"
except Exception:
    ...
```

unless there is a clear application-level reason to do so.

Prefer specific exceptions:

```python id="n7f2m6"
except ValueError:
```

```python id="y6q1w8"
except KeyError:
```

or domain-specific exceptions:

```python id="k3m9p5"
except InvalidIncidentError:
```

---

# 💡 Engineering Learnings

* Exceptions represent abnormal conditions that interrupt normal execution.
* Tracebacks provide valuable debugging information.
* `try` identifies code that may fail.
* `except` handles a specific failure condition.
* Catch specific exceptions when possible.
* `else` runs when the `try` block succeeds.
* `finally` runs regardless of success or failure.
* `raise` allows code to deliberately reject invalid data.
* Returning a default value is not always the correct way to handle invalid input.
* Exceptions can propagate through multiple layers of an application.
* Lower-level components can detect and raise problems while higher-level components decide how to respond.
* Custom exceptions provide domain-specific meaning.
* Exception chaining preserves the underlying cause of a higher-level exception.
* Silently swallowing exceptions can hide serious production failures.
* The correct place to handle an exception is generally the layer that can make a meaningful recovery or application-level decision.
* Exception handling should preserve useful debugging context rather than hide it.

---

# ⚠️ Things That Initially Needed Clarification

Initially, Exceptions felt like a difficult Python topic and were mainly viewed as errors that caused a program to fail.

The mental model became much clearer after understanding:

```text id="x8q1m4"
LOWER-LEVEL FUNCTION
        ↓
detect problem
        ↓
raise meaningful exception
        ↓
propagate
        ↓
HIGHER-LEVEL APPLICATION
        ↓
decide what to do
```

This helped separate two different responsibilities:

```text id="g3m7p1"
Detect and describe the problem
          VS
Decide how the application should respond
```

The difference between:

```python id="i9v2k6"
try / except
```

and:

```python id="r7m3q8"
raise
```

also became much clearer.

`except` is used when the current layer can meaningfully handle a problem.

`raise` is useful when the current layer should communicate that something is invalid or cannot safely continue.

---

# 🚀 Production Takeaways

Error handling is essential in SRE and production engineering because failures are expected.

A production system needs to distinguish between:

```text id="f8v4m2"
Expected / recoverable condition
        ↓
Handle appropriately


Invalid input
        ↓
Reject explicitly


Unexpected failure
        ↓
Preserve context
Propagate / fail safely
```

For example:

```text id="p3w7k9"
API Request
    ↓
Validation
    ↓
Invalid?
 /     \
YES     NO
 ↓       ↓
raise   process
exception
    ↓
application layer
    ↓
log / reject / retry / alert
```

This same model will later apply to:

* FastAPI APIs
* Kubernetes automation
* ML data pipelines
* Model loading
* Model inference
* MLOps workflows
* External API calls
* SRE Copilot tools

The most important production principle learned today is:

> **Do not hide failures. Detect them, preserve useful context, and handle them at the appropriate layer.**

---

# 🏆 End of Day Reflection

Today's biggest takeaway was that Exceptions are not simply something to be afraid of.

They are a structured way for Python programs to communicate:

> **"Something unexpected or invalid happened, and normal execution cannot safely continue."**

The concepts that initially felt confusing became much clearer after working through:

```text id="e6t2r8"
try
 ↓
except
 ↓
raise
 ↓
finally
 ↓
propagation
 ↓
custom exceptions
```

The most important mental model from today was:

```text id="x2m7q4"
LOWER-LEVEL FUNCTION
        ↓
detect problem
        ↓
raise meaningful exception
        ↓
propagate
        ↓
HIGHER-LEVEL APPLICATION
        ↓
decide what to do
```

This changed my perspective from:

> "An Exception means my Python program broke."

to:

> **"An Exception is a controlled mechanism for communicating failure between different layers of an application."**

I also learned that catching every Exception is not automatically good error handling. Good error handling means understanding the failure, preserving context, and making the recovery or failure decision at the right level.

From today onwards, when encountering an error I'll consciously ask:

> **"What exactly failed, can this layer meaningfully handle it, and if not, what useful exception information should propagate upward?"**

**Exceptions: Theory ✅ | Hands-on ✅**

**Error Handling: Theory ✅ | Hands-on ✅**

**Day 6: Complete ✅**
