---
title: "PID Controller"
description: "A simple proportional-integral-derivative controller."
lead: "A simple proportional-integral-derivative controller."
date: 2022-03-01T00:00:00+00:00
lastmod: 2022-03-01T00:00:00+00:00
draft: false
images: []
menu:
  docs:
    parent: "recipes"
weight: 100
toc: true
---

{{< alert icon="❓" context="info" text="For more background on PID controllers, see <a href=\"https://theory.ldstr.dev/pid/\">this theory page</a>." />}}

In this recipe, we will prepare a simple discretized PID controller. As a bonus, we show how to take this prebaked controller to the next level by programming a PID controller with anti-windup from scratch.

## Gathering the Ingredients

You will need the following standard library blocks:

- `ConstantBlock`, for a constant setpoint.
- `SumBlock`, to add and subtract signals.
- `GainBlock`, to multiply signals with a fixed gain.

Include them as follows:

```cpp
#include <Lodestar/blocks/std/ConstantBlock.hpp>
#include <Lodestar/blocks/std/SumBlock.hpp>
#include <Lodestar/blocks/std/GainBlock.hpp>
#include <Lodestar/blocks/std/SimplePIDBlock.hpp>
```

To make life easier for ourselves and reduce the amount of typing, let's add the following two `using namespace` declarations:

```cpp
using namespace ls::blocks;
using namespace ls::blocks::std;
```

{{< alert icon="⚠️" context="warning" text="Using `using` declarations pollutes your namespace and can cause name clashes; use them in moderation." />}}

## Preparing the Blocks

We start by declaring our constant setpoint, which we assume to be a `double`:

```cpp
ConstantBlock<double> constBlk{0.5};
```

You can change the constant any time before running the loop as follows:

```cpp
constBlk.constant() = -0.5;
```

{{< alert icon="⚠️" context="warning" text="Don't change constants while your program is running; there are better ways of doing this (shown just below)." />}}

Now it's time to introduce the summation that computes our error (setpoint minus the system's output):

```cpp
// The number of inputs to the summation is 2, and the type is `double`, hence <double, 2>.
SumBlock<double, 2> sumBlk;
```

You should know that a `SumBlock` assumes that all its inputs should just be added; we want to subtract the second input. To do this, we set the operators as follows:

```cpp
sumBlk.setOperators(decltype(sumBlk)::Plus, decltype(sumBlk)::Minus);
```

