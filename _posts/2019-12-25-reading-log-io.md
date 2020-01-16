---
layout: post
title:  "CPP Primer Reading Log: IO"
date:   2019-12-25
excerpt: "Personal reading log."
tag: [reading log, cpp]
---

Merry Christmas
{: .notice}

>2019/12/25 IO, cpp primer: 398 - 

## No Copy or Assign for IO Objects

We cannot copy or assign objects of the IO types
{% highlight cpp %}
ofstream out1, out2;
out1 = out2; // error: cannot assign stream objects
ofstream print(ofstream); // error: can't initialize the ofstream parameter
out2 = print(out2); // error: cannot copy stream objects
{% endhighlight %}
Because we can’t copy the IO types, we **cannot have a parameter or return type** that is one of the stream types. Functions that do IO typically pass and return the stream through references. Reading or writing an IO object changes its state, so the reference **must not be** `const`.

The `badbit` indicates a system-level failure, such as an unrecoverable read or write error. It is usually not possible to use a stream once `badbit` has been set. 

The `failbit` is set after a recoverable error, such as reading a character when numeric data was expected. It is often possible to correct such problems and continue using the stream. 

Reaching end-of-file sets both `eofbit` and `failbit`. 

The `goodbit`, which is guaranteed to have the value 0, indicates no failures on the stream. If any of badbit, failbit, or eofbit are set, then a condition that evaluates that stream will fail.

There are several conditions that cause the standard output buffer to be flushed - that is, to be written to the **actual output device or file**:
* The program completes normally.
* The buffer can become full, in which case it will be flushed before writing the next value.
* Flush the buffer explicitly using a manipulator such as `endl`.
* Use the `unitbuf` manipulator to set the stream’s internal state to empty the buffer after each output operation. By default, `unitbuf` is set for `cerr`, so that writes to `cerr` are flushed immediately.
* An output stream might be tied to another stream. In this case, the buffer of the tied stream is flushed whenever the tied stream is read or written. By default, `cin` and `cerr` are both tied to `cout`. Hence, reading `cin` or writing to `cerr` flushes the buffer in `cout`.

Some extra manipulators add extra char at the end of the prepending strings.
{% highlight cpp %}
cout << "hi!" << endl; // writes hi and a newline, then flushes the buffer
cout << "hi!" << flush; // writes hi, then flushes the buffer; adds no data
cout << "hi!" << ends; // writes hi and a null, then flushes the buffer
{% endhighlight %}

## The unitbuf Manipulator

If we want to flush after every output, we can use the `unitbuf` manipulator. This manipulator tells the stream to do a `flush` after every subsequent write. The `nounitbuf` manipulator restores the stream to use normal, system-managed buffer flushing:
{% highlight cpp %}
cout << unitbuf; // all writes will be flushed immediately
// any output is flushed immediately, no buffering
cout << nounitbuf; // returns to normal buffering
{% endhighlight %}
Output buffers are not flushed if the program terminates abnormally.
{: .notice}

## Tying Input and Output Streams Together
When an input stream is tied to an output stream, any attempt to read the input stream will first flush the buffer associated with the output stream. The library ties `cout` to `cin`, so the statement
{% highlight cpp %}
cin >> ival;
{% endhighlight %}
causes the buffer associated with `cout` to be flushed.

Interactive systems usually should tie their input stream to their output stream. Doing so means that all output, which might include prompts to the user, will be written before attempting to read the input.
{: .notice}

There are two overloaded versions of `tie`: One version takes no argument and returns a pointer to the output stream, if any, to which this object is currently tied. The function returns the null pointer if the stream is not tied.

The second version of `tie` takes a pointer to an `ostream` and ties itself to that `ostream`. That is, `x.tie(&o)` ties the stream `x` to the output stream `o`.

We can tie either an `istream` or an `ostream` object to another `ostream`:
{% highlight cpp %}
cin.tie(&cout); // illustration only: the library ties cin and cout for us
// old_tie points to the stream (if any) currently tied to cin
ostream *old_tie = cin.tie(nullptr); // cin is no longer tied
// ties cin and cerr; not a good idea because cin should be tied to cout
cin.tie(&cerr); // reading cin flushes cerr, not cout
cin.tie(old_tie); // reestablish normal tie between cin and cout
{% endhighlight %}
To tie a given stream to a new output stream, we pass `tie` a pointer to the new stream. To untie the stream completely, we pass a null pointer. Each stream can be tied to at most one stream at a time. However, multiple streams can tie themselves to the same `ostream`.

When an `fstream` object is destroyed, `close` is called automatically.

## Opening a File in `out` Mode Discards Existing Data

By default, a file opened in `out` mode is truncated even if we do not specify `trunc`. To preserve the contents of a file opened with `out`, either we must also specify `app`, in which case we can write only at the end of the file, or we must also specify `in`, in which case the file is open for both input and output.

By default, when we open an `ofstream`, the contents of the file are discarded. The only way to prevent an `ostream` from emptying the given file is to specify `app`:
{% highlight cpp %}
// file1 is truncated in each of these cases
ofstream out("file1"); // out and trunc are implicit
ofstream out2("file1", ofstream::out); // trunc is implicit
ofstream out3("file1", ofstream::out | ofstream::trunc);
// to preserve the file's contents, we must explicitly specify app mode
ofstream app("file2", ofstream::app); // out is implicit
ofstream app2("file2", ofstream::out | ofstream::app);
{% endhighlight %}

The only way to preserve the existing data in a file opened by an `ofstream` is to specify `app` or `in` mode explicitly.
{: .notice}