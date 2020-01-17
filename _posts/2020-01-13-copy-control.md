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