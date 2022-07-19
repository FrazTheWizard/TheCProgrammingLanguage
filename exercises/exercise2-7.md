# Exercise 2-7

> Write a function `invert(x,p,n)` that returns `x` with the `n` bits that begin at position `p` 
> inverted (i.e., `1` changed into `0` and vice versa), leaving the others unchanged.


## Process
Having spent week wrapping my head around the last exercise, this one took under an hour which is cool.
It seems there are some common ideas with Bitwise operations. Masking with Nots/Bitshifting and Anding seem to be cool tricks to merge bits.

As a side note, I've really found the windows calculator in programmer mode to be really helpful in walking through potential solutions

## Final Code

```c
#include <stdio.h>

unsigned int invert(unsigned int x, int p, int n);

int main(void) {
    
    printf("%d\n",invert(181, 3, 3));
    // 181: 1011 0101‬
    // 187: 1011 1011
    
    printf("%d",invert(2581, 7, 4));
    //‭ 2581: 1010 0001 0101
    //‭ 2789: 1010 1110 0101‬
    
    return 0;
}

unsigned int invert(unsigned int x, int p, int n) {
    int flippedBits = ~x & ((~(~0 << n)) << (p + 1 - n));
    int maskedX = x & ~(~(~0 << n) << (p + 1 - n));
    return flippedBits | maskedX;
}

```


## Output

<p align="center">
    <image src="../assets/exercise2-7_output1.jpg" alt="output for exercise2-7" />
</p>
<p align="center">
    <image src="../assets/exercise2-7_output2.jpg" alt="output for exercise2-7" />
</p>

[Back to Main](../readme.md)
