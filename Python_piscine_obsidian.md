
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

## 1 - Starting the Python Interpreter (CPython)

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

## 2 - Lexing (Lexical Analysis / Tokenization)

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

## 3 - Parsing (Syntactic Analysis)

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

## 4 - Bytecode Generation and Inspection

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

## 5 - Execution by the Python Virtual Machine (PVM)

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

## 6 - Generate .pyc Files

In Python, you can compile a script to bytecode without executing it. This produces a `.pyc` file, which is a cached version of the bytecode. Python uses these files to speed up future executions by loading the precompiled bytecode instead of recompiling the source code each time.

```bash
python3 -m py_compile your_file.py
## Output: __pycache__/your_file.cpython-312.pyc
```

**`.pyc` is not machine code.** A `.pyc` file looks like random symbols, but it stores Python bytecode — a low-level, platform-independent instruction set for the PVM. The CPU never runs `.pyc` directly; the PVM reads and interprets the bytecode.

**The "compiler" word:** Python does have a compilation step, but it is very different from C/C++ compilation. The Python compiler converts source code (`.py`) into tokens, then into an AST, and finally into bytecode. It can optionally save this bytecode to a `.pyc` file. Unlike C or C++, Python does **not** generate machine code at this stage.

**Library modules** are included at runtime, not compile time. When the PVM encounters an `import` statement, it searches for the module in the standard library, `site-packages`, and the current working directory. If a `.pyc` file exists, Python loads the precompiled bytecode; otherwise it compiles the `.py` source first.

---

## Python Interpreter Execution Flow

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
## Numeric types

| Type      | Example | Typical size                |
| --------- | ------- | --------------------------- |
| `bool`    | `True`  | 28 bytes                    |
| `int`     | `10`    | 28 bytes (grows with value) |
| `float`   | `3.14`  | 24 bytes                    |
| `complex` | `2+3j`  | 32 bytes                    |

Notes:

* `int` uses **arbitrary precision**, so very large numbers require more memory.
* `float` follows **IEEE-754 double precision (64 bits)** internally.

## Text type

| Type  | Example | Typical size           |
| ----- | ------- | ---------------------- |
| `str` | `"a"`   | ~50 bytes + characters |

Strings grow with length because characters are stored internally.

Example:

```python
sys.getsizeof("a")      # ~50 bytes
sys.getsizeof("hello")  # larger
```

## Container types

| Type    | Example   | Typical size         |
| ------- | --------- | -------------------- |
| `list`  | `[1,2,3]` | ~56 bytes + elements |
| `tuple` | `(1,2,3)` | ~48 bytes + elements |
| `set`   | `{1,2,3}` | ~216 bytes           |
| `dict`  | `{"a":1}` | ~232 bytes           |

Notes:

* Containers store **references to objects**, not the objects themselves.
* Their size grows as elements are added.

## Special types

| Type       | Example     | Typical size |
| ---------- | ----------- | ------------ |
| `NoneType` | `None`      | ~16 bytes    |
| `range`    | `range(10)` | ~48 bytes    |

## Example measurement

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
# **`sys` module**

The **`sys` module** is a built-in Python module that lets your program interact with the **Python interpreter and runtime environment**.

Import it with:

```python
import sys
```

### Key uses (quick view)

**1) Command-line arguments**

```python
sys.argv
```

List of values passed when running the script.

**2) Exit the program**

```python
sys.exit()
```

Stops execution immediately.

**3) Standard input/output streams**

```python
sys.stdin
sys.stdout
sys.stderr
```

Used for CLI tools and logging.

**4) Python runtime info**

```python
sys.version
sys.platform
```

**5) Module search paths**

```python
sys.path
```

Where Python looks for imports.

**6) Object memory size**

```python
sys.getsizeof(obj)
```

**In one line:**  
`sys` gives you access to the Python interpreter and how your script runs inside it.

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

## Module Object Inspection

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
# Program’s life cycle

**Load time, Import time, and Runtime** are the three moments that describe **when Python executes different kinds of code** during a program’s life cycle. The order is:

> **Load time → Import time → Runtime**

Let’s define each precisely and connect them.

## 1) Load time (file loading / compilation phase)

**Load time = when Python reads a `.py` file and compiles it to bytecode before executing it.**

When you run:

```bash
python main.py
```

Python first:

1. Opens the file
    
2. Parses the source code
    
3. Converts it into **bytecode**
    
4. Prepares a module object to execute
    

No statements run yet — Python is preparing the module to execute.

Think of this as:

> “Python is _understanding_ the file.”

### What happens here technically

- Syntax checking happens
    
- Indentation errors happen here
    
- The file becomes a **module object**
    

If syntax is wrong → program stops **before execution starts**.

Example syntax error (load time error):

```python
print("File started loading")

def crash():    
	print("About to crash")    
	return 1 / 0   # logical/runtime error

crash()

print("File finished loading")
```

Example of not load time error:

```python
print("File started loading")

def crash():    
	print("About to crash")    
	return 1 / 0   # logical/runtime error

print("File finished loading")
```

Because the function `crash()` was **defined**, not executed.
## 2) Import time (module execution phase)

**Import time = when Python executes the top-level code of a module the first time it is imported.**

This is extremely important:

> In Python, **importing a file runs it.**

When Python sees:

```python
import math_utils
```

It does:

1. Load the module (step 1)
    
2. Execute **all top-level code inside it**
    
3. Store the module in `sys.modules`
    
4. Future imports reuse the cached module (no re-execution)
    

### What counts as _top-level code_?

Everything **not inside a function or class**.

Example:

```python
print("Hello from module")   # runs at import time

x = 5                        # runs at import time

def func():
    print("inside function") # NOT run at import
```

So importing this module prints immediately.

### Why `if __name__ == "__main__"` exists

To prevent code from running at import time unintentionally.

```python
if __name__ == "__main__":
    main()
```

This block runs **only when file is executed directly**, not when imported.

## 3) Runtime (program execution phase)

**Runtime = when your program is actively running after all modules are loaded and imported.**

This is the phase where:

- Functions are called
    
- Loops run
    
- Input is processed
    
- Threads run
    
- The actual application logic happens
    

Everything inside functions/classes typically runs at runtime.

Example:

```python
def greet():
    print("Hello")

greet()  # runtime execution
```

## Short mental model

|Phase|What Python is doing|Typical errors|
|---|---|---|
|**Load time**|Parsing & compiling the file|SyntaxError|
|**Import time**|Executing module top-level code|Import side effects|
|**Runtime**|Running the program logic|Bugs, exceptions|

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
# `any()`

