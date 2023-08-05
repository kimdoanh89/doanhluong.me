---
layout: single
title:  "Introduction to SOLID Principles"
date:   2023-07-16
categories: oop
toc: true
toc_label: "On This Post"
toc_sticky: true
---

Hey friends! Today I'm talking about SOLID - five principles that will level up your object-oriented 
programming game.

I used to think SOLID was one of those abstract concepts that's great in theory but doesn't help
day-to-day coding. Boy, was I wrong!

Sticking to these principles has made my code way easier to reason about and modify. I encounter
way fewer bugs and hacky workarounds now.

Let's get into it!

SOLID is an acronym that stands for five key principles of object-oriented class design:

- S - Single Responsibility Principle 
- O - Open/Closed Principle
- L - Liskov Substitution Principle
- I - Interface Segregation Principle
- D - Dependency Inversion Principle

These principles encourage designs that are robust, extensible and maintainable. Applying them
consistently leads to code that is flexible, modular and easier to understand, refactor and test.

## 1. Single Responsibility Principle 

The single responsibility principle states that a class should have only one reason to change.
In other words, a class should have only one job and encapsulate that functionality entirely. 

Here are some symptoms that indicate code is not following the single responsibility principle:
- A class has too many methods or instance variables
- Methods in a class do unrelated things - e.g. calculate tax and send email
- Changes to one functionality breaks other functionality
- God object anti-pattern - one huge central class

To refactor code to follow single responsibility:
- Split large classes into smaller, logical classes 
- Group methods and variables based on functionality
- Create new classes for each responsibility with descriptive names based on their purpose
- Remove duplication by extracting common code to new classes 

For example, refactoring a Customer class:

```python
# BEFORE

class Customer:

  def __init__(self):
    self.name
    self.address
    self.orders = []
    self.cart = []

  def add_order(self, order):
    self.orders.append(order)

  def add_to_cart(self, item):
    self.cart.append(item)

  def get_billing_address(self):
    return self.address

  def calculate_loyalty_discount(self, amount):
    # logic to calculate discount
    return discounted_amount

# AFTER 

class Customer:

  def __init__(self, name, address):
    self.name = name
    self.address = address

class OrderService:
  
  def __init__(self, customer):
    self.customer = customer
  
  def add_order(self, order):
    self.customer.orders.append(order)

class CartService:

  def __init__(self, customer):
    self.customer = customer

  def add_to_cart(self, item):
    self.customer.cart.append(item)

class BillingService:

  def __init__(self, customer):  
    self.customer = customer

  def get_billing_address(self):
    return self.customer.address

class LoyaltyService:

  def __init__(self, customer):
    self.customer = customer

  def calculate_loyalty_discount(self, amount):
    # logic 
    return discounted_amount
```

This ensures the Customer class only handles core customer data, while other classes handle specific responsibilities.

## 2. Open/Closed Principle

The open/closed principle states that classes should be open for extension but closed for modification. We should be able to add new functionality without changing existing code.

Here are some symptoms that indicate code is not following the open/closed principle:
- Need to modify existing classes to add new features
- Adding new sub-classes breaks parent class or other subclasses
- Lots of conditional statements (`If/Else` or `Switch/Case`) to handle different cases

To refactor code to be open for extension and closed for modification:
- Use interfaces and abstract classes for stable class contracts 
- Favor composition over inheritance when adding new functionality
- Use dependency injection to inject implementations rather than hardcoding
- Implement new features in new classes that extend existing classes

For example, refactoring validation logic:

```python
# BEFORE

class Validator:

  def validate(self, name, age, phone):
    if len(name) < 5:
      return False
    
    if not age > 0:
      return False

    if not is_valid_phone(phone):
      return False

    return True

# Need to modify to add new validations

# AFTER

class Validator:

  def validate(self, validations):
    for v in validations:
      if not v.validate():
        return False
    return True

class NameValidation:

  # name validation 

class AgeValidation:

  # age validation

# New validations implement same interface
# No need to modify Validator

validator = Validator([
  NameValidation(),
  AgeValidation()
])
```

This makes it easy to add new functionality by creating new classes, without changing existing code.

## 3. Liskov Substitution Principle 

This principle states that child classes should be substitutable for their parent classes.
Behavior in the parent class should remain consistent when it is overridden in a child class.

