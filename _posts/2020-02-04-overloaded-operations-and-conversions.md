---
layout: post
title:  "CPP Primer Reading Log: Overloaded operations and conversions"
date:   2020-01-13
excerpt: "Personal reading log."
tag: [reading log, cpp]
---

> Basic Concepts

Overloaded operators are functions with special names: the keyword operator
followed by the symbol for the operator being defined.
It has a return type, a parameter list, and a body.

An overloaded operator function has the same number of parameters as the operator has operands. In a binary operator, the left-hand operand is passed to the first parameter and the right-hand operand to the second.
Except for the overloaded function-call operator, `operator()`, an overloaded operator may not have default arguments.

If an operator function is a member function, the first(left-hand) operand is bound to the implicit `this` pointer. Because the first operand is implicity bound to `this`, a member operator function has one less(explicit) parameter than the operator has operands.

An operator function must either be a member of a class or have at least one parameter of class type.
{% highlight cpp %}
// error: cannot redefine the built-in operator for ints
int operator+(int, int);
{% endhighlight %}

Some operators shouldn't be overloaded

The overloaded versions of `&&` or `||` operators do not preserve short-circuit evaluation properties of the built-in operators. Both oeprands are always evaluated.

We should not overload the comma and address-of operator(`&`). The language defines what the comma and adderss-of operators mean when applied to objhects of class type. Because these operators have built-in meaning, they ordinarily should not be overloaded.

Ordinarily, the comma, address-of, logical `AND`, and logical `OR` operators should `not` be overloaded.
{: .notice}

To be consistent with other output operators, `operator<<` normally returns its `ostream` parameter.
{% highlight cpp %}
ostream &operator<<(ostream &os, const Sales_data &item)
{
    os << item.isbn() << " " << item.units_sold << " " << item.revenue << " " << item.avg_price();
    return os;
}
{% endhighlight %}

## IO Operators Must Be Nonmember Functions

These operators cannot be members of your own class. If they were, then the left-hand operand would have to be an object of our class type.

IO operators usually need to read or write the non`public` data members. As a consequence, IO operators usually must be declared as friends.

We define the arithmetic and relational operators as nonmember functions in order to allow conversions for either the left- or right-hand operand. These operators shouldn't need to change the state of either operand, so the parameters are ordinarily references to `const`.

## Differentiating Prefix and Postfix Operators

Normal overloading cannot distinguish between these operators. The prefix and postfix versions use the same symbol. They also have the same number and type of operands.

To solve this problem, the postfix versions take an extra parameter of type `int`. When we use a postfix operator, the compiler supplies 0 as the argument for this parameter.

## Calling the Postfix Operators Explicitly

If we want to call the postfix version using a function call, then we must pas a value for the integer argument.
{% highlight cpp %}
StrBlobPtr p(al); // p points to the vector inside al
p.operator++(0); // call postfix operator++
p.operator++();  // call prefix operator++
{% endhighlight %}