The `any()` function evaluates whether **at least one element** in an iterable is truthy. It returns `True` if any element is considered true in a boolean context, and `False` if all elements are falsy. Falsy values include `False`, `None`, `0`, empty sequences like `[]` or `""`, and empty collections like `{}` or `set()`. If the iterable is empty, `any()` returns `False`.

`any()` has **short-circuit behavior** — evaluation stops as soon as a truthy value is encountered.

```python
print(any([False, False, True]))     # True
print(any([False, 0, None]))         # False

nums = [1, 3, 5, 8]
print(any(n % 2 == 0 for n in nums)) # True

def check(x):
    print(f"Checking {x}")
    return x > 0

print(any(check(n) for n in [-1, -2, 3, -4]))
```

---

# `strip()`

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

A circular dependency appears when two modules require each other **during import time**.

The key rule of Python imports:

> Importing a module = executing the file from top to bottom **once**, then caching it in `sys.modules`.

During this execution, the module exists in a **partially initialized state**.  
If another module tries to access objects that are not defined yet, the import fails.

## The Problem We Will Solve

We want two modules to cooperate:

- `one.py` owns **func_a**
    
- `two.py` owns **func_b**
    
- `func_a()` must call `func_b()`
    
- `func_b()` must call `func_a()`
    

This naturally creates a cycle.

## ❌ Broken Circular Import Example

### one.py

```python
from two import func_b

def func_a():
    print("Function A")
    func_b()

if __name__ == "__main__":
    func_a()
```

### two.py

```python
from one import func_a

def func_b():
    print("Function B")
```

### Running

```
python one.py
```

### What happens internally

```mermaid
flowchart TD
    A[Run one.py as script] --> B[Create module '__main__' from one.py]
    B --> C[Execute top-level of __main__]
    C --> D[__main__: from two import func_b]
    D --> E[Create empty module 'two']
    E --> F[Execute top-level of two]
    F --> G[two: from one import func_a]
    G --> H{Is module 'one' in sys.modules?}
    H -->|No| I[Create SECOND module 'one' from one.py]
    I --> J[Execute top-level of module 'one']
    J --> K[module 'one': from two import func_b]
    K --> L[Python finds partially initialized 'two']
    L --> M{Is func_b already defined?}
    M -->|No| N[ImportError: partially initialized module]
```

The dependency loop exists **during module initialization**, which is the dangerous moment.

## Solution 1 — Late Import (Lazy Import)

Move the import inside the function so the dependency is needed **at runtime**, not during module initialization.

Late imports work because Python only executes **top-level (global) code** when it imports it. Code inside functions is saved but not run until the function is called. If an import is at the top, it runs too early and the other file may not be ready yet, which causes the circular import error. Putting the import inside a function makes Python wait until the program is already running and both files are fully loaded, so the import happens safely.

### one.py

```python
def func_a():
    from two import func_b
    print("Function A")
    func_b()

if __name__ == "__main__":
    func_a()
```

### two.py

```python
def func_b():
    from one import func_a
    print("Function B")
```

### Output

```
Function A
Function B
```

### Execution Trace (step-by-step)

When running:

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
│       │       └── print("Function A")  ─────────── A ①
│       │
│       └── print("Function B")  ─────────────────── B ②
│
└── print("Function A")  ─────────────────────────── A ③
```

Important concept revealed here:

|Execution|Module name|
|---|---|
|Running file directly|`__main__`|
|Importing the same file|`one`|

So the file can execute twice under different names.

## Solution 2 — Dependency Injection

Instead of modules importing each other, the dependency is **passed from the outside**.

This removes the circular dependency entirely.

### two.py

```python
def func_b():
    print("Function B")
```

### one.py

```python
def func_a(func_b):
    print("Function A")
    func_b()

if __name__ == "__main__":
    from two import func_b
    func_a(func_b)
```

### Output

```
Function A
Function B
```

### Why this works

`one.py` no longer imports `two` at module level.  
The wiring happens only in the program entry point.

This pattern is called **inversion of control** and is heavily used in large systems.

## Solution 3 — Separate Shared Module

The circular dependency shows both modules are being composed in the wrong place.  
A third module becomes the **composition root**.
### one.py

```python
def func_a():
    print("Function A")
```

### two.py

```python
def func_b():
    print("Function B")
```

### app.py

```python
from one import func_a
from two import func_b

func_a()
func_b()
```

#### Output

```
Function A
Function B
```

Here:

- `one` and `two` are independent
    
- only `app.py` knows about both
    

This produces a **clean dependency graph**.
## Solution 4 — Import the Module, Not the Function

This technique relies on a subtle but very important behavior of Python’s import system:

`import module` is safer than `from module import name` in circular dependencies. Why?, Because `import module` only requires the **module object** to exist.
It does **not** require its contents to already be defined.

Python can provide a partially initialized module object safely.
### Apply this fix to the SAME problem

We still want:

* `func_a()` calls `func_b()`
* `func_b()` calls `func_a()`

### one.py

```python
print("Loading A")

import two   # ← import module, not function

def func_a():
    print("Function A")
    two.func_b()

if __name__ == "__main__":
    func_a()
```

### two.py

```python
print("Loading B")

import one   # ← import module, not function

def func_b():
    print("Function B")
```

### Run

```
python one.py
```

### Output

```
Loading A
Loading B
Function A
Function B
Function A
```

```mermaid
flowchart LR
    A[from module import name] --> B[Needs attribute NOW]
    B --> C[Circular import breaks]

    D[import module] --> E[Needs attribute LATER]
    E --> F[Circular import works]
```
## Comparing the Three Fixes

| Fix                  | Strategy                                     | Best use case                |
| -------------------- | -------------------------------------------- | ---------------------------- |
| Late Import          | Delay dependency until runtime               | Small/local cycles           |
| Dependency Injection | Pass dependency from entry point             | Clean architecture / testing |
| Shared Module        | Move composition to third module             | Structural refactor          |
| Import Module        | Use `import module` instead of symbol import | Mutual module collaboration  |

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

> NOW You will master **script mode** (directly via a file path) and **package mode** (via `python -m`).

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

## 🔥 The real lesson

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

## 🧠 Final mental rule

If your code imports using the package name:

```python
from ex0.something import X
```

You must run it as a module:

```
python -m ex0.main
```

---
# What is an Enum in Python?

An **Enum (enumeration)** is a class used to represent a **fixed set of named constants**.

It answers this problem:

> “This variable must only be one of a few allowed values.”

Instead of using random numbers or strings, you give them **names**.

## Example Enum

We define a set of allowed contact types:

```python
from enum import Enum

class ContactType(Enum):
    telepathic = 2
    visual = 1
    physical = 3
