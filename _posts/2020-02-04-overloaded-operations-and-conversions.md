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