# Exercise 1-9
> Write a program to copy its input to its output, replacing each string of one or more blanks by a single blank.

## Process
Another simple one

Count amount of blanks in a sequence, then reset. If sequence is below 1, print current character

## Final Code
```c
#include <stdio.h>

int main(){
	int c;
	int spaceCount = 0;
	
	while((c = getchar()) != EOF){
		if (c == ' ')
			++spaceCount;
		else
			spaceCount = 0;
		
		if (spaceCount <= 1)
			putchar(c);
	}
	return 0;
}
```
## Output
<p align="center">
    <image src="../assets/exercise1-9_output1.jpg" alt="output for exercise1-8" />
</p>
<p align="center">
    <image src="../assets/exercise1-9_output2.jpg" alt="output for exercise1-8" />
</p>

[Back to Main](../readme.md)
