---
layout: post
title: "A noticeable flag"
description: "A noticeable flag"
categories: [CMake]
tags: [CMake]
redirect_from:
  - /2018/06/17/
---

## A noticeable flag

I was reading CMU 15-445 (Fall 2017) database project cmake file and found one interesting flag defined in `CMakeList.txt`. 
```cmake
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -D__VTableFILE__='\"$(subst ${CMAKE_SOURCE_DIR}/ , , $(abspath $<))\"'")
```
This piece of code can be found in [stackoverflow](https://stackoverflow.com/questions/8487986/file-macro-shows-full-path/16658858#16658858). Let me explain it first. 
Here we need to add one flag to `CMAKE_CXX_FLAGS` variable `D__VTableFILE__` 
`subst` is a function in `makefile` which has the form `$(subst from,to,text)`. It will replace every `from` text to `to` text in `text`. And here we are substitute `${CMAKE_SOURCE_DIR}` with empty string in `abspath $<`. `$<` is one special variable defined in makefile. It means the name of the first prerequisite.
Then use `__FILENAME__` in place of `__FILE__`.

