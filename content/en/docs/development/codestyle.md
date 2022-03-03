---
title: "Code Style"
description: "Lodestar's coding guidelines."
lead: "Lodestar's coding guidelines."
date: 2022-03-01T00:00:00+00:00
lastmod: 2022-03-01T00:00:00+00:00
draft: false
images: []
menu:
  docs:
    parent: "development"
weight: 100
toc: true
---

{{< alert icon="⚠️" context="info" text="This is a work-in-progress." />}}

```cpp
#ifndef LODESTAR_CONTAINER_HPP
#define LODESTAR_CONTAINER_HPP

#include <array>
// Note that we do not use `using std` to limit namespace pollution/clashes

// Preprocessor directives and their arguments should be in upper snake case 
#define STATIC_GET(ARRAY, IDX) std::get<IDX>(ARRAY)

// Template arguments should be descriptive, written in upper camel case, and prefixed with T*
template <typename TType>
// Constexpr should be prefixed with K*, and the following word should be in lower camel case.
constexpr TType &Kmax(const TType &a, const TType &b)
{   // K&R-style brace placement (preceded by a linebreak) for functions
    // Indents are 4 spaces deep
    return (int &) (a > b ? a : b);
}

// Global variables follow the same naming rules as other variables
// Constant variables are prefixed with k*
const static unsigned int kDefaultSize = 10;

template <typename TType, unsigned int TSize = kDefaultSize>
// Class names should be in upper camel case
class Container {
public:
    // Typedefs should be descriptive and are prefixed with TD*
    typedef std::array<TType, TSize> TDArray;

    Container() : array_(TDArray{}) {}

    explicit Container(const TDArray &array) : array_(array) {}

    explicit Container(const TDArray *array) : array_(*array) {}

    Container(const TDArray &array, const unsigned int end) : array_(TDArray{})
    {
        unsigned int i = 0;
        while ((i < end) && (i < TSize)) {
            array_[i] = array[i];
            i++;
        }
    }

    // Use of lower camel case names is encouraged
    void clearArray()
    {
        for (unsigned int i = 0; i < TSize; i++)
            array_[i] = TType{};
    }

    TType getElement(const unsigned int idx) const
    {
        return array_[idx];
    }

    void setElement(const unsigned int idx, const TType &val)
    {
        array_[idx] = val;
    }

    TDArray copyArray() const {
        return array_;
    }

    void copyContainer(const Container &other) const
    {
        for (unsigned int i = 0; i < TSize; i++)
            array_[i] = other.getElement(i);
    }

    TType getFirst() const
    {
        return STATIC_GET(array_, 0);
    }

    TType getLast() const
    {
        return STATIC_GET(array_, Kmax(0, TSize - 1));
    }

    const unsigned int kSize = TSize;
protected:
    // Private/protected member variables are postfixed with *_
    std::array<TType, TSize> array_;

    // Static members may be prefixed with z* in case of ambiguities
    template<unsigned int TSizeOther>
    Container<TType, TSize> zCopyContainer(const Container<TType, TSizeOther> &other)
    {
        Container<TType, TSize> container{};

        unsigned int i = 0;
        while ((i < other.kSize) && (i < TSize))
            container.setElement(i, other.getElement(i));
        
        return container;
    }
}

// Preprocessor directives should be undef'd if they are not part of the public API
#undef STATIC_GET

#endif //LODESTAR_CONTAINER_HPP
```