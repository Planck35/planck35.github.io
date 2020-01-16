---
layout: post
title:  "CPP Primer Reading Log: Class"
date:   2019-12-20
excerpt: "Personal reading log."
tag: [reading log, cpp]
---

>2019/12/20 Class, cpp primer: 332 - 394

The only difference between `struct` and `class` is the default access level.
A class may define members before the first access specifier. Access to such
members depends on how the class is defined. For `struct` keyword, the
members are `public`; For `class`, the members are `private`.

Following type alias is equivalent:
{% highlight cpp %}
typedef std::string::size_type pos;
using pos = std::string::size_type;
{% endhighlight %}

## Returning *this a const member function

A `const` member function that returns `*this` as a reference should have a return type that is a reference to `const`.

We can refer to a class type directly, by using the class name as a type name.
Alternatively, we can use the class name following the keyword class or struct:

{% highlight cpp %}
Sales_data item1; // default-initialized object of type Sales_data
class Sales_data item1; // equivalent declaration
{% endhighlight %}

After a declaration and before a definition of a `class` is seen, this `class` is an **incomplete type**, A `class` must be defined-not just declared-before we can create objects of that type and a reference or pointer is used to access a member of the type.

## Friendship between Classes

{% highlight cpp %}
class Screen {
    // Window_mgr members can access the private parts of class Screen
    friend class Window_mgr;
    // ... rest of the Screen class
};
{% endhighlight %}
The member functions of a friend class can access all the members, including the `nonpublic` members of the class granting friendship.

It is important to understand that friendship is not transitive. That is, if class `Window_mgr` has its own friends, those friends have no special access to `Screen`.

Each class controls which classes or functions are its friends.
{: .notice}

Rather than making the entire `Window_mgr` class a friend, `Screen` can instead
specify that only the `clear` member is allowed access.

{% highlight cpp %}
class Screen {
    // Window_mgr::clear must have been declared before class Screen
    friend void Window_mgr::clear(ScreenIndex);
    // ... rest of the Screen class
};
{% endhighlight %}

Making a member function a friend requires careful structuring of the programs:
* First, define the `Window_mgr` class, which declares, but **cannot define** `clear()`.
* Define class `Screen`, including a friend declaration for `clear()`.
* Define `clear()`, which can now refer to the members in `Screen`.
  
I think why not define `clear()` first, is that if so, the clear will not have any idea about the member of `Screen` class, because it is not a friend function yet.
{: .notice}

Although overloaded functions share a common name, they are still different
functions. Therefore, a class must declare as a friend each function in a set of
overloaded functions that it wishes to make a friend.

## Friend Declarations and Scope

Classes and nonmember functions **need not have been declared** before they are used
in a friend declaration. When a name first appears in a friend declaration, that name
is implicitly assumed to be part of the surrounding scope. **However, the friend itself is not actually declared in that scope**.

**Even if we define the function inside the class, we must still provide a declaration outside of the class itself to make that function visible.** A declaration must exist even if
we only call the friend from members of the friendship granting class.

{% highlight cpp %}
struct X {
    friend void f() { /* friend function can be defined in the class body */ }
    X() { f(); } // error: no declaration for f
    void g();
    void h();
};
void X::g() { return f(); } // error: f hasn't been declared
void f(); // declares the function defined inside X
void X::h() { return f(); } // ok: declaration for f is now in scope
{% endhighlight %}

When a member function is defined outside the class body, any name used in the return type is outside the class scope. As a result, the return type must specify the class of which it is a member.

{% highlight cpp %}
Window_mgr::ScreenIndex Window_mgr::addScreen(const Screen &s);
{% endhighlight %}

The class member function bodies are not processed until the entire class is seen.

Names used in member function declarations, including names used for the return type and types in the parameter list, must be seen before they are used.

{% highlight cpp %}
typedef double Money;
string bal;
class Account {
public:
    Money balance() { return bal; }
private:
    Money bal;
    // ...
};
{% endhighlight %}
When the compiler sees the declaration of the balance function, it will look for a declaration of Money in the Account class. On the other hand, the function body of balance is processed only after the entire class is seen. Thus, the return inside that function returns the member named bal, not the string from the outer scope.

{% highlight cpp %}
class ConstRef {
public:
    ConstRef(int ii);
private:
    int i;
    const int ci;
    int &ri;
};
ConstRef::ConstRef(int ii): i(ii), ci(ii), ri(i) { }
{% endhighlight %}
We must use the constructor initializer list to provide values for members that are `const`, reference, or of a class type that does not have a default constructor.

## Order of Member Initialization
The order in constructor initializer list specifies only the values used to initialize the members, not the order in which those initializations are performed.

Members are initialized in the order in which they appear in the class definition.
{% highlight cpp %}
class X {
    int i;
    int j;
public:
// undefined: i is initialized before j
    X(int val): j(val), i(j) { }
};
{% endhighlight %}
A constructor that supplies default arguments for all its parameters also defines the default constructor.
{: .notice}

## Suppressing Implicit Conversions Defined by Constructors

We can prevent the use of a constructor in a context that requires an implicit conversion by declaring the constructor as `explicit`:

The explicit keyword is meaningful only on constructors that can be called with
a single argument. Constructors that require more arguments are not used to perform
an implicit conversion, so there is no need to designate such constructors as
explicit.

{% highlight cpp %}
class Sales_data {
public:
    Sales_data() = default;
    Sales_data(const std::string &s, unsigned n, double p):
        bookNo(s), units_sold(n), revenue(p*n) { }
    explicit Sales_data(const std::string &s): bookNo(s) { }
    explicit Sales_data(std::istream&);
    // remaining members as before
};
{% endhighlight %}

Now, neither constructor can be used to implicitly create a Sales_data object.
{% highlight cpp %}
item.combine(null_book); // error: string constructor is explicit
item.combine(cin); // error: istream constructor is explicit
{% endhighlight %}

One context in which implicit conversions happen is when we use the copy form of initialization (with an `=`). We cannot use an `explicit` constructor with this form of initialization; we must use direct initialization:
{% highlight cpp %}
Sales_data item1 (null_book); // ok: direct initialization
// error: cannot use the copy form of initialization with an explicit constructor
Sales_data item2 = null_book;
{% endhighlight %}

When defining the `static` member functions, the `static` keyword is used only on the declaration inside the class body.

{% highlight cpp %}
// define and initialize a static class member
double Account::interestRate = initRate();
{% endhighlight %}
Note also that even though initRate is private, we can use this function to initialize interestRate. The definition of interestRate, like any other member definition, has access to the private members of the class.

## Static member special use

A `static` data member can have incomplete type. In particular, a `static` data member can have the same type as the class type of which it is a member. Another difference between static and ordinary members is that we can use a static member as a default argument.
{% highlight cpp %}
class Bar {
public:
    // ...
private:
    static Bar mem1; // ok: static member can have incomplete type
    Bar *mem2; // ok: pointer member can have incomplete type
    Bar mem3; // error: data members must have complete type
};
{% endhighlight %}