---
title: "Introduction"
description: "Lodestar is a lightweight C++11 framework for rapidly prototyping and deploying real-time control systems."
lead: "Lodestar is a lightweight C++11 framework for rapidly prototyping and deploying real-time control systems."
date: 2022-03-01T00:00:00+00:00
lastmod: 2022-03-01T00:00:00+00:00
draft: false
images: []
menu:
  docs:
    parent: "prologue"
weight: 100
toc: true
---

_Lodestar_ aims to be a framework that provides _directly executable code_, unlike most modeling and simulation-centric
toolboxes, which have not been tailored for use in real-time applications. At the same time, _Lodestar_ also allows for
_code generation_ when performance is particularly important. It comes built-in with many compile-time checks that cover
most common typos and bugs (i.e., matrix-vector dimensions mismatch, connecting inputs to inputs, etc.).

To this end, _Lodestar_ adopts a function block description of systems, where
each `Block` provides a _pure function_ (i.e., one that only alters its internal state). Each of these block may have
any number of inputs, parameters, and outputs. The resulting functional block diagrams can easily be extended with new
user-defined block types, without incurring overhead.

```
                           Parameters
           +-----------------------------------------+
Inputs --->| Block : f(Input, Parameters) -> Outputs |---> Outputs
           +-----------------------------------------+
```

### Features

- Automatic resolution of circular data-dependencies (algebraic loops).
- Transparent compile-time error checking, as well as run-time checks prior to executing code.
- Easy extensibility with a simple yet powerful `Block` API based on template metaprogramming.
- Clean C++ code generation with predetermined function execution order, as well as resolved data-dependencies.
- Zero-overhead abstraction using templated classes; it does not matter if you have a thousand inputs, or just one.
- Out-of-the-box networking support, with efficient serialization.
- Automatic direct encryption and decryption of messages for enhanced security using state-of-the-art elliptic curve algorithms.

## Getting Started

### Dependencies

* CMake
* A C++11-compliant compiler
* [GiNaC](https://ginac.de/) (optional, for resolving algebraic loops)
* [nng](https://nng.nanomsg.org/) (optional, for networking)

### Building

Simply clone the [repository](https://github.com/helkebir/Lodestar) and build using CMake.
If you just want to grab a static library, run `cmake ..` instead of a debug build.

{{< details "Installing/disabling dependencies" >}}
You can install the dependencies as follows:

- On Ubuntu (Debian):
  ```bash
  sudo apt-get install libginac-dev libcln-dev libnng-dev
  ```
- On macOS:
  ```bash
  brew install ginac nng
  ```
You can disable GiNaC and NNG using the `-DWITH_GINAC=OFF -DWITH_NNG=OFF` flags when running the `cmake` command:

```cmake .. -DCMAKE_BUILD_TYPE=Debug -DWITH_GINAC=OFF -DWITH_NNG=OFF```
{{< /details >}}

```bash
git clone https://github.com/helkebir/Lodestar
cd Lodestar
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Debug
make
sudo make install
```

You can find some demos in the `examples/` folder, as well as unit tests of different components in the `tests/` folder.