```

This creates a class whose members are **constants**.

Each member has:

* a **name** → the label (`visual`)
* a **value** → the assigned value (`1`)

## Accessing enum members (3 ways)

```python
by_dot = ContactType.visual
by_name = ContactType['visual']
by_value = ContactType(1)
```

All three refer to the **same enum member**.

## Printing name and value

```python
print(by_dot.name)
print(by_dot.value)

print(by_name.name)
print(by_name.value)

print(by_value.name)
print(by_value.value)
```

Output:

```
visual
1
visual
1
visual
1
```

So:

| Expression              | Meaning                 |
| ----------------------- | ----------------------- |
| `ContactType.visual`    | access by attribute     |
| `ContactType['visual']` | access by name string   |
| `ContactType(1)`        | access by numeric value |

They all resolve to the same enum object.

## Now the comparision problem (very important)

Look at this enum:

```python
from enum import Enum

class ContactType(Enum):
    physical = "physical"
```

You might expect this to work:

```python
ContactType.physical == "physical"
```

But it does **not**.

Result:

```
False
```

Why?

Because left side = Enum object
Right side = string

They are different types.

You must access the value explicitly:

```python
ContactType.physical.value == "physical"  # True
```

This becomes annoying when working with:

* user input
* JSON
* APIs
* Pydantic
## Solution: make the Enum behave like a string

```python
from enum import Enum

class ContactType(str, Enum):
    physical = "physical"
```

Now the enum **inherits from `str`**.

Meaning the enum *is also a string*.

Now this works:

```python
ContactType.physical == "physical"   # True
```

Big usability improvement.

## Why this is useful

Without `str`:

```python
if "physical" == ContactType.physical.value:
```

With `str`:

```python
if "physical" == ContactType.physical:
```

Because it inherits from `str`, Python uses **string comparison** when you do `==`.

it is effectively evaluated as:

```python
str.__eq__(ContactType.physical, "physical")
```

Which compares the underlying string value → `"physical" == "physical"` → `True`. That's why you still need to do `print(ContactType.physical.value)` to get the value, becase `print(ContactType.physical)` will not work, because as we knew the `__eq__` is the one gets the string ;) .

---
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

## **complete minimal project**:

Here is a **complete minimal project** showing how a virtual environment fits into a real Python project. This is the simplest “professional-style” layout.

### Step 0 — Project goal

We’ll build a tiny app that:

* Uses an external library (`requests`)
* Runs inside a virtual environment
* Can be recreated by someone else using `requirements.txt`

**Explanation**

This is the real purpose of virtual environments:
You want a project that works the same on **your computer**, **your teammate’s computer**, and **a server**.
This is called **reproducibility**.
### Step 1 — Create the project folder

```bash
mkdir hello_venv_project
cd hello_venv_project
```

Project is empty now.

**Explanation**

* `mkdir hello_venv_project` → creates a new directory for the project.
* `cd hello_venv_project` → moves your terminal into that folder.

We always isolate projects in their own directory so their files don’t mix.

### Step 2 — Create the virtual environment

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

### Step 3 — Activate the environment

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

### Step 4 — Install a package inside the venv

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

### Step 5 — Create the application files

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

#### File 1 — app/main.py

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

#### File 2 — app/**init**.py

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

#### File 3 — requirements.txt

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

#### File 4 — README.md

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

### Step 6 — Run the project

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

### Step 7 — What happens on another computer (important)

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

### Step 8 — What NOT to commit

Create `.gitignore`:

```
venv/
__pycache__/
```

We never commit the venv folder because it can be recreated.

**Explanation**

The venv can be hundreds of MB and is machine-specific.
We only commit the **instructions to rebuild it**.

### Final project tree

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
# How to check if you’re in a virtual environment

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
# Pydantic

Pydantic is a Python library for **data validation, parsing, and settings management using type hints**. Think of it as a strict layer between raw data (JSON, env vars, user input, APIs) and your Python code. It contains many tools:

* `BaseModel` → data validation models
* `Field` → field metadata & constraints
* validators → custom validation
* `BaseSettings` → env/config management
* serialization utilities (JSON export, dict export)
* schema generation (used by FastAPI)
* dataclass integration
* etc.

## Core idea

You describe the **shape of data using Python type hints** → Pydantic automatically:

- validates input data
- converts types when possible
- raises clear errors if data is invalid
- gives you a typed object you can trust

## Why developers use it

Typical problems in Python apps:

- JSON from APIs is messy
- env variables are strings
- user input is unpredictable
- manual validation is repetitive and error-prone

Pydantic solves this by turning validation into **declarative schemas**.

## What BaseModel actually provides

When you inherit from `BaseModel`, your class automatically gains:

### 1) Parsing

```python
User(id="123")  # converts to int
```

### 2) Validation engine

```python
username: str = Field(min_length=3)
```

### 3) Parsing and validation

Python dictionaries (most common):

```python
data = {"id": "123", "name": "Alice"}
User.model_validate(data)
```

Json string:

```python
json_string = '{"id": 1, "username": "Neo", "is_admin": true}'
User.model_validate_json(json_string)
```

Python objects (ORM / custom classes):

Pydantic can read attributes from objects — this is huge for databases.

```python
class DbUser:
    def __init__(self):
        self.id = 1
        self.name = "Alice"

db_user = DbUser()
user = User.model_validate(db_user)
```

It reads attributes like:

```
db_user.id
db_user.name
```
### 3) Serialization

```python
from pydantic import BaseModel

class User(BaseModel):
    id: int
    username: str
    is_admin: bool = False

user = User(id=1, username="neo")

print(user.model_dump())
print(user.model_dump_json())
```

**Output**:

```text
{'id': 1, 'username': 'neo', 'is_admin': False}
{"id":1,"username":"neo","is_admin":false}
```
### 4) Error reporting

Clear `ValidationError` messages.

### 5) Schema generation

Used to generate OpenAPI docs in FastAPI.

## Basic example

```python
from pydantic import BaseModel

class User(BaseModel):
	id: int
	name: str
	is_admin: bool = False

````
  
### Validate incoming data

  
```python
data = {"id": "123", "name": "Alice"}
user = User(id="123", name="Alice")
print(user)
```

**Output**:

```
User(id=123, name='Alice', is_admin=False)
```
### What happened automatically

* `"123"` → converted to `int`
* missing `is_admin` → default applied

## Validation errors

```python
User(id="abc", name="Alice")
```

Raises:

```
ValidationError:
id Input should be a valid integer
```

Instead of silent bugs, you get **precise error messages**.

## Why there is no `__init__`

You don’t write `__init__` because **BaseModel generates one dynamically**. The constructor comes from the parent (`BaseModel`), but it is **generated at runtime using metaprogramming**.
## Nested models


```python
class Address(BaseModel):
	city: str
	zip: int

class User(BaseModel):
	name: str
	address: Address

```