Here are some symptoms that indicate code is violating the Liskov substitution principle:
- Child classes don't properly implement parent class methods, do some additional "stuff"
beyond parent class methods, and change expectations of how they work.
- Parent class methods throw unexpected errors when called on child objects, need to check
the type of child objects.

To refactor code to follow Liskov substitution principle:
- Make sure subclasses match full input/output behavior of parent classes
- Avoid throwing exceptions in child methods not thrown in parents
- Favor composition over inheritance if behavior differs substantially
- Make base classes abstract if methods need different implementations and split into separate class
hierarchies if behaviors differ
- Use dependency inversion - depend on abstractions not concrete classes

For example, refactoring validation classes:

```python 
# BEFORE

class Validator:

  def validate(self, obj):
     # validate object

class IntValidator(Validator):
  
  # Also logs validation?? 
  def validate(self, obj):
    super().validate(obj)  
    logger.log("Validating int object")

# Violates LSP - Extra logging behavior 

# AFTER 

class Validator:
  
  def validate(self):
    pass
    
class IntValidator(Validator):

  def validate(self, obj):
    # validate object

class LoggingValidator(Validator):

  def validate(self, obj):
    result = super().validate(obj)
    logger.log("Validating object")
    return result

# Uses composition to add logging without changing contract 
validator = LoggingValidator(IntValidator())
```

This ensures child classes have consistent contracts as parent classes, and unexpected behavior is avoided when substituting implementations.

## 4. Interface Segregation Principle

This principle states that clients should not be forced to depend on methods they don't use from
an interface. Interfaces should be split to serve specific client needs.

Here are some symptoms that indicate code is violating the interface segregation principle:
- Interfaces have many methods unrelated to each other and changes affect many classes.
- Clients are forced to depend on and implement unused methods
- Classes contain empty method stubs to satisfy interfaces

To refactor code to follow interface segregation principle: 

- Split large interfaces into smaller, role-specific interfaces
- Move unrelated methods to new interfaces
- Use composition from multiple interfaces rather than one large interface
- Avoid inheriting from interfaces just to gain single method

For example, splitting interface for devices:

```python
# BEFORE

class DeviceInterface:

  def power_on(self) 
  def power_off()
  def write_data()
  def read_data()
  def encrypt_data()

# Forces all devices to implement unneeded methods

# AFTER 

class PowerInterface:

  def power_on()
  def power_off()

class DataInterface:

  def write_data()
  def read_data() 

class EncryptInterface:

  def encrypt_data()
  
# Devices can implement only interfaces they need

class HardDrive(PowerInterface, DataInterface):

  ...

class EncryptedDrive(PowerInterface, DataInterface, EncryptInterface):

  ... 
```

This results in lean, focused interfaces that group logically related operations.

## 5. Dependency Inversion Principle
This principle states that high-level modules should not depend on low-level implementation details.
Details should depend on abstractions.

Here are some symptoms that indicate code is violating the dependency inversion principle:
- Tight coupling between high-level and low-level classes
- Business logic depends directly on concrete implementations
- Changing implementations requires changing lots of classes
- Unit testing is difficult due to many hardcoded dependencies
- Lack of abstraction makes code hard to extend

To refactor code to follow dependency inversion principle:
- Depend on abstractions (interfaces) rather than concrete classes
- Inject dependencies through constructor or setter methods 

For example, refactoring data access code:

```python
# BEFORE

class MySQLDataStore:

  def get_data(self):
    # implementation 

class DataAnalyzer:

  def __init__():  
    self.db = MySQLDataStore()
  
  def analyze(self):
    data = self.db.get_data()
    # analyze data

# Directly coupled to MySQLDataStore

# AFTER 

class DataStore:

  def get_data():
    pass

class MySQLDataStore(DataStore):

  def get_data():
     # implementation

class DataAnalyzer:

  def __init__(data_store):
    self.db = data_store

  def analyze(self):
    data = self.db.get_data()
    # analyze data 

# Decoupled from concrete implementation
```

This removes tight coupling between modules and enables easier extensibility and testing.

## 6. Conclusion

Applying the SOLID principles consistently improves code quality and maintainability. Code becomes
more modular, reusable and adaptable to change when needed. While principles like single 
responsibility may seem obvious, actually ollowing them strictly takes discipline. Use these
principles as guiding tenets when designing object oriented systems.


