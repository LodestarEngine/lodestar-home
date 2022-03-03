---
title: "Quick Start"
description: "How to get started with Lodestar."
lead: ""
date: 2022-03-01T00:00:00+00:00
lastmod: 2022-03-01T00:00:00+00:00
draft: false
images: []
menu:
  docs:
    parent: "prologue"
weight: 110
toc: true
---

## Dependencies

* CMake
* A C++11-compliant compiler
* [GiNaC](https://ginac.de/) (optional, for resolving algebraic loops)
* [nng](https://nng.nanomsg.org/) (optional, for networking)

## Building

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

---

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

## First Example

Here's an example to get you started. We will be using `ConstantBlock`, `SumBlock`, and `GainBlock` objects to construct
a simple program.

```cpp
#include <Lodestar/blocks/std/ConstantBlock.hpp>
#include <Lodestar/blocks/std/SumBlock.hpp>
#include <Lodestar/blocks/std/GainBlock.hpp>

#include <Lodestar/blocks/BlockUtilities.hpp>
#include <Lodestar/blocks/aux/Executor.hpp>

using namespace ls::blocks;
using namespace ls::blocks::std;

/*
 *                (+)
 * +---+    +---+    +---+
 * | c |--->| g |--->| s |--->
 * +---+    +---+    +---+
 *                     ^ (-)
 * +----+              |
 * | c2 |--------------+
 * +----+
 */

int main()
{
    ConstantBlock<double> c{5}, c2{2};
    SumBlock<double, 2> s;
    GainBlock<double> g{0.5};
    
    s.setOperators(decltype(s)::Plus, decltype(s)::Minus);
    
    // We now establish the interconnections:
    connect(c.o<0>(), g.i<0>());
    connect(g.o<0>(), s.i<0>());
    connect(c2.o<0>(), s.i<1>());
    
    // We group all our blocks in a BlockPack object,
    // which contains all components of our system.
    BlockPack bp{c, c2, s, g};
    
    // We pass the BlockPack onto the Executor,
    // which will allow us to resolve the execution order,
    // providing a single trigger function for the entire system.
    aux::Executor ex{bp};
    ex.resolveExecutionOrder();

    // Triggering the entire system is as simple as
    // calling the trigger function of the executor.
    ex.trigger();
    
    // We obtain +5*0.5 -2 = 0.5  
    auto res = (s.o<0>().object == 0.5);
    
    return 0;
}
```

## Next Steps

Look around in the sidebar on the left to read on whatever topic is of direct interest. We recommend browsing through the [Concepts](/docs/concepts) tab, especially the [Rationale](/docs/concepts/rationale). Some assorted code snippets and recipes are laid out in the [Recipes](/recipes/) section. If you want to directly dive into the source code, check out the [C++ reference](/doxygen). 