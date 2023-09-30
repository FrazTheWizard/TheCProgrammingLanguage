# Exercise 3-5

> Write the function itob(n,s,b) that converts the integer n
> into a base b character representation in the string s.
> In particular, itob(n,s,16) formats n as a hexadecimal integer in s.

## Process
This one took about a week to wrap my head around number base conversion. 
Found a bunch of helpful youtube videos, but found this [article](https://betterexplained.com/articles/understanding-exponents-why-does-00-1/)
to be the thing that helped complete the loop in my understanding. And as usual my tactic is to step through the code using gdb and checking
my answers with windows calculator in programmer mode.

I started with a brand new solution (Code 1) to the problem which finds the largest number and counts down, like I'd seen how to 
convert between number bases in YouTube videos. I thought it was finished, but then checked this guys [solution](https://www.youtube.com/watch?v=87M-Lz1InAc)

I realised my solution didn't handle negative numbers. In fact I didn't catch how similar this exercise was to the previous exercise 3.4

So I've basically copied his code for the solution (code 2)

In the beginning I didn't understand his code, but since then I've boiled my understanding of `itob()` to be:

```
Is negative? Then turn positive

do {
  get next char
} while (number not finished? Then slide down number using base)
       
Was negative sign? Then add negative sign back
```





## Code 1
```c
int itob(int n, char s[], int b)
{
    int divisor, i, j, sign;
    
    if ((sign = n) < 0) /* record sign */ 
    {
        n = -n;         /* make n positive */
        s[0] = '-';
        j = 1;
    } else j = 0;
    
    for (i = 1; i < INT_MAX; i*= b)
    {
        if (n / i == 1) break;
        if (n / i < 1)
        {
            i /= b;
            break;
        }
    }
        
    for (j = 0; i > 0; i /= b)
    {
        divisor = n / i;
        n = n % i;
        if (divisor < 10)
            s[j] = divisor + '0';
        else
            s[j] = divisor - 10 + 'A';
        j++;
    }
    s[j] = '\0';
}
```

## Code 2
```c
#include <stdio.h>
#include <string.h>
#include <limits.h>
#define size 30

int itob(int, char[], int);
void reverse(char s[]);

int main()
{
    int n;
    char s[size];
    for (int b = 2; b <= 16; b++)
    {
        n = INT_MIN;
        itob(n, s, b);
        printf("%d in base %d is %s\n", n, b, s);    
    }    
}

int itob(int n, char s[], int b)
{
    int i, sign, r;

    if ((sign = n) < 0 && b == 10) /* record sign */ 
        n = -n;         /* make n positive */
        
    i = 0;
    do {        /* generate digits in reverse order */
        r = (unsigned) n % b;  /* get next digit */
        switch (r)
        {
            case 10: s[i++] = 'A'; break;
            case 11: s[i++] = 'B'; break;
            case 12: s[i++] = 'C'; break;
            case 13: s[i++] = 'D'; break;
            case 14: s[i++] = 'E'; break;
            case 15: s[i++] = 'F'; break;
            default: s[i++] = r + '0';
        }
    } while ((n = (unsigned) n / b) > 0);    /* delete it */
    
    if (sign < 0 && b == 10)
        s[i++] = '-';
    s[i] = '\0';
        reverse(s);
}

/* reverse: reverse string s in place */
void reverse(char s[])
{
    int c, i, j;
    for (i = 0, j = strlen(s) -1; i < j; i++, j--) {
        c = s[i];
        s[i] = s[j];
        s[j] = c;
    }
}
```
## Output
<p align="center">
    <image src="../assets/exercise3-5.jpg" alt="output for exercise3-5" />
</p>

[Back to Main](../readme.md)
