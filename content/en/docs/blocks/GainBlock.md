---
title: "GainBlock"
description: "Multiplication or convolution by a constant value."
lead: "Multiplication or convolution by a constant value."
date: 2022-03-01T00:00:00+00:00
lastmod: 2022-03-01T00:00:00+00:00
draft: false
images: []
menu:
  docs:
    parent: "blocks"
weight: 100
toc: true
---

## Description

Multiplies/convolves input by a value (gain).

## Interface Definition

`GainBlock<TInput, TGain, TOps, TConv>`

### Template Parameters

| Index | Parameter type | Name | Description | Default value |
|:------|----------------|------|-------------|---------------|
| 0     | `typename`     | `TInput` | Input value type | - |
| 1     | `typename`     | `TGain` | Gain value type | `TInput` |
| 2     | `GainBlockOperator` | `TOps` | Gain block operator type | `::Left` |
| 3     | `GainBlockConvolutionMode` | `TConv` | Gain block convolution type | `::Reflect` |

### Inputs

| Index | Parameter type | Name | Description | Default value |
|:------|----------------|------|-------------|---------------|
| 0     | `<TInput>`     | -    | Input value | - |

### Outputs

| Index | Parameter type | Name | Description | Default value |
|:------|----------------|------|-------------|---------------|
| 0     | `<TOutput>`    | -    | Output value | - |

Determination of `TOutput` is not straightforward; in the case of multiplication, we must first check if multiplication is possible.
In the case of a scalar gain, or convolution, the output type is the same as `TInput`.

### Parameters

| Index | Parameter type | Name | Description | Default value |
|:------|----------------|------|-------------|---------------|
| 0     | `<TGain>`      | `gain` | Gain value | - |
| 1     | `double`       | `constant` | Constant value | - |

The function of `gain` varies with `TOps`; see [Effects](#effects).

## Effects

If `TOps` is:
- `::Left`, left-multiply the input by the gain.
- `::Right`, right-multiply the input by the gain.
- `::Convolution`, convolve the input with the gain, where the boundary condition is determined by `TConv`.

If `TOps` is `::Convolution`, and `TConv` is:
- `::Reflect`, reflect about the edge of the last pixel. Also known as half-sample symmetric.
- `::Constant`, fill all values beyond edges with a constant value (see parameter 1 in [Parameters](#parameters)).
- `::Nearest`, extend by replicating last pixel.
- `::Mirror`, extend by reflecting about the center of the last pixel. Also known as whole-sample symmetric.
- `::Wrap`, wrap around to the opposing edge.

## Static Asserts

- Multiplicability in case of `TOps` in `{::Left, ::Right}`.
- Scalar type congruence in case of `TOps = ::Convolution`.