Input:

```python
User(name="John", address={"city": "Paris", "zip": "75000"})
```

Automatically becomes nested objects.
## Settings / environment variables

Very popular feature:

```python
from pydantic_settings import BaseSettings, SettingsConfigDict

class Settings(BaseSettings):
	
	model_config = SettingsConfigDict(env_file=".env", extra="ignore")
	
	api_key: str
	debug: bool = False
	db_url: str
	port: int = 8000
	settings = Settings()

print(settings.api_key) # super-secret-key
print(settings.debug) # True (auto-converted from "true")
print(settings.db_url)
print(settings.port) # 5000 (overrides default)
```

**.env**:

```
API_KEY=super-secret-key
DEBUG=true
DB_URL=postgresql://user:password@localhost:5432/app_db
PORT=5000
```

Reads from environment variables automatically:

```
super-secret-key
True
postgresql://user:password@localhost:5432/app_db
5000
```

Widely used in backend apps and DevOps tooling.
## Where you’ll see Pydantic used

Most common in:
* FastAPI backends (request/response validation)
* CLI tools
* Data pipelines
* Config management
* Microservices

It’s basically the **standard validation layer in modern Python backends**.

## `Field()`

`Field()` is a **helper function from Pydantic used to add validation rules and metadata to a model attribute**.

Think of it as:
Type hint → *what type is this?*
`Field()` → *what rules must the value follow?*

## The simple definition

`Field()` lets you attach **constraints, defaults, and documentation** to a Pydantic model field.

## Why it exists

Without `Field()`, a model only knows the **type**:

```python
class User(BaseModel):
name: str
```

This accepts:

```python
name=""
name="a"*10000
```

Type is correct → but data is bad.

`Field()` adds **validation rules**.

## Basic syntax

  
```python
field_name: type = Field(...)
```

Example:

```python
name: str = Field(min_length=1, max_length=50)
```

Now Pydantic enforces rules when the model is created.

## What Field actually returns

`Field()` does **not store the value**. It returns a special object (`FieldInfo`) that contains:

* default value
* validation constraints
* metadata

Pydantic reads this metadata when building the model and generates the validation logic automatically.

So this:

```python
name: str = Field(min_length=1)
```

Internally becomes something like:

```text
Schema:
- field name: name
- type: string
- rules: min_length = 1
```

## The 3 main things Field is used for

### 1) Validation rules

```python
age: int = Field(ge=0, le=120)
```

Constraints the value range.

### 2) Default values

  
```python
notes: str | None = Field(default=None)
```

Same as writing:

```python
notes: str | None = None
```  

But now you can combine default + validation.

### 3) Required values

In Pydantic, `...` has a very specific meaning:

> **This field is required. There is no default value.**

```python
name: str = Field(..., min_length=3, max_length=10)
```

This means:

- The field **must be provided**
- It must be a string
- Length must be between 3 and 10

If the field is missing → validation error.
### 4) Documentation / metadata (very important in APIs)

```python
name: str = Field(
min_length=1,
description="User full name",
example="Alice"
)
```

Frameworks like FastAPI use this to generate API docs automatically.  

---
# What is schema generation?

Pydantic can automatically create a **machine-readable description of your model**, often used for:

- JSON Schema (`OpenAPI`)
- API validation
- Documentation

Basically, Pydantic reads your type hints, `Field()` constraints, default values, and validators, and produces a **dictionary that describes the model structure**.

Use the `model_dump_schema()` or `model_json_schema()` (for Pydantic v2) methods on your `BaseModel`.

```python
from pydantic import BaseModel, Field

class User(BaseModel):
    id: int = Field(..., gt=0)
    username: str = Field(..., min_length=3, max_length=50)
    is_admin: bool = Field(default=False)
```

```python
json_schema = User.model_json_schema()
print(json_schema)
```

**Output (pretty JSON string):**

```json
{
  "title": "User",
  "type": "object",
  "properties": {
    "id": {"title": "Id", "type": "integer", "exclusiveMinimum": 0},
    "username": {"title": "Username", "type": "string", "minLength": 3, "maxLength": 50},
    "is_admin": {"title": "Is Admin", "type": "boolean", "default": false}
  },
  "required": ["id", "username"]
}
```

---
# Dual Representation of Validation Errors in Pydantic

When working with Pydantic, a single validation failure can be represented in two different ways depending on the audience. The same ValidationError object exposes both a human-readable view and a machine-readable structured view of the error.

* `print(e)` → human-readable string
* `e.errors()` → machine-readable structured data

## What `e` really is

When validation fails, Pydantic raises:

```python
pydantic.ValidationError
```

This class stores errors internally in a **structured list of dictionaries**. That structured form is the **real data**. else (the pretty text) is just formatting layered on top.


## Internal structure (the real source of truth)

Inside the exception, Pydantic stores something like:

```python
[
    {
        "type": "less_than_equal",
        "loc": ("crew_size",),
        "msg": "Input should be less than or equal to 20",
        "input": 25,
        "ctx": {"le": 20},
        "url": "https://errors.pydantic.dev/2.12/v/less_than_equal"
    }
]
```

This is what `e.errors()` returns directly.

This format is designed for:

* APIs
* logging
* JSON responses
* FastAPI error responses

It is **stable and machine-friendly**.

## Then what does `print(e)` do?

When you do:

```python
print(e)
```

Python calls:

```python
e.__str__()
```

Pydantic overrides `__str__()` to **format the structured errors into a readable report**.

So this:

```text
1 validation error for SpaceStation
crew_size
  Input should be less than or equal to 20
```

is just a **pretty rendering** of the structured data.

## key idea

Inside Pydantic validators, **your `ValueError` is NOT propagated as a raw `ValueError`.**
Pydantic **catches it internally** and converts it into a **`ValidationError`**.
So even though _you raise `ValueError`_, the outside world receives a `ValidationError`.

---

# `@field_validator`

`@field_validator` is a **decorator used to write custom validation logic for a specific field** in a Pydantic model. You use `@field_validator` when `Field()` is not enough to express the rule.

It lets you run Python code to check or transform a value after (or before) Pydantic parses it.

Think of it as:

> “Run this function whenever this field is validated.”

for a `@field_validator` It must return the value, Pydantic expects the validated (or transformed) field value back.  If you don’t return it → the field becomes None. 

It should be a class method `(cls)`, Field validators run before the model instance exists, so there is no `self`. That’s why the first parameter is the class: `cls`.
## Basic example

```python
from pydantic import BaseModel, field_validator  
  
class User(BaseModel):  
	username: str  
	  
	@field_validator("username")  
	def must_not_contain_spaces(cls, v):  
		if " " in v:  
			raise ValueError("No spaces allowed")  
		return v
```

