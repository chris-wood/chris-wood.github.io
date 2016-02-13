---
layout: post
title: Polymorphism in C
---

Polymorphic behavior is an incredibly useful construct in software systems.
There are generally two ways by which a program can be constructed to exhibit
polymorphic behavior. Namely, through composition or inheritance. Polymorphism
via composition relies on (1) well-defined and narrow interfaces and (2) other objects
or types *containing* references to things that implement those interfaces. To give
a modern example, consider the following Go program.

```go
package main

import "fmt"

type description struct {
    breedName string
    color     string
}

func (d description) toString() string {
    return fmt.Sprintf("%s %s", d.color, d.breedName)
}

type animal interface {
    identify() string // oof.
}

type cat struct {
    desc description
    name string
}
type dog struct {
    desc description
    name string
}

func (c cat) identify() string {
    return fmt.Sprintf("%s -- %s\n", c.name, c.desc.toString())
}

func (d dog) identify() string {
    return fmt.Sprintf("%s -- %s\n", d.name, d.desc.toString())
}

func main() {
    c := cat{name: "Salem", desc: description{breedName: "Siamese Cat", color: "Blue"}}
    d := dog{name: "Buddy", desc: description{breedName: "Labrador Retriever", color: "Black"}}
    fmt.Println(c.identify())
    fmt.Println(d.identify())
}
```

We have a generic ```animal``` interface and then two structs that implement
this interface: ```dog``` and ```cat```. Each animal *has* a description which
is able to produce a simple description of the animal. When animals are asked
to identify themselves they invoke ask their specific description implementation
for a description. This is the fundamental idea behind polymorphism behind
composition. Since the description fields embedded in the animal structs
implements some well-known interface (in this case, just the description one),
the specific instance of that field determines the behavior of the animal
at runtime. Thus, we say that polymorphic behavior is achieved by delegating
to the *implementation of an object*.

Inheritance, on the other hand, achieves polymorphic behavior through the
*interface* of a class. Or, put another way, composition allows
reuse at the object level whereas inheritance allows for reuse at the
class level. For example, if a class called Child inherits from
a class called Parent then it must be true that a Child instance
can be substituted in any place where an instance of type Parent is
needed. This is the basic idea behind the Liskov Substitution Principle [1]
and is the L in the famed SOLID principles for OO programming.

Most pragmatic programmers recognize that code re-use is more easily
achieved through composition than inheritance since polymorphic behavior
is not bound to compile-time types but rather run-time object instances.
Many design patterns such as the Strategy pattern [2] use composition
(object delegation) for polymorphic behavior. This is regarded
as the Composite Reuse Principle in the literature [3].

Even still, inheritance has its place for a reason. When an object
truly does exemplify the *is-a* relationship instead of a *has-a*
relationship, inheritance is perfectly plausible. For example, an
array list is most certainly a specific type of list, and so making
an array list implementation inherit from a generic list implementation
(or implement a list interface) is quite acceptable.

This leads me to the main topic of this post. Since I spend the majority
of my time programming in C, which is not an object-oriented language,
I often find myself yearning for the benefits of inheritance. To that end,
I will discuss one way of implementing polymorphism in C via C++-like inheritance.

# Polymorphic Shapes

