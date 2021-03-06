Administrative Scripting with Julia
===================================

Introduction
------------
If you know anything about Julia_, you probably know it's an interpreted
technical/scientific computing language that competes with the likes of
R, Matlab, NumPy and others. You've probably also heard that it compiles
to LLVM bytecode at runtime and can often be optimized to within a
factor of two of C or Fortran.

Given these promises, it's not surprising that it's attracted some very
high-profile users_.

I don't do any of that kind of programming. I like Julia for other
reasons altogether. It just feels good to write. The language features
scratch all my itches and the standard library has all the right
abstractions.

Semantically and syntactically, it feels similar to Python and Ruby
though it promotes functional design patterns and doesn't support
classic OO patterns in the same way. Instead, it relies on structs,
abstract types, and multiple dispatch as tools for working with types.
Julia favors immutability, but it's not strict. Julia's metaprogramming
story is simple yet deep. It allows operator overloading and other kinds
of magic methods. If that isn't enough, it has Lips-like AST macros.

Finally, reading the standard library (which is implemented mostly in
very readable Julia), you see just how pragmatic it is. It is happy to
call into libc for the things libc is good at. It's equally happy to
shell out if that's the most practical thing. Check out the code for
the download_ function for an instructive example. Julia is very happy
to rely on PCRE_ for regular expressions. On the other hand, Julia is
fast enough that many of the bundled data structures and primitives
are implemented directly in Julia.

While keeping the syntax fairly clean and straightforward, the Julia
ethos is ultimately about getting things done and empowering the
programmer. If that means performance, you can do you can optimizing
to your heart's content. If it means downloading files with ``curl``,
it will do that, too!

This ethos fits very well with system automation. The classic
languages in this domain are Perl and Bash. Perl has the reputation of
being "write only," and Bash is much worse than that! [#]_ However,
both are languages that emphasize pragmatism over purity, and that
seems to be a win for short scripts. Julia is more readable than
either of these, but it is not less pragmatic. [#]_

This tutorial follows roughly the approach of my `Python tutorial`_ on
administrative scripting and will refer to it at various points.

.. _Julia: https://julialang.org/
.. _users: https://juliacomputing.com/case-studies/
.. _download:
  https://github.com/JuliaLang/julia/blob/e7d15d4a013a43442b75ba4e477382804fa4ac49/base/download.jl
.. _PCRE: https://pcre.org/
.. _Python tutorial:
  https://github.com/ninjaaron/replacing-bash-scripting-with-python

.. [#] Anyone who fully undersands the semantics of the Bash they write
       even at the time they write it is something of a domain expert.
       Simple, straightforward Bash is vulnerable to injection more
       often than not. A completely correct Bash script is either
       delegating all control flow to other processes, or it is using
       non-obvious, ugly language features to ensure safety.

.. [#] This is not to fault the creators of Perl or Bourne Shell. They
       are much older langauges, and all interpreted languages,
       including Julia, have followed in their footsteps. Later
       languages learned from their problems, but they also learned from
       what they did right, which was a lot!

.. contents:: 

Why You Shouldn't Use Julia for Administrative Scripts
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
It's just a bad idea!

- Julia is not the most portable. It's a relatively new language and has
  only had it's 1.0 release this year (2018). Many Linux distros don't
  have a package available. Ubuntu 18.04 (the latest one as I write
  this) doesn't have a package in the repository, though it is available
  as a snap. Even if you can compile from source, it has that LLVM
  dependency.
- Julia has a fat runtime. It's more than 100KB and it has a
  human-perceptible load time on a slower system. For the kinds of
  intensive problems it's targeted at, this is nothing. On a
  constrained server or an embedded system, it's bad.
- Julia's highly optimizing JIT compiler also takes a little time to
  warm up. There are ways to recompile some things, but who wants to
  bother for little scripts? The speed of the compiler is impressive for
  how good it actually is, but it's not instant.

The above are reasonable arguments against using Julia on a certain
class of servers. However, none of this stuff really matters on a PC
running an OS with current packages. If your system can run a modern web
browser, Julia's ~140K runtime is a pittance.

If you already want to learn Julia, which there are many good reasons to
do, writing small automation scripts is a gentle way to become
acquainted with the basic features of the language.

The other reason you might want to try administrative scripting in Julia
is because the abstractions it provides are surprisingly well suited to
the task. Translating a Bash script to Julia is very easy but will
automatically make your script safer and easier to debug.

One final reason to use Julia for administrative is that it means you're
not using Bash! I've made a `case against Bash`_ for anything but
running and connecting other processes in Bash in my Python tutorial. In
short, Bash is great for interactive use, but it's difficult to do
things in a safe and correct way in scripts, and dealing with data is an
exercise in suffering. Handle data and complexity in programs in other
languages.

.. _case against bash:
  https://github.com/ninjaaron/replacing-bash-scripting-with-python#if-the-shell-is-so-great-what-s-the-problem


Learning Julia
~~~~~~~~~~~~~~
This tutorial isn't going to show you how to do control flow in Julia
itself, and it certainly isn't going to cover all the ways of dealing
with the rich data structures that Julia provides. To be honest, I'm
still in the process of learning Julia myself, and I'm relying heavily
on the `official docs`_ for that, especially the "Manual" section. As an
experienced Python programmer, the interfaces provided by Julia feel
very familiar, and I suspect the feeling will be even stronger for Ruby
programmers. For programmers with this background, becoming productive
in Julia should only take a few hours, though there are rather major
differences as one progresses in the language.

For a quick introduction to the language, the `learing`_ page has some
good links. The `Intro to Julia`_ with Jane Herriman goes over
everything you'll need to know to understand this tutorial. If you
choose to follow this tutorial, you will be guided to log into
juliabox.com, but you don't need to unless you want to. You can
download and run the `Jupyter Notebooks`_ locally if you wish, and you
can also simply follow along in the Julia REPL in a terminal.

The `Fast Track to Julia`_ is a handy cheatsheet if you're learning
the language

.. _official docs: https://docs.julialang.org
.. _learning: https://julialang.org/learning/
.. _Intro to Julia: https://www.youtube.com/watch?v=8h8rQyEpiZA&t=
.. _Jupyter Notebooks: https://github.com/JuliaComputing/JuliaBoxTutorials
.. _Fast Track to Julia: https://juliadocs.github.io/Julia-Cheat-Sheet/

Reading and Writing Files
-------------------------

