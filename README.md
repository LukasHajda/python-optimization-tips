# 🐍 Python Performance Optimization Guide

This repository presents practical techniques and illustrative examples to enhance the **performance**, **efficiency**, and **memory utilization** of Python applications.

## 🚀 Topics Covered

- [Using `__slots__` to Reduce Memory Consumption](#using-__slots__-to-reduce-memory-consumption)
- [Optimizing Garbage Collection Thresholds](#optimizing-garbage-collection-thresholds)
- [Comparing List Comprehensions and Traditional Loops](#comparing-list-comprehensions-and-traditional-loops)
- [Leveraging Built-in Functions and Generators](#leveraging-built-in-functions-and-generators)
- [Local Variable Access Optimization](#local-variable-access-optimization)
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

- **Reduced memory usage:** Eliminates per-instance `__dict__`, significantly decreasing memory consumption.
- **Faster attribute access:** Fixed attribute layout enables quicker lookups.
- **Enhanced performance:** Particularly effective when creating large numbers of instances.

### Disadvantages

- **Limited flexibility:** Attributes cannot be added dynamically beyond those specified in `__slots__`.
- **Inheritance constraints:** Subclasses must also define `__slots__` to maintain memory benefits.
- **Compatibility issues:** Some libraries or frameworks that rely on dynamic attribute assignment may not function properly.

## Garbage Collection Threshold Optimization

Python’s built-in garbage collector (GC) periodically frees unused memory by cleaning up unreachable objects. For applications that generate a large number of short-lived objects, tuning the GC thresholds can help reduce collection frequency, thereby minimizing pause times and enhancing performance.

### Viewing and Adjusting GC Thresholds

The garbage collector manages memory in three generations (0, 1, and 2), each with its own threshold that determines when collection occurs. You can inspect and modify these thresholds as demonstrated below:

```python
import gc

# Perform a full collection of generation 2
gc.collect(2)

# Display current GC thresholds (default values are typically 700, 10, 10)
print(gc.get_threshold())

# Adjust thresholds to increase collection intervals
gc.set_threshold(10_000, 20, 20)
```
### Advantages

- **Reduced GC overhead:** Increasing thresholds decreases the frequency of garbage collection, which can reduce the performance impact of GC pauses.
- **Improved performance in high-allocation workloads:** Especially beneficial for applications with rapid creation and disposal of many short-lived objects.
- **Fine-grained control:** Allows tuning of memory management to better match application-specific behavior and resource constraints.

### Disadvantages

- **Increased memory usage:** Delaying garbage collection may cause higher memory consumption as unused objects remain longer before being collected.
- **Risk of memory leaks:** Improper tuning or disabling GC can cause memory leaks if unreachable objects are not collected promptly.
- **Requires careful profiling:** Without profiling, threshold adjustments may degrade performance rather than improve it.

> **Note:** There is no universal or "magic" formula for tuning garbage collection thresholds. Optimal settings vary depending on the application's workload and behavior. It is essential to profile and test your specific application to determine the best configuration.

For a detailed discussion, see [Michael Kennedy's article](https://mkennedy.codes/posts/python-gc-settings-change-this-and-make-your-app-go-20pc-faster/).

