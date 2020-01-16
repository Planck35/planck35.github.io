---
layout: post
title:  "CPP Primer Reading Log: Function"
date:   2019-12-18
excerpt: "Personal reading log."
tag: [reading log, cpp]
---

`initializer_list` can be used to pass varying number of arguments of the **same** type

The use of `initialzer_list` is similar to `vector`, but its element value is constant.

When we pass a sequence of values to an `initializer_list` parameter, we
must enclose the sequence in curly braces:

{% highlight cpp %}
void error_msg(ErrCode e, initializer_list<string> il)
{
    cout << e.msg() << ": ";
    for (const auto &elem : il)
        cout << elem << " " ;
    cout << endl;
}
{% endhighlight %}
{% highlight cpp %}
error_msg(ErrCode(0), {"functionX", "okay"});
{% endhighlight %}

**Ellipsis** can also be used to pass a varying number of arguments. But it should be used only for types that are common to both C and C++. Objects of most cpp classes are not copied properly when using ellipsis.

***Never Return a Reference or Pointer to a Local Object***

***Reference Returns Are LValues***

Calls to functions that return references are lvalues; other return types
yield rvalues. Lvalues can be used on the left-hand side of an assignment.

* `int (*func(int i))[10]` is a function that returns a pointer to array of 10 elements.
* `func(int)` says that we can call `func` with an `int` argument.
* `(*func(int))` says we can dereference the result of that call.
* `(*func(int))[10]` says that dereferencing the result of a call to `func` yields
an array of size ten.
* `int (*func(int))[10]` says the element type in that array is `int`.

***Trailing Return Type in new standard***

{% highlight cpp %}
// fcn takes an int argument and returns a pointer to an array of ten ints
auto func(int i) -> int(*)[10];
{% endhighlight %}

THe return type follows the parameter list and is preceded by ->. We use `auto` where the return type ordinarily appears.

{% highlight cpp %}
int odd[] = {1,3,5,7,9};
int even[] = {0,2,4,6,8};
// returns a pointer to an array of five int elements
decltype(odd) *arrPtr(int i)
{
    return (i % 2) ? &odd : &even; // returns a pointer to the array
}
{% endhighlight %}

It is an error for two functions to differ only in terms of their return types. If the
parameter lists of two functions match but the return types differ, then the second
declaration is an error.
{: .notice}

If we declare a name in an inner scope, ***that name hides uses of that name declared in an outer scope***.

{% highlight cpp %}
string read();
void print(const string &);
void print(double); // overloads the print function
void fooBar(int ival)
{
    bool read = false; // new scope: hides the outer declaration of read
    string s = read(); // error: read is a bool variable, not a function
    // bad practice: usually it's a bad idea to declare functions at local scope
    void print(int); // new scope: hides previous instances of print
    print("Value: "); // error: print(const string &) is hidden
    print(ival); // ok: print(int) is visible
    print(3.14); // ok: calls print(int); print(double) is hidden
}
{% endhighlight %}

In C++, name lookup happens before type checking.
{: .notice}

***`inline mechanism`*** is meant to optimize small, straight-line functions. The inline specification is only a request to the compiler. The compiler may choose to ignore this request.

## Pointers to Functions

A function pointer is just that - a pointer that denotes a function rather than an object.

{% highlight cpp %}
// pf points to a function returning bool that takes two const string references
bool (*pf)(const string &, const string &); // uninitialized
// declares a function named pf that returns a bool*
bool *pf(const string &, const string &);
{% endhighlight %}

## Using Function Pointers

When we use the name of a function as a value, the function is automatically
converted to a pointer. For example, we can assign the address of `lengthCompare`
to `pf` as follows:

{% highlight cpp %}
pf = lengthCompare; // pf now points to the function named lengthCompare
pf = &lengthCompare; // equivalent assignment: address-of operator is optional
{% endhighlight %}

Moreover, we can use a pointer to a function to call the function to which the
pointer points. We can do so directly—there is no need to dereference the pointer:

{% highlight cpp %}
bool b1 = pf("hello", "goodbye"); // calls lengthCompare
bool b2 = (*pf)("hello", "goodbye"); // equivalent call
bool b3 = lengthCompare("hello", "goodbye"); // equivalent call
{% endhighlight %}

{% highlight cpp %}
int (*f1(int))(int*, int);
{% endhighlight %}

Reading this declaration from the inside out, we see that `f1` has a parameter list, so
`f1` is a function. `f1` is preceded by a `*` so `f1` returns a pointer. The type of that
pointer itself has a parameter list, so the pointer points to a function. That function
returns an `int`.