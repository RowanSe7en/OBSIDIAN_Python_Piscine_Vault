
---
# PYTHON PISCINE

---

# Interpreting in Python

Interpreting is the process where Python executes code by translating it into instructions the Python Virtual Machine (PVM) can run, rather than compiling it directly into native machine code. Python is technically compiled to bytecode first, then interpreted by the PVM.

**Main steps in Python interpretation:**

1. Starting the Python Interpreter (CPython)
2. Lexing (Tokenization) – Converts raw source code into tokens, the smallest meaningful units of the program.
3. Parsing (Syntactic Analysis) – Checks the grammar of the token stream and builds an Abstract Syntax Tree (AST) representing program structure.
4. Bytecode Generation – Translates the AST into Python bytecode, a low-level, platform-independent instruction set.
5. Execution by PVM – The Python Virtual Machine reads and executes the bytecode.
6. Optional .pyc Compilation – Generates cached bytecode files for faster future execution without running the program.

---

# 1 - Starting the Python Interpreter (CPython)

Before any Python source code is read, the operating system starts the Python interpreter itself, which is the CPython executable. When you run `python3 script.py`, the OS locates the compiled `python3` binary (built from C code using a compiler such as GCC), loads it into memory, sets up the process environment (stack, heap, and CPU registers) like every native program (Python, Chrome, GCC, Bash, etc.), and jumps to its entry point (`main()` in C). CPython then initializes its runtime environment by setting up memory management, the garbage collector, built-in types, the import system, and the Python Virtual Machine (PVM). All of this work is performed in machine code, before any lexing, parsing, or bytecode generation of your `.py` file begins. Only after this initialization phase does CPython read and compile the Python source code.

```
1) OS starts python3 process
  |
  ├── sets up stack, heap, registers
  ├── loads python3 binary
  └── jumps to main()

2) CPython initializes runtime
  |
  ├── memory manager
  ├── garbage collector
  ├── built-in types
  ├── import system
  └── Python Virtual Machine

3) Python reads your .py file
  |
  ├── lexing
  ├── parsing
  ├── bytecode generation
  └── execution by PVM
```

---

# 2 - Lexing (Lexical Analysis / Tokenization)

Lexing is the first step in Python's frontend, performed by the Python compiler component inside the CPython interpreter, where the raw source code is converted into a stream of meaningful symbols called **tokens**. It turns the human-readable text into units the parser can understand.

You can see the lexer's output by running: `python3 -m tokenize test.py`

**What the lexer does:**
- Reads characters left to right
- Groups them into tokens based on rules
- Ignores irrelevant characters (spaces, comments)
- Tracks indentation (very important in Python)

**Token types include:**
- Keywords: `if`, `for`, `def`, `return`
- Identifiers: variable and function names
- Literals: numbers, strings
- Operators: `+ - * / =`
- Delimiters: `() [] {} , :`
- Structural tokens: `INDENT`, `DEDENT`, `NEWLINE`

**Example source file (`test.py`):**

```python
## test.py

a = 2
x = a + 3

def greet(name):
    print("Hello,", name)

greet("Alice")
```

**Tokenizer output:**

```
└─$ python3 -m tokenize test.py
0,0-0,0:            ENCODING       'utf-8'
1,0-1,9:            COMMENT        '# test.py'
1,9-1,10:           NL             '\n'
2,0-2,1:            NAME           'x'
2,2-2,3:            OP             '='
2,4-2,5:            NAME           'a'
2,6-2,7:            OP             '+'
2,8-2,9:            NUMBER         '3'
2,9-2,10:           NEWLINE        '\n'
3,0-3,1:            NL             '\n'
4,0-4,3:            NAME           'def'
4,4-4,9:            NAME           'greet'
4,9-4,10:           OP             '('
4,10-4,14:          NAME           'name'
4,14-4,15:          OP             ')'
4,15-4,16:          OP             ':'
4,16-4,17:          NEWLINE        '\n'
5,0-5,4:            INDENT         '    '
5,4-5,9:            NAME           'print'
5,9-5,10:           OP             '('
5,10-5,18:          STRING         '"Hello,"'
5,18-5,19:          OP             ','
5,20-5,24:          NAME           'name'
5,24-5,25:          OP             ')'
5,25-5,26:          NEWLINE        '\n'
6,0-6,1:            NL             '\n'
7,0-7,0:            DEDENT         ''
7,0-7,5:            NAME           'greet'
7,5-7,6:            OP             '('
7,6-7,13:           STRING         '"Alice"'
7,13-7,14:          OP             ')'
7,14-7,15:          NEWLINE        '\n'
8,0-8,0:            ENDMARKER      ''
```

---

# 3 - Parsing (Syntactic Analysis)

Parsing is the second step in Python's frontend, performed by the Python compiler component inside the CPython interpreter. The stream of tokens from the lexer is analyzed according to Python's grammar rules. The parser checks that the code is structurally correct and builds a hierarchical representation called an **Abstract Syntax Tree (AST)**, which captures the meaning of the program without irrelevant details like whitespace or comments.

- **Input:** stream of tokens from the lexer
- **Output:** Abstract Syntax Tree (AST)
- Run: `python3 -m ast test.py`

**What the parser does:**
- Ensures tokens follow Python's syntax rules
- Organizes statements, expressions, and blocks hierarchically
- Detects syntax errors (e.g., missing colons, unmatched parentheses, incorrect indentation)
- Removes unnecessary details like formatting and comments

**AST output:**

```
└─$ python3 -m ast test.py
Module(
  body=[
     Assign(
        targets=[
           Name(id='x', ctx=Store())],
        value=BinOp(
           left=Name(id='a', ctx=Load()),
           op=Add(),
           right=Constant(value=3))),
     FunctionDef(
        name='greet',
        args=arguments(
           args=[
              arg(arg='name')]),
        body=[
           Expr(
              value=Call(
                 func=Name(id='print', ctx=Load()),
                 args=[
                    Constant(value='Hello,'),
                    Name(id='name', ctx=Load())]))]),
     Expr(
        value=Call(
           func=Name(id='greet', ctx=Load()),
           args=[
              Constant(value='Alice')]))])
```

---

# 4 - Bytecode Generation and Inspection

After parsing, Python converts the AST into **bytecode** — a low-level, platform-independent set of instructions that the Python Virtual Machine (PVM) can execute. Bytecode is analogous to assembly code in C, but instead of running directly on the CPU, it runs on Python's virtual machine. The PVM interprets Python bytecode and executes its semantics by invoking precompiled C functions, whose machine instructions are executed directly by the CPU.

Inspect bytecode with: `python3 -m dis test.py`

**What this shows:**
- Each line represents a single bytecode instruction.
- Instructions include operations like:
  - `LOAD_NAME` → load a variable
  - `LOAD_CONST` → load a constant value
  - `BINARY_ADD` → perform addition
  - `STORE_NAME` → store a value in a variable
  - `RETURN_VALUE` → return from a function or module

**Disassembly output:**

```
└─$ python3 -m dis test.py
 0           RESUME                   0

 2           LOAD_NAME                0 (a)
             LOAD_CONST               0 (3)
             BINARY_OP                0 (+)
             STORE_NAME               1 (x)

 4           LOAD_CONST               1 (<code object greet at 0x7f2cc09375a0, file "test.py", line 4>)
             MAKE_FUNCTION
             STORE_NAME               2 (greet)

 7           LOAD_NAME                2 (greet)
             PUSH_NULL
             LOAD_CONST               2 ('Alice')
             CALL                     1
             POP_TOP
             RETURN_CONST             3 (None)

Disassembly of <code object greet at 0x7f2cc09375a0, file "test.py", line 4>:
 4           RESUME                   0

 5           LOAD_GLOBAL              1 (print + NULL)
             LOAD_CONST               1 ('Hello,')
             LOAD_FAST                0 (name)
             CALL                     2
             POP_TOP
             RETURN_CONST             0 (None)
```

---

# 5 - Execution by the Python Virtual Machine (PVM)

After Python generates bytecode, it does not run directly on your CPU. Instead, the **Python Virtual Machine (PVM)** executes it. The PVM is the interpreter inside Python that reads bytecode instructions one by one and performs the corresponding operations in memory.

**How the PVM works:**
1. **Fetch** – Reads the next bytecode instruction.
2. **Decode** – Understands what operation it represents (addition, function call, variable assignment).
3. **Execute** – Performs the operation using Python's runtime environment:
   - Manages memory for variables and objects
   - Handles function calls and returns
   - Performs arithmetic and logical operations
   - Manages control flow (`if`, `for`, `while`)

**Key features of the PVM:**
- **Platform-independent** – Runs the same bytecode on different systems.
- **Manages memory automatically** – Includes garbage collection for unused objects.
- **Handles dynamic features** – Python can create and modify objects at runtime, including functions and classes.
- **Interpreted execution** – Unlike C, instructions are executed one at a time instead of being compiled to machine code beforehand.

**Example — how `x = a + 3` is executed:**

```python
LOAD_NAME a       # Fetch the value of 'a'
LOAD_CONST 3      # Load constant 3
BINARY_ADD        # Add them together
STORE_NAME x      # Store the result in 'x'
```

**The PVM's evaluation loop (C pseudocode):**

```c
while (1) {
   opcode = *ip++;
   switch (opcode) {
       case LOAD_FAST:
           ...
       case BINARY_ADD:
           ...
   }
}
```

When the PVM encounters the `BINARY_ADD` opcode, it invokes a C function that pops operands from the stack, performs the addition, and pushes the result back. Thus, the actual CPU instructions executed originate from precompiled C code, not from any direct translation of Python bytecode.

---

# 6 - Generate .pyc Files

In Python, you can compile a script to bytecode without executing it. This produces a `.pyc` file, which is a cached version of the bytecode. Python uses these files to speed up future executions by loading the precompiled bytecode instead of recompiling the source code each time.

```bash
python3 -m py_compile your_file.py
## Output: __pycache__/your_file.cpython-312.pyc
```

**`.pyc` is not machine code.** A `.pyc` file looks like random symbols, but it stores Python bytecode — a low-level, platform-independent instruction set for the PVM. The CPU never runs `.pyc` directly; the PVM reads and interprets the bytecode.

**The "compiler" word:** Python does have a compilation step, but it is very different from C/C++ compilation. The Python compiler converts source code (`.py`) into tokens, then into an AST, and finally into bytecode. It can optionally save this bytecode to a `.pyc` file. Unlike C or C++, Python does **not** generate machine code at this stage.

**Library modules** are included at runtime, not compile time. When the PVM encounters an `import` statement, it searches for the module in the standard library, `site-packages`, and the current working directory. If a `.pyc` file exists, Python loads the precompiled bytecode; otherwise it compiles the `.py` source first.

---

# Python Interpreter Execution Flow

```
┌─────────────────────────────────────┐
│ 1. OS starts CPython                │
│    OS locates python3, loads binary │
└─────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────┐
│ 2. CPython runtime initialization   │
│    Initialize memory, GC, types, PVM│
└─────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────┐
│ 3. Read source code (.py file)      │
│    CPython reads Python source file │
└─────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────┐
│ 4. Lexing & Parsing                 │
│    Tokenize → parse into AST        │
└─────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────┐
│ 5. Bytecode generation              │
│    Compile AST into bytecode        │
└─────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────┐
│ 6. Create the __main__ module       │
│    Setup namespace, __name__="..."  │
└─────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────┐
│ 7. Execute bytecode (PVM)           │
│    PVM runs instructions, output    │
└─────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────┐
│ 7.1. Library Modules                │
│    (Imported at runtime)            │
└─────────────────────────────────────┘
```

