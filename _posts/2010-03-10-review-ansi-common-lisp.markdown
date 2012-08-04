---
layout: post
title: Review of "ANSI Common Lisp"
summary: |
    <p><em><strong>ANSI Common Lisp</strong><br/>
    Paul Graham<br/>
    Prentice Hall (November 12, 1995)</em></p>
    
    This is a really good book for newcomers to Common Lisp, however take note
    that this is not a book for beginning programmers. This book will not
    guide you on the details of how to get, install and launch a Common Lisp
    environment. If you don't feel able to do that on your own then this book
    might not be for you.
    
    The pace of the book is fast, and the information is presented ...
---

<p><em><strong>ANSI Common Lisp</strong><br/>
Paul Graham<br/>
Prentice Hall (November 12, 1995)</em></p>

This is a really good book for newcomers to Common Lisp, however take note
that this is not a book for beginning programmers. This book will not
guide you on the details of how to get, install and launch a Common Lisp
environment. If you don't feel able to do that on your own then this book
might not be for you.

The pace of the book is fast, and the information is presented in a very
compressed form, meaning that the author does not ramble unnecessarily on
any topic. The concepts are introduced one after another, many times
building new concepts on top of the ones that have been introduced before;
in this sense this book is meant to be read in order.

To give you an idea of the compactness of the book, the 17 chapters of the
book (not counting the appendixes) add up to only 285 pages.

There are exercises at the end of each chapter, with varying degrees of
difficulty. Personally, I would've liked it better if there were more
exercises that required the student to construct more levels of
abstraction to find a solution.

The topics of CLOS and Macros receive a fair treatment on this book, but
not an exhaustive one. For further study into these two topics I would
suggest the books *"Object-Oriented Programming in Common Lisp: A
Programmer's Guide to CLOS"* by Sonya E. Keene, and *"On LISP: Advanced
Techniques for Common LISP"* also by Paul Graham.

Although the latter book is out of print and very hard to find a copy of,
a PDF version is offered by the author at his website
[http://www.paulgraham.com/onlisp.html][onlisp]

[onlisp]: http://www.paulgraham.com/onlisp.html

One thing I like about this book is its thorough treatment of *cons*
objects and how they are used in Common Lisp, along with discussions on
destructive and non-destructive *(side-effect free)* functions, and the
issues of sharing structure.

Other highlights are, a simple implementation of a Ray-tracer; an
implementation of a custom Object-Oriented language within Common Lisp; an
implementation of a program that makes inferences from a set of rules
*(very similar to Prolog)*; and utilities to generate HTML.

Finally, it's got a reference of all the language at the end of the book,
which makes it very convenient to take the book with you, away from the
computer, and try understanding the topics and do the exercises using only
pen and paper. In fact, I recommend you do that, and use the computer only
to verify that the solutions you found are correct.

It took me 48 hours over the space of 39 days (averaging nearly 1:15 hrs a
day) to read this book and do the exercises.

Conclusion: Highly recommended as an introductory book in Common Lisp.


<!-- vim: set tw=74 sw=4 ts=4 et spell filetype=mkd: -->
