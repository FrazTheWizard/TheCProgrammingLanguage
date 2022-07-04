# Exercise 1-15
> Rewrite the temperature conversion program of Section 1.2 to use a function for conversion

## Process
--
## Final Code
```c
#include <stdio.h>

#define MIN  0
#define MAX  300
#define STEP 10

float fahrToCel(int fahr);

int main(){
	float i;

	printf("Fahrenheit to Celcius table using function");
	for (i = MIN; i <= MAX; i = i + STEP)
		printf("\n%3.0ff  %6.1fc", i, fahrToCel(i)); 
    return 0;
}


float fahrToCel(int fahr){
	return (5.0 / 9.0) * (fahr - 32.0);
}
```

## Output
<p align="center">
    <image src="../assets/exercise1-15_output.jpg" alt="output for exercise1-15" />
</p>


[Back to Main](../readme.md)
