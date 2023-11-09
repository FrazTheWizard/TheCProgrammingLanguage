# Exercise 4-12

> Adapt the ideas of printd to write a recursive version of itoa; that is, convert an integer into a string by calling a recursive routine.

## Process
This one was kind of fun!

I felt my brain moaning when I had to go back to the `itoa` function, I think it want's to go onto new and exciting things. However I actually appreciate going back
because it's a good learning experience to really make sure I know it. **Integer** to **Char** conversion seems like a pretty basic idea that you should be able to do easily
and it still takes me a little bit to figure it out

Initally implemented the recurvise routine with a static `i` variable and then checked some other solutions when mine was working.

Realised that a static variable won't be able to handle multiple calls, because the static variable is never reset.
One [solution](https://github.com/zer0325/KnR-Exercise-Solution/blob/master/Chapter/04/Exercise/4-12.c) was to make the variable global, 
but I didn't like having a global `i` floating around. So combined two solutions from [clc-wiki](https://clc-wiki.net/wiki/K%26R2_solutions:Chapter_4:Exercise_12)
in using a way to detect inital recursive call without counting; and then separating it to an exclusive variable.

I also took my previous solution from [Exercise 3-4](./exercises/exercise3-4.md) to handle `INT_MIN`

## Code
```c
#include <stdio.h>
#include <limits.h>
#define MAX_SIZE 20

void reverse(char s[]);
void itoa_recursive(int n, char s[]);

int main()
{
    char s[MAX_SIZE];
    
    itoa_recursive(123, s);
    printf("%s\n", s);
    
    itoa_recursive(-456, s);
    printf("%s\n", s);
    
    itoa_recursive(INT_MAX, s);
    printf("%s\n", s);
    
    itoa_recursive(INT_MIN, s);
    printf("%s", s);
    
    return 0;
}

void itoa_recursive(int n, char s[])
{
    static int i;
    static int calls = 0;   /* to handle inital recursive call */
    
    if (calls == 0) {       /* inital recursive call */
        i = 0;
        if(n < 0) {             /* handle negative number */
            n = -n;
            s[i++] = '-';
        }    
        calls = 1;
    }
    
    /* unsigned cast to handle INT_MIN */
    if(n / 10)
        itoa_recursive((unsigned) n / 10, s);
    s[i++] = ((unsigned) n % 10) + '0';
    s[i] = '\0';
    
    calls = 0;              /* reset recursive call flag */
}
```

## Output
<p align="center">
  <image src="../assets/exercise4-12.jpg" alt="output for exercise4-12" />
</p>

[Back to Main](../readme.md)