Transformation (not just validation)

```python
@field_validator("username")
def normalize(cls, v):
    return v.lower().strip()
```

Whenever `username` is validated, these functions runs automatically.

## Why does it exist?

Because not all validation rules are simple.

Some rules require:

- custom logic
- business rules
- transformations
- checks that Pydantic does not provide by default

That’s what `@field_validator` is for.

## Side-by-side comparison with the `field()`

|Feature|`Field(...)` constraints|`@field_validator`|
|---|---|---|
|Type of rule|Built-in|Custom Python logic|
|Where written|Inside field definition|Decorator function|
|Runs in Rust engine|Yes|No (runs in Python)|
|Performance|Very fast|Slightly slower|
|Complexity supported|Simple constraints|Any logic you want|

---

# `@model_validator`

`@model_validator` is a **Pydantic decorator used to validate the entire model**, not just one field.

If `@field_validator` checks _one attribute_,  
`@model_validator` checks the **relationship between fields**.

`@model_validator(mode="after")` must return self. Because in after mode the model is already created. Pydantic gives you the finished object, lets you check/modify it, and expects you to return the final instance.

Also In before mode `@model_validator(mode="before")` , the model does not exist yet.
Pydantic gives you the raw input data before parsing and validation happen. So you must return the data that Pydantic should continue validating.
## Why it exists

Some rules require **multiple fields together**.

Examples:

- password == confirm_password
- start_date < end_date
- either email or phone must exist    
- total == sum(items)

A single field validator cannot see the whole picture.

## Basic idea

```python
@model_validator(mode="after")
def check_model(cls, model):
    ...
    return self
```

It runs after all fields have already been parsed and validated.

So inside the validator you get a **fully typed model object**.

## Example: password confirmation

```python
from pydantic import BaseModel, model_validator

class User(BaseModel):
    password: str
    confirm_password: str

    @model_validator(mode="after")
    def passwords_match(self):
        if self.password != self.confirm_password:
            raise ValueError("Passwords do not match")
        return self
```

Now this fails:

```python
User(password="secret", confirm_password="123")
```

Because the model-level rule is violated.

## When model validators run

Model validation happens in this order:

```text
1) Raw input
2) Field parsing + conversion
3) Field validators
4) Model validators   ← here
5) Final model object
```

So model validators see **final cleaned data**.

## Two modes of model validators

### 1) `mode="after"` (most common)

Runs **after validation**.  
You receive the **model instance**.

Use this for:

- comparing fields
- business rules
- derived checks

Signature:

```python
@model_validator(mode="after")
def validate_model(self):
    return self
```


### 2) `mode="before"`

Runs **before validation**.  
You receive the **raw input dictionary**.

Use this when you need to:

- modify incoming data
- merge fields
- accept alternative input formats

Example:

```python
@model_validator(mode="before")
def preprocess(cls, data):
    if "full_name" in data:
        first, last = data["full_name"].split()
        data["first_name"] = first
        data["last_name"] = last
    return data
```

This runs **before any field parsing happens**.

## field_validator vs model_validator

|Feature|field_validator|model_validator|
|---|---|---|
|Sees one field|✓|✗|
|Sees entire model|✗|✓|
|Used for relationships|✗|✓|
|Can modify raw input|✗|✓ (before mode)|
## Mental model

- `@field_validator` → validate a value
    
- `@model_validator` → validate the object

You **cannot (and should not) call a `@model_validator` yourself**, even it is technically callable.
Even though it looks like a normal method, Pydantic registers it as a **validation hook** during class creation. It is executed **automatically inside the model validation pipeline** when you create or validate a model (e.g., `User(**data)` or `User.model_validate(data)`).

Here’s a **simple example showing what it would look like if you tried to call a `@model_validator` manually** — and why that’s not how it’s meant to be used.

### Model with a `@model_validator`

```python
from pydantic import BaseModel, model_validator

class User(BaseModel):
    username: str
    password: str

    @model_validator(mode="after")
    def check_password_length(self):
        if len(self.password) < 8:
            raise ValueError("Password too short")
        return self
```

### Normal (correct) usage — Pydantic calls it automatically

```python
user = User(username="neo", password="12345678")
print(user)
```

Flow:

```
User(...)
  → validation starts
  → model_validator runs automatically
  → object created
```

### Manually calling the validator (NOT recommended)

Because it’s just a method, Python technically lets you call it:

```python
u = User(username="neo", password="12345678")

# Manually calling the validator
u.check_password_length()
```

This works **technically**, but it is wrong conceptually because:

• It runs _outside_ the validation pipeline  
• It skips parsing and field validation  
• It can break assumptions Pydantic relies on  
• It defeats the purpose of validators

---

# `Callable` VS `callable()`

Two different things here: **`Callable` (type hint)** vs **`callable()` (runtime check)**.
## 1) From which package should you import `Callable`?

Recommended modern import:

```python
from typing import Callable
```

or:

```python
from collections.abc import Callable
```

Example:

```python
from typing import Callable
#from collections.abc import Callable

def apply_spell(spell: Callable[[str, int], str]) -> str:
    return spell("orc", 10)
```

Meaning of the hint:

```
Callable[[arg_types], return_type]
```

Example meanings:

|Type hint|Meaning|
|---|---|
|`Callable[[], int]`|function taking no args → returns int|
|`Callable[[int], str]`|takes int → returns str|
|`Callable[[str, int], bool]`|takes str & int → returns bool|

So `Callable` is purely for **static typing** (mypy, IDE, linters).

It does **nothing at runtime**.

## 2) What is `callable()`?

This is a **built-in function** in Python.

It checks at runtime if an object can be called with `()`.

```python
callable(object) -> bool
```

### Examples

```python
callable(len)          # True (function)
callable(print)        # True (function)
callable(lambda x: x)  # True (lambda)
callable(42)           # False (int)
callable("hello")      # False (string)
```

---
# Closure

A **closure** is a function that keeps the variables from the environment where it was created, so it can use them later even after that environment no longer exists.

In short:

> A closure = function + remembered variables.

## Use your example step-by-step

```python
def make_multiplier(n):
    def multiply(x):
        return x * n
    return multiply
```

### What happens when Python runs this?

1. Python defines `make_multiplier`.
2. Nothing special yet — no closure exists.
## Step 1 — Create the closure

```python
double = make_multiplier(2)
```

Execution flow:

1. `make_multiplier(2)` runs.
    
2. Local variable created inside it:
    
    ```
    n = 2
    ```
    
3. Inner function `multiply` is created.
    
4. `multiply` **uses `n` from outer scope** → closure is formed.
    
5. `make_multiplier` returns `multiply`.
    

