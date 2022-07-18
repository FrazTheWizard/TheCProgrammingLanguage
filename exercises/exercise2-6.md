# Exercise 2-6

> Write a function `setbits(x,p,n,y)` that returns `x` with the `n` bits that begin 
> at position `p` set to the rightmost `n` bits of `y`, leaving the other bits unchanged.


## Process
This one took about a week to wrap my head around bitwise operations. It was quite confusing in the beginning, however as I had a look
at it everyday, it started becoming more intuitive, until the solution became obvious

The idea is:
1. replace the first number `x` field of length `n` at position `p` with just zeros
2. mask out only the field of length `n` of the second number `y` and move it to position `p`
3. combine both numbers using the or operator `|`

So for instance
1. `x = 255` which is `1111 1111` in binary
2. then we say from position `p = 5` for length of `n = 4` replace with 0's which is `1100 0011` in binary
3. then grab the second number `y = 85` which is `‭01010101‬` in binary
4. mask out only the field of `n` and move over into position `p` which is `0001 0100` in binary
5. finally, combine both numbers `1100 0011` and `0001 0100` using `|`
6. final answer is `1101 0111`


## Final Code

```c
#include <stdio.h>

unsigned int setbits(unsigned int x, unsigned int p, unsigned int n, unsigned int y);

int main(void) {
    unsigned int x = 128;
    unsigned int p = 3;
    unsigned int n = 2;
    unsigned int y = 3;
    
    printf("x: %d\np: %d\nn: %d\ny: %d\n\n", x, p, n, y);
    printf("Result: %d\n\n", setbits(x, p, n, y));
    
    x = 255; p = 5; n = 6; y = 85;
    printf("x: %d\np: %d\nn: %d\ny: %d\n\n", x, p, n, y);
    printf("Result: %d\n", setbits(x, p, n, y));
    
    return 0;
}

unsigned int setbits(unsigned int x, unsigned int p, unsigned int n, unsigned int y) {
    unsigned int old = x & ~(~(~0 << n) << (p + 1 - n));    /* replace field for replacement with 0's */
    unsigned int new = (y & ~(~0 << n)) << (p + 1 - n);     /* get only rightmost field and bitshift into position */
    printf("Old: %d\nNew: %d\n", old, new);
    return old | new; /* combine old and new */
}
```


## Output

<p align="center">
    <image src="../assets/exercise2-6.jpg" alt="output for exercise2-6" />
</p>




[Back to Main](../readme.md)