---

# Python3 Command

`python3` is the command used to run the Python3 interpreter. It exists because Python has two major versions: Python 2 (now obsolete) and Python 3 (current and supported). On many systems, `python` may refer to Python 2 or may not exist at all, so `python3` is provided to explicitly run Python 3.

When you run `python3` and see something like `[GCC 15.2.0]`, this does not mean GCC is compiling your Python code; it simply indicates that the Python interpreter itself (CPython) was compiled using GCC 15.2.0.

On most modern systems, `python`, `python3`, and `python3.13` all ultimately execute the same binary executable, all found in `/usr/bin/`.

---
# Data Types sizes

In Python, variable **types do not have fixed sizes like in C or Java**. Python objects are dynamic and include metadata (reference count, type pointer, etc.), so their memory size is larger and can vary by system and Python build.

You can inspect the size of an object using:

```python
import sys
sys.getsizeof(obj)
```

Below are **typical sizes on a 64-bit CPython implementation** (values may vary slightly).
# Numeric types

| Type      | Example | Typical size                |
| --------- | ------- | --------------------------- |
| `bool`    | `True`  | 28 bytes                    |
| `int`     | `10`    | 28 bytes (grows with value) |
| `float`   | `3.14`  | 24 bytes                    |
| `complex` | `2+3j`  | 32 bytes                    |

Notes:

* `int` uses **arbitrary precision**, so very large numbers require more memory.
* `float` follows **IEEE-754 double precision (64 bits)** internally.

# Text type

| Type  | Example | Typical size           |
| ----- | ------- | ---------------------- |
| `str` | `"a"`   | ~50 bytes + characters |

Strings grow with length because characters are stored internally.

Example:

```python
sys.getsizeof("a")      # ~50 bytes
sys.getsizeof("hello")  # larger
```

# Container types

| Type    | Example   | Typical size         |
| ------- | --------- | -------------------- |
| `list`  | `[1,2,3]` | ~56 bytes + elements |
| `tuple` | `(1,2,3)` | ~48 bytes + elements |
| `set`   | `{1,2,3}` | ~216 bytes           |
| `dict`  | `{"a":1}` | ~232 bytes           |

Notes:

* Containers store **references to objects**, not the objects themselves.
* Their size grows as elements are added.

# Special types

| Type       | Example     | Typical size |
| ---------- | ----------- | ------------ |
| `NoneType` | `None`      | ~16 bytes    |
| `range`    | `range(10)` | ~48 bytes    |

# Example measurement

```python
import sys

# numeric types
print("int:", sys.getsizeof(10))
print("bool:", sys.getsizeof(True))
print("float:", sys.getsizeof(3.14))

# string
print("empty str:", sys.getsizeof(""))
print("char:", sys.getsizeof("a"))
 
# containers
print("empty list:", sys.getsizeof([]))
print("list:", sys.getsizeof([1, 2, 3]))
print("empty tuple:", sys.getsizeof(()))
print("tuple:", sys.getsizeof((1, 2, 3)))
print("empty set:", sys.getsizeof({}))
print("set:", sys.getsizeof({1, 2, 3}))
print("dict:", sys.getsizeof({"a": 1, "b": 2}))
 
# special types

print("None:", sys.getsizeof(None))
print("range:", sys.getsizeof(range(10)))
```

**output**:

```
int: 28
bool: 28
float: 24
empty str: 41
char: 42
list: 88
list: 56
tuple: 64
set: 216
dict: 184
None: 16
range: 48
```

✅ **Key takeaway**

Python objects include **extra memory for metadata**, so they are much larger than primitive types in low-level languages. Sizes are **implementation-dependent** and **can grow dynamically**.

---
# Type hints

**Type hints** in Python are annotations that specify the **expected type of variables, function parameters, and return values**. They help with **readability, static analysis, and tooling**, but they **do not change how Python runs the code** (Python does not enforce them at runtime by default).
## 1. Function type hints

You annotate parameters and return values.

```python
def add(a: int, b: int) -> int:
    return a + b
```

Meaning:

* `a: int` → `a` should be an integer
* `b: int` → `b` should be an integer
* `-> int` → function returns an integer

Python will still run this:

```python
add("3", "4")   # no runtime error from type hints
```

Type hints are mainly checked by tools like mypy.

## 2. Variable type hints

You can annotate variables.

```python
age: int = 17
name: str = "Alex"
```

You can also declare without assigning:

```python
score: float
```


## 3. Container types

For lists, dictionaries, sets, etc.

```python
numbers: list[int] = [1, 2, 3]
names: list[str] = ["Ali", "Sara"]

scores: dict[str, int] = {
    "Ali": 90,
    "Sara": 95
}
```

Meaning:

* `list[int]` → list containing integers
* `dict[str, int]` → dictionary with `str` keys and `int` values


## 4. Multiple possible types

Using `|` (Python 3.10+):

```python
def parse(value: int | str) -> int:
    return int(value)
```

Before 3.10 this used:

```python
from typing import Union
Union[int, str]
```

## 5. Optional values

If something can be `None`:

```python
def find_user(name: str) -> str | None:
    ...
```

Meaning the function returns either:

* `str`
* `None`
---
# The `typing` module 

In Python there are **two common styles of type hints**:

1. **Built-in generic types (lowercase)** (The one mentioned before)
2. **Typing module types (Capitalized, imported)**

Before Python 3.9, containers could **not be subscripted with types**, so Python introduced the typing module.

Example:

```python
from typing import List, Dict, Set

numbers: List[int]
scores: Dict[str, int]
tags: Set[str]
```

Here:

* `List`
* `Dict`
* `Set`

are **typing objects**, not the real containers.

---

## Direct comparison

| Modern style      | Old typing style  |
| ----------------- | ----------------- |
| `list[int]`       | `List[int]`       |
| `dict[str, int]`  | `Dict[str, int]`  |
| `set[str]`        | `Set[str]`        |
| `tuple[int, str]` | `Tuple[int, str]` |

Modern Python prefers the **left column**.

---

## When you still need `typing`

Some types **only exist in `typing`**, for example:

```python
from typing import Iterable, Iterator, Generator, Callable
```

Example:

```python
from typing import Iterable

def process(data: Iterable[int]) -> None:
    ...
```

`Iterable` is **not a built-in container**, so it must come from `typing`.

---
# Module

A **module** is a Python file that contains reusable code, including functions, classes, and variables, designed to organize and simplify your programs. Modules can be:
- **Standard** – built-in modules like `math`, `os`
- **Third-party** – installed via package managers like `pip`
- **Custom** – created by users

Modules promote modularity and code reuse, reducing redundancy and improving maintainability.

**Example (`Test1.py`):**

```python
def function(number):
    print(f"From Test{number}: {number * 10}")

function(1)
```

**Example (`Test2.py`):**

```python
import Test1

Test1.function(2)
```

**Output:**
```
python Test2.py

From Test1: 10
From Test2: 20
```

---

# How Python Executes a Module on Import

When a Python file is executed, the interpreter processes its statements sequentially from top to bottom. If it encounters an `import` statement, Python temporarily pauses execution and begins loading the target module. During this import process, Python:

1. Creates a module object
2. Sets the module's `__name__` variable to the module's filename (e.g., `"file"`)
3. Executes all top-level code line by line
4. Evaluates any conditional logic that depends on `__name__`
5. Stores the fully executed module in `sys.modules` for caching

Only after this execution finishes does Python make the module's functions, variables, and objects available to the importing file. **A useful mental model: "import means run the file, then give me access to its names."**

**Example (`one.py`):**

```python
print("one.py: top-level code running")

def hello():
    print("Hello from one")

print("one.py: module loaded")
```

**Example (`two.py`):**

```python
print("two.py: start")

import one

print("two.py: after import")

one.hello()
```

**Output:**
```
python two.py

two.py: start
one.py: top-level code running
one.py: module loaded
two.py: after import
Hello from one
```

**Module execution timeline (import case):**

```
two.py starts
└── two.py set (__name__="__main__")
└── import one.py
     ├── create module object and set __name__ = "one"
     ├── execute top-level code
     └── store in sys.modules
└── continue two.py
```

**What happens when you import `one` inside `two.py`:**

1. Python is already running: `two.py` is executing inside the `__main__` module object.
2. Python sees: `import one`.
3. Python checks: Is `"one"` already loaded? It looks in `sys.modules`.
4. If NOT loaded yet:
   - Python creates a new Module object named `"one"`, sets `__name__ = "one"`
   - Reads `one.py`, lexes → parses → compiles → executes it
   - Stores the executed module object in `sys.modules`
5. Execution returns to `two.py`. `one` is now a reference to that module object.
6. If imported again: Python does NOT re-run the file. It reuses the existing module object.

---

# Module Object Inspection

**`t1.py`:**

```python
#t1.py
import t2

print("Module object:", t2)
print("Type of module:", type(t2))
print("Module attributes:", dir(t2))
print("Access Module attributes:", t2.x)
print("Access Module attributes:", t2.__name__)

import sys

print("\nIs 't2' in sys.modules?", "t2" in sys.modules)
print("sys.modules['t2']:", sys.modules["t2"])
```

**`t2.py`:**

```python
#t2.py
print("t2 module is being executed")

x = 10

def greet():
    print("Hello from t2")
```

**Output:**

```
python t1.py

t2 module is being executed
Module object: <module 't2' from '/home/kali/1337/python/test/t2.py'>
Type of module: <class 'module'>
Module attributes: ['__builtins__', '__cached__', '__doc__', '__file__', '__loader__', '__name__', '__package__', '__spec__', 'greet', 'x']
Access Module attributes: 10
Access Module attributes: t2

Is 't2' in sys.modules? True
sys.modules['t2']: <module 't2' from '/home/kali/1337/python/test/t2.py'>
```

---

# `if __name__ == "__main__"`

The `if __name__ == "__main__":` check determines whether the Python script is being **run directly** or **imported as a module**.

- When run directly → `__name__` is set to `"__main__"`
- When imported → `__name__` is set to the name of the module

**`Test1.py`:**

```python
def greet():
    print("Hello from Test1!")

if __name__ == "__main__":
    greet()
    print(f'Script is being run directly, in \"{__name__}\" file')
else:
    print(f"This Code is Imported to {__name__} file")
```

**`Test2.py`:**

```python
import Test1
Test1.greet()
```

**Output when running `Test1.py` directly:**
```
pyhton Test1.py

Hello from Test1!
Script is being run directly, in "__main__" file
```

**Output when running `Test2.py` (which imports `Test1`):**
```
pyhton Test2.py

This Code is Imported to Test1 file
Hello from Test1!
```

---

# How Python Stores Variables, Functions, and Classes

Python uses both a **stack** and a **heap**, but in a highly abstracted and automatic way compared to languages like C. In Python, **everything is an object**, including integers, strings, functions, classes, modules, lists, and dictionaries.

Variables do not store values directly; they store **references (pointers)** to objects in memory. All objects are allocated in the **heap**. Variables themselves exist as references within a namespace (such as a local stack frame or module scope), pointing to those heap-allocated objects.

When `x = 10` is executed:
- Python creates (or reuses) the integer object `10` in the heap
- The name `x` becomes bound to that object
- If you write `x = 20`, Python binds `x` to a **different** integer object `20` — the original `10` object remains unchanged (because integers are immutable)