Important:  
`make_multiplier` is now finished, its stack frame is gone…  
BUT Python keeps `n = 2` alive because `multiply` still needs it.

So `double` now equals a function that secretly carries `n = 2`.
## Step 2 — Prove the function remembers

```python
print(double.__closure__)
```

Output (shape):

```
(<cell at 0x...: int object at 0x...>,)
```

That cell is the “memory backpack”.
### Extract the stored value

```python
print(double.__closure__[0].cell_contents)
```

Output:

```
2
```

This proves the function still holds the value from the outer function.
## Step 3 — Call the closure

```python
result = double(5)
print(result)
```

When this runs:

```
multiply(5)
→ return 5 * n
→ return 5 * 2
→ 10
```

Output:

```
10
```

Even though `make_multiplier` finished earlier, the value `n = 2` is still available.

That is the closure in action.
## Why this is powerful

Each call creates a **separate memory**:

```python
double = make_multiplier(2)
triple = make_multiplier(3)
```

Now we have two different closures:

|Function|Remembered value|
|---|---|
|`double`|n = 2|
|`triple`|n = 3|

They share the same code, but carry different hidden state.
## How to recognize a closure (final checklist)

A closure exists when:

1. A function is defined inside another function ✔
    
2. The inner function uses a variable from the outer function ✔
    
3. The outer function returns the inner function ✔
    

All three happen in `make_multiplier`.

---
# `global` VS `nonlocal`

`global` and `nonlocal` both let a function modify a variable that is **outside its own local scope**.  
The difference is _where that variable lives_ and _how risky it is to modify it_.

`global` modifies a variable at the **module level** (shared by the whole program).  
`nonlocal` modifies a variable in the **enclosing function** (private to a closure).

Why projects forbid `global` but allow `nonlocal`:

When you use `global`, you change shared state that _any function anywhere_ can also change. That makes programs harder to reason about, test, and debug because the value can change from many places.

With `nonlocal`, the state stays **inside a small, controlled scope** (the outer function). Only the inner functions of that closure can touch it, so it stays predictable and encapsulated.

Simple example with `global`:

```python
count = 0

def increment():
    global count
    count += 1

increment()
increment()
print(count)  # 2
```

Here `count` belongs to the whole file. Any function could modify it.

Simple example with `nonlocal`:

```python
def counter():
    count = 0

    def increment():
        nonlocal count
        count += 1
        return count

    return increment

c = counter()
print(c())  # 1
print(c())  # 2
```

Here `count` is private inside `counter`. Nothing outside can touch it, but the inner function can safely update it.

So the idea is simple:

- `global` changes **shared program state** → risky.
- `nonlocal` changes **private closure state** → controlled and safe.

---
# Functions are first-class citizens

When we say **functions are first-class citizens in Python**, we mean they behave like any other object (like ints, strings, lists, dicts).  
Anything you can do with normal objects, you can do with functions.

There are 5 main properties.
## 1) Functions can be assigned to variables

```python
def greet(name):
    return f"Hello {name}"

say_hi = greet
print(say_hi("Ali"))
```

You didn’t copy the function — you created another reference to the same object.

This proves functions are **values**.
## 2) Functions can be stored in data structures

```python
def add(x): return x + 1
def double(x): return x * 2

operations = [add, double]
print(operations)  # 6
```

Functions live inside lists, dicts, tuples, etc.
## 3) Functions can be passed as arguments

This creates **higher-order functions**.

```python
def apply(func, value):
    return func(value)

apply(len, "magic")  # 5
```

You passed a function just like you’d pass a number.
## 4) Functions can be returned from functions

```python
def make_multiplier(n):
    def multiply(x):
        return x * n
    return multiply

double = make_multiplier(2)
double(10)  # 20
```

Functions can be **created dynamically** and returned.
## 5) Functions exist at runtime (they are objects)

You can inspect them:

```python
print(type(len))
```

Output:

```
<class 'builtin_function_or_method'>
```

Functions have:

- attributes
- identity
- memory address
- can be passed around

They are real objects in memory.

## The big picture

Because functions are first-class objects, Python supports:

- higher-order functions (`map`, `filter`, `sorted`)
- decorators    
- callbacks
- functional programming patterns
## Short definition to remember

> Functions are first-class citizens because they can be treated like any other value: stored, passed, returned, and manipulated at runtime.

---
# Functional Programming 

Functional Programming is a programming style where **functions are the main building blocks** and computation is done by **combining and transforming data using functions**, rather than changing program state.

Core idea: instead of telling the computer _how to do steps_, you focus on _what result you want from functions_.

### Key characteristics

**1) Functions are first-class values**  
You can store them, pass them, and return them.

```python
def add(a, b):
    return a + b

operation = add
print(operation(2, 3))  # 5
```

**2) Pure functions**  
A pure function:

- depends only on its inputs
    
- has no side effects
    
- always returns the same output for the same input
    

```python
def square(x):
    return x * x
```

No global variables, no printing, no modifying outside data.

**3) Immutability**  
Instead of changing data, you create new data.

Imperative style:

```python
numbers = [1,2,3]
numbers.append(4)  # mutated
```

Functional style:

```python
numbers = [1,2,3]
new_numbers = numbers + [4]  # new list
```

**4) Higher-order functions**  
Functions that take or return other functions.

Examples in Python:

```python
map()
filter()
sorted()
```

```python
nums = [1,2,3,4]
squared = list(map(lambda x: x*x, nums))
```

**5) Function composition**  
Building complex behavior by combining small functions.

```python
def double(x): return x*2
def add3(x): return x+3

result = add3(double(5))  # 13
```
### Simple mental model

Imperative programming:

> Do this, then this, then change that.

Functional programming:

> Transform this data into new data using functions.

---
# Decorator

A decorator is a function that takes another function, adds behavior to it, and returns a new function. The original function’s code stays the same, but what happens when you call it changes.

## Start **without** the `@` syntax:

```python
def outerfunc(func):
    def innerfunc():
        print("Before")
        func()
        print("After")
    return innerfunc

def great():
    print("Hello")

say_hi = outerfunc(great)
say_hi()
```

What happens here:

1. `great` is created normally.
    
2. We pass that function into `outerfunc`.
    
3. Inside the decorator, a new function called `innerfunc` is created.
    
4. `innerfunc` remembers the original function (`func`) thanks to a closure.
    
5. The decorator returns `innerfunc`.
    
6. The name `say_hi` now refers to `innerfunc`, not the original function.
    

So calling `say_hi()` actually calls `innerfunc`, which calls the original function inside it.

That’s why the output becomes:

```
Before
Hello
After
```

The original function didn’t change, but its behavior did.
## Now the `@` syntax is just a shortcut.

