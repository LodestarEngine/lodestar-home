---
title: "Rationale"
description: "Rationale behind Lodestar."
lead: "Rationale behind Lodestar."
date: 2022-03-01T00:00:00+00:00
lastmod: 2022-03-01T00:00:00+00:00
draft: false
images: []
menu:
  docs:
    parent: "concepts"
weight: 100
toc: true
---

The greatest driver behind this project is the lack of a fast and
accessible library for applying control theoretic concepts on real-life
(embedded) systems. While platforms such as
[ROS](https://www.ros.org/)[^ros] are popular in robotics, people
working with applications that require real-time viable code and thread
safety are often compelled to write their own solutions, with
propiertary implementations being the norm, and a single unified
solution is left to be desired.

With this in mind, I set out to develop a unified and approachable
control library, with the explicit intention to make it as real-time
viable as possible while maintaining code transparency and providing a
flexible API[^api].

For more information, please check out our paper on arXiv entitled ["Lodestar: An Integrated Embedded Real-Time Control Engine"](https://arxiv.org/abs/2203.00649).

## Core Principles

Lodestar is built on the following core principles:

1. ***Firm real-time* first**:
    * Memory demands should be known at compile-time, minimum/no dynamic
        memory allocation/paging.
    * Dynamically and statically allocated objects should share a common
        API.
    * Exceptions should be thrown with care; status-based fallbacks are
        preferred.
    * Processes should be timed, with fallback mechanisms and deadline
        violation reporting.
2. **Wide platform support**:
    * Maintain minimum demands should be imposed on compiled libraries,
        preferring header-only libraries instead.
    * Use only ISO/ANSI C++11 features.
    * Make full use of POSIX[^posix] RT (real-time) mechanisms
        (`pthread`s, `sigaction`s, etc.). 
3. **Flexible interfacing and extension**:
    * Extendible, documented, and transparent core API.
    * Inproc and outproc non-blocking communication framework.
    * Thread-safe data types, with object-bound `mutex`es.

### Levels of Real-time

The following is a common classification of levels of real-time:

1. **Soft real-time**: Delayed process completion may degrade a system's
    quality of service, but is tolerable.
2. **Firm real-time**: Delayed process completion may degrade a system's
    quality of service, but is tolerable if infrequent.
3. **Hard real-time**: Delayed process completion results in system
    failure.

At least for the initial stages, Lodestar will try to get to the level
where one could reliably call it a firm real-time framework. With
sufficient testing and profiling in a hardware emulator, it is possible
to extend a firm real-time system to a hard real-time system, but it
could be the case that additional safety features and fallbacks are
expected. This is, for now, beyond the scope of the library.

[^ros]: **ROS**: Robot Operating System.

[^api]: **API**: Application Programming Interface.

[^posix]: **POSIX**: Portable Operating System Interface, a family of standards
described in IEEE 1003 and ISO/IEC 9945.

[^slicot]: **SLICOT**: Subroutine Library in Systems and Control Theory.

[^ginac]: **GiNaC**: GiNac is Not a CAS (Computer Algebra System).

[^bsd]: **BSD**: Berkeley Software Distribution, a family of licenses.