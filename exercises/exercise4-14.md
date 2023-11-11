# Exercise 4-14

> Define a macro swap(t,x,y) that interchanges two argument of type t. (Block structure will help.)

## Process
Pretty straightforward. I quite like this macro business

## Code
```c
#include <stdio.h>

#define swap(t,x,y) t temp = y; y = x; x = temp;

int main()
{
    int x = 5;
    int y = 7;
    
    printf("x:%d y:%d\n", x, y);
    swap(int, x, y);
    printf("x:%d y:%d\n", x, y);
    return 0;
}
```

## Output
<p align="center">
  <image src="../assets/exercise4-14.jpg" alt="output for exercise4-14" />
</p>

[Back to Main](../readme.md)
