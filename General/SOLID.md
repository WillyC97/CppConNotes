## SOLID Principles

* Speaker: Klaus Iglberger
* Talk: [Breaking Dependencies: The SOLID Principles](https://youtu.be/Ntraj80qN2k)

### Overview

> "Coupling is the enemy of change, because it **links together things that
> must change in parallel**.”
>
> _-- David Thomas, Andrew Hunt, The Pragmatic Programmer_

> ”Dependency is the **key problem** in software development at all scales.”
>
> _-- Kent Beck, TDD by Example_

* **S**ingle-Responsibility Principle
* **O**pen-Closed Principle
* **L**iskov Substitution Principle
* **I**nterface Segregation Principle
* **D**ependency Inversion Principle

#### Single-Responsibility Principle (SRP)

> ”Orthogonality: … We want to design components that are **self-contained: independent, and with a single, well-defined purpose**.
> When components are isolated from one another, you know
> that you can **change one without having to worry about the rest**.”
>
> -- _Andrew Hunt, David Thomas, The Pragmatic Programmer_

Elements inside a module should be highly cohesive - They should be a collection
of statements and data items that should be trated as a whole because they are so
close together.

---
**Guideline** Prefer cohesive software entities. Everything that does not strictly
belong together, should be separated.

---

#### Open-Closed Principle (OCP)

> ”Software artifacts (classes, modules, functions, etc.) should be open
> for extension, but closed for modification.”
>
> -- _Bertrand Meyer, Object-Oriented Software Construction_

Example: Typed-based programming descended from C code.

The procedural approach taken by the above should be replaced by an OOP approach.

In an example of a shape class, that uses an enum-type to determine which type of
shape to construct, using virtual methods in the `Shape` class should be favoured,
as it allows for the addition of more shapes, without requiring modification to the
existing code.

Favour:
``` c ++
class Shape
{
 public:
 Shape() = default;
 virtual ~Shape() = default;

 virtual void translate( Vector3D const& ) = 0;
 virtual void rotate( Quaternion const& ) = 0;
 virtual void draw() const = 0;
};
```

Over:
``` c ++
enum ShapeType
{
 circle,
 square,
 rectangle
};

class Shape
{
 public:
 explicit Shape( ShapeType t )
 : type{ t }
 {}
 virtual ~Shape() = default;
 ShapeType getType() const noexcept;
 private:
 ShapeType type;
};
```

---
**Guideline**: Prefer software design that allows the addition of types or
operations without the need to modify existing code.
___

#### The Liskov Substitution Principle (LSP)

Subtypes must be substitutable for their base types.

> "What is wanted here is something like the following substitution
> property: If for each object o1 of type S there is an object o2 of type T
> such that for all programs P defined in terms of T, the behavior of P is
> unchanged when o1 is substituted for o2 then S is a subtype of T."
>
> -- _Barbara Liskov, Data Abstraction and Hierarchy_


**Behavioral** subtyping (aka “IS-A” relationship):

* Contravariance of **method arguments** in a subtype
* Covariance of **return types** in a subtype
* **Preconditions** cannot be strengthened in a subtype
* **Postconditions** cannot be weakened in a subtype
* **Invariants** of the super type must be preserved in a subtype

---
**Guideline**: Make sure that inheritance is about behaviour, not about data.

**Guideline**: Make sure that the contract of base types is adhered to.

**Guideline**: Make sure to adhere to the required concept.
___

#### The Interface Segregation Principle (ISP)

> "Many client specific interfaces are better than one general-purpose interface"


Referring back to the Shapes exmaple, Rather than including the drawing methods
within each shape class, implement DrawingStrategies for each shape.

``` c ++
class Circle;
class Square;

class DrawCircleStrategy
{
 public:
 virtual ~DrawCircleStrategy() {}
 virtual void draw( const Circle& circle ) const = 0;
};

class DrawSquareStrategy
{
 public:
 virtual ~DrawSquareStrategy() {}
 virtual void draw( const Square& square ) const = 0;
};
```

``` c ++
class Circle : public Shape
{
 public:
 explicit Circle( double rad, std::unique_ptr<DrawCircleStrategy> ds )
 : radius{ rad }
 , // ... Remaining data members
 , drawing{ std::move(ds) }
 {}
 virtual ~Circle() = default;
 double getRadius() const noexcept;
 // ... getCenter(), getRotation(), ...
 void translate( Vector3D const& ) override;
 void rotate( Quaternion const& ) override;
 void draw() const override;
 private:
 double radius;
 // ... Remaining data members
 std:unique_ptr<DrawCircleStrategy> drawing;
};
```

---
**Guideline**: Make sure interfaces don't induce unnecessary dependencies

---

#### Dependency Inversion Principle (DIP)

> ”The Dependency Inversion Principle (DIP) tells us that the most flexible
> systems are those in which source code dependencies refer only to
> abstractions, not to concretions.”
>
> -- _Robert C. Martin, Clean Architecture_

> "a. High-level modules should not depend on low-level modules. Both
> should depend on abstractions.
> b. Abstractions should not depend on details. Details should depend on
> abstractions."
>
> -- _Robert C. Martin, Agile Software Development_

___
**Guideline**: Prefer to depend on abstractions (i.e. abstract classes or
concepts) instead of concrete types.

___
