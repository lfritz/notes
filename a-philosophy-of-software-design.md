# A Philosopy of Software Design

*John Ousterhout: A Philosopy of Software Design (2018)*


## Chapter 1: Introduction

Programmers aren't bound by practical limitations such as the laws of physics, so the greatest
limitation in writing software is our ability to understand the systems we are creating. As software
evolves, complexity accumulates and it becomes harder for programmers to keep all of the relevant
factors in their minds as they modify the system. This slows down development and leads to bugs,
which slow down development even more.

There are two approaches to fighting complexity. one is to eliminate complexity by making code
simpler and more obvious. The second is to encapsulate it; this is called modular design. The book
discusses both.


## Chapter 2: The Nature of Complexity

John uses a practical definition of complexity:

> Complexity is anything related to the structure of a software system that makes it hard to
> understand and modify the system.

Complexity manifests itself in three general ways:

* Change amplification: a seemingly simple change requires code modifications in many different
  places.
* Cognitive load: how much a developer needs to know in order to complete a task.
* Unknown unknowns: it's not obvious which pieces of code must be modified or what information a
  developer needs to complete a task.

Complexity is caused by two things, dependencies and obscurity.

Dependencies exist when a given piece of code can't be understood and modified in isolation.
Dependencies can't be completely eliminated; however, one of the goals of software design is to
reduce the number of dependencies and to make the dependencies that remain as simple as possible.

Obscurity occurs when important information is not obvious. A simple example is a variable name
that doesn't carry much useful inforamtion. Obscurity also comes about because of inadequate
documentation, although if a system has a clean and obvious design then it will need less
documentation.


## Chapter 3: Working Code Isn't Enough

The chapter discusses two approaches to software development: tactical and strategic programming.

With tactical programming, the main focus is to get something working, such as a new feature or a
bug fix. You don't spend time looking for the best design. This works well at the beginning, but
over time complexities accumulate and productivity declines.

With strategic programming, the primary goal is to produce a great design which also happens to
work. This requires an investment mindset. Instead of taking the fastest path to finish your current
task, you invest time to improve the design of the system. Those investments spped you pu in the
long term.

John's suggestion is to spend 10-20% of your time on improving the design. He also estimates that
the time until strategic programming pays off is in the range of 6-18 months, although he doesn't
know of any studies or hard data on this.

## Chapter 4: Modules Should Be Deep

In modular design, software is decomposed into modules that are relatively independent. We can think
of modules in two parts: an interface and an implementation. The interface consists of everything
that a developer working in a different module must know in order to use the module.

The interface to a module contains two kinds of information: formal and informal. The format parts
are specificed explicitly in the code, for example the names and types of a method's parameters in
Java. The informal elements include everything else that a programmer needs to use the module.

The best modules are those that provide powerful functionality yet have a simple interface. John
calls these deep modules. The opposite is a shallow module, which has a complex interface but not
much functionality.

Module depth is a way of thinking about cost versus benefit. The benefit provided by a module is its
functionality. The cost, in terms of system complexity, is its interface. Deep modules provide
greater benefit at lower cost.

The next few chapter discuss techniques for creating deep modules.


## Chapter 5: Information Hiding (and Leakage)

One of the most important techniques is information hiding. The basic idea is that each module
should encapsulate a few pieces of knowledge that represent design decisions. The knowledge is
embedded in the module's implementation but doesn't appear in its interface.

Inforamtion hiding reduces complexity in two ways. First, it simplifies the interface to a module.
Second, it makes it easier to evolve the system. If a module hides a piece of information, there
are fewer dependencies on that information from other modules, so a design chagne related to that
information will affect only the one module.

The opposite is information leakage, which occurs when a design decision is reflected in multiple
modules. If a piece of information is refleced in the interface for a module, then by definition it
has been leaked; thus, simpler interfaces tend to correlate with better information hiding. However,
inforamtion can be leaked even if it doesn't appear in a module's interface. Suppose two classes
both have knowledge of a particular file format (perhaps one reads and the other writes file in that
format).Even if neither class exposes that information in its interface, they both depend on the
file format: if the format chagnes, both classes will need to be modified. Back-door leakage like
this is more pernicious than leakage through an interface, because it isn't obvious.

