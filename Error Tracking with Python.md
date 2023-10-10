# Overview
Python's built-in error tracking system includes:
- **Syntax Errors**: These are parsing errors that occur when the Python parser is unable to understand a line of code.
- **Exceptions**: These are errors that occur during the execution of a program. When an exception is raised, it propagates up the call stack until it is caught and handled. If it's not caught, the program will terminate.
- **Tracebacks**: When an exception is raised, Python will print a traceback to the standard error. The traceback contains the sequence of function calls that led to the exception, which can help you identify where the error occurred.

## How do error tracking programs like Sentry work under the hood in Python?
When you initialize Sentry using `sentry_sdk.init()`, it sets up a global error handler that can catch unhandled exceptions. This is done by hooking into the Python standard library's logging module and the built-in `sys.excepthook` function, which is called whenever an unhandled exception occurs. This is how libraries like Sentry can capture uncaught exceptions: they assign a custom function to `sys.excepthook` that sends exception details to their service.

`sys.excepthook` is a built-in function in Python's `sys` module that allows you to handle uncaught exceptions. By default, `sys.excepthook` is called whenever an exception is not caught in a `try/except` block, and it handles the exception by printing out a traceback to `sys.stderr` and terminating the program. The default behavior of `sys.excepthook` can be overridden by assigning a new function to `sys.excepthook`. The new function should accept three arguments: the exception type, the exception value, and a traceback object. This allows you to define custom error handling for uncaught exceptions.

Here's an example:

```python
import sys

def handle_exception(exc_type, exc_value, exc_traceback):
    # This function will be called for any uncaught exceptions
    print("An uncaught exception occurred:")
    print(exc_type, exc_value)
    print("Don't divide by 0, Mate!")

sys.excepthook = handle_exception

# This will raise an uncaught ZeroDivisionError
1 / 0
```

Result:
```
An uncaught exception occurred:
<class 'ZeroDivisionError'> division by zero
Don't divide by 0, Mate!
```

In this example, we define a new function `handle_exception` and assign it to `sys.excepthook`. Now, when an uncaught `ZeroDivisionError` occurs, our `handle_exception` function is called instead of the default `sys.excepthook`, and it prints out a custom error message. This is how libraries like Sentry can capture uncaught exceptions: they assign a custom function to `sys.excepthook` that sends exception details to their service.

## Inspecting the Traceback
Here's an example:
```python
import sys
import traceback


def handle_exception(exc_type, exc_value, exc_traceback):
    print("An uncaught exception occurred:")
    print(exc_type, exc_value)
    print("\nTraceback:")

    # Print the traceback
    traceback.print_exception(exc_type, exc_value, exc_traceback)

    # Walk the traceback
    for frame, lineno in traceback.walk_tb(exc_traceback):
        # Print local variables in this frame
        print(f"\nIn {frame.f_code.co_name}:")
        for var_name, var_value in frame.f_locals.items():
            print(f"  {var_name} = {var_value}")


sys.excepthook = handle_exception


def divide_10_by_20_then_by_0():
    x = 10
    y = 20
    return (x / y) / 0  # This will raise a ZeroDivisionError


def divide_by_0():
    zero = 0
    val = divide_10_by_20_then_by_0()
    return val / zero  # This will raise a ZeroDivisionError


divide_by_0()
```

Result:
```
An uncaught exception occurred:
<class 'ZeroDivisionError'> float division by zero

Traceback:

In <module>:
  __name__ = __main__
  __doc__ = None
  __package__ = None
  __loader__ = <_frozen_importlib_external.SourceFileLoader object at 0x100a73c10>
  __spec__ = None
  __annotations__ = {}
  __builtins__ = <module 'builtins' (built-in)>
  __file__ = /Users/buiducphong/Projects/PythonPlayground/main2.py
  __cached__ = None
  sys = <module 'sys' (built-in)>
  traceback = <module 'traceback' from '/opt/homebrew/Cellar/python@3.10/3.10.13/Frameworks/Python.framework/Versions/3.10/lib/python3.10/traceback.py'>
  handle_exception = <function handle_exception at 0x100b4c0d0>
  divide_10_by_20_then_by_0 = <function divide_10_by_20_then_by_0 at 0x100bf7640>
  divide_by_0 = <function divide_by_0 at 0x100acf9a0>

In divide_by_0:
  zero = 0

In divide_10_by_20_then_by_0:
  x = 10
  y = 20
Traceback (most recent call last):
  File "/Users/buiducphong/Projects/PythonPlayground/main2.py", line 36, in <module>
    divide_by_0()
  File "/Users/buiducphong/Projects/PythonPlayground/main2.py", line 32, in divide_by_0
    val = divide_10_by_20_then_by_0()
  File "/Users/buiducphong/Projects/PythonPlayground/main2.py", line 27, in divide_10_by_20_then_by_0
    return (x / y) / 0  # This will raise a ZeroDivisionError
ZeroDivisionError: float division by zero
```
