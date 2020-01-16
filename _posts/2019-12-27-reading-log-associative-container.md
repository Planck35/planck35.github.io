---
layout: post
title:  "CPP Primer Reading Log: Associative Container"
date:   2019-12-27
excerpt: "Personal reading log."
tag: [reading log, cpp]
---

> Requirement on Key Type for ordered container

For the ordered containers - `map`, `multimap`, `set` and `multiset` - the key type must define a way to compare the elements. By default, the library uses the `<` operator for the key type to compare the keys.

Each type inside the angle brackets is just that, a type. We supply a particular comparison operation as a constructor argument when we create a container.

For example, we can’t directly define a `multiset` of `Sales_data` because `Sales_data` doesn’t have a `<` operator. That function defines a strict weak ordering based on their ISBNs of two given `Sales_data` objects. The `compareIsbn` function should look something like
{% highlight cpp %}
bool compareIsbn(const Sales_data &lhs, const Sales_data &rhs)
{
    return lhs.isbn() < rhs.isbn();
}
{% endhighlight %}
To use our own operation, we must define the `multiset` with two types: the key type, `Sales_data`, and the comparison type, which is a function pointer type that can point to `compareIsbn`. When we define objects of this type, we supply a pointer to `compareIsbn`:
{% highlight cpp %}
// bookstore can have several transactions with the same ISBN
// elements in bookstore will be in ISBN order
multiset<Sales_data, decltype(compareIsbn)*> bookstore(compareIsbn);
{% endhighlight %}
Here, we use `decltype` to specify the type of our operation, remembering that when we use `decltype` to form a function pointer, we must add a `*` to indicate that we’re using a pointer to the given function type. We initialize `bookstore` from `compareIsbn`, which means that when we add elements to `bookstore`, those elements will be ordered by calling `compareIsbn`. We can write `compareIsbn` instead of `&compareIsbn` as the constructor argument because when we use the name of a function, it is automatically converted into a pointer if needed.

> A Function to Create `pair` Objects
Imagine we have a function that needs to return a pair. Under the new standard we can list initialize the return value
{% highlight cpp %}
pair<string, int> process(vector<string> &v) {
    // process v
    if (!v.empty())
        return {v.back(), v.back().size()}; // list initialize
    else
        return pair<string, int>(); // explicitly constructed return value
}
{% endhighlight %}

Each element in `map` is a `pair` object containing a key and a associated value. Because we cannot change an element’s key, the key part of these `pair`s is `const`.

> Associative Container Additional Type Aliases

| key_type | Type of the key for this container type |
| mapped_type | Type associated with each key; `map types only` |
| value_type | For `sets`, same as the key_type, For `maps`, `pair<const key_type, mapped_type>` | 
{: rules="groups"}

Iterators for `set`s are `const`, the keys in a `set` are also `const`. We can use a `set` iterator to read, but not write.
{% highlight cpp %}
set<int>::iterator set_it = iset.begin();
*set_it = 42; // error: keys in a set are read-only
{% endhighlight %}

Because the subscript operator might insert an element, we may use subscript only on a `map` that is not `const`.

When we subscript a `map`, we get a `mapped_type` object; when we dereference a `map` iterator, we get a `value_type` object. In common with other subscripts, the `map` subscript operator returns an lvalue. Because the return is an lvalue, we can read or write the element.

Elements with the same key are stored adjacent to one another in both the ordered and unordered containers.