This:

```python
@outerfunc
def great():
    print("Hello")
```

is automatically rewritten by Python as:

```python
def great():
    print("Hello")

great = outerfunc(great)
```

So when Python sees `@outerfunc`, it:

1. Creates the function object `great`
    
2. Passes it to `outerfunc`
    
3. Replaces `great` with whatever the decorator returns
    

Inside the decorator:

- `func` becomes the original `great`
    
- `innerfunc` is created and remembers `func`
    
- `innerfunc` is returned
    
- `great` now points to `innerfunc`
    

When you call `great()`:

- You are really calling `innerfunc()`
    
- `innerfunc()` runs code before and after the original function
    

Mental model:

```
great → innerfunc → original great
```

A decorator is simply a function that wraps another function to add extra behavior around it.
## Notice:

A decorator is executed **immediately when Python defines the function**, not when you call it.

It takes the original function, builds an `innerfunc`, and replaces the original name with it.

Simple example:

```python
def outerfunc(func):
    def innerfunc():
        print("Before")
        func()
        print("After")
    return innerfunc

@outerfunc
def great():
    print("Hello")

great()
```

## What happens:

When Python sees `@outerfunc`, it instantly does:

```python
great = outerfunc(great)
```

So:

- `outerfunc` runs immediately
    
- it creates `innerfunc`
    
- `innerfunc` wraps `great`
    
- `great` becomes `innerfunc`
    

Important rule: **a function is not always passed automatically to a decorator.**
It depends on how the decorator is written.

There is a **different situations**.

When you write Decorator WITH parentheses → function is NOT passed yet:

```python
@power_validator(50)
def fireball(power):
    ...
```

Python rewrites it in **two steps**:

Step 1 — call the decorator factory:

```python
decorator_function = power_validator(50)
```

At this moment:

* Python is only configuring the decorator
* The function `fireball` does **not exist yet**
* So nothing can be passed to it

Step 2 — after the function is created:

```python
def fireball(power):
    ...

fireball = decorator_function(fireball)
```

Now the function is finally passed.

## The key idea

If you see:

```
@decorator
```

→ function is passed immediately.

If you see:

```
@decorator(...)
```

→ first Python calls the decorator,
then later it passes the function to the returned inner decorator.

---
# functools

`functools` is a built-in Python module that provides **tools for functional programming**, especially utilities that work with functions.

Short idea:  
It contains helpers that make working with higher-order functions, decorators, and function composition easier.

### Most important things inside `functools`

**1) `wraps`**  
Used in decorators to preserve the original function’s metadata.

**2) `partial`**  
Creates a new function with some arguments already fixed.

```python
from functools import partial

def power(base, exp):
    return base ** exp

square = partial(power, exp=2)
print(square(5))  # 25
```

You created a new function from another function.

**3) `reduce`**  
Applies a function repeatedly to combine values.

```python
from functools import reduce

nums = [1, 2, 3, 4]
result = reduce(lambda a, b: a + b, nums)
print(result)  # 10
```

It reduces a list to one value.

**4) `lru_cache`**  
Caches function results (memoization) for speed.

```python
from functools import lru_cache

@lru_cache
def fib(n):
    if n < 2:
        return n
    return fib(n-1) + fib(n-2)
```

Very useful for expensive calculations.

---
# `wraps`

`wraps` is a helper from `functools` used **inside decorators** to preserve the original function’s metadata.

When you decorate a function, the original function gets replaced by the wrapper.  
Without `wraps`, Python thinks the wrapper _is_ the function.

### The problem without `wraps`

```python
def outerfunc(func):
    def innerfunc():
        return func()
    return innerfunc

@outerfunc
def greet():
    """Say hello"""
    print("Hello")

print(greet.__name__)
print(greet.__doc__)
```

Output:

```
innerfunc
None
```

Python lost the original function info 😬  
This breaks debugging, help(), documentation, frameworks, etc.
### The solution: `functools.wraps`

```python
from functools import wraps

def outerfunc(func):
    @wraps(func)
    def innerfunc():
        return func()
    return innerfunc
```

Now:

```python
@outerfunc
def greet():
    """Say hello"""
    print("Hello")

print(greet.__name__)
print(greet.__doc__)
```

Output:

```
greet
Say hello
```

### What `wraps` copies

It copies metadata from the original function:

- function name (`__name__`)
    
- docstring (`__doc__`)
    
- module name
    
- annotations
    
- other metadata
    

---
# `partial`

`partial` (from `functools`) lets you **create a new function by fixing some arguments of an existing function**.

Think of it as **pre-filling parameters**.

`partial` is different from default parameters because default values change the original function itself, while `partial` leaves the original function unchanged and creates a new specialized function with some arguments already pre-filled.

### Basic idea

```python
from functools import partial
```

Suppose we have:

```python
def multiply(a, b):
    return a * b
```

Create a new function that always multiplies by 2:

```python
double = partial(multiply, 2)

print(double(5))  # 10
```

What happened?

```text
double(x) → multiply(2, x)
```

We “locked” the first argument. Python matches arguments **from left to right** (positional order).

### Example with named arguments

```python
def power(base, exp):
    return base ** exp

square = partial(power, exp=2)
cube   = partial(power, exp=3)

print(square(4))  # 16
print(cube(4))    # 64
```

You created new specialized functions from one generic function.

### Why it’s useful

Without `partial`:

```python
def square(x):
    return power(x, 2)
```

With `partial`:

```python
square = partial(power, exp=2)
```

Less code, more reusable.

---
# `operator`

The `operator` module is a built-in Python module that provides **functions for common operators**.

Instead of writing operators like `+`, `*`, or `==`, it gives you **function versions** of them.

This is useful in functional programming when you need to pass an operation as a function (to `map`, `sorted`, `reduce`, etc.).
### Example idea

Normally:

```python
3 + 5
```

With `operator`:

```python
from operator import add
add(3, 5)  # 8
```

Same operation, but now it’s a function you can pass around.
### Why it exists

Many functional tools expect a **function**, not an operator symbol.

Example with `reduce`:

```python
from functools import reduce
from operator import mul

nums = [1, 2, 3, 4]
result = reduce(mul, nums)
print(result)  # 24
```

Instead of writing a lambda:

```python
reduce(lambda a, b: a * b, nums)
```

Cleaner and faster.
### Very common functions in `operator`

Arithmetic:

```python
operator.add(a, b)   # a + b
operator.sub(a, b)   # a - b
operator.mul(a, b)   # a * b
operator.truediv(a,b)# a / b
```

Comparisons:

```python
operator.eq(a, b)    # a == b
operator.gt(a, b)    # a > b
operator.lt(a, b)    # a < b
```

