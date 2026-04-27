# Many functions with no grouping section

**Trigger:** A file has many top-level definitions (more than 7-10 functions, classes, or constants) with no visual grouping. The reader must scan linearly through a long flat list.

**Why:** Without grouping, a file with 15+ definitions feels like an undifferentiated wall. Headings create scannable sections that let readers jump to the relevant area.

**Fix:** Introduce comment headings to group related definitions. Use the heading style appropriate to the scope:

**Top-level definitions** (module-level functions, classes, constants):
```python
# ------------------------------------------------------------------------------
# Section Name

def some_function() -> None: ...
```

**Within class definitions** (methods, properties, nested types):
```python
class MyClass:
    # === Properties ===

    @property
    def name(self) -> str: ...

    # === Operations ===

    def do_something(self) -> None: ...

    # === Utility ===

    def _helper(self) -> int: ...
```

Aim for groups of 7-10 items each. If a group grows beyond that, consider whether it should be split further or whether some definitions should move to a separate module.

**When NOT to revise:** In small files where all definitions are naturally visible without scrolling. Headings in a file with 5 functions add noise rather than structure.
