# Exercise 1-3
> Modify the temperature conversion program to print a heading above the table. 

## Process
Straight forward, just added a `printf` statement. Interesting that float conversion is implicit though...
## Final Code
```c
#include <stdio.h>

main() {
    float fahrenheit, celcius;

    int max = 300;	/* upper limit of conversion */
    int min = 0;	/* lower limit of conversion */
    int step = 10;	/* step value of conversion */

    fahrenheit = min; /* initialise NOTE: float conversion implicit */

    printf("\nTitle:\tFahrenheit to Celcius conversion program\n");

    while (fahrenheit <= max) {
        celcius = (5.0 / 9.0) * (fahrenheit - 32);
        printf("%3.0f\t%6.1f\n", fahrenheit, celcius);
        fahrenheit = fahrenheit + step;
    }
}
```

## Output
<image src="../assets/exercise1-3output.jpg" alt="output for exercise1-3" height=200 />

[Back to Main](../readme.md)
