---
title: "Features"
description: "Lodestar's feature list."
lead: "Lodestar's feature list."
date: 2022-03-01T00:00:00+00:00
lastmod: 2022-03-01T00:00:00+00:00
draft: false
images: []
menu:
  docs:
    parent: "concepts"
weight: 90
toc: true
---

- Automatic resolution of circular data-dependencies (algebraic loops).
- Transparent compile-time error checking, as well as run-time checks prior to executing code.
- Easy extensibility with a simple yet powerful `Block` API based on template metaprogramming.
- Clean C++ code generation with predetermined function execution order, as well as resolved data-dependencies.
- Zero-overhead abstraction using templated classes; it does not matter if you have a thousand inputs, or just one.
- Out-of-the-box networking support, with efficient serialization.
- Automatic direct encryption and decryption of messages for enhanced security using state-of-the-art elliptic curve algorithms.

## Control Routines

For now, this section will be a checkboxed list; more details will
follow. We adhere in most cases to the classification system used in 
[SLICOT](http://slicot.org/the-control-and-systems-library-slicot/slicot-library-organization)[^slicot].
Cursive identifiers are extensions to the SLICOT system that we devised
for additional features that Lodestar provides.

- [ ] **A**: Analysis Routines
    - [ ] **AB**: State-Space Analysis
        - [x] **AB04**: Continuous/Discrete Time
            - [x] **AB04MD**: Discrete-time <-> continuous-time
                conversion by bilinear transformation.
                    - Generalized bilinear transformations have been
                        implemented, with specialization for Tustin, Euler,
                        and backward differencing transforms.<br>
                        [<code lang=C++>ls::analysis::BilinearTransformation</code>](https://github.com/helkebir/Lodestar/blob/master/Lodestar/analysis/BilinearTransformation.hpp)
                    - Beyond the SLICOT spec, zero-order hold
                        transformations based on exponential and logarithmic
                        matrices have been implemented.<br>
                        [<code lang=C++>ls::analysis::ZeroOrderHold</code>](https://github.com/helkebir/Lodestar/blob/master/Lodestar/analysis/ZeroOrderHold.hpp)
        - [ ] **AB07**: Inverse and Dual Systems
            - [x] **AB07ND**: Inverse of a given state-space representation
                - Continuous-time state space inverses have been
                    implement, but error handling is not properly done.
                    <br>
                    [<code lang=C++>ls::analysis::LinearSystemInverse</code>](https://github.com/helkebir/Lodestar/blob/master/Lodestar/analysis/LinearSystemInverse.hpp)

- [ ] **F**: Filtering
    - [ ] **FB**: Kalman Filters
        - [ ] **FB01**: _N/A_
            - [ ] **FB01VD**: One recursion of the conventional Kalman
                filter
                    - We currently have an implementation for finite
                        horizon Kalman gain computation for
                        discrete-time linear time invariant systems.<br>
                        [<code lang=C++>ls::filter::DiscreteLQE</code>](https://github.com/helkebir/Lodestar/blob/master/Lodestar/filter/DiscreteLQE.hpp)

- [ ] **S**: Synthesis Routines
    - [ ] **SB**: State-Space Synthesis
        - [ ] **SB02**: Riccati Equations
            - [ ] **SB02MD**: Solution of continuous- or discrete-time
                algebraic Riccati equations (Schur vectors method)
                    - A discrete-time ARE solver is currently in
                        development.<br>
                        [<code lang=C++>ls::synthesis::AlgebraicRiccatiEquation::solveDARE</code>](https://github.com/helkebir/Lodestar/blob/master/Lodestar/synthesis/AlgebraicRiccatiEquation.hpp)

- [ ] **T**: Transformation Routines
    - [ ] **TD**: Rational Matrix
        - [x] **TD05**: Rational Matrix to State-Space Conversion
            - [x] **TD05AD**: Minimal state-space representation for a
                proper transfer matrix 
                    - This is currently implemented for scalar transfer
                        functions, but can easily be extended to
                        transfer function matrices.<br>
                        [<code lang=C++>ls::systems::TransferFunction::toStateSpace</code>](https://github.com/helkebir/Lodestar/blob/master/Lodestar/systems/TransferFunction.hpp)

- [ ] **U**: Utility Routines
    - [ ] **UD**: Numerical Data Handling
        - [ ] **UD01**: _N/A_
            - [x] **UD01BD**: Reading a matrix polynomial
                - This is achieved using _GiNaC_[^ginac] routines; currently
                    only scalar transfer functions are implemented, but
                    this is easily extendable.<br>
                    [<code lang=C++>ls::systems::TransferFunction::copyFromExpression</code>](https://github.com/helkebir/Lodestar/blob/master/Lodestar/systems/TransferFunction.hpp)
    - [ ] ***US***: _Symbolic Manipulation/Data Handling_
        - [ ] ***US01***: _Ordinary Differential Equations_

[^ros]: **ROS**: Robot Operating System.

[^api]: **API**: Application Programming Interface.

[^posix]: **POSIX**: Portable Operating System Interface, a family of standards
described in IEEE 1003 and ISO/IEC 9945.

[^slicot]: **SLICOT**: [Subroutine Library in Systems and Control Theory](http://slicot.org/).

[^ginac]: **GiNaC**: [GiNac is Not a CAS](http://ginac.de/) (Computer Algebra System).

[^bsd]: **BSD**: Berkeley Software Distribution, a family of licenses.