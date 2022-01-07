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