A common cause of leakage is a design style that John calls temporal decomposition. In temporal
decomposition, the structure of a system corresponds to the order in which operations occur. This
often results in information leakage because most design decisions manifest at several different
times over the life of an application. When designing modules, focus on the knowledge that's needed
to perform each task, not the order in which tasks occur.


## Chapter 6: General-Purpose Modules are Deeper

One of the most common decisions that you will face when designing a new module is whether to
implement it in a general-purpose or a special-purpose fashion. Specializing each module for today's
needs and making it more general only once you discover additional uses that a generalized module
could serve fits with an iterative approach, which seems like an argument for starting with
specialized modules.

However, general-purpose interfaces are often simpler and deeper than specialized ones, mostly
because generality leads to better information hiding. John recommends building new modules in a
*somewhat general-purpose* fashion. The interface should be easy to use for today's needs without
being tied specifically to them. The word "somewhat" is important: don't get carried away and build
something so general-purpose the it's difficult to use for your current needs.

Here are some questions you can ask yourself to find the right balance between general-purpose and
special-purpose:

* What is the simplest interface that will cover all my current needs? If you reduce the number of
  methods in an API without reducing its overall capabilities, then you are potentially creating
  more general-purpose methods.
* In how many situations will this method be used? If a method is designed for one particular use,
  that's a red flag that it may be too special-purpose.
* Is this API easy to use for my current needs? This question can help you determine when you have
  gone too far in making an API simple and general-purpose.


## Chapter 7: Different Layer, Different Abstraction

Software systems have layers, where higher layers use the facilities provided by lower layers. If a
layer implements a similar abstraction to the layer below, there's a good chance that it doesn't
provide enough benefit for the complexity it adds.

A red flag here are pass-through methods, which are methods that do nothing except pass their
arguments to another method. The decorator (or wrapper) design pattern can also be a sign of
interface duplication across layers.


## Chapter 8: Pull Complexity Downward

When developing a module, it is often tempting to punt complexity to users of the module. If you're
not sure how to deal with come condition, just throw an exception; if you don't know which policy
will work best, just define some configuration parameters and let the user set them. This is usually
the wrong thing to do: most modules have more users than developers, so it's better for developers
to deal with an issue once. Another way to look at it is taht it's more important for a module to
have a simple interface than a simple implementation.

You can take this too far, though, and increase overall system complexity. Pulling complexity down
makes the most sense if (a) the complexity being pulled down is closely related to the module's
existing functionality, (b) pulling the complexity down will result in simplifications elsewhere in
the application, and (c) pulling the complexity down simplifies the module's interface.


## Chapter 9: Better Together or Better Apart?

One of the most fundamental questions in software design is: given two pieces of functionality,
should they be implemented together in the same place, or should their implementations be separated?
This applies at all levels in a system, from functions to classes to services.

The goal is always to reduce the complexity of the sytem as a whole and improve its modularity. It
might appear that the best weay to achieve this is to divide the system into a large nubmer of small
components: the smaller the components, the simpler each component is likely to be. However, the act
of subdividing also creates additional complexity that wasn't present before: more components means
more interfaces and often additional code to manage the components, and it may separate pieces of
code that are closely related.

In general, pick the structure that results in the best information hiding, the fewest dependencies,
and the deepest interfaces. Some guidelines are:

* Combine components if they share information, e.g. both depend on a particular file format.
* Combine components if they're used together, i.e. anyone using one is like to also use the other,
  and vice versa (it has to be bidirectional).
* Bring modules together if the interface for the new module is simpler or easier to use than the
  original interfaces. This often happens when the original modules each implement part of the
  solution to a problem.
* Separate general-purpose and special-purpose code. If a module contains a mechanism that can be
  used for several different purposes, then it should provide just that one general-purpose
  mechanism.
* Each function should do one thing and do it completely. Splitting a function only makes sense if
  it results in cleaner abstractions overall.

A red flag is if two pieces of code are physically separated, but each implementation can only be
understood by looking at the other.