Global variables are maintained in a **module namespace dictionary** accessible via `globals()`. Every scope in Python is backed by a dictionary (`module.__dict__`, `frame.f_locals`, `class.__dict__`, or `object.__dict__`) rather than fixed memory segments. Python manages memory automatically using reference counting and a garbage collector.

**`*1` — Everything is an object:**

```python
## Integers
x = 10
print(x, "→ type:", type(x), "→ id:", hex(id(x)))

## Strings
s = "hello"
print(s, "→ type:", type(s), "→ id:", hex(id(s)))

## Lists
lst = [1, 2, 3]
print(lst, "→ type:", type(lst), "→ id:", hex(id(lst)))

## Dictionaries
d = {"a": 1, "b": 2}
print(d, "→ type:", type(d), "→ id:", hex(id(d)))

## Functions
def greet():
    return "Hello"
print(greet, "→ type:", type(greet), "→ id:", hex(id(greet)))

## Classes
class Car:
    pass
print(Car, "→ type:", type(Car), "→ id:", hex(id(Car)))

## Modules
import math
print(math, "→ type:", type(math), "→ id:", hex(id(math)))
```

**Output:**
```
10 → type: <class 'int'> → id: 0x73753836c430
hello → type: <class 'str'> → id: 0x737537a34d80
[1, 2, 3] → type: <class 'list'> → id: 0x73753794e640
{'a': 1, 'b': 2} → type: <class 'dict'> → id: 0x7375379d7fc0
<function greet at 0x7375379731a0> → type: <class 'function'> → id: 0x7375379731a0
<class '__main__.Car'> → type: <class 'type'> → id: 0x62a2f96b8c50
<module 'math' from '...'> → type: <class 'module'> → id: 0x737537a3a840
```

**`*2` — Immutable vs mutable objects:**

```python
## Immutable object (int)
x = 10
y = x

print("x id:", hex(id(x)))
print("y id:", hex(id(y)))

## Change y
y = 20

print("After changing y:")
print("x id:", hex(id(x)), "x value:", x)
print("y id:", hex(id(y)), "y value:", y)

## Mutable object (list)
a = [1, 2, 3]
b = a

print("Before change:")
print("a id:", hex(id(a)), "b id:", hex(id(b)))

## Modify b — it's like a is modified
b.append(4)

print("After change:")
print("a:", a, "a id:", hex(id(a)))
print("b:", b, "b id:", hex(id(b)))
```

**Output:**
```
x id: 0xa4fc30
y id: 0xa4fc30
After changing y:
x id: 0xa4fc30 x value: 10
y id: 0xa4fd70 y value: 20
Before change:
a id: 0x7ffa63906480 b id: 0x7ffa63906480
After change:
a: [1, 2, 3, 4] a id: 0x7ffa63906480
b: [1, 2, 3, 4] b id: 0x7ffa63906480
```

**`*3` — globals():**

```python
## Global variable
x = 42
y = "hello"

def fun():
    not_g = 15
    print(not_g)

print("Global variables stored in globals():")
print(globals())
print("Access x via globals():", globals()['x'])
print("Access y via globals():", globals()['y'])
```

**Output:**
```
{'__name__': '__main__', ..., 'x': 42, 'y': 'hello', 'fun': <function fun at 0x...>}
Access x via globals(): 42
Access y via globals(): hello
```

---

# Shebang

A **shebang** is a special line at the very beginning of a script that starts with `#!`. It is a Unix convention indicating which interpreter should be used to execute the file.

Although the shebang looks like a comment in Python and is ignored by the Python interpreter itself, it is read by the **operating system** before execution. When a script is run directly, the OS reads the shebang, locates the specified interpreter, and executes the file using that interpreter.

```python
#!/usr/bin/env python3
```

Using `#!/usr/bin/env python3` does **not** mean that the `python3` interpreter is located inside `/usr/bin/env`. Instead, `/usr/bin/env` is a standard Unix utility that searches for the specified program in the system's `PATH` environment variable.

Alternatively, you can use a direct shebang like `#!/usr/bin/python3`, which explicitly specifies the absolute path. This works on systems where Python is installed at that exact location but is less portable. Using `#!/usr/bin/env python3` is generally preferred because it allows the script to locate the appropriate Python interpreter dynamically.

---

# Try & Except

Python provides a structured mechanism to handle runtime errors using the `try` statement and its associated blocks.

- **`try` block:** Code that might cause an exception. If an error occurs, Python immediately stops executing the `try` block and looks for a matching `except` block.
- **`except` block:** Handles specific exceptions. You can catch individual exception types or multiple types in a single block. If the exception is not caught, Python propagates it up the call stack.
- **`else` block:** Optional. Runs only if the `try` block succeeds without raising any exceptions.
- **`finally` block:** Optional but always executes, regardless of whether an exception occurred. Used for cleanup tasks (closing files, releasing resources).

**Example 1 — basic try/except/else/finally:**

```python
try:
    print("Trying to water plants...")
    plant = None
    if not plant:
        print("Error: No plant to water!")
except ValueError:
    print("Caught a ValueError!")
else:
    print("Watering successful!")
finally:
    print("Closing watering system (cleanup)")
```

**Example 2 — unhandled exception (finally still runs):**

```python
try:
    x = 10 / 0  # This raises ZeroDivisionError
except ValueError:
    print("Caught a ValueError!")
finally:
    print("This always runs!")
```

---

# Custom Exception Class

A **custom exception class** is a user-defined class that inherits from the built-in `Exception` class and is used to represent application-specific error conditions. In Python, only classes that inherit from `BaseException` can be raised or caught as exceptions.

**Basic example:**

```python
class Exeption(Exception):
    ...

def set_age(age):
    if age < 0:
        raise Exeption("Age cannot be negative")
    print(f"Age set to {age}")

try:
    set_age(-5)
except Exeption as e:
    print("Error:", e)
```

---

# Exception Inheritance in Python

Exception classes fully support inheritance. All exceptions ultimately inherit from `BaseException`, with most application-level exceptions inheriting from `Exception`. Custom exceptions can inherit from other exceptions, creating a logical hierarchy.

**Example — GardenError hierarchy:**

```python
class GardenError(Exception):
    pass

class PlantError(GardenError):
    pass

class WaterError(GardenError):
    pass

errors = [PlantError(), WaterError()]

for error in errors:
    try:
        raise error
    except GardenError:
        print("Caught a garden error")
```

---

# Custom Exception Classes and Messages

Using `super().__init__(message)` within the constructor calls the parent class's constructor, ensuring the message is properly passed up the inheritance chain.

```python
## --- Custom Exception Classes ---
class GardenError(Exception):
    pass

class PlantError(GardenError):
    def __init__(self, message="There is a problem with a plant!"):
        super().__init__(message)

def check_plant():
    raise PlantError("The tomato plant is wilting!")

try:
    check_plant()
except PlantError as e:
    print(f"Caught PlantError: {e}")

try:  # Catching all garden-related errors
    check_plant()
except GardenError as e:
    print(f"Caught a garden error: {e}")
```

`super().__init__(message)` means:

> "Call the `__init__` method of the parent class of `PlantError` and pass `message` to it."

The parent here is Exception through GardenError.

## What `super()` Actually Returns

`super()` returns a **proxy object** that lets you access methods of the **next class in the Method Resolution Order (MRO)**.

For your classes, the MRO looks like this:

```text
PlantError
   ↓
GardenError
   ↓
Exception
   ↓
BaseException
   ↓
object
```

So when Python sees:

```python
super().__init__(message)
```

it interprets it as:

```python
GardenError.__init__(self, message)
```

But since `GardenError` doesn't define `__init__`, Python continues up the chain until it finds one in `Exception`.

## Why It Looks Like "Sending a Message"

It feels like sending a message because the call is **forwarded up the inheritance chain**.

Conceptually:

```text
PlantError.__init__()
      │
      ▼
super() → ask next class in MRO
      │
      ▼
Exception.__init__(message)
```

So instead of directly naming the parent class, `super()` says:

> "Let the next class in the hierarchy handle this."

When the message reaches the base exception classes, it is **stored inside the exception object** so Python can later display or retrieve it.

Your call:

```python
super().__init__(message)
```

eventually reaches the constructor of Exception (which inherits from BaseException).

## What the Base Exception Does With the Message

Internally, the constructor stores the message inside the attribute **`args`**.

Conceptually it behaves like this:

```python
class BaseException:
    def __init__(self, *args):
        self.args = args
```

So when your code runs:

```python
raise PlantError("The tomato plant is wilting!")
```

the base exception receives:

```text
args = ("The tomato plant is wilting!",)
```

and stores it inside the exception object.

## Where the Message Lives

After creation, the exception object looks roughly like this:

```text
PlantError instance
├── args = ("The tomato plant is wilting!",)
└── type = PlantError
```

## Why `print(e)` Shows the Message

When you do:

```python
except PlantError as e:
    print(e)
```

Python calls the exception's `__str__()` method (defined in BaseException).

That method returns a string based on `args`.

Conceptually:

```python
def __str__(self):
    return str(self.args[0])
```

So Python prints:

```
The tomato plant is wilting!
```

## You Can Access It Directly

Example:

```python
try:
    raise PlantError("The tomato plant is wilting!")
except PlantError as e:
    print(e.args)
```

Output:

```
('The tomato plant is wilting!',)
```

## Full Flow in Your Program

```
raise PlantError("The tomato plant is wilting!")
        │
        ▼
PlantError.__init__(message)
        │
        ▼
super().__init__(message)
        │
        ▼
Exception.__init__(message)
        │
        ▼
BaseException.__init__(message)
        │
        ▼
self.args = ("The tomato plant is wilting!",)
```

Later:

```
print(e)
   │
   ▼
BaseException.__str__()
   │
   ▼
returns "The tomato plant is wilting!"
```

 **Summary**

The base exception doesn't "handle" the message in a complex way. It simply:

1. **stores it in `args`**
2. **returns it when the exception is printed**

---

# Comprehensions & Generators

In Python there are **two ways to create generators**, and they differ mainly by **syntax**, not by concept. Both produce **lazy iterators** that generate values on demand.

  

These generators are part of the iteration system defined in the Python iterator protocol.

## 1. Generator Expressions (Comprehension-style)

  

These look like **list comprehensions but with parentheses**.

  

Example:

  

```python
gen = (x * x for x in range(5))
```

  

This creates a **generator object** that produces values one by one.

  

Equivalent step-by-step use:

  

```python

print(next(gen)) # 0

print(next(gen)) # 1

print(next(gen)) # 4

```

  

Key points:

  

* Uses **parentheses `( )`**

* Looks like a comprehension

* No `yield`

* Produces values lazily

  

Comparison:

  

```python

[x*x for x in range(5)] # list comprehension → builds full list

(x*x for x in range(5)) # generator expression → lazy generator

```

Memory difference:

```text

List comprehension → stores all values

Generator expression → produces values when needed

```



## 2. Generator Functions (`yield`)

  

These are **normal functions that use `yield`**.

  

Example:

  

```python
def squares(n):
	for i in range(n):
		yield i * i
```

  

Calling the function **does not run it immediately**:

  

```python
gen = squares(5)
```

  

Instead it returns a **generator object**.

  

Execution happens when iterated:

  

```python
for x in gen:
	print(x)
```

  

Key mechanism:

  

* `yield` **pauses the function**

* State is preserved

* Execution resumes on the next `next()` call


## Execution Model of Generator Functions

  

Normal function:

  

```text

call → run entire function → return result

```

  

Generator function:

  

