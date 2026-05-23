# Python OOP Concepts

## 1️⃣ Classes and Objects

* **Class** – Blueprint for creating objects.
* **Object (Instance)** – A real entity created from a class.

**Example:**
```python
class Car:
    def __init__(self, brand, color):
        self.brand = brand
        self.color = color
    
    def drive(self):
        return f"Driving a {self.color} {self.brand}"

# Creating objects (instances)
car1 = Car("Toyota", "Red")
car2 = Car("Honda", "Blue")

print(car1.drive())  # Output: Driving a Red Toyota
print(car2.drive())  # Output: Driving a Blue Honda
```

---

## 2️⃣ Methods

* **Instance Methods** – Work on object data; first parameter is `self`.
* **Class Methods** – Work on class data; first parameter is `cls`; use `@classmethod`.
* **Static Methods** – Don't use object or class data; utility functions; use `@staticmethod`.
* **Constructors (`__init__`)** – Initialize object data when it's created.
* **Destructors (`__del__`)** – Cleanup before object is deleted.

**Example:**
```python
class Dog:
    species = "Canine"  # Class variable
    
    def __init__(self, name, age):
        self.name = name
        self.age = age
    
    def bark(self):  # Instance method
        return f"{self.name} says Woof!"
    
    @classmethod
    def get_species(cls):  # Class method
        return f"This is a {cls.species}"
    
    @staticmethod
    def is_adult(age):  # Static method
        return age >= 2

# Creating object
dog = Dog("Buddy", 3)
print(dog.bark())  # Output: Buddy says Woof!
print(dog.get_species())  # Output: This is a Canine
print(Dog.is_adult(3))  # Output: True
```

---

## 3️⃣ Variables

* **Instance Variables** – Belong to objects; accessed via `self`.
* **Class Variables** – Shared across all objects of the class.
* **Local Variables** – Exist inside a method; temporary.
* **Global Variables** – Defined outside class/method; accessible globally.

---

## 4️⃣ Encapsulation

* **Private Variables/Methods** – Prefix with `_` or `__` to hide from outside.
* **Public Variables/Methods** – Accessible from anywhere.
* **Getters and Setters** – Methods to access or update private variables.

---

## 5️⃣ Inheritance

* **Single Inheritance** – One class inherits from another.
* **Multiple Inheritance** – A class inherits from multiple classes.
* **Multilevel Inheritance** – Chain of inheritance.
* **Hierarchical Inheritance** – One parent, multiple children.
* **Method Overriding** – Child class replaces parent method.

---

## 6️⃣ Polymorphism

* **Compile-time / Method Overloading** – Same method name, different parameters (Python doesn't support traditional overloading).
* **Run-time / Method Overriding** – Child class changes parent method.
* **Operator Overloading** – Same operator behaves differently for objects.

---

## 7️⃣ Abstraction

* **Abstract Classes** – Cannot instantiate; only subclass.
* **Abstract Methods** – Must be implemented in child class.
* **In Python**, implemented using the `abc` module.

---

## 8️⃣ Other Features

* `self` – Refers to object instance.
* `cls` – Refers to class itself.
* `super()` – Access parent class methods and variables.
* **Composition / Aggregation** – Classes can contain objects of other classes.