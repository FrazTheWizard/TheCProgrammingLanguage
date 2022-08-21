# Exercise 2-9

> In a two's complement number system, `x &= (x-1)` deletes the rightmost 1-bit in `x`. Explain why. Use this observation to write faster version of bitcount.


## Process
An interesting case which took me two days

Playing around with numbers in windows calculator and using some python for loops I found that:

`x - 1` turns the right most 1-bit to `0` and everything lower than it to `1`

<br>

So when `&`ing that number with the original number would 
 - leave the right most 1-bit at `0`
 - turn all bits to the right of the right most 1-bit to `0`

This is how it deletes the right most 1-bit


The bitshifting method described in the book compares all bit positions whereas this method skips any 0-bits. This speeds up the process for numbers with many 0-bits.



## Final Code

```c
#include <stdio.h>

int fastbitcount(unsigned x);

int main(void) {    
        
    printf("Decimal: 183\nBinary: 1011 0111\nNum of 1-bits: %d\n\n",
        fastbitcount(183)
    );
    
    printf("Decimal: 1669\nBinary: 0110 1000 0101\nNum of 1-bits: %d ", 
      fastbitcount(1669)
    );
    
    return 0;
}

/* count 1 bits in x using & method*/
int fastbitcount(unsigned x)
{
    int b = 0;
    
    while (x != 0)
    {
        x &= (x-1);
        b++;
    }
    return b;
}

```

## Output

<p align="center">
    <image src="../assets/exercise2-9.jpg" alt="output for exercise2-9" />
</p>

[Back to Main](../readme.md)