To show how we might implement polymorphic behavior via inheritance we will
implement a general Shape hierarchy. For now, shapes will only have the ability
to compute their area. We will implement concrete Square and Circle shapes that
will be "instances of" the Shape "class." Before showing the code, I'll first
describe the general design. The Shape base will be responsible for two things:
(1) storing a table of function pointers that point to the implementations of
specific (concrete) functions (e.g., Circle's ```Area``` function) and (2)
invoking the correct concrete function with the appropriate concrete instance.
This is done by storing a function table and pointer to the concrete type in
a Shape instance. This code is shown below.

```C
typedef struct shape_interface {
    double (*Area)(void *instance);
} ShapeInterface;

typedef struct {
    void *instance;
    const ShapeInterface *interface;
} Shape;

Shape *
shape_Create(void *instance, ShapeInterface *interface)
{
    Shape *shape = (Shape *) malloc(sizeof(Shape));
    shape->instance = instance;
    shape->interface = interface;
    return shape;
}

double
shape_Area(Shape *shape)
{
    return (shape->interface->Area)(shape->instance);
}
```

Shapes are created with a pointer to the instance and an specific function
table (```ShapeInterface```). In the ```shape_Area``` function, we simply
call the ```Area``` function in the function table and pass in the
instance provided during construction.

To implement concrete Shape types we need to write functions that adhere
to the interface specified in the shape interface. Specifically, we need
to implement the ```Area``` function, which takes as input a pointer to
_something_ and returns a ```double```. The following code shows how we
might do this for a Square.

```C
typedef struct {
    double x;
} Square;

ShapeInterface *SquareAsShape = &(ShapeInterface) {
    .Area = (double (*)(void *)) = square_Area
};

Square *
square_Create(double sideLength)
{
    Square *square = (Square *) malloc(sizeof(Square));
    square->x = sideLength;
    return square;
}

double
square_Area(Square *square)
{
    return square->x * square->x;
}
```

And now for a Circle.

```C
#incluce <math.h>

typedef struct {
    double radius;
} Circle;

ShapeInterface *CircleAsShape = &(ShapeInterface) {
    .Area = (double (*)(void *)) = circle_Area
};

Circle *
circle_Create(double radius)
{
    Circle *circle = (Circle *) malloc(sizeof(Circle));
    circle->radius = radius;
    return circle;
}

double
circle_Area(Circle *circle)
{
    return M_PI * (circle->radius * circle->radius);
}
```

That's it. We have the pieces necessary to create concrete Shape types
and then instantiate general Shapes from them. This allows us to pass
around Circles and Squares wherever Shapes are needed, thereby abiding
by Liskov's priniciple. The code below shows how we might create a
Circle and Square instance, "cast" them to general Shape instances,
and then compute their area through the ```shape_Area``` function.

```C
int
main(int argc, char **argv)
{
    // Create concrete types.
    Circle *circle = circle_Create(5.0);
    Square *square = square_Create(10.0);

    // Wire up the tables.
    Shape *circleShape = shape_Create(circle, CircleAsShape);
    Shape *squareShape = shape_Create(square, SquareAsShape);

    // Sanity check.
    printf("Equal circle areas? %d\n", circle_Area(circle) == shape_Area(circleShape));
    printf("Equal square areas? %d\n", square_Area(square) == shape_Area(squareShape));

    // ... free up memory

    return 0;
}
```

# Differences from C++

As an object oriented language, C++ also provides polymorphism through
inheritance. And it does so using a similar technique to what I described
above for shapes. Specifically, it uses what's called a virtual function
table (vtable) to point to the appropriate implementations of functions [4].
In C++, the compiler is responsible for configuring the vtable for a class
hierarchy. It inserts a reserved field called ```__vptr``` in the base
class of a hierarchy. This field points to the vtable for an object belonging
to that hierarchy. Specifically, when an object from this hierarchy is
created, the ```__vptr``` field is initialized to point to the vtable of
the object's class (not the base class... unless the object is an instance
of the base class). The fields of this vtable instance point to
implementations of each function for the respective class. For example,
using our shape example again, if a Circle object was created then the
vtable Area function would point to the Circle's implementation of
the Area function rather than the Shape's implementation.

Whenever a virtual function is invoked, the resolution procedure works
as follows:

1. Retrieve the vtable pointed to by ```__vptr```.
2. Retrieve the function pointer for the association function.
3. Invoke the function via its pointer.

In our Shape example, the vtable configuration would look like the following.

![The Shape hierarchy vtable layout.](/images/posts/vtables.png)

The C implementation I provided does not do any of this automatic table
wiring for us. We must do this at compile time. This can be seen as a benefit
since we can rely on the compiler to catch function table wiring problems
for us. However, it is quite cumbersome to define new "subclasses." You have
to make the decision about whether it's important yourself.

# references

[1] [https://en.wikipedia.org/wiki/Liskov_substitution_principle](https://en.wikipedia.org/wiki/Liskov_substitution_principle)
[2] Gamma, Erich. Design patterns: elements of reusable object-oriented software. Pearson Education India, 1995.
[3] [http://www.cs.sjsu.edu/~pearce/cs251b/principles/crp.htm](http://www.cs.sjsu.edu/~pearce/cs251b/principles/crp.htm)
[4] [http://www.learncpp.com/cpp-tutorial/125-the-virtual-table/](http://www.learncpp.com/cpp-tutorial/125-the-virtual-table/)