---
layout: post
title:  "CPP Primer Reading Log: Dynamic Memory"
date:   2020-01-06
excerpt: "Personal reading log."
tag: [reading log, cpp]
---

- The `shared_ptr`, which allows multiple pointers to refer to the same object.
- The `unique_ptr`, which "owns" the object to which it points.
- The `weak_ptr` that is a weak reference to an object managed by a `shared_ptr`.

A default initialized smart pointer holds a nulll pointer.

> Copying and Assigning `shared_ptr`s

We think of a `shared_ptr` as if it has an associated counter, usually referred to as a **reference count**. Whenever we copy a `shared_ptr`, the count is incremented. For example, the counter associated with a `shared_ptr` is incremented:

- When we use it to initialize another `shared_ptr`.
- When we use it as the right-hand operand of an assignment.
- When we pass it to or return it from a function by value.

Once a `shared_ptr`'s counter goes to zero, the `shared_ptr` automatically frees the object that it manages.

> `shared_ptr`s automatically destroy their objects and automatically free the associated memory

When the last `shared_ptr` pointing to an object is destroyed, the `shared_ptr` class automatically destroy the object to which that `shared_ptr` points. It does so through another special member function known as a **destructor**.

By default, dynamically allocated objects are default initialized, which means that objects of built-in or compound type have undefined value; objects of class type are initializedd by their default constructor.

{% highlight cpp %}
auto p1 = new auto(obj); // p points to an object of the type of obj that object is initialized from obj
{% endhighlight %}
The type of `p1` is a pointer to the `auto`-deduced type of `obj`. If `obj` is an `int`, then `p1` is `int*`; if `obj` is a `string`, then `p1` is a `string*`;

It is legal to use `new` to allocate `const` objects.
{% highlight cpp %}
const int *pci = new const int(1024)
{% endhighlight %}

By default, if `new` is unable to allocate the requested storage, it throws an exception of type `bad_alloc`. We can use different form of new to prevent
{% highlight cpp %}
int *p2 = new (nothrow) int;
{% endhighlight %}
If this form of `new` is unable to allocate the requested storage, it will return a null pointer.

> Pointer values and delete
The pointer we pass to `delete` must either point to dynamically allocated memory or be a null pointer. Deleting a pointer to memory that was not allocated by `new`, or deleting the same pointer value more than once, is undefined. In general, compilers cannot tell whether a pointer points to statically or dynamically allocated object.

A dynamic object managed through a built-in pointer exists until it is explicitly deleted.

We cannot implicitly convert a built-in pointer to a smart pointer; we must use the direct form of initialization to initialize a smart pointer. A function that returns a `shared_ptr` cannot implicitly convert a plain pointer in its return statement:
{% highlight cpp %}
shared_ptr<int> p1 = new int(1024); // error: must use direct initialization
shared_ptr<int> p2(new int(1024)); // ok: uses direct initialization
{% endhighlight %}

> Don't use `get` to initialize or assign another smart pointer

The smart pointer types define a function named `get` that returns a built-in pointer to the object that the smart pointer is managing. This function is intended for cases when we need to pass a built-in pointer to code that can’t use a smart pointer. The code that uses the return from `get` must not `delete` that pointer.

> Smart pointers with Exceptions

When we use a smart pointer, the smart pointer class ensures that memory is freed when it is no longer needed even if the block is exited prematurely.

{% highlight cpp %}
template <class U, class D> shared_ptr (U* p, D del);
template <class D> shared_ptr (nullptr_t p, D del);

void f(destination &d /* other parameters */)
{
    connection c = connect(&d);
    shared_ptr<connection> p(&c, end_connection);
    // use the connection
    // when f exits, even if by an exception, the connection will be properly closed
}
{% endhighlight %}

When `p` is destroyed, it won’t execute `delete` on its stored pointer. Instead, `p` will
call `end_connection` on that pointer. In turn, `end_connection` will call
`disconnect`, thus ensuring that the connection is closed. If `f` exits normally, then `p`
will be destroyed as part of the return.

If you use a smart pointer to manage a resource other than memory allocated by `new`, remember to pass a deleter.

> `weak_ptr`

Once the last `shared_ptr` pointing to the object goes away, the object itself will be deleted. That object will be deleted even if there are `weak_ptr`s pointing to it.

> Freeing Dynamic Arrays

To free a dynamic array, we use a special form of delete that includes an empty pair of square brackets:
{% highlight cpp %}
delete p; // p must point to a dynamically allocated object or be null
delete [] pa; // pa must point to a dynamically allocated array or be null
{% endhighlight %}
The second statement destroys the elements in the array to which `pa` points and frees the corresponding memory. Elements in an array are destroyed in reverse order. That is, the last element is destroyed first, then the second to last, and so on.

> Smart pointers and dynamic arrays

The library provides a version of `unique_ptr` that can manage arrays allocated by `new`. To use a `unique_ptr` to manage a dynamic array, we must include a pair of empty brackets after the object type.
{% highlight cpp %}
// up points to an array of ten uninitialized ints
unique_ptr<int[]> up(new int[10]);
up.release(); // automatically uses delete[] to destroy its pointer
{% endhighlight %}
The brackets in the type specifier (`<int[]>`) say that `up` points not to an `int` but to an array of `int`s.

Unlike `unique_ptr`, `shared_ptr`s provide no direct support for managing a dynamic array. If we want to use a `shared_ptr` to manage a dynamic array, we must provide our own deleter.