---
title: "Porting C++ code to python using the pybind11 library"
author: Freeman Irabaruta
date: 2021-04-04 19:55:00 +0800
categories: [Blogging,Coding]
tags: [Python,C_lang, C++]
math: true
mermaid: true
image:
  src: https://raw.githubusercontent.com/snow-blade/blog/master/content/assets/lite.png
---

As a c++ and python developer, have you ever wondered on how you can use both languages in a single project ?

Using C and python, it is extremely easy to do so as python even has a built-in module called [cython](https://cython.org)  but i kind of struggled to find a decent and easy to use c++ library which can port c++ code to python.

In the past, an excellent library named [Boost.python](http://www.boost.org/doc/libs/1_58_0/libs/python/doc) has been made, but the problem resided in the fact that  Boost is an enormously large and complex suite of utility libraries that works with almost every C++ compiler in existence. This compatibility has its cost: arcane template tricks and workarounds are necessary to support the oldest and buggiest of compiler specimens. Now that C++11-compatible compilers are widely available, this heavy machinery has become an excessively large and unnecessary dependency. 

# Installing

<div class= "tip">  <div class="tip-header">📓 📝 😎 tip</div> <br>
    Be sure to have gcc, make and cmake installed to be able to follow these steps.
 </div>



```bash
$ git clone https://github.com/pybind/pybind11 # clone the project's github repository
$ cd pybind11 
$ mkdir build
$ cd build
$ cmake ..
$ make check -j 4 # compile and run the tests
```

# Example : py_fast_int_stack

Ok, now that we've installed all the required modules, let's build a simple project with the library.

We are going to build a very simple stack module using c++ then we will use it in our python code, then we will use a benchmarking library to check which code is faster and if it is, by how much it is.

<div class="tip">  <div class="tip-header">📓 📝 😎 tip</div><br>
 First of all I assume the reader has a basic understanding of python and a solid understanding of some c++ principles.</div>

Let's start by building the project's folder.

```bash
$ mkdir ffr-py
$ cd ffr-py
$ touch module.cpp
```

Then, using your favorite text editor, open the module.cpp file.

<div class = "tip">  <div class="tip-header">📓 📝 😎 tip</div><br>
Ensure that these 2 lines are always at the start of your code every time you want to make a project with pybind11. </div>

```c++
#include <pybind11/pybind11.h>

namespace py = pybind11;
```

Let me first showcase the code, then we'll explain it later.

```c++
#include <pybind11/pybind11.h>

namespace py = pybind11;

class element{
public:
   int value;
};
class stack{
  public:
    stack(){
      size_ = 0;
    }
    void push(int x){
      els[size_].value = x;
      size_ ++;
    }
    void pop(){
      els[size_ - 1].value = NULL;
      size_ --;
    }
    int size(){
      return size_;
    }
    int top(){
      return els[size_ -1].value;
    }

private:

    element els[2000];
    int size_;
};

PYBIND11_MODULE(module, m) {
        m.doc() = "pybind11 example plugin"; // optional module docstring
        py::class_<stack>(m, "Stack")
          .def(py::init<>())
          .def("push",&stack::push)
          .def("pop",&stack::pop)
          .def("size", &stack::size)
          .def("top", &stack::top);
    }
```

I shall not explain the code from line 5 to line 33 as it is pretty obvious for someone who has a basic understanding of c++.

### PYBIND11_MODULE
Now let me explain this seemingly complicated code.
```c++
PYBIND11_MODULE(module, m) {
        m.doc() = "pybind11 example plugin"; // optional module docstring
        py::class_<stack>(m, "Stack")   
          .def(py::init<>()) // the init method
          .def("push",&stack::push) // push function
          .def("pop",&stack::pop) // pop function
          .def("size", &stack::size) // size function
          .def("top", &stack::top); // top function
}
```

This code simply transforms our c++ class and it's method into a python class. If you want to know more on using pybind11 on object oriented code, checkout the [docs](https://pybind11.readthedocs.io/en/latest/classes.html#creating-bindings-for-a-custom-type) .

Let us compile the code :

```bash
$ c++ -O3 -Wall -shared -std=c++11 -fPIC $(pkg-config --cflags --libs python3) `python3 -m pybind11 --includes` module.cpp -o module `python3-config --extension-suffix
```

A file called ``` module.cpython-38-x86_64-linux-gnu.so ``` should be generated.

Be sure to be in the same folder as that file when you launch the python REPL.

```console
$ python
>>> from module import Stack
>>> stack = Stack()
>>> stack.push(12)
>>> stack.push(2000)
>>> stack.top()
2000
>>> stack.pop()
>>> stack.top()
12
```

