# Exercise 1-1

> Run the `"hello, world"` program on your system. Experiment with leaving out parts of the program, to see what error messages you get.

## Process
A glorious start, simple stuff. 
One thing that did catch me out was using the `-ansi` flag on the gcc compilier so it can compile the `main` function without the `int` return type as stated in the book

## Final Code
```c
#include <stdio.h>

main() {
    printf("hello, world\n");
}
```
[Back to Main](../readme.md)
