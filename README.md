# Basic Rewriting Library: Automated Reasoning

This repository contains a basic rewriting library to be used as a basis for
the submission of assignments on the automated reasoning course.

## Library structure and important files

The library is organized as follows.

```txt
.
├── LICENSE
├── README.md
├── lpo_solver.py
├── main.py
├── term.py
├── test.py
├── trs.py
└── utils.py
```

The most important modules are ``term.py``, ``trs.py``, and ``lpo_solver.py``.

- ``term.py`` defines the data structures for terms and implement some simple
functions over it: term equality, and printing facilities.
- ``trs.py`` defines the data structures for rules and rewriting systems.
- ``lpo_solver.py`` implements a lpo solving algorithm.
It can be used as a basis for more complicated versions of RPO implementations.

## Installation and Basic Requirements

This codebase uses features of python3 version 3.11.6 or newer so make sure that
your environment meets this minimum requirement.

What do you need to use it then:

- Python3 version 3.11.6 or later
- Z3 SMT Solver should be installed system-wide
  - See, [https://github.com/Z3Prover/z3](https://github.com/Z3Prover/z3)
  for installation instructions.
- You will need to install the python bindings for Z3.

  ```bash
       pip install z3-solver
    ```

- A code editor. I recommend either ``vscode``, with the python extension,
or PyCharm, there is a free version available.
In my experience, the linting and error checking in PyCharm is superior to that of
``vscode``. It should provide you with better error reporting and better
Python virtual environments.

## Using and extending this library with your own assignment code

The fundamental piece of code in the library is that defining the data structure for terms.
Recall the formal definition: a term is either a variable or a functional application
f(t_1, ..., t_n) where t_1, ..., t_n are themselves terms.
Each function symbol f has an arity number, which sets how many arguments it can take.

In this library, we define terms in ``terms.py`` and it is essentially written as follows:

```python
@dataclass
class Var:
    name: str
```

This defines variables. Next, we define the data for function symbols:

```python
@dataclass
class FnSym:
    name: str
    arity: int
```

Reflecting the fact that every function symbol has a name
and an arity, which is an integer number.
Now we need to apply function symbols to terms.
Which we do recursively as follows:

```python
@dataclass
class FnApp:
    fnApp: tuple[FnSym, list[Term]]
```

So a functional application is encoded as a pair of a symbol
and a list of ``Term`` instances.
Finally, ``Term`` is then encoded using recursion:

```python
Term: UnionType = Var | FnApp
```

This code finally says that a term is either a variable or a functional application.
It should be very close to the mathematical notation and recursive definition
you are also using in the class.

If you look at the code in ``term.py``, you will see more code
than only defining the datatype.
Don't worry about it. It is there just to guarantee
printing format and whatnot.
You don't need to actually read nor understand it.

### Defining functions over terms

The best way to write functions that operate over terms
is then to do pattern matching over the arguments.
So we consider all possible cases.
An example from ``terms.py``:

```python
def term_eq(s: Term, t: Term) -> bool:
    match (s, t):
        case (Var(x), Var(y)):
            return x == y
        case (Var(_), FnApp(_)):
            return False
        case (FnApp(_), Var(_)):
            return False
        case (FnApp(l), FnApp(r)):
            return (l[0] == r[0]) and list_eq(term_eq, l[1], r[1])
        case _:
            raise TypeError("Arguments to term_eq should be instances of Terms.")
```

Notice that we do a case-analysis on all possible combinations
and produce a result for each case.
This library uses pattern matching on all functions
that operate over ``Term``.
So you will see this a lot in the code.
Your own implementations should also use it.
Take a look at the functions in
``term.py`` and ``lpo_solver.py`` for instance.

## Encoding Term Rewriting Systems

Now let us look on how to encode TRSs internally. 
This might be useful for debugging and testing your code.
Consider the TRS described as follows:
```text
x : var
y : var

f : 1
g : 2

Rule 1: f(x) -> x
Rule 2: g(f(x), y) -> f(y)
```

In a separate file, we have to import the ``term.py`` module.

```python
import term as tm
```

We can then define the objects to encode the TRS.

```python
x = tm.Var("x")
y = tm.Var("y")
```

The two function symbols.

```python
f = FnSym("f", 1)
g = FnSym("g", 2)
```

```python
# Now we encode the first rule, f(x) -> x

fx = FnApp((f, [x]))

rule1 = trs.Rule(fx, x)

# Now we encode the second rule, g(f(x), y) -> f(y)
fy = FnApp((f, [y]))
gf = FnApp((g, [fx, y]))

rule2 = trs.Rule(gf, fx)
```