```text

call → create generator object

next() → run until yield

next() → resume from yield

```


## Conceptual Equivalence

  

These two are roughly equivalent:

  

Generator expression:

  

```python
gen = (x*x for x in range(5))
```

  

Generator function:

  

```python
def gen():
	for x in range(5):
yield x*x
```

  

Both produce a **generator object**.

## Summary

  

| Feature    | Generator Expression | Generator Function |
| ---------- | -------------------- | ------------------ |
| Syntax     | `(x for x in ...)`    | `def ... yield`    |
| Complexity | Simple cases         | Complex logic      |
| State      | Automatic            | Explicit control   |
| Code size  | Short                | Longer             |
 **Key idea**

  

Both produce **generator objects** implementing the iterator protocol:

  

```text
__iter__()
__next__()
```

  

The difference is **syntax and complexity of the logic**.

  

---

**Generator vs normal function:**

```python
## Generator function example
def generate_numbers():
    print("Generator starting")
    for i in range(3):
        print(f"Producing {i}")
        yield i  # pauses here and produces value only when requested

## Create a generator object (the function will not execute)
gen_obj = generate_numbers()
print("Generator created (nothing produced yet)\n")

## Normal function example
def normal_numbers():
    print("Normal function starting")
    for i in range(3):
        print(f"Producing {i}")
        return i  # returns immediately, loop stops after first iteration

## Call normal function
normal_result = normal_numbers()
```

**Output:**
```
Generator created (nothing produced yet)

Normal function starting
Producing 0
```

---

**List comprehension vs generator expression:**

```python
## List comprehension (eager)
numbers = range(5)

## This builds the full list immediately
squares_list = [n * n for n in numbers]
print("List comprehension (all values computed at once):")
print(squares_list)

## Generator expression (lazy)
## This creates a generator object; values are computed only when requested
squares_gen = (n * n for n in numbers)
print("\nGenerator expression (values computed lazily):")
print(squares_gen)  # just shows generator object

## Iterate to get values one by one
print("Iterating generator:")
for val in squares_gen:
    print(val)
```

**Output:**
```
List comprehension (all values computed at once):
[0, 1, 4, 9, 16]

Generator expression (values computed lazily):
<generator object <genexpr> at 0x7f7a53feeb50>
Iterating generator:
0
1
4
9
16
```

---

**`lazy_squares` generator:**

```python
## Generator function that produces squares
def lazy_squares(n):
    for i in range(n):
        print(f"Computing square of {i}")
        yield i * i  # value is produced only when requested

## Create generator object
squares = lazy_squares(5)

print("Generator created")  # Nothing has been computed yet

## Request the first value
print(next(squares))

## Request the next value
print(next(squares))

## Iterate over the remaining values
for val in squares:
    print(val)
```

**Output:**
```
Generator created
Computing square of 0
0
Computing square of 1
1
Computing square of 2
4
Computing square of 3
9
Computing square of 4
16
```

---

**`counter` generator — state and StopIteration:**

```python
def counter():
    print("Start")
    yield 1
    print("Middle")
    yield 2
    print("End")
    return 99

try:
    g = counter()
    print(g)
    print(next(g))
    print(g)
    print(next(g))
    print(g)
    print(next(g))
except StopIteration:
    print("Generator finished")
```

**Output:**
```
<generator object counter at 0x776f10a402e0>
Start
1
<generator object counter at 0x776f10a402e0>
Middle
2
<generator object counter at 0x776f10a402e0>
End
Generator finished
```

A generator function returns a **single generator object** that remains the same throughout its lifetime. It does not store the values it yields. The generator preserves only its execution state: the current instruction pointer, the function's local variables, and its call stack frame.

---

**For loop vs manual `next()`:**

```python
def numbers():
    print("Generator started")
    for i in range(1, 3):
        print(f"Yielding {i}")
        yield i
    print("Generator finished")


print("=== Using for loop ===")
for n in numbers():
    print(f"Received: {n}")


print("\n=== Using manual next() ===")
gen = numbers()  # create generator once

try:
    for _ in range(10):
        value = next(gen)
        print(f"Received: {value}")
except StopIteration:
    print("Generator exhausted")
```

**Output:**
```
=== Using for loop ===
Generator started
Yielding 1
Received: 1
Yielding 2
Received: 2
Generator finished

=== Using manual next() ===
Generator started
Yielding 1
Received: 1
Yielding 2
Received: 2
Generator finished
Generator exhausted
```

---
# The `next()` Function
In Python, `next()` is a **built-in function** used to **retrieve the next item from an iterator**. It is the standard way to move through any object that implements the **iterator protocol**.


## 1. Syntax

```python id="fr5p38"
next(iterator[, default])
```

* `iterator` → an **iterator object**, like a generator or anything with `__next__()`.
* `default` → optional value returned if the iterator is exhausted (instead of raising `StopIteration`).


## 2. What It Does

* Calls the iterator's `__next__()` method internally.
* Returns the **next value** in the sequence.
* If there are no more values:

  * Without `default` → raises `StopIteration`
  * With `default` → returns the `default` value


## 3. Example With a Generator Expression

```python id="1w9e8p"
gen = (x*x for x in range(2))

print(next(gen))  # 0
print(next(gen))  # 1
print(next(gen))  # StopIteration
```

Here, `next()` **pauses the generator, resumes it, and returns the next value**. When no more values exist, it raises `StopIteration`.



## 4. Using Default to Avoid StopIteration and Using `next()` with a List

next() works with any iterator, not just generators. In Python, lists, tuples, dictionaries, and sets can all provide iterators.

```python id="eqm6sy"
gen = iter([1, 2])
print(next(gen, "end"))  # 1
print(next(gen, "end"))  # 2
print(next(gen, "end"))  # "end"
```
Explanation:

* `iter(numbers)` creates an iterator object.
* `next(it)` retrieves elements one by one from that iterator.
* Lists themselves are **iterable**, but not iterators — that's why we call `iter()` first.


## 5. Behind the Scenes

`next()` essentially does:

```python id="7rnwnu"
iterator.__next__()
```

For a generator, this means:

1. Resume execution from the last `yield`
2. Run until the next `yield` (or end of function)
3. Return the yielded value

For an Iterator (like list, tuple, dict, etc.) Python internally does:

1. **Look at the current position in the iterator**
   Every iterator keeps track of **where it is** in the sequence of elements.

2. **Retrieve the next element** from the underlying iterable

   * For a list, it's the next index in the list.
   * For a dictionary, it's the next key (or value/items if that iterator).
   * For a file, it's the next line.

3. **Advance the internal state** of the iterator so the next call to `next()` will get the following element.

4. **Return that element**.

5. **Raise `StopIteration`** if there are no more elements.

---

# The `iter()` Function

The `iter()` function creates an iterator from an iterable object (list, tuple, string, etc.). An iterator keeps track of its current position and can produce the next item using `next()`.

While generators are already iterators and do not require `iter()`, standard iterables cannot be advanced with `next()` unless first converted using `iter()`.

`iter()` has two forms:
- `iter(obj)` — calls `obj.__iter__()` to produce an iterator
- `iter(callable, sentinel)` — repeatedly invokes a callable until a specified sentinel value is returned

**How Python Does This Internally**

When you call:

```python id="c5jkts"
it = iter(some_iterable)
```

Python roughly does:

```python id="9hgbky"
it = some_iterable.__iter__()
```

* The iterable's `__iter__()` method returns an **iterator object**.
* The iterator object has `__next__()` to give you the next value.

**Example — converting a list to an iterator:**

```python
## A normal list is iterable, but not an iterator
numbers = [10, 20, 30]

## Convert the list into an iterator
it = iter(numbers)

## Manually access elements using next()
print(next(it))  # 10
print(next(it))  # 20
print(next(it))  # 30

## The iterator is now exhausted
## next(it)  # Would raise StopIteration
```

**How `for` uses `iter()` internally:**

```python
## This:
for x in [1, 2, 3]:
    print(x)

## Is roughly equivalent to:
it = iter([1, 2, 3])
while True:
    try:
        x = next(it)
        print(x)
    except StopIteration:
        break
```

**`iter()` with sentinel form:**

```python
it = iter([1, 2, 3])

print(next(it))
print(next(it))
print(next(it))

def f():
    return input()

for line in iter(f, "quit"):
    print("You typed:", line)

## output:
    # 1
    # 2
    # 3
    # test
    # You typed: test
    # quit
```

---

# Command-Line Arguments

**Command-line arguments** are values provided to a Python program at the moment it is executed from the terminal, allowing the user to pass information without modifying the source code. In Python, these arguments are accessed through the `sys.argv` list.

- `sys.argv[0]` → always the program name
- `sys.argv[1:]` → the arguments supplied by the user

```python
## Example command:
## python3 script.py hello 42
## Inside the program:
## sys.argv = ["script.py", "hello", "42"]
```

**Example:**

```python
import sys

print("Program name:", sys.argv[0])

if len(sys.argv) == 1:
    print("No arguments provided")
else:
    i = 1
    while i < len(sys.argv):
        print(f"Arguments {i}: {sys.argv[i]}")
        i += 1
```

**Output:**
```
└─$ python file.py hello 42
Program name: file.py
Arguments 1: hello
Arguments 2: 42

└─$ python file.py
Program name: file.py
No arguments provided
```

---

# Packing & Unpacking

**Packing** collects multiple values into a single composite object. This occurs in function definitions using `*args` and `**kwargs`:
- `*args` → gathers all extra positional arguments into a **tuple**
- `**kwargs` → gathers extra keyword arguments into a **dictionary**

**Unpacking** is the expansion of a composite object into individual elements:
- `*` for iterables
- `**` for mappings

Conceptually, `a, b = iterable` is roughly equivalent to:
```python
it = iter(iterable)
a = next(it)
b = next(it)
```

**Summary:**

```python
## Packing — Positional (*args):
def func(*args):
    print(args)

## Packing — Keyword (**kwargs):
def func(**kwargs):
    print(kwargs)

## Unpacking in Assignment:
a, b, c = [1, 2, 3]

## Extended Unpacking:
a, *middle, c = [1, 2, 3, 4, 5]
all_unique = set().union(*player_sets.values())
```

---

# `isinstance()`

`isinstance()` is a built-in Python function used for **runtime type checking**. It determines whether an object is an instance of a specified class (or a subclass thereof).

**Syntax:** `isinstance(object, classinfo)`
- `object` — the value you want to check
- `classinfo` — a type, class, or a tuple of types
- Returns `True` if the object is an instance of the given type (or its subclass)

```python
x = 10
print(isinstance(x, int))          # True
print(isinstance(x, str))          # False

x = 3.14
print(isinstance(x, (int, float))) # True

class Animal:
    pass

class Dog(Animal):
    pass

d = Dog()

print(isinstance(d, Dog))          # True
print(isinstance(d, Animal))       # True (because Dog inherits from Animal)
```

---

# `all()`

The `all()` function evaluates whether **every element** in an iterable is truthy. It returns `True` if all elements are considered true in a boolean context, and `False` if at least one element is falsy. Falsy values include `False`, `None`, `0`, empty sequences like `[]` or `""`, and empty collections like `{}` or `set()`. If the iterable is empty, `all()` returns `True` by default (vacuous truth).

`all()` has **short-circuit behavior** — evaluation stops as soon as a falsy value is encountered.

```python
print(all([True, True, True]))        # True
print(all([True, False, True]))       # False

nums = [2, 4, 6, 8]
print(all(n % 2 == 0 for n in nums)) # True

def check(x):
    print(f"Checking {x}")
    return x > 0

print(all(check(n) for n in [1, 2, -3, 4]))
```

