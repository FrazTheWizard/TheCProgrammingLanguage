# Exercise 3-4

> In a two's complement number representation, our version of itoa does not handle the largest negative number, that is,
> the value of n equal to -(2^wordsize-1). Explain why not. Modify it to print that value correctly,
> regardless of the machine on which it runs.

## Process
I've been playing around with gnu debugger or gdb. I've found it really helpful in my understanding on what is actually going on with the code, 
where I can investigate the inner workings and memory usage. I found [this](https://www.youtube.com/watch?v=wIuZajISL-E) video helpful in finally 
understanding it and how to get assembly working in windows cmd line.


At first with this exercise I had no idea how to tackle it or even what it was asking. Interestingly I had just decided to try to understand signed
variables this morning, so I was already onto the subject. Of course [Professor Brailsford](https://www.youtube.com/watch?v=lKTsv6iVxV4&pp) and
[Ben Eater](https://www.youtube.com/watch?v=4qH4unVtJkE&pp) had the best explanations for twos complement.

## Solution 1 - my attempt
The reason it doesn't get the lowest negative number is because when it does the positive conversion, the number wraps around to the
highest positive value, which is -1 lower than it needs to be, due to twos complement.

In order to solve this, I used a method from a previous exercise 2-1, which uses limits.h and the INT_MIN constant to find the minimum value.

Subtract 1 from the number to convert to char then add 1 to the lowest individual char value.

Of course, I suppose this wouldn't work on a system that had a minimum value ending in 9 because it would need to carry the one to the next number.
But this solution seems to be enough for me..

This exercise took an afternoon of mostly research and playing around.

## Solution 2 - after looking at other solutions
Came across this [solution](https://www.youtube.com/watch?v=Aj3hOHe-JqI) that just implemented a cast to unsigned int, which sent me down a rabbit hole, trying to understand. 

```c
do {     /* generate digits in reverse order */
    s[i++] = (unsigned) n % 10 + '0';     /* get next digit */
} while ((n = (unsigned) n / 10) > 0);    /* delete it */
```

I think it's a much better solution than mine as its shorter and also is more targeted to the actual problem or signed and unsigned conversion.

Also, found this [explanation](https://onlinetoolz.net/unsigned-signed) helpful to understanding

Basically the unsigned cast has no effect on any number that get's flipped in the line `n = -n` because those numbers are positive. The only number it will change the result is on the largest negative number `-2147483648` which will turn it into the positive number that is needed to make the correct conversion.

## Note
Also, had a confusing problem where the result was changing randomly after it had been calculated and discovered that the array length I created for the outputs didn't account for the `'\0'` null string termination character at the end and they overflowed into other arrays.

## Code - my attempt
```c
#include <stdio.h>
#include <string.h>
#include <limits.h>

/* reverse: reverse string s in place */
void reverse(char s[])
{
    int c ,i ,j;
    
    for (i = 0, j = strlen(s) -1; i < j; i++, j--) {
        c = s[i];
        s[i] = s[j];
        s[j] = c;
    }
}

/* itoa: convert n to characters in s */
void itoa(int n, char s[])
{
    int i, sign, foundmin;
    foundmin = 0;

    if ((sign = n) < 0) /* record sign */
    {
        if(n == INT_MIN){ /* reach lowest number */
            n = n + 1;
            foundmin = 1;
        }        
        n = -n;         /* make n positive */
    }
        
    i = 0;
    do {        /* generate digits in reverse order */
        s[i++] = n % 10 + '0';  /* get next digit */
        if(foundmin == 1){ 
            s[i-1] = s[i-1] + 1; /* add 1 to the char number */
            foundmin = 0;
        }            
    } while ((n /= 10) > 0);    /* delete it */
    
    if (sign < 0)
        s[i++] = '-';
    s[i] = '\0';
    reverse(s);
}

int main(){
    int num = -2147483648;
    char output[12];
    int i;
    itoa(num, output);
    
    for(i=0;output[i]!='\0';i++){
        putchar(output[i]);
    }
}
```

### Code 2 - after looking at other solutions
```c
/* itoa: convert n to characters in s */
void itoa(int n, char s[])
{
    int i, sign;

    if ((sign = n) < 0) /* record sign */ 
        n = -n;         /* make n positive */
        
    i = 0;
    do {        /* generate digits in reverse order */
        s[i++] = (unsigned) n % 10 + '0';  /* get next digit */
    } while ((n = (unsigned) n / 10) > 0);    /* delete it */
    if (sign < 0)
        s[i++] = '-';
    s[i] = '\0';
    reverse(s);
}
```

## Output
<p align="center">
    <image src="../assets/exercise3-4.jpg" alt="output for exercise3-4" />
</p>

[Back to Main](../readme.md)
