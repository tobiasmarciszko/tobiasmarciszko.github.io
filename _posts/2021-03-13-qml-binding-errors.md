---
layout: post
title: How to avoid QML binding errors on app exit
date: 2021-03-13 09:35
category: 
author: 
tags: [Qt, QML]
summary: 
image: qt-logo-big.png
---

In my project [Freeside Raytracer](https://github.com/tobiasmarciszko/qt_raytracer_challenge) I'm using a C++ backend together with a QML "frontend". The C++ backend is a QObject class that is exposed to the QML layer via a root context property. 

```cpp
engine.rootContext()->setContextProperty("raytracer", &raytracer);
```

What I noticed is that when the application exists, I get binding errors from the QML engine since the backend has already been destructed. The reason for that is that the QML engine "outlives" the backend, expecting the backend to be there.

This is an excerpt from the output:

```txt
qrc:/raytracer.qml:423: TypeError: Cannot read property 'progress' of null
qrc:/raytracer.qml:424: TypeError: Cannot read property 'rendering' of null
qrc:/raytracer.qml:409: TypeError: Cannot read property 'lastRenderTime' of null
```

There are two solutions for this. One way is to make sure the backend is constructed _before_ the QML engine, so that the backend object is destructed last, or at least after the engine itself: 

```cpp
RaytracerBackend raytracer; // Backend constructed before the QML engine
QQmlApplicationEngine engine;
```

The other option is to create the backend on the heap and set the engine as the parent. This means that when the engine is destructed, the backend is in turn also destructed, like so: 

```cpp
QQmlApplicationEngine engine;
const auto raytracer = new RaytracerBackend(&engine);
```

I don't know what the recommended approach is, but I think the first solution is cleaner and you create the objects directly on the stack. On the other hand, it's more fragile since the order is the solution but also prone to bugs if you start refactoring :) The second option is more robust in the way that you cannot accidentally change the order (you cannot declare the backend unless the engine already exists).

I hope that helps. It took me a while to figure out why this was the case. The errors themselves are harmless, since the application is already exiting, but nevertheless they are errors that shouldn't be there :)

That's all folks! :)