---

# `.strip()`

The `.strip()` method removes unwanted characters from the **beginning and end** of a string. By default, it strips all types of whitespace (spaces, tabs, newlines, carriage returns, vertical tabs, form feeds). It can also accept a custom string of characters to remove.

```python
## Example 1: Default stripping (whitespace)
user_input = "   Alice \n"
clean_input = user_input.strip()
print(repr(clean_input))  # Output: 'Alice'

## Example 2: Removing specific characters
filename = ">>>report.txt<<<"
clean_filename = filename.strip("><")
print(clean_filename)     # Output: 'report.txt'
```

---

# `sys.stdin`, `sys.stdout`, `sys.stderr`

Special file-like objects from the `sys` module that handle input and output at the system level.

- **`sys.stdin`** — Standard input (usually keyboard). Read data from the user through it.
- **`sys.stdout`** — Standard output (usually the console). `print()` actually writes to `sys.stdout`.
- **`sys.stderr`** — Standard error. Used for error messages or logs. Can be redirected separately from `sys.stdout`.

`readline()` reads a single line from input; `read()` reads all remaining data from the input stream.

**`flush()`** forces any buffered data to be written immediately instead of waiting. Without `flush()`, "Processing..." might appear only after the sleep or after a newline is printed.

**Example:**

```python
import sys

## 1. sys.stdin → reading input
print("Enter your name:", end=' ')
name = sys.stdin.readline().strip()  # reads one line from standard input
print(f"Hello, {name}!\n")

## 2. sys.stdout → writing output
sys.stdout.write("This is written directly to standard output.\n")

## 3. sys.stderr → writing error messages
sys.stderr.write("Warning: This is an error message!\n")
```

**Using flush:**

```python
import sys
import time

sys.stdout.write("Processing...")
sys.stdout.flush()  # forces it to show
time.sleep(2)       # simulate a delay
print("Done")
```

---

# Package

A **package** is a directory that organizes related modules into a structured namespace, allowing large programs to be divided into logical, reusable components. Packages group multiple modules and subpackages together so they can be imported using hierarchical names.

A package typically contains an `__init__.py` file — the **package initializer**. It is executed automatically when the package is imported, allowing Python to recognize the folder as an importable package and to perform setup actions.

**Main roles of `__init__.py`:**
- Control the package's public interface (decides which functions, classes, or variables are exposed)
- Run initialization code
- Define metadata (e.g., `version`, `author`)

---

### How `__init__.py` Controls What the Package Exposes

**`printer.py`:**

```python
def print_hello():
    print("Hello!")

def print_secret():
    print("This is a secret message!")
```

**`__init__.py`:**

```python
from .printer import print_hello
```

**`main.py`:**

```python
import mypackage

mypackage.print_hello()    # works
mypackage.print_secret()   # ERROR
```

`__init__.py` does **not** hide functions completely; it simply controls what is conveniently exposed through the package namespace. Direct module access remains possible (e.g., `mypackage.printer.print_secret()`).

### What Happens During Import

When Python sees for example `import alchemy`, it:
1. Compile the file to bytecode
2. Check cache (`sys.modules`) for `sys.modules["alchemy"]`, If already imported → reuse it, If not → continue.
3. Execute the bytecode and locates folder `alchemy` in `sys.path`
4. Executes `alchemy/__init__.py`
5. Python creates package object (`alchemy = <module object 'alchemy'>`) This object exists **only in RAM while the program runs** as `sys.modules["alchemy"]`. It contains things like:
	- `alchemy.__name__`
	- `alchemy.__package__`
	- `alchemy.__file__`
	- `alchemy.create_fire` (if exposed in `__init__.py`)
6. During the step of compilation (AST to bytecode), Python may save the compiled bytecode `alchemy/__pycache__/__init__.cpython-312.pyc`, this is just a cache to speed up future runs.  It is **not required** for execution.
7. Exposes names defined there
### What is `__pycache__`

`__pycache__` is a directory automatically created by Python to store **compiled bytecode versions (`.pyc` files)** of modules that were **imported during program execution**.

For example, if your structure is:

```
alchemy/
├── __init__.py
├── printer.py
tester.py
```

and `tester.py` contains:

```python
from alchemy import printer
```

When you run:

```
python3 tester.py
```

Python compiles `printer.py` and `__init__.py`, then creates:

```
alchemy/
├── __init__.py
├── printer.py
└── __pycache__/
    ├── __init__.cpython-311.pyc
    └── printer.cpython-311.pyc
```

These `.pyc` files allow Python to **load the module faster in future executions**, because it can reuse the compiled bytecode instead of recompiling the `.py` file every time.
### Namespace and API

**Namespace:** A container that holds names (identifiers) and maps them to objects.

When you write:

```python
from animals.dog import bark
```

`animals` is a **package namespace**.

It’s a container holding modules:

```
animals namespace
└── dog
```

**API (Application Programming Interface):** The set of functions, classes, and variables a module or package exposes for others to use.

```python
## mypackage/__init__.py
from .printer import print_hello
## print_hello() is part of the package's API
## print_secret() in printer.py is internal, not part of the API
```

### Sacred Scroll

In the subject, the **"Sacred Scroll"** is a fancy name for `__init__.py`:
- Transforms a folder into a Python package
- Controls the public API
- Can contain metadata (author, version)
- Decides which functions are exposed and which stay hidden

### Alchemy Package Example

**`elements.py`:**

```python
def create_fire():
    return "Fire element created"

def create_water():
    return "Water element created"

def create_earth():
    return "Earth element created"

def create_air():
    return "Air element created"
```

**`__init__.py`:**

```python
from .elements import create_fire, create_water
```

**`ft_sacred_scroll.py`:**

```python
import alchemy

## Accessible because they were exposed in __init__.py
alchemy.create_fire()
alchemy.create_water()

## Not accessible at package level
## alchemy.secret_spell()  -> AttributeError

## But direct module access still works
alchemy.elements.secret_spell()
```

---

# Import Styles

### 1️⃣ Basic `import module`

Imports the whole module. Access functions with the module name as a prefix.

```python
import math
print(math.sqrt(16))  # Access function using module name
```

✅ Avoids name conflicts, clear which module a function comes from.
❌ Must type module name every time.

---

### 2️⃣ `import module as alias`

Imports the whole module, but gives it a shorter or custom name.

```python
import math as m
print(m.sqrt(16))
```

✅ Shorter, convenient for long module names.

---

### 3️⃣ `from module import name`

Imports specific functions, classes, or variables from a module.

```python
from math import sqrt
print(sqrt(16))  # Use directly, no module prefix
```

✅ Cleaner code (no prefix), only imports what you need.
❌ Can cause name conflicts if different modules have the same function names.

---

### 4️⃣ `from module import name as alias`

Imports a specific function or variable with a custom name.

```python
from math import sqrt as square_root
print(square_root(16))
```

✅ Avoids conflicts, can give meaningful names.

---

### 5️⃣ `from module import *`

Imports everything from the module into the current namespace.

```python
from math import *
print(sqrt(16))   # sqrt is directly accessible
print(pi)         # pi is directly accessible
```

❌ Can overwrite existing names, hard to track where names come from. Not recommended in production code.

---

### 6️⃣ Importing Submodules in Packages

```python
## package/
## └── subpackage/
##     └── module.py: def func(): ...

import package.subpackage.module
package.subpackage.module.func()

## Or, using from:
from package.subpackage.module import func
func()
```

---

# Circular Dependency

A **circular dependency** happens when two or more modules try to import each other, creating a loop that Python can't resolve during loading.

```python
## module a.py
from b import func_b

def func_a():
    print("Function A")
    func_b()
```

```python
## module b.py
from a import func_a

def func_b():
    print("Function B")
    func_a()
```

If you try to `import a` or `import b`, Python gets stuck because:
- `a.py` imports `b.py`
- `b.py` imports `a.py`
- Python is still trying to finish loading `a.py` → `func_a` doesn't exist yet
- Crash or `ImportError` occurs
### How to Fix Circular Dependencies

**1 Late Import (Recommended)**

**`one.py`**
```python
def func_a():
    from two import func_b
    print("Function A")

func_a()
```

**`two.py`**
```python
def func_b():
    from one import func_a
    print("Function B")

func_b()
```

**Execution Trace**

When you run `python one.py`, here's exactly what happens:

```
python one.py
│
├── [__main__] func_a is defined
├── [__main__] func_a() is called
│   │
│   └── from two import func_b  →  triggers full execution of two.py
│       │
│       ├── [two] func_b is defined
│       ├── [two] func_b() is called
│       │   │
│       │   └── from one import func_a  →  triggers full execution of one.py
│       │       │                           (as module "one", NOT __main__)
│       │       │
│       │       ├── [one] func_a is defined
│       │       ├── [one] func_a() is called
│       │       │   └── from two import func_b  →  already cached, no re-run
│       │       │
│       │       └── print("Function A")  ──────────────────────────── A ①
│       │
│       └── print("Function B")  ────────────────────────────────── B ②
│
└── print("Function A")  ──────────────────────────────────────── A ③
```

The trick is that **`__main__` ≠ module `one`** in Python's module registry (`sys.modules`).

| Execution | Module name in `sys.modules` |
|---|---|
| `python one.py` | Registered as `__main__` |
| `from one import ...` inside two.py | Registered as `one` (a fresh import!) |

Because of this, `one.py` runs **twice** — once as `__main__` and once as `one` — while `two.py` runs only once, giving you **A → B → A**.

---

**2 Dependency Injection**

```python
# two.py
def func_b():
    print("Function B")
```

```python
# one.py
def func_a(func_b):
    print("Function A")
    func_b()

if __name__ == "__main__":
    from two import func_b
    func_a(func_b)          # wiring happens right here
```

Run: `python one.py` → works fine, no third file needed.

---

**3 Separate Shared Module**

Move shared functions to a new module that both modules can safely import. or use an other file that imports both of them.

`shared.py` is the only one that imports — `one.py` and `two.py` know nothing about each other.

**`one.py`** ← no imports at all

```python
def func_a():
    print("Function A")
```

**`two.py`** ← no imports at all

```python
def func_b():
    print("Function B")
```

**`shared.py`** ← the only file that imports from both

```python
from one import func_a
from two import func_b

func_a()
func_b()
```

**Output of `python shared.py`:**

```
Function A
Function B
```

---

# Absolute & Relative Imports

Both are ways to tell Python where to find a module or function.
### 1️⃣ Absolute Import

Specifies the full path from the top-level package.

```python
## alchemy/
##     __init__.py
##     elements.py
##     transmutation/
##         __init__.py
##         basic.py

## absolute import
from alchemy.elements import create_fire

def lead_to_gold():
    return f"Lead turned to gold using {create_fire()}"
```

✅ Clear and unambiguous, works no matter where the importing file is located.
❌ Can be long in deep hierarchies.

### Top-Level Import (`from basic import lead_to_gold`)

A top-level import does not reference the current package. Instead, Python searches for the module `basic` in the directories listed in `sys.path`, which include the current working directory, installed packages, and other configured paths. Because no package context is specified, Python expects `basic.py` to exist as a **top-level module**. If the actual file is located inside a package such as `alchemy/transmutation/basic.py`, Python will not find it using this statement and will raise a `ModuleNotFoundError`. To import it correctly in that case, the full package path must be used, such as `from alchemy.transmutation.basic import lead_to_gold`.

