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


## Chapter 10: Define Errors Out Of Existence

Dealing with exceptions (or any other mechanism to signal errors) is one of the worst sources of
complexity in programs. Code dealing with special conditions is inherently harder to write than code
that deals with normal cases; exceptions disrupt the normal flow of the code; and error conditions
are often hard to test. The best way to reduce the damage is to reduce the number of places where
exceptions have to be handled. John presents three strategies to do so.

The best one, where it is possible, is to define errors out of existence. For example, Java's
substring method throws an exception if a given index is out of range. Instead, the method could
have been defined to return the overlap between the string and the given indexes, so the error case
simply disappears.

The second technique is exception masking. With this approach, an exceptional condition is detected
and handled at a low level, for example by retrying an operation, so the module's callers don't need
to handle it. This results in deeper modules since it reduces the module's interface and adds
functionality.

The third technique is exception aggregation. The diea is so refactor your code to handle many
exceptions in one place instead of writing many individual handlers.

Finally, in some cases it's best to simply crash the application instead of invoking error handling.
A good example is running out of memory.


## Chapter 11: Design It Twice

Designing software is hard, so it's unlikely that your first thoughts about how to structure a
module or system will produce the best design. You'll end up with a much better result if you
consider multiple options for each major design decision: design it twice.

Try to pick approaches that are radically different from each other. Even if you think there's only
one reasonable approach, consider a second design. It'll be instructive to compare the two options.
After you've roughed out the designs, make a list of pros and cons for each.

The most important consideration for an itnerface is ease of use for higher-level software. Other
points worth considering:

* Does one alternative have a simpler interface than the other?
* Is one interface more general-purpose than another?
* Does one interface enable a more efficient implementation than another?


## Chapter 12: Why Write Comments? The Four Excuses

In-code documentation plays a crucial role in software design. Comments capture information that
was in the mind of the designer but couldn't be represented in code. Without comments, users must
read the code implementing an interface to be able to use it, and then there is no abstraction.

Unfortunately, a lot of code essentially has no comments. When developers don't write comments,
they usually justify their behavior with one or more of these excuses:

* "Good code is self-documenting." There are things you can do when writing code to reduce the need
  for comments, but there is still a significant amount of design information that can't be
  represented in code. A high-level description of what each function does, the rationale for a
  particular design decision, or the conditions under which it makes sense to call a particular
  method, can only be described in comments.
* "I don't have time to write comments." The counter-argument is the investment mindset discussed in
  chapter 3. In the long run, good documentation will speed up development.
* "Comments get out of date and become misleading." Keeping documentation up-to-date actually
  doesn't take much effort; updating comments for a change in the code usually takes much less time
  than the change in the code itself.
* "All the comments I've seen are worthless." This is probably the one with the most merit. Every
  software developer has seen comments that provide no useful information. Fortunately, this problem
  is solvable; writing solid documentation is not hard once you know how.


## Chapter 13: Comments Should Describe Things that Aren't Obvious from the Code

The purpose of comments is to record information that was in the mind of the developer when they
wrote the code but that can't be expressed as code. That means comments should describe things that
aren't obvious from the code. Documentation augments the code by providing information at a
different level of detail.

Some comments provide information at a lower, more detailed level than the code; these comments add
precision by clarifying the exact meaning of the code. A good example are comments for variable
declarations, arguments, and return types. The name and type in these declarationsare usually not
very precise; comments can fill in missing detail such as units for a variable, whether null values
are permitted, and certain properties that are always true (invariants).

Higher-level comments enhance code by providing intuition. They omit details and help the reader to
understand the overall intent and structure of the code.

High-level comments are more difficult to write than lower-level comments because you must think
about the code in a different way. This means taking a step back from the details, deciding which
aspects of the system are most important, and finding a simple way to think about a complex entity.
This is the essence of abstraction.

One tricky kind of documentation is comments describing cross-module design decisions. Where
possible, each design decision should be encapsulated in a module, but real systems inevitably end
up with design decisions that affect multiple classes. It's important to document those, but where
do you put that documentation so it will be discovered by developers working on any of the affected
modules? There's no perfect solution. John recommends adding a section to a central file on design
notes, then referring to that section wherever it is relevant.


## Chapter 14: Choosing Names

Good names for variables, methods, and other entities are a form of documentation: they make code
easier to understand, reduce the need for other documentation, and make it easier to detect errors.
Conversely, poor names increase the complexity of code and create ambiguities and misunderstandings
that can result in bugs.

When choosing a name, the goal is to create an image in the mind of the reader about the nature of
the thing being named. The challenge is to find just a few words that capture the most important
aspects of the entity.

John presents three guidelines for naming:

* Names should be precise. If a name is generic or vague, it's hard for readers to tell what the
  name refers to.
* Use names consistently. Consistent naming reduces cofnitive load: once the reader has seen the
  name in one context, they can reuse this knowledge and instantly make assumptions when they see
  the name in a different context.
* Avoid extra words. Words that don't help to clarify the variable's meaning just add clutter.

If it's hard to find a simple name that creates a clear image of the object, that's a hint that the
underlying object doesn't have a clear design. The process of choosing good names can improve your
design by exposing weaknesses.


## Chapter 15: Write The Comments First

Some developers put off writing documentation until the end of a project, after coding and unit
testing are complete. This is a sure way to produce poor-quality documentation: by that time your
memories of the design process will have become fuzzy and you'll be ready to move on to the next
project, so you're likely to do add just enough comments to look respectable, or to skip commenting
altogether.

John recommends to write interface documentation before implementing the interface, and
implementation comments while writing the implementation. This leads to better comments, but more
importantly it improves the system design. Comments are the only way to fully capture abstractions.
If you write comments describing the abstractions at the beginning, you can review and tune them
before writing implementation code.

Comments also serve as a canary in the coal mine of complexity. If it's difficult to write a comment
that describes an abstraction completely and concisely, perhaps it's time to revisit that
abstraction.


## Chapter 16: Modifying Existing Code

Software development is iterative and incremental, which means a system's design is constantly
evolving. How do you keep the design from deteriorating and complexity from creeping in?

The most important tool is to keep a strategic programming mindset. When changin existing code to
fix a bug or add a feature, it's often tempting to make the smallest change that does what you need,
but over time this approach accumulates special cases, dependencies, and other forms of complexity.
Instead, with each change, think about whether the current design is still the best one. If not,
take the time to refactor and improve the design.

It's especially easy to forget updating comments that have been invalidated by code changes. To keep
documentation up-to-date:

* Always position comments close to the code they describe.
* Avoid duplicating comments.
* Put comments in the code, not just in the commit log, if future developers should see them.
* Comments that are higher-level than the code are usually easier to maintain.


## Chapter 17: Consistency

Consistency is a powerful tool for reducing the complexity of a system. If similar things are done
in a similar way and dissimilar things are done in different ways, developers can work more quickly
with fewer mistakes.

Some tips for ensuring consistency:

* Document conventions such as coding style guides in a spot where developers are likely to see it.
* Enforce consistency with automated checkers.
* When you join an existing project, follow the conventions you see.
* Don't take consistency so far that you make disimilar things look the same.
