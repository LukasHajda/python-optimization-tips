# 🐍 Python Performance Optimization Guide

This repository presents practical techniques and illustrative examples to enhance the **performance**, **efficiency**, and **memory utilization** of Python applications.

## 🚀 Topics Covered

- [Using `__slots__` to Reduce Memory Consumption](#using-__slots__-to-reduce-memory-consumption)
- [Optimizing Garbage Collection Thresholds](#optimizing-garbage-collection-thresholds)
- [Comparing List Comprehensions and Traditional Loops](#comparing-list-comprehensions-and-traditional-loops)
- [Local Variable Access Optimization](#local-variable-access-optimization)
- [List Preallocation for Faster Write](#list-preallocation-for-faster-write)
- [Leveraging Built-in Functions and Generators](#leveraging-built-in-functions-and-generators)
- [Efficient String Concatenation Methods](#efficient-string-concatenation-methods)
- [Using `set` for Fast Membership Testing](#using-set-for-fast-membership-testing)

---

## Using `__slots__` to Reduce Memory Consumption

The `__slots__` construct can be employed in class definitions to avoid the per-instance `__dict__`, thereby minimizing memory overhead when instantiating many objects.

```python
from pympler import asizeof

class RegularUser:
    def __init__(self, username, email, is_active, roles, preferences):
        self.username = username
        self.email = email
        self.is_active = is_active
        self.roles = roles
        self.preferences = preferences

# Optimized class using __slots__
class SlottedUser:
    __slots__ = ['username', 'email', 'is_active', 'roles', 'preferences'] # <-- Here

    def __init__(self, username, email, is_active, roles, preferences):
        self.username = username
        self.email = email
        self.is_active = is_active
        self.roles = roles
        self.preferences = preferences

user1 = RegularUser("alice", "alice@example.com", True, ["admin"], {"theme": "dark"})
user2 = SlottedUser("bob", "bob@example.com", False, ["user"], {"theme": "light"})

print(f"RegularUser memory size: {asizeof.asizeof(user1)}")
print(f"SlottedUser memory size: {asizeof.asizeof(user2)}")
```
```bash
RegularUser memory size: 1080
SlottedUser memory size: 680
```
In this example, using a slotted class leads to an approximate **37% reduction** in memory consumption compared to a regular class.

### Advantages

- ✅ **Reduced memory usage:** Eliminates per-instance `__dict__`, significantly decreasing memory consumption.
- ✅ **Faster attribute access:** Fixed attribute layout enables quicker lookups.
- ✅ **Enhanced performance:** Particularly effective when creating large numbers of instances.

### Disadvantages

- ❌ **Limited flexibility:** Attributes cannot be added dynamically beyond those specified in `__slots__`.
- ❌ **Inheritance constraints:** Subclasses must also define `__slots__` to maintain memory benefits.
- ❌ **Compatibility issues:** Some libraries or frameworks that rely on dynamic attribute assignment may not function properly.

## Optimizing Garbage Collection Thresholds

Python’s built-in garbage collector (GC) periodically frees unused memory by cleaning up unreachable objects. For applications that generate a large number of short-lived objects, tuning the GC thresholds can help reduce collection frequency, thereby minimizing pause times and enhancing performance.

### Viewing and Adjusting GC Thresholds

The garbage collector manages memory in three generations (0, 1, and 2), each with its own threshold that determines when collection occurs. You can inspect and modify these thresholds as demonstrated below:

```python
import gc

# Perform a full collection
gc.collect()

# Display current GC thresholds (default values are typically 700, 10, 10)
print(gc.get_threshold())

# Adjust thresholds to increase collection intervals
gc.set_threshold(10_000, 20, 20)
```

This example demonstrates the impact of Python’s garbage collector (GC) on an application that creates a large number of short-lived objects. The code measures the runtime of object creation under different GC settings, illustrating how tuning GC thresholds can affect performance.

## Code Overview

```python
import gc
import time

class MyObject:
    def __init__(self, value):
        self.value = value

def create_objects(n):
    objs = []
    for i in range(n):
        objs.append(MyObject(i))
    return objs

# Number of objects to create
N = 5_000_000
```

This code benchmarks the runtime of creating a large number of objects (`N = 5,000,000`) under different garbage collection (GC) configurations to illustrate the performance impact of tuning GC thresholds.

```python
# Run with default GC thresholds
gc.collect()
start_default = time.time()
create_objects(N)
end_default = time.time()
default_duration = end_default - start_default

print(f"Runtime with default GC thresholds: {default_duration:.4f} seconds")

# Tune GC thresholds to reduce collection frequency
gc.set_threshold(10_000, 50, 50)

gc.collect()
start_tuned = time.time()
create_objects(N)
end_tuned = time.time()
tuned_duration = end_tuned - start_tuned

print(f"Runtime with tuned GC thresholds: {tuned_duration:.4f} seconds
```

## Performance Comparison of Garbage Collection Thresholds

| GC Thresholds (gen0, gen1, gen2) | Approximate Runtime (seconds) |
|----------------------------------|-------------------------------|
| 700, 10, 10                     | 3.4186                        |
| 10,000, 20, 20                  | 2.4594                        |
| 10,000, 50, 50                  | 2.0037                        |
| 50,000, 50, 50                  | 1.9703                        |

*Note: Runtimes represent the duration to create 5,000,000 objects in a test environment. Actual results may vary depending on hardware and workload.*

### Advantages

- ✅ **Reduced GC overhead:** Increasing thresholds decreases the frequency of garbage collection, which can reduce the performance impact of GC pauses.
- ✅ **Improved performance in high-allocation workloads:** Especially beneficial for applications with rapid creation and disposal of many short-lived objects.
- ✅ **Fine-grained control:** Allows tuning of memory management to better match application-specific behavior and resource constraints.

### Disadvantages

- ❌ **Increased memory usage:** Delaying garbage collection may cause higher memory consumption as unused objects remain longer before being collected.
- ❌ **Risk of memory leaks:** Improper tuning or disabling GC can cause memory leaks if unreachable objects are not collected promptly.
- ❌ **Requires careful profiling:** Without profiling, threshold adjustments may degrade performance rather than improve it.

> **Note:** There is no universal or "magic" formula for tuning garbage collection thresholds. Optimal settings vary depending on the application's workload and behavior. It is essential to profile and test your specific application to determine the best configuration.

For a detailed discussion, see [Michael Kennedy's article](https://mkennedy.codes/posts/python-gc-settings-change-this-and-make-your-app-go-20pc-faster/).

## Comparing List Comprehensions and Traditional Loops
One common Python optimization technique is replacing traditional for loops with list comprehensions. List comprehensions are generally faster and more concise, especially when you're building new lists.

```python
import time

N = 35_000_000

def traditional_loop(n):
    squares_loop = []
    for i in range(n):
        squares_loop.append(i)

def comprehension_loop(n):
    squares_comp = [i for i in range(n)]

# Traditional loop benchmark
start = time.time()
traditional_loop(N)
print(f"Traditional loop time: {time.time() - start:.4f} seconds")

# List comprehension benchmark
start = time.time()
comprehension_loop(N)
print(f"Comprehension loop time: {time.time() - start:.4f} seconds")
```
### List Comprehensions

#### Advantages

- ✅ **Faster execution:** Compiled into optimized C code under the hood.
- ✅ **More concise:** Less code for simple list transformations.
- ✅ **Improved readability** for straightforward operations.
- ✅ **Functional-style syntax** encourages declarative thinking.

#### Disadvantages

- ❌ **Hard to read** with complex logic or nested conditions.
- ❌ **No side effects allowed** (e.g. printing or logging inside the expression).
- ❌ **May use more memory** if not careful (vs. generators).

---

### Traditional For Loops

#### Advantages

- ✅ **Clearer for complex logic**, multiple operations, or side-effects.
- ✅ **Easier to debug** and maintain line-by-line.
- ✅ **Flexible for conditional branching**, logging, or interacting with other parts of a program.

#### Disadvantages

- ❌ **Slower** due to method calls (e.g., `.append()`) in Python-level code.
- ❌ **More verbose**, especially for simple operations.
- ❌ **Slightly less efficient** in tight performance-critical loops.

### 📊 Benchmark: Traditional Loop vs List Comprehension

| N (Elements) | Traditional Loop (s) | List Comprehension (s) | % Improvement |
|--------------|----------------------|--------------------------|----------------|
| 15,000,000   | 0.8969               | 0.5821                   | 35.09%         |
| 25,000,000   | 1.4530               | 1.0087                   | 30.59%         |
| 35,000,000   | 2.0529               | 1.3764                   | 32.96%         |
| 70,000,000   | 4.0457               | 2.7963                   | 30.88%         |

List comprehensions consistently outperform traditional loops by 30–35%, making them a strong choice for large-scale data generation tasks where simplicity and speed matter.

## Local Variable Access Optimization
In Python, accessing local variables is faster than accessing global variables or attributes. Python’s interpreter resolves local variables more efficiently because they are stored in an internal structure that is quicker to look up (like a C array under the hood).

```python
import time
import math

def use_global(n):
    for _ in range(n):
        val = math.pi * 2

def use_local(n):
    local_pi = math.pi
    for _ in range(n):
        val = local_pi * 2

N = 30_000_000

# Global variable access benchmark
start = time.time()
use_global(N)
end = time.time()
print(f"Global access time: {end - start:.4f} seconds")

# Local variable access benchmark
start = time.time()
use_local(N)
end = time.time()
print(f"Local access time:  {end - start:.4f} seconds")
```

### Advantages

- ✅ Faster access to variables (especially in tight loops).  
- ✅ Better performance in functions that access the same variable many times.  
- ✅ Encourages encapsulation and cleaner function scopes.  

### Disadvantages

- ❌ Requires manual refactoring to copy global values into locals.  
- ❌ May clutter code with unnecessary assignments if misused.  
- ❌ Minimal gain in functions with few variable lookups or outside of tight loops.

### 📊 Benchmark: Global vs Local Variable Access

| N (Elements) | Global Access (s) | Local Access (s) | % Improvement |
|--------------|-------------------|------------------|---------------|
| 10,000,000   | 0.5725            | 0.3575           | 37.55%        |
| 20,000,000   | 1.3555            | 0.7128           | 47.40%        |
| 30,000,000   | 1.8159            | 1.1028           | 39.28%        |

This benchmark clearly shows that local variable access in Python is consistently faster than accessing variables globally, with improvements ranging roughly between **37% to 47%** in our tests

## List Preallocation for Faster Writes
Preallocating a list by creating it with a fixed size (e.g. `[0] * N`) can offer slight performance improvements in scenarios where the list size is known in advance. This avoids the overhead of repeated memory resizing that occurs with `.append()` operations.

While this optimization is generally **not critical** and often results in **negligible gains** for most real-world applications, it’s an interesting detail that can be worth considering in tight loops or large-scale numerical tasks. It’s more of a micro-optimization than a game-changer—but useful to know when squeezing out every bit of performance.

```python
import time

N = 40_000_000

def benchmark_dynamic_append(n):
    start = time.time()
    lst = []
    for i in range(n):
        lst.append(i)
    end = time.time()
    return end - start

def benchmark_preallocated(n):
    start = time.time()
    lst = [0] * n
    for i in range(n):
        lst[i] = i
    end = time.time()
    return end - start

dynamic_time = benchmark_dynamic_append(N)
print(f"Dynamic append: {dynamic_time:.4f} sec")

prealloc_time = benchmark_preallocated(N)
print(f"Preallocated assignment: {prealloc_time:.4f} sec")
```
### 📊 Benchmark: Dynamic Append vs. Preallocated List

| N Elements       | Dynamic Append (sec) | Preallocated (sec) | Improvement (%)  |
|------------------|----------------------|---------------------|------------------|
| 10,000,000       | 0.5837               | 0.3120              | **46.5% faster** |
| 30,000,000       | 1.6121               | 0.9348              | **42.0% faster** |
| 100,000,000      | 5.4583               | 3.1222              | **42.8% faster** |
| 500,000,000      | 17.4425              | 9.4788              | **45.7% faster** |

### 🧠 Insight

Preallocating a list reduces the need for dynamic memory resizing during appends, which can result in performance gains of **40–46%** in large-scale list construction. While the absolute time saved may not be significant for small workloads, this optimization becomes more impactful as the number of elements grows.

## Leveraging Generators for Memory Efficiency
This example demonstrates the difference in memory usage between list comprehensions and generator expressions in Python.

## Overview

- **List comprehension** creates an entire list in memory immediately, which can consume significant memory for large datasets.

- **Generator expression** produces items one at a time, generating values on-the-fly without storing the entire sequence in memory. This leads to much lower memory consumption, especially beneficial for processing large or infinite sequences.

```python
import tracemalloc

N = 1_000_000

# Profile list comprehension (creates full list in memory)
tracemalloc.start()
lst = [x * x for x in range(N)]
current, peak = tracemalloc.get_traced_memory()
print(f"List comprehension — Current memory usage: {current / 1024**2:.2f} MB; Peak: {peak / 1024**2:.2f} MB")
tracemalloc.stop()

# Profile generator expression (lazy evaluation, low memory)
tracemalloc.start()
gen = (x * x for x in range(N))
total = sum(gen)  # Consume generator to measure memory use
current, peak = tracemalloc.get_traced_memory()
print(f"Generator expression — Current memory usage: {current / 1024**2:.2f} MB; Peak: {peak / 1024**2:.2f} MB")
tracemalloc.stop()
```

## Memory Usage Comparison

| Method               | Memory Usage       |
|----------------------|--------------------|
| List Comprehension   | 38.45 MB           |
| Generator Expression | Less than 1 MB     |

### Advantages
- ✅ **Reduced memory usage:** Generators only keep one item in memory at a time.
- ✅ **Suitable for large datasets:** Can process sequences too large to fit in memory.
- ✅ **Lazy evaluation:** Values are generated only as needed, improving performance in many cases.

### Disadvantages
- **Single iteration:** Generators can only be iterated once.
- **No random access:** Unlike lists, you cannot index or slice generators.
- **Slightly more complex code:** Sometimes requires more careful handling (e.g., converting to list if needed multiple times).