---
# `lru_cache`

`lru_cache` is a decorator from `functools` that **caches (stores) the results of a function** so it doesn’t recompute the same input again.

### Basic idea

If you call a function with the same input multiple times, Python normally recalculates it each time.

With `lru_cache`, Python:

- saves the result the first time
    
- reuses it next time
    
- makes execution much faster
    

### Example

```python
from functools import lru_cache

@lru_cache
def add(x, y):
    print("computing...")
    return x + y
```

Now:

```python
print(add(2, 3))
print(add(2, 3))
```

Output:

```
computing...
5
5
```

👉 The second call does NOT recompute.

### Why “LRU”?

LRU = **Least Recently Used**

If the cache gets full:

- Python removes the oldest unused results
    
- keeps the most recently used ones
    

### Real use case (classic: Fibonacci)

Without cache (slow):

```python
def fib(n):
    if n < 2:
        return n
    return fib(n-1) + fib(n-2)
```

With cache (fast):

```python
from functools import lru_cache

@lru_cache
def fib(n):
    if n < 2:
        return n
    return fib(n-1) + fib(n-2)
```

Now repeated calculations are reused instead of recomputed.

In Fibonacci, the whole point of `lru_cache` is that **many calls reuse values that were already computed before**.

### Fibonacci rule

```text
fib(n) = fib(n-1) + fib(n-2)
```

So every value depends on smaller values

### Scenario: computing `fib(5)`

Let’s expand it **without cache first**:

```
fib(5)
→ fib(4) + fib(3)

fib(4)
→ fib(3) + fib(2)

fib(3)
→ fib(2) + fib(1)
```

Notice something important:

#### `fib(3)` is computed multiple times

* once in `fib(5)`
* once in `fib(4)`

#### `fib(2)` is also repeated many times

### Where cached values are reused

Now imagine we use:

```python id="l1c9r2"
from functools import lru_cache

@lru_cache
def fib(n):
    if n < 2:
        return n
    return fib(n-1) + fib(n-2)
```

### Step-by-step reuse scenario

### 1. First time:

```text id="k8p3xv"
fib(2) → computed → stored
```

Cache:

```
{2: 1}
```

### 2. Later in recursion:

When Python reaches again:

```text id="m3n9qv"
fib(3)
→ needs fib(2)
```

Instead of recomputing:

* it **fetches fib(2) from cache**

### 3. Even bigger reuse:

When computing `fib(5)`:

```text id="w7c2bd"
fib(5)
→ fib(4)
→ fib(3)
→ fib(2)  ← already computed earlier
```

So `fib(2)` is reused multiple times instantly.

---
# `cache_info()`

To verify that caching is working with `lru_cache`, you use:

```python
memoized_fibonacci.cache_info()
```

This returns a named tuple showing **how effective the cache is**.

## What `cache_info()` shows

It gives 4 values:

```text
CacheInfo(hits, misses, maxsize, currsize)
```

### Meaning of each field

- **hits** → how many times Python reused a cached result
    
- **misses** → how many times Python had to compute a new value
    
- **maxsize** → maximum cache capacity (None = unlimited)
    
- **currsize** → current number of stored cached results
    

## Example

```python
from functools import lru_cache

@lru_cache
def fib(n):
    if n < 2:
        return n
    return fib(n-1) + fib(n-2)

fib(10)

print(fib.cache_info())
```

## Typical output

```text
CacheInfo(hits=8, misses=11, maxsize=128, currsize=11)
```

## How to interpret it

- `misses=11` → 11 unique computations were done
    
- `hits=8` → 8 times Python reused already computed values
    
- This proves caching is actively working
    

## How to _prove caching is working_

If caching is effective:

```text
hits > 0
```

If caching is NOT working:

```text
hits = 0
```

## Extra check (very clear proof)

Call the same function twice:

```python
fib(20)
fib(20)

print(fib.cache_info())
```

Second call increases:

- **hits increases**
    
- **misses does NOT increase**
    

That’s the strongest proof caching is working.

---
# `singledispatch`

`functools.singledispatch` is a decorator that lets you create a **single function with multiple implementations depending on the type of the first argument**.

This is called **generic function dispatch**.

## Core idea

Instead of writing:

```python
if type(x) is int:
    ...
elif type(x) is str:
    ...
```

You write **separate functions per type**, and Python chooses the correct one automatically.

## Basic example

```python
from functools import singledispatch

@singledispatch
def process(value):
    print("Default:", value)
```

Now we register special behavior for types:

```python
@process.register(int)
def _(value):
    print("Integer:", value * 2)

@process.register(str)
def _(value):
    print("String:", value.upper())
```

## Usage

```python
process(10)
process("hello")
process(3.14)
```

Output:

```text
Integer: 20
String: HELLO
Default: 3.14
```

## How it works

When you call:

```python
process(value)
```

Python:

1. Looks at the **type of `value`**
    
2. Finds the registered function for that type
    
3. Executes it
    

If no match → uses the **default implementation**

## Key rule

Only the **first argument** is used for dispatch.

```python
process(value, other)
```

→ dispatch depends only on `value`

---
# `multipledispatch`

`multipledispatch` is a library that lets you **choose which function to run based on the types of all arguments**, not just the first one.

### Simple idea

> Same function name, different behavior depending on argument types.

### Example with `(str, int)`

```python
from multipledispatch import dispatch

@dispatch(str, int)
def process(name, power):
    return f"{name} boosted by {power} points"
```

### Usage

```python
print(process("fire", 10))
```

Output:

```text
fire boosted by 10 points
```

---
# Why use `*args, **kwargs` with decorators
### First idea: decorators don’t know the future

When you write a decorator, you should assume it might be used on **any function**, not just the one you are testing with.

Functions can have many possible signatures:

- no parameters
- positional parameters
- keyword parameters
- many parameters
- methods with `self`
- any mix of the above


Because of this uncertainty, decorators are usually written using:

```python
*args, **kwargs
```

This lets the decorator accept **any possible function signature** and forward the arguments safely.

### Why your `spell_timer` works _by coincidence_

Current version:

```python
def wrapper():
    result = func()
```

It works only because it decorates this function:

```python
def fireball():
```

which takes **no arguments**.

But imagine you later decorate a function that needs parameters:

```python
@spell_timer
def fireball(power):
    ...
```

Now Python will try to pass `power` to the wrapper, but the wrapper accepts **no parameters**, so the program crashes with:

```
TypeError: wrapper() takes 0 positional arguments but 1 was given
```

### The correct real-world version

A proper decorator must forward _any_ arguments:

```python
def wrapper(*args, **kwargs):
    result = func(*args, **kwargs)
```

This makes the decorator **future-proof and reusable**.