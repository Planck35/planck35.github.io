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