At this point, everything is ready for a PID controller. We are taking a shortcut here, directly using the `SimplePIDBlock` (don't worry, a more advanced example follows just after this).

```cpp
SimplePIDBlock<double> pidBlk;
pidBlk.pGain() = 1;
pidBlk.iGain() = 0.2;
pidBlk.dGain() = 0.1;
pidBlk.samplingPeriod() = 1e-3;
```

Based on the code above, it should be clear that we are setting the proportional, derivative, and integral gains, as well as the sampling period, in that exact order. We link these blocks together next.

## Linking the Blocks

We will need to introduce two more `include`s add the top of the code: one for our `connect` function, and the other for the `Executor`, which will figure out how to run our blocks.

```cpp
#include <Lodestar/blocks/BlockUtilities.hpp> // connect function
#include <Lodestar/blocks/aux/Executor.hpp> // Executor
```

We're now getting to the fun part: connecting the blocks. It should be quite obvious to see what we're after: the error should go into the PID block, and the constant should go into the first input of the sum.

```cpp
connect(sumBlk.o<0>(), pidBlk.i<0>());
connect(constBlk.o<0>(), sumBlk.i<0>());
```

Here's a pro tip for when you start taking on more advanced systems; you can _alias_ signals as follows:

```cpp
auto &e = sumBlk.o<0>();
auto &s0 = sumBlk.i<0>();
auto &s1 = sumBlk.i<1>();
auto &sp = constBlk.o<0>();
auto &pid = pidBlk.i<0>();
auto &u = pidBlk.o<0>();

connect(sp, s0);
connect(e, pid);
```

If you keep track of your naming conventions, this can feel a lot closer to drawing a block diagram by hand!

## Executing the Program

We're missing one crucial component: a physical system. This is were the possibilities start to expand; you can declare your _own_ blocks in Lodestar and directly interface with them.

For now, let's keep things simple, going for a one-step time delay (`DelayBlock`) instead:

```cpp
#include <Lodestar/blocks/std/DelayBlock.hpp>

DelayBlock<double> sysBlk;
// Set the initial value to be 5.
sysBlk.clear(5);
```

The final interconnections read:

```cpp
connect(pidBlk.o<0>(), sysBlk.i<0>());
connect(sysBlk.o<0>(), sumBlk.i<1>());
```

Or, in case you're using those handy aliases:

```cpp
connect(u, sysBlk.i<0>());
connect(sysBlk.o<0>(), s1);
```

{{< alert icon="⚠️" context="warning" text="Be aware of your `connection`s; Lodestar will complain if you connect incompatible signals, or if you try to connect two or more outputs to a single input. Also, you can't use and input as an output (for obvious reasons)." />}}

We're now ready to run the code. First, however, we need to let Lodestar know what we're working with. To do that, Lodestar uses `BlockPack`s; as the name suggests, these are packs of blocks:

```cpp
BlockPack bp{pidBlk, constBlk, sumBlk, sysBlk};
```

Now, we can pass this `BlockPack` to an `Executor`, which allows us to resolve the execution order:

```cpp
aux::Executor ex{bp};
ex.resolveExecutionOrder();
```

That's it, you can now run a single iteration of your loop (known as `trigger`ing in Lodestar parlance):

```cpp
ex.trigger();
```

To actually see our program in action, let's include `<iostream>`, and run the following code:

```cpp
for (int i=0; i<50; i++) {
    ex.trigger();
    
    // Print the system output
    ::std::cout << i << ": " << sysBlk.o<0>().object << ::std::endl;
}
```

And there you have it, your first PID controller in Lodestar!

## Going Beyond

We will need some more blocks to cook up our _advanced_ PID controller:

- `SaturationBlock`, to limit our integral value.
- `DeadzoneBlock`, to prevent small oscillations in case of small errors.
- `DelayBlock`, to be able to perform Euler integration and backward differentiation.

```cpp
#include <Lodestar/blocks/std/SaturationBlock.hpp>
#include <Lodestar/blocks/std/DeadzoneBlock.hpp>
#include <Lodestar/blocks/std/DelayBlock.hpp>
```

Now, for the block initialization:

```cpp
GainBlock<double> Kp{1};
GainBlock<double> Ki{0.2};
GainBlock<double> Kd{0.1};
GainBlock<double> periodInv{1e3};
GainBlock<double> period{1e-3};

DelayBlock<double> delayDiff, delayInt;

SaturationBlock<double> satInt;

satInt.lower() = -1;
satInt.upper() = 1;

SumBlock<double, 2> sumDiff, sumInt;
sumDiff.setOperators(decltype(sumDiff)::Plus, decltype(sumDiff)::Minus);

SumBlock<double, 3> sumPID;

SaturationBlock<double> satBlk;
satBlk.lower() = -2;
satBlk.upper() = 2;

DeadzoneBlock<double> dzBlk;
dzBlk.lower() = -0.05;
dzBlk.upper() = 0.05;
```

You can see that we introduced a couple of gains, some delays and sums, and a saturation and deadzone block. Initializing these block is pretty straightforward as shown.
We can now start linking them together.

```cpp
// Proportional
connect(sumBlk.o<0>(), Kp.i<0>());
connect(Kp.o<0>(), sumPID.i<0>());

// Integral
connect(sumInt.o<0>(), satInt.i<0>());
connect(satInt.o<0>(), delayInt.i<0>());
connect(delayInt.o<0>(), sumInt.i<0>());
connect(sumBlk.o<0>(), period.i<0>());
connect(period.o<0>(), sumInt.i<1>());
connect(satInt.o<0>(), Ki.i<0>());
connect(Ki.o<0>(), sumPID.i<1>());

// Differential
connect(sumBlk.o<0>(), sumDiff.i<0>());
connect(sumBlk.o<0>(), delayDiff.i<0>());
connect(delayDiff.o<0>(), sumDiff.i<1>());
connect(sumDiff.o<0>(), periodInv.i<0>());
connect(periodInv.o<0>(), Kd.i<0>());
connect(Kd.o<0>(), sumPID.i<2>());

// Error computation
connect(constBlk.o<0>(), sumBlk.i<0>());
connect(dzBlk.o<0>(), sumBlk.i<1>());

// Saturation
connect(sumPID.o<0>(), satBlk.i<0>());

// Deadzone
connect(sysBlk.o<0>(), dzBlk.i<0>());

// Control interconnection
connect(satBlk.o<0>(), sysBlk.i<0>());
```

Most of the code above is straightforward, but the integral and differential parts might need to be reread to see how things link up. This is where a block diagram could come in handy (Lodestar can generate those for you, but we'll get into that in another article!).

The `BlockPack`-`Executor` interaction is the same as above, but now we have a couple more blocks to declare:

```cpp
BlockPack bp{sysBlk, constBlk, sumBlk, satBlk, sumDiff, sumInt, sumPID, satInt,
             Kp, Ki, Kd, period, periodInv, delayInt, delayDiff};

aux::Executor ex{bp};
ex.resolveExecutionOrder();
```

Try running the code below and see what has changed:

```cpp
for (int i=0; i<50; i++) {
    ex.trigger();
    
    // Print the system output
    ::std::cout << i << ": " << sysBlk.o<0>().object << ::std::endl;
}
```