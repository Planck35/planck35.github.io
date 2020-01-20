---
layout: post
title:  "CPP Primer Reading Log: Copy Control"
date:   2020-01-13
excerpt: "Personal reading log."
tag: [reading log, cpp]
---

- The copy and move constructors define what happens when an object is initialized from another object of the same type.
- The copy- and move-assignment operators define what happens when we assign an object of a class type to another object of that same class type.
- The destructor defines what happens when an object of the type ceases to exist.

If a class does not define all of the copy-control members, the compiler automatically defines the missing operations. As a result, many classes can ignore copy control.

A constructor is the copy constructor if its first parameter is a reference to the class type and any additional parameters have default values:
{% highlight cpp %}
class Foo {
public:
    Foo(); // default constructor
    Foo(const Foo&); // copy constructor
    // ...
};
{% endhighlight %}
In default synthesize copy constructor, the synthesized copy constructor memberwise copies the members of its argument into the object being created. The compiler copies each non`static` member in turn from the given object into the one being created.

The type of each member determines how that member is copied:
- Members of class type are copied by the copy constructor for that class.
- Members of built-in type are copied directly.
- Members of array type by copying each element.

> Copy initialization

Copy initialization happens not only when we define variables using an `=`, but also when we
*   Pass an object as an argument to a parameter of nonreference type
*   Return an object from a function that has a nonreference return type
*   Brace initialize the elements in an array or the members of an aggregate class
Some class types also use copy initialization for the objects they allocate. For example, the library containers copy initialize their elements when we initialize the container, or when we call an `insert` or `push` member. sBy contrast, elements created by an `emplace` member are direct initialized.

The `explicit` constructors can not be used in copy initialization period.

> The compiler can bypass the copy constructor

During copy initialization, the compiler is permitted (but not obligated) to skip the copy/move constructor and create the object directly. However, even if the compiler omits the call to the copy/move constructor, the copy/move constructor must exist and must be accessible at that point in the program.

What happens when a member is destroyed depends on the type of the member. Members of class type are destroyed by running the member’s own destructor. The built-in types do not have destructors, so nothing is done to destroy members of built-in type.

The implicit destruction of a member of built-in pointer type does not `delete` the object to which that pointer points.

Members that are smart pointers are automatically destroyed during the destruction phase.

> The Synthesized Destructor

The synthesized destructor has an empty function body.

The destructor body does not directly destroy the members themselves. Members are destroyed as part of the implicit destruction phase that follows the destructor body.

> Class that need destructors need copy and assignment

If the class needs a destructor, it almost surely needs a copy constructor and copy-assignment operator as well.
{% highlight cpp %}
class HasPtr {
public:
    HasPtr(const std::string &s = std::string()):
    ps(new std::string(s)), i(0) { }
    ~HasPtr() { delete ps; }
    // WRONG: HasPtr needs a copy constructor and copy-assignment operator
    // other members as before
};
{% endhighlight %}
This version of the class uses the synthesized versions of copy and assignment. Those functions copy the pointer member, meaning that multiple HasPtr objects may be pointing to the same memory.

If the class needs an assignment operator, it almost surely needs a copy constructor as well, vice versa.

> Defining a function as deleted

We can prevent copies by defining the copy constructor and copy-assignment operator as deleted functions. 
A deleted function is one that is declared but may not be used in any other way. 
We indicate that we want to define a function as deleted by following its parameter list with = delete.

{% highlight cpp %}
struct NoCopy {
    NoCopy() = default; // use the synthesized default constructor
    NoCopy(const NoCopy&) = delete; // no copy
    NoCopy &operator=(const NoCopy&) = delete; // no assignment
    ~NoCopy() = default; // use the synthesized destructor
    // other members
};
{% endhighlight %}

If the destructor is deleted, then there is no way to destroy objects of that type. 
The compiler will not let us define variables or create temporaries of a type that has a deleted destructor.

Although we cannot define variables or members of such types, we can dynamically
allocate objects with a deleted destructor. However, we cannot free them:
{% highlight cpp %}
struct NoDtor {
    NoDtor() = default; // use the synthesized default constructor
    ~NoDtor() = delete; // we can't destroy objects of type NoDtor
};
NoDtor nd; // error: NoDtor destructor is deleted
NoDtor *p = new NoDtor(); // ok: but we can't delete p
delete p; // error: NoDtor destructor is deleted
{% endhighlight %}
It is not possible to define an object or delete a pointer to a dynamically
allocated object of a type with a deleted destructor.
{: .notice}

In essence, the copy-control members are synthesized as deleted when it is impossible to copy, assign, or destroy a member of the class.
{: .notice}

Classes that want to prevent copying should define their copy constructor and copy-assignment operators using = delete rather than making those members private.
{: .notice}

The classes that manage resources that do not reside in the class must define the copy-control members. Such classes will need destructors to free the resources allocated by the object. Once a class needs a destructor, it almost surely needs a copy constructor and copy-assignment operator.

> Lvalues and Rvalues

In C++, an lvalue expression yields an object or a function.
However, some lvalues, such as `const` objects, may not be the left-hand operand of an assignment. 
Moreover, some expressions yield objects but return them as rvalues, not lvalues.
Roughlt speaking, when we use an object as an rvalue, we use the object's value(its contents).
When we use an object as an lvalue, we use the object's identity(its location in memory).
The important point is that we can use an lvalue when an rvalue is required, but we cannot use a rvalue when an lvalue is required.

> Lvalues persist & Rvalue are ephemral

Lvalues have persistent state, whereas rvalues are either literals or temporary objects created in the course of evaluating expressions.

Because rvalue references can only be bound to temporaries, we know that:
- The referred-to object is about to be destroyed.
- There can be no other users of that object.
Code that uses an rvalue reference is free to take over resources from the object to which the reference refers.