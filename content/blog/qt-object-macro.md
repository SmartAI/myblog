+++
title = "Qt What is Q_OBJECT"
author = ["SmartAI"]
date = 2025-04-22T22:41:00-07:00
tags = ["bug", "qt"]
categories = ["development", "coding"]
draft = false
+++

I encountered a link error after adding `Q_OBJECT` macro to the class definition. The error message was similar to `vtable for XXX is missing, NOTE: a missing vtable usually means the first non-inline virtual member function has no definition.`. However, the error disappeared if I removed the `Q_OBJECT` macro.

The `Q_OBJECT` macro, used in `Qt`, enables features like `slot` and `signal`. This macro requires the Meta Object Compiler(`moc`) to process the class definition. I presume that the `moc` generates source code that must be compiled and linked into the final target.

Even though I have enabled the `AUTOMOC` in my `CMakeLists.txt` file, I still encountered this issue. After searching online, I realized that the header file(`hpp`) containing the class with `Q_OBJECT` macro needed to be explicitly listed as a source file for the target in `CMakeLists.txt`. For instance,

```shell
set(GUI_SOURCES
  MainWindow.cpp
  MainWindow.hpp # <--- here!!!
)

set(GUI_UIS
  MainWindow.ui
)

set(GUI_RESOURCES
    # Add resource files here as they are created
)

add_library(gui STATIC
    ${GUI_SOURCES}
    ${GUI_UIS}
    ${GUI_RESOURCES}
    )
```

After adding the `MainWindow.hpp` (which contains the `Q_OBJECT` macro) to the source list for `gui` target in `CMakeLists.txt`, the project compiled and linked successfully.
I hope this explanation helps others who encounter similar `vtable missing` errors when use `Q_OBJECT` with `CMake`, even when `AUTOMOC` is enabled.
