---
layout: post
title: Ok, this is the second post!
date: 2019-11-04 16:08
category: 
author: 
tags: [test, dummy, c++]
summary: 
image: cpp-logo.png
---

Cool, this is the second post!

Some code: 

```cpp
const auto dx = abs(x1 - x0);
    const auto dy = abs(y1 - y0);
    const auto sx = (x0 < x1) ? 1 : -1;
    const auto sy = (y0 < y1) ? 1 : -1;
    auto err = dx - dy;

    while (true) {
       if (x0 >= 0 &&
           y0 >= 0 &&
           x0 < m_camera.hsize &&
           y0 < m_camera.vsize) {
            m_framebuffer.setPixel(x0, y0, color);
       }

       if ((x0 == x1) && (y0 == y1)) break;
       const auto e2 = 2 * err;
       if (e2 > -dy) { err -= dy; x0 += sx; }
       if (e2 < dx) { err += dx; y0 += sy; }
    }
```
