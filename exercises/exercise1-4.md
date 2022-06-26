# Exercise 1-4
> Write a program to print the corresponding Celsius to Fahrenheit table

## Process
Just continued on from the previous exercise, nothing complicated really..

## Final Code
```c
#include <stdio.h>

main() {
    float celcius, fahrenheit;

    int max = 300;	/* upper limit of conversion */
    int min = 0;	/* lower limit of conversion */
    int step = 10;	/* step value of conversion */

    celcius = min; /* initialise NOTE: float conversion implicit */

    printf("\nTitle:\tCelcius to Fahrenheit conversion program\n");
    printf("Date:\t29 Sep 2021\n");

    while (celcius <= max) {
        fahrenheit = (celcius / (5.0 / 9.0)) + 32;
        printf("%3.0f\t%6.1f\n", celcius, fahrenheit);
        celcius = celcius + step;
    }
}
```

## Output
<p align="center">
    <image src="../assets/exercise1-4_output.jpg" alt="output for exercise1-4" />
</p>

[Back to Main](../readme.md)
