---
title: "C++ Goodies Episode 2: Better unit tests using the Catch2 library"
author: Freeman Irabaruta
date: 2021-04-06 11:33:00 +0800
categories: [Blogging,Coding, Cpp-Goodies]
tags: [C++,C, libraries, unit-tests]
math: true
mermaid: true
image:
  src: https://raw.githubusercontent.com/snow-blade/blog/master/content/assets/lite.png
---

# Method 1 : Using cassert (The faster but harder way)

Most of the C++ programmers I know usually write their unit tests using the ``` cassert``` library but this method in not very effective as it holds many limitations.

## Simple examples

### First Case, assertion fails

Let me showcase a simple example using the cassert library then point out it's limitations.

```c++
#include <cassert>
#include <iostream>
unsigned int Factorial( unsigned int number ) {
    return number <= 1 ? number : Factorial(number-1)*number;
}
int main(){
    assert(Factorial(3)==7);
    std::cout << "Hello" << std::endl;
}
```

The assertion will fail and it will make the program stop. Line 8 won't compile as the test case is not fulfilled.

![image-20210406150436195](/home/free/.config/Typora/typora-user-images/image-20210406150436195.png)

### Second Case, assertion succeeds

Let us correct the error and then re-compile our program.

```c++
#include <cassert>
#include <iostream>
unsigned int Factorial( unsigned int number ) {
    return number <= 1 ? number : Factorial(number-1)*number;
}
int main(){
    assert(Factorial(3)==7);
    std::cout << "Hello" << std::endl;
}
```

Now our program will just print "Hello" in our prompt as the assertion is fulfilled. But we might remark that there is no message that shows us that our test cases have been fulfilled.

# Method 2: Using Catch2 (The slightly slower but better method)




## Installing

To install the Catch2 library either go to it's [Github repository](https://github.com/catchorg/Catch2/) then follow the steps in the readme or  copy these 2 files to proceed with a single-header file: 

- [https://raw.githubusercontent.com/catchorg/Catch2/devel/extras/catch_amalgamated.hpp](https://raw.githubusercontent.com/catchorg/Catch2/devel/extras/catch_amalgamated.hpp) 

- [https://raw.githubusercontent.com/catchorg/Catch2/devel/extras/catch_amalgamated.cpp](https://raw.githubusercontent.com/catchorg/Catch2/devel/extras/catch_amalgamated.hpp)

  

To create a new project, create a folder which contains a file named test.cpp, then copy the 2 header files into the folder. Then if you want to compile the test file, run: ``` g++ test.cpp catch_amalgamated.cpp -o test ``` 

## Simple examples

### The assertion fails

```c++
#include <iostream>
#include <string>
#include "catch_amalgamated.hpp"

unsigned int Factorial( unsigned int number ) {
    return number <= 1 ? number : Factorial(number-1)*number;
}

TEST_CASE( "Factorials are computed", "[factorial]" ) {
    REQUIRE( Factorial(1) == 1 );
    REQUIRE( Factorial(2) == 2 );
    REQUIRE( Factorial(3) == 8 );
    REQUIRE( Factorial(10) == 3628800 );
}
```

The third assertion will fail, then we will get the following error message and the program will stop. We will get this beautiful and concise error message :

![image-20210406150926007](/home/free/.config/Typora/typora-user-images/image-20210406150926007.png)

### The assertion succeeds

```c++
#include <iostream>
#include <string>
#include "catch_amalgamated.hpp"

unsigned int Factorial( unsigned int number ) {
    return number <= 1 ? number : Factorial(number-1)*number;
}

TEST_CASE( "Factorials are computed", "[factorial]" ) {
    REQUIRE( Factorial(1) == 1 );
    REQUIRE( Factorial(2) == 2 );
    REQUIRE( Factorial(3) == 6 );
    REQUIRE( Factorial(10) == 3628800 );
}
```

All the tests will pass and we will get this beautiful message :

![image-20210406152103420](/home/free/.config/Typora/typora-user-images/image-20210406152103420.png)

# Conclusion

The main drawback of using catch2 as a single header file is that the compilation time will be greatly increased and you'll have to wait for a fair amount of time for the program to finish compiling. 

Also, as we can see, the logs are way more beautiful than the default and the error messages are more concise and easier to read even by non-programmers standards. 

<div class= "tip">   <div class="tip-header">📓 📝 😎 tip</div> <br />  <br />
 That's it, if you found any typos or anything that could be improved, pleased make a PR to <a href ="https://github.com/snow-blade/jekyll_blog">this github repo </a> by going to the _posts folder and adding your change to the cpp-goodies-ep-2.md file</div>