---
### 2️⃣ Relative Import

Specifies the path relative to the current module using dots:
- `.` → current package
- `..` → parent package
- `...` → grandparent, etc.

```python
## relative import
from .basic import lead_to_gold
from ..elements import create_water

def philosophers_stone():
    return f"Philosopher's stone made using {lead_to_gold()} and {create_water()}"
```

✅ Shorter paths in deeply nested packages, easier to reorganize packages internally.
❌ Can be confusing if you move files around, only works inside a package (not from top-level scripts).

### Relative Import (`from .basic import lead_to_gold`)

A relative import uses dots to reference modules **relative to the current package location**. BTW to use relative imports with dots (`from .module import ...`), your code must be inside a Python package (a folder containing an `__init__.py` file). In the statement `from .basic import lead_to_gold`, the dot (`.`) means “start from the current package.” Python determines the current package using the `__package__` variable that is automatically set when the module is imported. If the module belongs to the package `alchemy.transmutation`, then Python resolves the import as `alchemy.transmutation.basic`. It then loads the module `basic.py` located in the same package and imports the function `lead_to_gold` from it. Relative imports are typically used for **internal communication between modules inside the same package**, allowing modules to reference nearby files without writing the full package path.
### Parent Relative Import (`from ..basic import lead_to_gold`)

```
alchemy/
├── elements.py
├── potions.py
└── transmutation/
    ├── basic.py         # <-- basic is here
    └── potions/
        └── codes/
            └── advanced.py
```

In `advanced.py`, `from ..basic import lead_to_gold` uses `..` to move one package level up from the current module (`alchemy.transmutation.potions.codes`), navigating up to `alchemy.transmutation.potions`, then looking for `basic` relative to that level.

---
# Is `__init__.py` Necessary in Python Packages?

Below are two complete mini-projects that do the same thing.
Both let you run python main.py and import a package called mathpack.

One uses `__init__.py` , the other uses namespace package (no init).

## 1️⃣ Package WITH empty `__init__.py`

## Folder structure

```
project_with_empty_init/
│
├── main.py
└── mathpack/
    ├── __init__.py   ← empty file
    ├── add.py
    └── mul.py
```

---

## mathpack/**init**.py

```python
# empty on purpose
```

### mathpack/add.py

```python
def add(a, b):
    return a + b
```

### mathpack/mul.py

```python
def multiply(a, b):
    return a * b
```

### main.py

```python
from mathpack.add import add
from mathpack.mul import multiply

print("Add:", add(5, 3))
print("Multiply:", multiply(5, 3))
```

### ▶️ Run

```
cd project_with_empty_init
python main.py
```

Output:

```
Add: 8
Multiply: 15
```

---

## 2️⃣ Package WITHOUT `__init__.py`

## Folder structure

```
project_without_init/
│
├── main.py
└── mathpack/
    ├── add.py
    └── mul.py
```

### mathpack/add.py

```python
def add(a, b):
    return a + b
```

### mathpack/mul.py

```python
def multiply(a, b):
    return a * b
```

### main.py

```python
from mathpack.add import add
from mathpack.mul import multiply

print("Add:", add(5, 3))
print("Multiply:", multiply(5, 3))
```

### ▶️ Run

```
cd project_without_init
python main.py
```

Output:

```
Add: 8
Multiply: 15
```

## 🎯 So what changed?

Nothing in runtime behavior for this simple case.
The only difference is **package type**:

| Folder type              | What Python calls it |
| ------------------------ | -------------------- |
| With empty `__init__.py` | Regular package      |
| Without it               | Namespace package    |

---
# Python Script Mode vs Package Mode

In Python, code can run in script mode (directly via a file path) or package mode (via python -m). Script mode treats the file as standalone, often breaking absolute imports, while package mode respects the package structure, sets sys.path correctly, and allows imports to work as intended. Using package mode is essential when running code inside a package.

> You will master **script mode** (directly via a file path) and **package mode** (via `python -m`).

## Example:

Your working directory:

```
python07/
└── ex0/
    ├── __init__.py
    ├── main.py
    └── CreatureCard.py
```

Your import inside `main.py`:

```python
from ex0.CreatureCard import CreatureCard
```

This is an **absolute import** that requires Python to see `python07` as the project root.

---

## 1️⃣ Why this failed

```
python ex0/main.py
```

### What Python does internally

When you run a script by path, Python sets:

```
sys.path[0] = folder containing the script
```

So Python sets:

```
sys.path[0] = python07/ex0
```

Python now searches for packages relative to **ex0**, not the project root.

So when this line runs:

```python
from ex0.CreatureCard import CreatureCard
```

Python looks for:

```
python07/ex0/ex0/CreatureCard.py
```

It expects another `ex0` folder inside `ex0`.

Which obviously doesn’t exist → 💥

Error:

```
ModuleNotFoundError: No module named 'ex0'
```

### Root cause

Running a file directly removes it from its package context.

Your file became a **standalone script**, not part of the package.

---

## 2️⃣ Why this failed

```
python -m ex0/main.py
```

This mixes **two incompatible syntaxes**.

`-m` expects a **module name**, not a file path.

You gave:

```
ex0/main.py  ❌ filesystem path
```

Python tried to interpret it as a module:

```
ex0/main.py  → module name "ex0/main.py"
```

But module names:

* use dots `.`
* never use `/`
* never include `.py`

So Python complains:

```
Try using 'ex0/main' instead of 'ex0/main.py'
```

---

## 3️⃣ Why this failed

```
python ex0.main
```

Here you forgot `-m`.

Without `-m`, Python assumes you’re giving a **file path**.

It literally tries to open a file named:

```
ex0.main
```

It searches for:

```
python07/ex0.main
```

That file does not exist → 💥

Error:

```
can't open file 'ex0.main'
```

---

## 4️⃣ Why this worked ✅

```
python -m ex0.main
```

This is the **correct combination**:

* `-m` → run as module
* `ex0.main` → module path

### What Python sets now

When using `-m`, Python sets:

```
sys.path[0] = current working directory
```

Your working directory:

```
python07/
```

So Python searches from the project root and finds:

```
python07/ex0/__init__.py
python07/ex0/main.py
python07/ex0/CreatureCard.py
```

Then it executes as if Python did:

```python
import ex0.main
```

Now your import works perfectly:

```python
from ex0.CreatureCard import CreatureCard
```

Because Python can finally see the package root.

---

# 🔥 The real lesson

Running a file directly:

```
python ex0/main.py
```

👉 **Filesystem execution**

Running with `-m`:

```
python -m ex0.main
```

👉 **Import system execution**

And your code uses **import system semantics**, so only the last command matches.

# 🧠 Final mental rule

If your code imports using the package name:

```python
from ex0.something import X
```

You must run it as a module:

```
python -m ex0.main
```

---
# Protocol

In Python, a **Protocol** is a type-hinting mechanism that defines a set of methods or behaviors an object must implement so Python knows how to interact with it. It is essentially an interface defined by **behavior**, not by inheritance. This idea is called **duck typing**.

Protocols do **not** enforce anything at runtime — Python does not inject methods or check types while the program runs. Instead, Protocols are used by static type checkers like `mypy` to verify that objects you pass to functions match the expected "shape."

**Benefits:**
- **Static analysis:** Tools like `mypy` check types before running the code.
- **Clear documentation:** A type hint like `obj: Reader` clearly communicates that `obj` must have a `.read()` method that returns a string.
- **IDE support:** Autocomplete can suggest `.read()` on `obj`, reducing guesswork.
- **Flexible duck typing:** Any object with the required methods satisfies the Protocol — no inheritance needed.

**Example:**

```python
from typing import Protocol

## 1 Define a Protocol
class Reader(Protocol):
	def read(self) -> str:
		...

## 2 Class satisfiec the protocol
class FileReader:
	def read(self) -> str:
		return "file data"

## 3 Function uses the Protocol in type hint
def process(obj: Reader) -> None:
	data = obj.read()
	print(data)

process(FileReader()) # Works

process(123) # ❌ Runtime error: int has no read()
```

**Output:**
```
└─$ python3 one.py
file data
Traceback (most recent call last):
  File "/home/kali/1337/cursus/Python_modules/test/one.py", line 20, in <module>
    process(123)           # ❌ Runtime error: int has no read()
    ~~~~~~~^^^^^
  File "/home/kali/1337/cursus/Python_modules/test/one.py", line 15, in process
    data = obj.read()
           ^^^^^^^^
AttributeError: 'int' object has no attribute 'read'. Did you mean: 'real'?
```

```
└─$ mypy one.py
one.py:20: error: Argument 1 to "process" has incompatible type "int"; expected "Reader"  [arg-type]
Found 1 error in 1 file (checked 1 source file)
```

# Virtual environment

A **virtual environment** in Python is an **isolated workspace** that contains its own Python interpreter and installed packages, separate from the system Python and other projects.

Think of it as a **self-contained mini Python installation per project**.
## Why virtual environments exist

Without a virtual environment:

* All packages install globally on your computer.
* Different projects may need **different versions of the same library**.
* Updating one project can **break another**.

Example problem:

* Project A needs `Django 3.2`
* Project B needs `Django 5`
* Global Python cannot safely satisfy both.

A virtual environment solves this by isolating dependencies.

## What a virtual environment contains

When you create one, Python copies or links:

* A Python interpreter
* A private `site-packages` directory
* Package manager (`pip`)
* Activation scripts

Typical structure:

```
my_project/
│
├── venv/
│   ├── bin/ (Linux/macOS)
│   ├── Scripts/ (Windows)
│   ├── lib/pythonX/site-packages/
│   └── pyvenv.cfg
│
└── app.py
```

Everything installed with `pip` while the environment is active goes into that `venv` folder.

## How it works conceptually

When you **activate** a virtual environment:

* Your terminal’s PATH changes.
* `python` and `pip` now point to the environment’s interpreter.
* Installs stay local to that project.

Deactivate → system Python is restored.

## Why this is critical in real projects

Virtual environments enable:

* Reproducibility (others can recreate your setup)
* Dependency isolation
* Clean project structure
* Safe upgrades and experimentation
* Deployment consistency

This is standard practice in **data engineering, web development, ML, and DevOps**.

---

Here is a **complete minimal project** showing how a virtual environment fits into a real Python project. This is the simplest “professional-style” layout.

# Step 0 — Project goal

We’ll build a tiny app that:

* Uses an external library (`requests`)
* Runs inside a virtual environment
* Can be recreated by someone else using `requirements.txt`

**Explanation**

This is the real purpose of virtual environments:
You want a project that works the same on **your computer**, **your teammate’s computer**, and **a server**.
This is called **reproducibility**.

---

# Step 1 — Create the project folder

```bash
mkdir hello_venv_project
cd hello_venv_project
```

Project is empty now.

**Explanation**

* `mkdir hello_venv_project` → creates a new directory for the project.
* `cd hello_venv_project` → moves your terminal into that folder.

We always isolate projects in their own directory so their files don’t mix.

---

# Step 2 — Create the virtual environment

```bash
python3 -m venv venv
```

Now your folder looks like:

```
hello_venv_project/
└── venv/
```

The `venv/` folder is the isolated Python installation.

**Explanation**

This command tells Python:

> “Create a mini Python installation inside a folder named `venv`.”

Inside that folder Python creates:

* its own interpreter
* its own pip
* its own site-packages directory

From now on, anything installed will stay **inside this folder only**.

---

# Step 3 — Activate the environment

Linux / macOS:

```bash
source venv/bin/activate
```

Windows:

```bash
venv\Scripts\activate
```

Your terminal becomes:

```
(venv) user@pc: hello_venv_project$
```

This means Python is now isolated.

**Explanation**

Activation changes your terminal’s **PATH** so that:

* `python` → points to `venv/bin/python`
* `pip` → points to `venv/bin/pip`

So when you install packages, they go into the venv instead of the global system.

The `(venv)` in the prompt is just a visual indicator.

---

# Step 4 — Install a package inside the venv

```bash
pip install requests
```

This installs **only inside venv**, not globally.

**Explanation**

`requests` is an external library not included in Python.

Because the venv is active:

* pip installs it into `venv/lib/pythonX/site-packages/`
* your system Python stays clean.

This is the core benefit of isolation.

---

# Step 5 — Create the application files

Create this structure:

```
hello_venv_project/
│
├── venv/
├── app/
│   ├── __init__.py
│   └── main.py
├── requirements.txt
└── README.md
```

**Explanation**

We separate **environment** from **code**.

* `venv/` → the Python environment
* `app/` → your actual program
* `requirements.txt` → dependency list
* `README.md` → instructions for humans

This separation is standard in real projects.

---

## File 1 — app/main.py

```python
import requests

def get_quote():
    url = "https://api.quotable.io/random"
    response = requests.get(url, timeout=5)
    data = response.json()
    return f"{data['content']} — {data['author']}"

def main():
    print("Fetching a random quote...")
    quote = get_quote()
    print(quote)

if __name__ == "__main__":
    main()
```

**Explanation**

This file is the application.

Key points:

* We import `requests` → proves external packages work.
* The app calls a public API → demonstrates real dependency usage.
* `if __name__ == "__main__":` → makes the file executable as a script.

---

## File 2 — app/**init**.py

Empty file (marks folder as Python package)

```python
# empty
```

**Explanation**

This tells Python:

> “The `app` folder is a package.”

It enables future imports like:

```python
from app.main import get_quote
```

Even if empty, it’s important in real projects.

---

## File 3 — requirements.txt

Generate automatically:

```bash
pip freeze > requirements.txt
```

It will contain something like:

```
requests==2.31.0
certifi==2024.2.2
charset-normalizer==3.3.2
idna==3.6
urllib3==2.2.1
```

This file is **critical** for reproducibility.

**Explanation**

`pip freeze` lists **every installed dependency with exact versions**.

This is what makes environments reproducible:
Someone else can install the exact same versions.

---

## File 4 — README.md

```markdown

#Hello Venv Project

Small example showing how to use Python virtual environments.

##Setup

Create venv:
python3 -m venv venv

Activate:
source venv/bin/activate

Install dependencies:
pip install -r requirements.txt

Run app:
python app/main.py
```

**Explanation**

README files explain:

* how to install the project
* how to run it

In real life, this is essential for teammates and future you.

---

# Step 6 — Run the project

```bash
python app/main.py
```

Output example:

```
Fetching a random quote...
Life is really simple, but we insist on making it complicated — Confucius
```

**Explanation**

Because the venv is active:

* Python finds the `requests` package inside the venv.
* The program runs successfully.

Without the venv, this could fail on another machine.

---

# Step 7 — What happens on another computer (important)

Someone cloning your project does:

```bash
git clone <repo>
cd hello_venv_project

python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
python app/main.py
```

They get **the exact same environment**.

**Explanation**

This is the entire reason virtual environments exist:
The environment is **recreated from instructions**, not shared as files.

---

# Step 8 — What NOT to commit

Create `.gitignore`:

```
venv/
__pycache__/
```

We never commit the venv folder because it can be recreated.

**Explanation**

The venv can be hundreds of MB and is machine-specific.
We only commit the **instructions to rebuild it**.

---

# Final project tree

```
hello_venv_project/
│
├── venv/              (not committed)
├── app/
│   ├── __init__.py
│   └── main.py
├── requirements.txt
├── README.md
└── .gitignore
```


This is the canonical beginner → professional workflow.

If you want, next we can upgrade this into a **real package with pyproject.toml**, which is the modern standard.

There are several reliable ways to verify whether your shell is currently using a virtual environment.

---
# Check if you’re in a virtual environment

## **1) Check which Python executable is being used (most reliable)**

### Linux / macOS

```bash
which python
```

### Windows

```bash
where python
```

If you are inside the venv, the path will point to the project:

Example:

```
/home/user/hello_venv_project/venv/bin/python
```

If you are NOT in a venv, you’ll see something like:

```
/usr/bin/python
```

This is the **best verification method**.

---

## **2) Check environment variable (quick check)**

Linux / macOS:

```bash
echo $VIRTUAL_ENV
```

Windows:

```bash
echo %VIRTUAL_ENV%
```

If active → prints path to venv
If empty → no virtual environment active.

---

## **3) Programmatic check inside Python code**

When Python runs inside a virtual environment:

* `sys.prefix` → points to the virtual environment
* `sys.base_prefix` → points to the global/system Python

So we just compare them.

---

## Minimal check

```python
import sys

if sys.prefix != sys.base_prefix:
    print("Running inside a virtual environment")
else:
    print("Running in system Python")
```

---

## Why this works

Python keeps two installation paths:

| Variable          | Meaning                      |
| ----------------- | ---------------------------- |
| `sys.base_prefix` | Original Python installation |
| `sys.prefix`      | Current active environment   |

Inside a venv:

```
sys.base_prefix = /usr
sys.prefix      = /home/user/project/venv
```

Outside a venv:

```
sys.base_prefix = /usr
sys.prefix      = /usr
```

So if they differ → you are in a venv.

This is the **official recommended detection method**.

---

Here’s a **clean table** with **Command / Code**, **Purpose**, and **Example Output**.

| Command / Code                                          | Purpose                         | Example Output                                                                     |
| ------------------------------------------------------- | ------------------------------- | ---------------------------------------------------------------------------------- |
| `python3 -m venv venv`                                  | Create virtual environment      | *(no output)*                                                                      |
| `source venv/bin/activate`                              | Activate venv (Linux/macOS)     | `(venv) user@pc: hello_venv_project$`                                              |
| `deactivate`                                            | Deactivate venv                 | Prompt returns to normal: `user@pc: hello_venv_project$`                           |
| `pip install requests`                                  | Install a package in venv       | `Collecting requests ... Successfully installed requests-2.31.0`                   |
| `pip freeze`                                            | List installed packages         | `requests==2.31.0` <br> `urllib3==2.2.1`                                           |
| `pip install -r requirements.txt`                       | Recreate environment from file  | `Installing collected packages: requests, ...`                                     |
| `import sys; print(sys.executable)`                     | Show running Python interpreter | `/home/user/project/venv/bin/python`                                               |
| `import sys; print(sys.prefix)`                         | Current Python environment path | `/home/user/project/venv`                                                          |
| `import sys; print(sys.base_prefix)`                    | Original/system Python path     | `/usr`                                                                             |
| `import sys; print(sys.prefix != sys.base_prefix)`      | Check if inside venv            | `True` → inside venv <br> `False` → system Python                                  |
| `import site; print(site.getsitepackages())`            | List global site-packages dirs  | `['/usr/lib/python3.13/site-packages', '/usr/local/lib/python3.13/site-packages']` |
| `import os; print(os.path.basename("/pathto/main.py"))` | Get last part of path           | `main.py`                                                                          |

## **`site` module**

In Python, the **`site` module** is a **standard library module** that handles **site-specific configuration** for the Python environment. Its main role is to **manage and locate package directories** when Python starts. Essentially, it acts as a **helper for package paths** — it tells Python where to find installed modules and provides tools to **inspect, add, or customize those locations**.

## **`os` module**

In Python, the **`os` module** is a **standard library module** that provides a way to **interact with the operating system**. It lets you **work with files, directories, paths, environment variables, and system commands** in a platform-independent way. Essentially, it acts as a **bridge between Python and your OS**, enabling your programs to perform tasks like navigating the filesystem, reading/writing files, and managing processes.

> A Python **virtual environment (venv)** works **per shell session (terminal) once activated**, not “globally” for the whole system or permanently in a directory.

---

# How to install a package (`Pandas` for example)

To install **pandas**, you use **`pip`**, Python’s package manager. Here’s the proper way depending on whether you are in a virtual environment or not.

### 1️⃣ If you are **inside a virtual environment** (recommended)

Activate your venv first:

**Linux/macOS:**

```bash
source venv/bin/activate
```

Then install pandas:

```bash
pip install pandas
```

### 2️⃣ If you are **using system Python**

```bash
pip install --user pandas
```

> The `--user` flag installs it for your user account only, avoiding the need for admin rights.

### 3️⃣ Optional: Install a specific version

```bash
pip install pandas==2.0.3
```

### Exact location

Packages go into the venv’s **site-packages** directory:

```
venv/lib/pythonX.Y/site-packages/
```

So your pandas files will be here:

```
venv/
└── lib/
    └── python3.13/
        └── site-packages/
            ├── pandas/
            ├── pandas-*.dist-info/
            ├── numpy/
            └── ...
```

That folder is the **local package storage** for that environment.

---
# What is Pandas

In Python, **`pandas`** is a **popular open-source library** used for **data manipulation and analysis**. It provides **easy-to-use data structures** and functions to handle **tabular data** (like spreadsheets or SQL tables).

### Key features of `pandas`:

1. **Data structures**

   * `DataFrame` → 2D table with rows and columns
   * `Series` → 1D labeled array

2. **Data manipulation**

   * Filtering, sorting, grouping, merging, reshaping
   * Handling missing data
   * Applying functions to data

3. **Data input/output**

   * Read/write CSV, Excel, JSON, SQL, and more

4. **Integration**

   * Works well with **NumPy**, **Matplotlib**, and **scikit-learn** for analytics, plotting, and machine learning

### Example

Here’s a **simple example of `pandas.DataFrame`**:

```python
import pandas as pd

# Create a simple DataFrame
data = {'Name': ['Alice', 'Bob'], 'Age': [25, 30]}
df = pd.DataFrame(data)

print(df)
```

**Output:**

```
    Name  Age
0  Alice   25
1    Bob   30
```

Here’s a **simple example of `pandas.Series`**:

```python
import pandas as pd

# Create a Series
ages = pd.Series([25, 30, 22, 28], index=['Alice', 'Bob', 'Charlie', 'Diana'])

# Print the Series
print(ages)
```

**Output:**

```
Alice      25
Bob        30
Charlie    22
Diana      28
dtype: int64
```

### Explanation:

* `pd.Series(data, index=...)` creates a **1-dimensional labeled array**.
* **Data:** `[25, 30, 22, 28]` → values of the Series
* **Index:** `['Alice', 'Bob', 'Charlie', 'Diana']` → labels for each value
* You can access values by **label** or **position**:

```python
print(ages['Bob'])   # 30
print(ages[2])       # 22
```

**Here’s a table for most useful pandas functions:**

| Example                                  | What it does                                                  |
| ---------------------------------------- | ------------------------------------------------------------- |
| `df = pd.read_csv("data.csv")`           | Load a CSV file into a DataFrame                              |
| `df.head()`                              | Show first 5 rows (quick preview)                             |
| `df.info()`                              | Summary of DataFrame: columns, types, missing values          |
| `df.shape`                               | Get number of rows and columns                                |
| `df["age"]`                              | Select a single column by name                                |
| `df.loc[df["age"] > 18]`                 | Select/filter rows by label or condition                      |
| `df.sort_values("age", ascending=False)` | Sort DataFrame by a column                                    |
| `df.to_csv("output.csv", index=False)`   | Save DataFrame to CSV file                                    |

