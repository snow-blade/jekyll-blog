---
title: "C++ Goodies ep1: Porting C++ code to python using the pybind11 library"
author: Freeman Irabaruta
date: 2021-04-04 19:55:00 +0800
categories: [Blogging,Coding,Cpp-Goodies]
tags: [Python,C_lang, C++,libraries]
math: true
mermaid: true
image:
  src: https://raw.githubusercontent.com/snow-blade/blog/master/content/assets/lite.png
---

As a c++ and python developer, have you ever wondered on how you can use both languages in a single project ?

Using C and python, it is extremely easy to do so as python even has a built-in module called [cython](https://cython.org)  but i kind of struggled to find a decent and easy to use c++ library which can port c++ code to python.

In the past, an excellent library named [Boost.python](http://www.boost.org/doc/libs/1_58_0/libs/python/doc) has been made, but the problem resided in the fact that  Boost is an enormously large and complex suite of utility libraries that works with almost every C++ compiler in existence. This compatibility has its cost: arcane template tricks and workarounds are necessary to support the oldest and buggiest of compiler specimens. Now that C++11-compatible compilers are widely available, this heavy machinery has become an excessively large and unnecessary dependency. 

## Installing

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

## Example : py_fast_int_stack

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

## PYBIND11_MODULE
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
## Benchmarking

Here is the code that we will use to run our benchmark.

```python
from line_profiler import *
import module
 
@profile
def normal_stack():
    stack = []
    stack.append(80)
    stack.append(90)
    stack.append(100)
    stack.remove(stack[0])

@profile
def cpp_stack(): 
    stack = module.Stack()
    stack.push(80)
    stack.push(90)
    stack.push(100)
    stack.pop()
spp_stack()
normal_stack()
```
Let's use the line_profiler python plugin(```pip install line_profiler```) to benchmark our 2 functions:

## First Case : test cases < 10 
```console
$ kernprof -l -v test-speed.py
Wrote profile results to test-speed.py.lprof
Timer unit: 1e-06 s

Total time: 5e-06 s
File: test-speed.py
Function: normal_stack at line 4

Line #      Hits         Time  Per Hit   % Time  Line Contents
==============================================================
     4                                           @profile
     5                                           def normal_stack():
     6         1          1.0      1.0     20.0      stack = []
     7         1          1.0      1.0     20.0      stack.append(80)
     8         1          1.0      1.0     20.0      stack.append(90)
     9         1          1.0      1.0     20.0      stack.append(100)
    10         1          1.0      1.0     20.0      stack.remove(stack[0])

Total time: 2.4e-05 s
File: test-speed.py
Function: cpp_stack at line 12

Line #      Hits         Time  Per Hit   % Time  Line Contents
==============================================================
    12                                           @profile
    13                                           def cpp_stack(): 
    14         1         16.0     16.0     66.7      stack = module.Stack()
    15         1          4.0      4.0     16.7      stack.push(80)
    16         1          2.0      2.0      8.3      stack.push(90)
    17         1          1.0      1.0      4.2      stack.push(100)
    18         1          1.0      1.0      4.2      stack.pop()
```
As we can see, the stack we implemented ourselves is slightly faster than the normal stack that we implemented in python.

## Second Case: test cases >= 1000

Let's test our code on a more realistic test case where the number of iterations is equal to 1000.
```python
from line_profiler import *
import module
 
@profile
def normal_stack():
    stack = []
    for i in range(1,1000):
        stack.append(i)
    stack.remove(stack[0])

@profile
def cpp_stack(): 
    stack = module.Stack()
    for i in range(1,1000):
        stack.push(i)
    stack.pop()
cpp_stack()
normal_stack()
```
When we compile with ``` kernprof -l -v test-speed.py ```, we get:
```console
$ kernprof -l -v test-speedpy       

Wrote profile results to test-speed.py.lprof
Timer unit: 1e-06 s

Total time: 0.001273 s
File: test-speed.py
Function: normal_stack at line 4

Line #      Hits         Time  Per Hit   % Time  Line Contents
==============================================================
     4                                           @profile
     5                                           def normal_stack():
     6         1          1.0      1.0      0.1      stack = []
     7      1000        532.0      0.5     41.8      for i in range(1,1000):
     8       999        737.0      0.7     57.9          stack.append(i)
     9         1          3.0      3.0      0.2      stack.remove(stack[0])

Total time: 0.00206 s
File: test-speed.py
Function: cpp_stack at line 11

Line #      Hits         Time  Per Hit   % Time  Line Contents
==============================================================
    11                                           @profile
    12                                           def cpp_stack():
    13         1         22.0     22.0      1.1      stack = module.Stack()
    14      1000        617.0      0.6     30.0      for i in range(1,1000):
    15       999       1419.0      1.4     68.9          stack.push(i)
    16         1          2.0      2.0      0.1      stack.pop()
```
## Wraping Up

Unfortunately, our code is slightly slower than the normal python stack because of the messy implementation of the stack data structure that I made in C++, I am sure that the code would be way faster if I had used a proper stack instead of the...uh...thing I made instead. <br>
The time for computation is exponential per iteration as it increases by every iteration because of the implementation I made of the Push method. We can see easily in the benchmark that the stack.pop() method takes up 68% of the computation time.


Here is a better implementation of the stack:
```c++
class stack
{
    int *arr;
    int top;
    int capacity;
 
public:
    stack(int size = SIZE);         // constructor
    ~stack();                       // destructor
 
    void push(int);
    int pop();
    int peek();
 
    int size();
    bool isEmpty();
    bool isFull();
};
 
// Constructor to initialize the stack
stack::stack(int size)
{
    arr = new int[size];
    capacity = size;
    top = -1;
}
 
// Destructor to free memory allocated to the stack
stack::~stack() {
    delete[] arr;
}
 
// Utility function to add an element `x` to the stack
void stack::push(int x)
{
    if (isFull())
    {
        cout << "Overflow\nProgram Terminated\n";
        exit(EXIT_FAILURE);
    }
 
    cout << "Inserting " << x << endl;
    arr[++top] = x;
}
 
// Utility function to pop a top element from the stack
int stack::pop()
{
    // check for stack underflow
    if (isEmpty())
    {
        cout << "Underflow\nProgram Terminated\n";
        exit(EXIT_FAILURE);
    }
 
    cout << "Removing " << peek() << endl;
 
    // decrease stack size by 1 and (optionally) return the popped element
    return arr[top--];
}
 
// Utility function to return the top element of the stack
int stack::peek()
{
    if (!isEmpty()) {
        return arr[top];
    }
    else {
        exit(EXIT_FAILURE);
    }
}
 
// Utility function to return the size of the stack
int stack::size() {
    return top + 1;
}
 
// Utility function to check if the stack is empty or not
bool stack::isEmpty() {
    return top == -1;               // or return size() == 0;
}
 
// Utility function to check if the stack is full or not
bool stack::isFull() {
    return top == capacity - 1;     // or return size() == capacity;
}
```

<div class= "tip">   <div class="tip-header">📓 📝 😎 tip</div> <br />  <br />
 That's it, if you found any typos or anything that could be improved, pleased make a PR to <a href ="https://github.com/snow-blade/jekyll_blog">this github repo </a> by going to the _posts folder and adding your change to the cplusplus-ffi-python.md file</div>

