# Exercise 1-4
> Modify the temperature conversion program to print the table in reverse order, that is, from 300 degrees to 0.

## Process

Same temperature conversion program as previous program, but using for loop and in reverse

## Final Code
```c
#include <stdio.h>

int main(){
    int fahr;

    for(fahr = 300; fahr >= 0; fahr = fahr - 10) {
        printf("%3d\t%6.1f\n", fahr, (5.0/9.0) * (fahr - 32) );
    }
}
```

## Output
<p align="center">
    <image src="../assets/exercise1-5_output.jpg" alt="output for exercise1-5" />
</p>

[Back to Main](../readme.md)
