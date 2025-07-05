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
