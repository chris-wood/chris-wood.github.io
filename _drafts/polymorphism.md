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

```
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
    identify() string
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
needed. This is the basic idea behind the Liskov Substitution Principle [x]
and is the L in the famed SOLID principles for OO programming.

Most pragmatic programmers recognize that code re-use is more easily
achieved through composition than inheritance since polymorphic behavior
is not bound to compile-time types but rather run-time object instances.
Many design patterns such as the Strategy pattern [y] use composition
(object delegation) for polymorphic behavior. This is regarded 
as the Composite Reuse Principle in the literature [z].


[x] LIskov
[y] gang of four
[z] http://www.cs.sjsu.edu/~pearce/cs251b/principles/crp.htm

Even still, inheritance has its place for a reason. When an object
truly does exemplify the *is-a* relationship instead of a *has-a*
relationship, inheritance is perfectly plausible. For example, an
array list is most certainly a specific type of list, and so making
an array list implementation inherit from a generic list implementation
(or implement a list interface) is quite acceptable. 

This leads me to the main topic of this post. Since I spend the majority
of my time programming in C, which is not an object-oriented language, 
I find myself yearning the great benefits of inheritance. To that end,
I will give an overview of how to implement C++-like inheritance in C. 

# Polymorphic Shapes

TODO: discuss how one might do it, and then give the code example in three parts (interface and concrete class, instances, instantiating)

how would we go about building this type of polymorphism in C, a statically 
typed and compiled language without support for objects? have a thought experiment about how 
this might work. function pointers allow us to pass around functions to invoke freely. 
We could use function pointers and compiled structs to determine which function
pointers to invoke for a given function. XXX

# Polymorphic Shapes

```
typedef struct shape_interface {
    double (*Area)(void *instance);
} ShapeInterface;

Shape *
shape_Create(void *instance, ShapeInterface *interface);

Shape *
shape_Create(void *instance, ShapeInterface *interface)
{
    Shape *shape = (Shape *) malloc(sizeof(Shape));
    shape->instance = instance;
    shape->interface = interface;
    return shape;
}

struct shape {
    void *instance;
    const ShapeInterface *interface;
}

double
shape_Area(Shape *shape)
{
    return (shape->interface->Area)(shape->instance);
}


# square header

extern ShapeInterface *SquareAsShape;

struct square {
    double x;
}

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

# circle header

extern ShapeInterface *CircleAsShape;


#incluce <math.h>
struct circle {
    double radius;
}

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

# describe how it's different from C++'s dispatch table (compile time binding vs runtime binding)
# Differences from C++
In C++, the virtual table (vtable) is configured by the compiler when an object is created.
Each class has its own vtable which contains function pointers. During compilation, a delegated 
field called __vptr [XX] is silently inserted into the base type of a class hierarchy. 
When an object from this hierarchy is created, that pointer is initialized to the vtable
of the new class. Thus, whenever a virtual function is invoked, the invocation procedure is as follows:

1. Retrieve teh vtable pointed to by __vptr.
2. Retrieve the function pointer for the association function.
3. Invoke the function via its pointer.

In our Shape example, the vtable configuration would look like the following. 

TODO: draw the figure of the vtable arrangement here

The C implementation I provided does not do any of this table wiring for us. All of this wiring
is done at compile time. This process is split into two parts: (1) functions in the base class 
(i.e., the Shape Area function invokes the Area function of its child) and (2) creation of
the generic types from concrete types (i.e., creating a Shape from a Circle). 

# Assessment

What are some of the benefits of this approach?
- compile-time checks for correctness.
- can be generalized to create complex hierarchies and even a full-blown object system
- ??

Drawbacks:
- it's a lot of code to define new sub "classes"
- it can be cumbersome to create subtypes
- ??
 
