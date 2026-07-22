- [Initialize All Parent Attributes in A Child Class without Re-declare](#initialize-all-parent-attributes-in-a-child-class-without-re-declare)
  - [1. Using `*args` and `**kwargs` (Recommended)](#1-using-args-and-kwargs-recommended)
  - [2. Using `@dataclass` (Cleanest Modern Python Solution)](#2-using-dataclass-cleanest-modern-python-solution)
  - [Key Takeaways](#key-takeaways)

# Initialize All Parent Attributes in A Child Class without Re-declare

To initialize all parent attributes in a child class without having to re-declare or manually pass every single parameter, you can use **`super().__init__()`** along with Python's variable positional and keyword arguments (**`*args`** and **`**kwargs`**).

---

## 1. Using `*args` and `**kwargs` (Recommended)

By using `**kwargs` (or `*args` for positional arguments), the child class accepts extra keyword arguments meant for the parent class and passes them directly up via `super().__init__()`.

```python
class Parent:
    def __init__(self, name, age, address, email, phone, occupation):
        self.name = name
        self.age = age
        self.address = address
        self.email = email
        self.phone = phone
        self.occupation = occupation


class Child(Parent):
    def __init__(self, student_id, grade, **kwargs):
        # 1. Call the parent constructor passing all extra keyword arguments
        super().__init__(**kwargs)
        
        # 2. Initialize the new child-specific attributes
        self.student_id = student_id
        self.grade = grade


# Example Usage:
child_obj = Child(
    student_id="S12345",
    grade="10th",
    name="Alice",
    age=15,
    address="123 Main St",
    email="alice@example.com",
    phone="555-0199",
    occupation="Student"
)

print(child_obj.name)        # Output: Alice
print(child_obj.student_id)  # Output: S12345

```

---

## 2. Using `@dataclass` (Cleanest Modern Python Solution)

If you are using Python 3.7+ and want a clean structure without manually writing `__init__` boilerplate at all, use `dataclasses`. Child dataclasses automatically inherit and initialize parent dataclass fields.

```python
from dataclasses import dataclass

@dataclass
class Parent:
    name: str
    age: int
    address: str
    email: str
    phone: str
    occupation: str

@dataclass
class Child(Parent):
    # Just list the new attributes; __init__ and super().__init__() are generated automatically!
    student_id: str
    grade: str


# Example Usage:
child_obj = Child(
    name="Bob",
    age=16,
    address="456 Elm St",
    email="bob@example.com",
    phone="555-0188",
    occupation="Student",
    student_id="S98765",
    grade="11th"
)

print(child_obj.email)       # Output: bob@example.com
print(child_obj.student_id)  # Output: S98765

```

---

## Key Takeaways

* **`super().__init__(**kwargs)`**: Best for traditional class definitions. It prevents you from needing to type out every single parent parameter inside the child `__init__`.
* **Dataclasses**: Best when your classes primarily serve to hold data, eliminating constructors altogether.