---

# NumPy

In Python, **`NumPy`** (short for *Numerical Python*) is a **powerful library for numerical computing**. It provides:

1. **`ndarray`** → a fast, multi-dimensional array for numbers.
2. **Mathematical functions** → vectorized operations, linear algebra, statistics, etc.
3. **Integration** → works with **pandas**, **Matplotlib**, and machine learning libraries like **scikit-learn**.

### Why it’s useful

* **Fast calculations** on large datasets (faster than regular Python lists)
* **Memory efficient** storage of numbers
* **Vectorized operations**: do math on arrays without loops

### Example

```python
import numpy as np

# Create an array
arr = np.array([1, 2, 3, 4])

# Add 5 to all elements
arr = arr + 5

print(arr)
```

**Output:**

```python
[6 7 8 9]
```

✅ Here, the addition was applied **to the whole array at once**, which is much faster than using a Python list with a loop.

**Here’s a table for most usefull NumPy functions:**

| Example / Function                            | What it does                                   | Example Output                       |
| --------------------------------------------- | ---------------------------------------------- | ------------------------------------ |
| `np.array([1,2,3])`                           | Create a NumPy array                           | `[1 2 3]`                            |
| `np.arange(0, 10, 2)`                         | Create an array with evenly spaced values      | `[0 2 4 6 8]`                        |
| `np.zeros((3,3))`                             | Create a 3x3 array of zeros                    | `[[0. 0. 0.] [0. 0. 0.] [0. 0. 0.]]` |
| `np.ones((2,4))`                              | Create a 2x4 array of ones                     | `[[1. 1. 1. 1.] [1. 1. 1. 1.]]`      |
| `np.linspace(0, 1, 5)`                        | Create 5 evenly spaced numbers between 0 and 1 | `[0.   0.25 0.5  0.75 1. ]`          |
| `np.eye(3)`                                   | Create a 3x3 identity matrix                   | `[[1. 0. 0.] [0. 1. 0.] [0. 0. 1.]]` |
| `np.reshape(np.arange(6), (2,3))`             | Reshape an array into a new shape              | `[[0 1 2] [3 4 5]]`                  |
| `np.mean(np.array([1,2,3,4]))`                | Calculate the mean of an array                 | `2.5`                                |
| `np.sum(np.array([[1,2,3],[4,5,6]]), axis=0)` | Sum array elements along axis 0 (columns)      | `[5 7 9]`                            |
|                                               |                                                |                                      |

---
# Matplotlib

In Python, **`Matplotlib`** is a **popular plotting library** used to **create static, animated, and interactive visualizations**. It’s especially useful for data analysis and scientific computing.

### Key Points

1. **Main module** → `matplotlib.pyplot` (commonly imported as `plt`)

   * Provides functions similar to MATLAB for plotting

2. **Types of plots**

   * Line plots, scatter plots, bar charts, histograms, pie charts, etc.

3. **Integration**

   * Works well with **NumPy** and **pandas** data
   * Often used alongside **Seaborn** for advanced statistical plots

### Example

```python id="6k7ovw"
import matplotlib.pyplot as plt

# Sample data
x = [1, 2, 3, 4, 5]
y = [2, 3, 5, 7, 11]

# Create a line plot
plt.plot(x, y)
plt.title("Simple Line Plot")
plt.xlabel("X-axis")
plt.ylabel("Y-axis")
plt.show()
```

**Output:**

* A line graph connecting points `(1,2)`, `(2,3)`, `(3,5)`, `(4,7)`, `(5,11)`
* Labeled axes and a title

**Here’s a table for most useful pandas functions:**

| Example / Code                             | What it does                                       |
| ------------------------------------------ | -------------------------------------------------- |
| `plt.figure()`                             | Create a new figure / plotting canvas              |
| `plt.scatter(data["x"], data["y"], s=20)`  | Create a scatter plot with point size = 20         |
| `plt.plot(x, y)`                           | Create a line plot connecting points               |
| `plt.title("Random Points Visualization")` | Set the title of the plot                          |
| `plt.xlabel("X values")`                   | Set the label for the X-axis                       |
| `plt.ylabel("Y values")`                   | Set the label for the Y-axis                       |
| `plt.show()`                               | Display the figure in a window                     |
| `plt.savefig("matrix_analysis.png")`       | Save the current figure to a file (PNG, PDF, etc.) |
| `plt.subplot(rows, cols, index)`           | Create subplots in a single figure                 |
| `plt.grid(True)`                           | Show grid lines on the plot                        |

---
# Difference between pip and Poetry

**Pip** is Python’s standard package installer. It installs libraries from PyPI based on what you ask for (like a `requirements.txt` file). It’s simple and works well for small projects, but it **does not manage conflicts or reproducibility automatically**.

**Poetry** is a full dependency and project management tool. It not only installs packages but also **resolves all dependency versions**, ensures **no conflicts**, and creates a reproducible environment with its `poetry.lock` file.
###  **1- Pip behavior**

* Installs packages in the order you specify or from `requirements.txt`.
* If two requirements are incompatible (e.g., `numpy>=1.25` and `numpy<1.25`), pip **just fails at install**, showing an error.
* Pip does **not try to resolve conflicts automatically**; it only reports the problem.
* If you ignore the conflict and manually override, pip can leave your environment **broken or inconsistent**.
### **2- Poetry behavior**

* Uses a **dependency resolver**: it analyzes all declared dependencies and their sub-dependencies **before installing anything**.
* When it detects a conflict, Poetry **stops immediately**, showing a clear message about **which packages caused the conflict**.
* It will **not partially install anything**, so your environment stays clean.
* Poetry generates a **`poetry.lock` file** that locks exact versions — this ensures **reproducible installations** on other machines.


## Examples:


## **1-`requirements.txt` (for pip)**

* Plain text file
* Each line specifies a package and optionally a version constraint

**Example:**

```text
numpy>=1.25
pandas==2.1.0
matplotlib>=3.8,<4.0
requests
```

You install it with:

```bash
pip install -r requirements.txt
```

## **2- `pyproject.toml` (for Poetry)**

* TOML file with sections for project metadata and dependencies
* Supports **version constraints** and more metadata

**Example:**

```toml
[tool.poetry]
name = "matrix-loader"
version = "0.1.0"
description = "Matrix data analysis"
authors = ["student"]

[tool.poetry.dependencies]
python = "^3.10"
pandas = "2.1.0"
matplotlib = ">=3.8,<4.0"
requests = "*"

```

Install with:

```bash
poetry install
```

* Poetry creates a **`poetry.lock`** file with exact versions to **guarantee reproducibility**.

## Examples of conflict:

**Here’s a concise conflict example of each:**
### **1-`Pip` conflict example**

`requirements.txt`:

```
numpy>=1.25
numpy<1.25
```

**What happens:**

* Pip tries to install in order.
* Sees conflicting constraints and **fails immediately**, showing an error.

### **2-`Poetry` conflict example**

`pyproject.toml`:

```toml
[tool.poetry.dependencies]
numpy = "^1.25"

[tool.poetry.dependencies]
numpy = "1.10"
```

**What happens:**

* Poetry analyzes all dependencies **before installing anything**.
* Detects conflict between `numpy` versions.
* **Stops cleanly** without installing anything, shows which packages caused the conflict.

---

# What is `.env` file

A **`.env` file** is a simple text file used to store **environment variables** for your project. These variables can be things like **API keys, database URLs, secret tokens, or configuration settings** that you don’t want to hard-code in your Python scripts.

### **Key points about `.env`**

1. **Format** – plain text, `KEY=VALUE` per line:

```text
API_KEY=123456789abcdef
DATABASE_URL=postgresql://user:pass@localhost:5432/mydb
DEBUG=True
```

2. **Purpose** – separates configuration from code:
* Keeps sensitive information out of your source code
* Makes it easier to change settings without touching code
* Works across different environments (dev, staging, production)

2. **How to use in Python** :
* usually with the `python-dotenv` library:

3. **Best practice** – add `.env` to `.gitignore`:
* So you don’t accidentally push secrets to GitHub.

## Analogy

if your code depends on a `.env` file and you **didn’t push it**, anyone cloning your repo **won’t have the environment variables**, so the code will likely **break or raise errors** when it tries to access them.

### **How the person cloning your repo can fix it**

1. **You need to provide a template** for the `.env` file (without real secrets).

Example: `example.env`:

```text
API_KEY=your_api_key_here
DATABASE_URL=your_database_url_here
DEBUG=True
```

2. **Instructions for the user:**

* Copy the template to `.env`:

```bash
cp example.env .env
```

* Fill in real values (like their own API keys, database credentials, etc.).

3. **Run the code** normally:

```bash
python app/main.py
```

* The code will now read the `.env` values correctly.

### ✅ **Best practice**

* Never commit real secrets.
* Always provide a **template** (`example.env` or `.env.example`).
* Document it in `README.md` so others know to copy it.

---

# Dotenv libary

`dotenv` is a **tool/library that loads environment variables from a `.env` file into your Python program** so your code can use them with `os.getenv()` or `os.environ`.

### **Key points about `dotenv`**

1. **Purpose**
- Python itself doesn’t automatically read `.env` files.
- `python-dotenv` makes it easy to **load all the key=value pairs from `.env` into the environment** at runtime.

1. **Installation** (if not already installed)

```bash
pip install python-dotenv
```

3. **Usage Example**

```python
from dotenv import load_dotenv
import os

# Load environment variables from .env file
load_dotenv()

API_KEY = os.getenv("API_KEY")
DEBUG = os.getenv("DEBUG") == "True"

print(API_KEY, DEBUG)
```

4. **How it works**
- `load_dotenv()` reads `.env` line by line.
- Adds each variable to **Python’s environment variables**, so `os.getenv("VAR")` works.

5. **Best practice**
- Keep `.env` **out of version control** (add to `.gitignore`).
- Provide a **template file** (`.env.example`) for others.

💡 **Analogy:**

Think of `python-dotenv` as a **mail carrier**: it reads the secret `.env` box and delivers each key/value to your program so it can use it safely.

---

# Python’s environment variables

### **Scenario**

You run this command in the shell:

```bash
$ MATRIX_MODE=production API_KEY=secret123 python3 oracle.py
```

Even though the variables are “before the program,” Python can read them automatically.

### **Step by step**

1. **Args passed in shell**

   ```bash
   MATRIX_MODE=production API_KEY=secret123 python3 oracle.py
   ```

   → Go directly into **Python’s environment** (`os.environ`) **before the script starts**. Python automatically sees them.

2. **`load_dotenv()` runs**

   ```python
   from dotenv import load_dotenv
   load_dotenv()
   ```

   → Reads variables from a `.env` file and **adds them to `os.environ` only if they don’t already exist**.
   → **Shell variables are NOT overwritten** unless you pass `load_dotenv(override=True)`.

3. **`config = {k: os.getenv(k) for k in REQUIRED}`**

   → Reads the **current values from `os.environ`**, giving priority to shell variables.
   → Stores them in a Python dictionary for easy access in your code.

### **Priority**

```text
Shell variables > .env variables > defaults (if any)
```

* If a variable exists **both in shell args and `.env`**, the **shell version wins**.
* If a variable exists **only in `.env`**, that’s what `config` will get.
* If it’s **in neither**, `os.getenv()` returns `None` (unless you provide a default).

---
