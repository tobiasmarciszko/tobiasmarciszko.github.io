---
layout: post
title: Tests using data sets and structured bindings
date: 2019-11-04 10:32
category:  
author: 
tags: [Catch2, c++, c++11, c++17]
summary: 
image: asm-bg.png
---
Quack! This is apparently the first post. Let's see if we can write something interesting :)

First off we add the data set. Here we use a vector of tuples which contain three values each. Those are the values I want to test against in my [Catch2](https://github.com/catchorg/Catch2) tests.

```cpp
    const std::vector<std::tuple<uint, float, float>> data{
        {0, 1.0, 1.5},
        {1, 1.5, 2.0},
        {2, 2.0, 2.5},
        {3, 2.5, 2.5},
        {4, 2.5, 1.5},
        {5, 1.5, 1.0},
    };
```
Next step is to iterate over our data with a [range-based for loop](https://en.cppreference.com/w/cpp/language/range-for), from C++11, and then use [structured bindings](https://en.cppreference.com/w/cpp/language/structured_binding), from C++17, to extract the data!

```cpp
    for (const auto& values: data) {
        auto [index, n1, n2] = values;

        Computations comps = prepare_computations(xs.at(index), r, xs);
        REQUIRE(equal(comps.n1, n1));
        REQUIRE(equal(comps.n2, n2));
    }

```

The structured binding inside the for loop can also be assigned directly in the for statement: 

```cpp
    for (const auto& [index, n1, n2]: data) {
        Computations comps = prepare_computations(xs.at(index), r, xs);
        REQUIRE(equal(comps.n1, n1));
        REQUIRE(equal(comps.n2, n2));
    }
```
---

For full details, see commit [34f2671a](https://github.com/tobiasmarciszko/qt_raytracer_challenge/commit/34f2671a6ee63beb58a731ca9c1267595fbab8c6).

That's all folks! :)
