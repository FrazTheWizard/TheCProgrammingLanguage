# Exercise 2-3

> Write a function `rightrot(x,n)` that returns the value of the integer `x` rotated to the right by `n` bit positions


## Process
This one took me a few weeks due to work commitments and needing to wrap my head around what the question actually meant.
In the end it was good, because I have a much greater understanding of bitshifting and unsigned/signed properties

I didn't even realise what rotation meant and found this [video](https://www.youtube.com/watch?v=2QhqZ4zJFYA) quite helpful

I also found this [explanation](https://arduinogetstarted.com/reference/arduino-bitshift-right) helpful

**Assumption 1** - the machine is using 32 bit unsigned integers

**Assumption 2** - we are using unsigned ints


## Final Code

```c
#include <stdio.h>

#define RIGHTROTBIT ~(~0U >> 1) // 1 in left most place in unsigned int 32

unsigned int rightrot(unsigned int x, int n);

int main(void) {
    
    /**
     *  21 Decimal = 0001 0101 Binary
     *  RightRotated by 4 -> ‭01010000000000000000000000000001‬
     *  Which is the Decimal value = ‭1,342,177,281‬
     */
    printf("%d\n", rightrot(21,4)); 
    
    /**
     *  11 Decimal = 0101 Binary
     *  RightRotated by 3 -> ‭‭01100000000000000000000000000001‬‬
     *  Which is the Decimal value = ‭‭1,610,612,737‬
     */
    printf("%d\n", rightrot(11,3));
    
    return 0;
}

unsigned int rightrot(unsigned int x, int n) {
    
    unsigned int rotated = x;
    
    /* rotation loop */
    for (int i = 0; i < n; i++) {
        
        /* if least significant bit is 0, bitshift as normal */
        if ((rotated & 1) == 0){
            rotated = rotated >> 1;
        }
        
        /* if least significant bit is 1, bitshift as normal 
           and add a 1 at most significant bit position */
        else {
            rotated = rotated >> 1;
            rotated = rotated | RIGHTROTBIT;
        }
    }
    
    return rotated;
}
```


## Output

<p align="center">
    <image src="../assets/exercise2-8.jpg" alt="output for exercise2-8" />
</p>

[Back to Main](../readme.md)
