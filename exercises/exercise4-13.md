# Exercise 4-13

> Write a recursive version of the function reverse(s), which reverses the string s in place

## Process
Used the same process as the previous exercise of a static variable that is initialised on the first call

I like this process because 
- it maintains the function signature of `reverse(s)` and doesn't require any external variables. 
- it can just be called by the user without noticing a difference from the original function
- it's fairly procedural with a little amount branching, which I think makes it pretty easy to read and understand

A downside might be that it's longer, but maybe that's ok?

## Code
```c
#include <stdio.h>
#include <string.h>

void recursive_reverse(char s[])
{
    static int i, j, calls;
    char temp;
    
    /* inital call */
    if(calls == 0) {
        i = 0, j = strlen(s) - 1;
        calls = 1;
    }
    
    /* swap */
    temp = s[j];
    s[j] = s[i];
    s[i] = temp;
    
    i++, j--;
    
    if(i < j)
        recursive_reverse(s);
    
    calls = 0; /* reset */
}

int main()
{
    char str[] = "hello world";
    printf("%s\n", str);
    recursive_reverse(str);
    printf("%s\n", str);
    recursive_reverse(str);
    printf("%s", str);
    return 0;
}
```

## Output
<p align="center">
  <image src="../assets/exercise4-13.jpg" alt="output for exercise4-13" />
</p>

[Back to Main](../readme.md)
