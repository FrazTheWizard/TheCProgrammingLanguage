# Exercise 1-6
> Verify that the expression **getchar() != EOF** is 0 or 1.

## Process

Used CRTL + Z to type EOF character which looks like this: ^Z

## Final Code
```c
#include <stdio.h>

int main(){
  int value = getchar() != EOF;
  printf("%d", value);
}
```

## Output
Ran program twice. First time input a random character which returns true. Second time input EOF character value, which returns false.
<p align="center">
    <image src="../assets/exercise1-6_output.jpg" alt="output for exercise1-6" />
</p>

[Back to Main](../readme.md)
