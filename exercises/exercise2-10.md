# Exercise 2-10

> Rewrite the function `lower`, which converts upper case letters to lower case,
with a conditional expression instead of `if-else`.


## Process
Wasn't too hard, took a morning. 

I used to think the ternary operator was unnessacarily complicated, but actually 
I'm coming to like it and think I might use it more.

I wrote the lower process to take an array of characters, rather than go a char at a time



## Final Code

```c
#include <stdio.h>

void lower1(char ca[], int n);
void lower2(char ca[], int n);

int main(void) {
    
    char ca[8];
    
    ca[0] = 'A'; ca[1] = 'B'; ca[2] = 'c'; ca[3] = 'F';
    ca[4] = 's'; ca[5] = 'Z'; ca[6] = 'z'; ca[7] = 'a';
    
    for (int i = 0; i < 8; i++)
      printf("%c ", ca[i]);
    
    putchar('\n');
    lower2(ca, 8);
    
    for (int i = 0; i < 8; i++)
        printf("%c ", ca[i]);
    
    return 0;
}

/* convert to lower case using if-else */
void lower1(char ca[], int n)
{
    for (int i = 0; i < n; i++)
    {
        if (ca[i] >= 'A' && ca[i] <= 'Z')
            ca[i] = ca[i] + 32;
    }
}

/* convert to lower case using ternary operator */
void lower2(char ca[], int n)
{
    for (int i = 0; i < n; i++)
        ca[i] = (ca[i] >= 'A' && ca[i] <= 'Z') ? ca[i] + 32 : ca[i];
}

```
## Output

<p align="center">
    <image src="../assets/exercise2-10.jpg" alt="output for exercise2-10" />
</p>

[Back to Main](../readme.md)
