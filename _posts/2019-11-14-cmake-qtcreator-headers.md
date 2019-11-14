---
layout: post
title: CMake and header files in Qt Creator
date: 2019-11-14 12:47
category: 
author: 
tags: [Qt, Qt Creator, CMake, C++]
summary: 
image: cmake-logo.png
---

I'm using [CMake](https://cmake.org/) as the build system in [Qt Creator](https://doc.qt.io/qt-5/cmake-manual.html). One of the reasons is to simply transition of the project to and from [CLion](https://www.jetbrains.com/clion) which is a nice alternative when it comes to C++ and Qt development. 

Another reason to use CMake is the [Qt ambition](https://wiki.qt.io/CMake_Port) to start using it in Qt 6. 

I discovered that when I was using CMake (I'm not very experienced in CMake), the header files in the IDE (Qt Creator) wasn't showing up properly in the Project's sidebar. The reasons for this is how you declare your files in the **add_executables** directive in your CMakeLists.txt file.

If your header files are listed here as in: 

![add_executable](/img/cmake_headers_1.png)

Then they will appear nicely as part of the project in the sidebar:

![headers showing](/img/cmake_headers_2.png)

If you remove them from your CMakeLists.txt **add_executable** directive, they will be greyed out and moved out as external dependencies but not **part** of the project. Note that your project will still build just fine.

![headers not showing](/img/cmake_headers_3.png)

See the [add_executable documentation](https://cmake.org/cmake/help/v3.16/command/add_executable.html?highlight=add_executable).

A small bonus: If you're using Qt Creator and want the CMake documentation as .qch file, get it [here](https://cmake.org/cmake/help/latest/CMake.qch). Read more about adding [external documentation to Qt Creator](https://doc.qt.io/qtcreator/creator-help.html#adding-external-documentation).

That's all folks! :)
