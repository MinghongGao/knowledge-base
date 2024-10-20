---
cover: >-
  https://images.unsplash.com/photo-1653074281018-c08f358059ab?crop=entropy&cs=srgb&fm=jpg&ixid=M3wxOTcwMjR8MHwxfHNlYXJjaHwxfHxmdW5jdGlvbnN8ZW58MHx8fHwxNzI5NDU5MzM1fDA&ixlib=rb-4.0.3&q=85
coverY: 0
---

# What are side effects ?

## Understanding Side Effects in Programming

In programming, understanding how functions and operations interact with the state of your program is crucial. One of the key concepts that often comes up in this context is **side effects**. This blog will explain what side effects are, why they matter, and how they can affect the behavior of your code.

### What Are Side Effects?

In computer science, an operation, function, or expression is said to have a side effect if it has any observable effect other than its primary effect of reading the value of its arguments and returning a value to the invoker of the operation. Side effects can include:

* Modifying a non-local variable
* Changing a static local variable or a mutable argument passed by reference
* Raising errors or exceptions
* Performing I/O operations (like reading or writing files, printing to the console)
* Calling other functions that have side effects

The presence of side effects means that a program's behavior may depend on its history, making the order of evaluation important. For example, if two functions modify the same variable, the final value of that variable will depend on which function is executed first.

### Why Do Side Effects Matter?

Side effects play an important role in the design and analysis of programming languages. They can make functions harder to understand and debug because you need to consider not only what the function does but also how it interacts with the rest of the program.

For instance, if a function reads from a global variable and changes it, the function's output might vary based on the global variable's state. This makes it difficult to reason about the function's behavior in isolation. Conversely, functions without side effects, often called **pure functions**, are easier to test and reason about because they always produce the same output for the same input.

### Side Effects Across Different Paradigms

The use of side effects depends on the programming paradigm:

* **Imperative Programming**: In imperative programming (e.g., C, Java), side effects are often used to update a program's state. This includes changing variables, reading and writing files, or communicating with external systems.
* **Declarative Programming**: In contrast, declarative programming (e.g., SQL, Haskell) minimizes or eliminates side effects. Declarative programs describe what should be done, rather than how to do it, often leading to functions that report on the state of the system without altering it.

