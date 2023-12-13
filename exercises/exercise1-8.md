# Exercise1-8
> Write a program to count blanks, tabs, and newlines.

## Process
Straight forward
## Edit - 14 Dec 2023
Going back through this program, I noticed the doubles are not initialised to zero, which could result in junk values giving a wrong result, so fixed that.

## Final Code
```c
#include <stdio.h>

int main() {
	int c;

	double newlineCount = 0.0;
	double tabCount = 0.0;
	double blankCount = 0.0;

	while((c = getchar()) != EOF) {
		if ( c == '\n') {++newlineCount;};
		if ( c == '\t') {++tabCount;};
		if ( c == ' ' ) {++blankCount;};
	}

	printf("newlines: %.0f, tabs: %.0f, spaces: %.0f", newlineCount, tabCount, blankCount);
	return 0;
}
```

## Output
<p align="center">
    <image src="../assets/exercise1-8_output1.jpg" alt="output for exercise1-8" />
</p>
<p align="center">
    <image src="../assets/exercise1-8_output2.jpg" alt="output for exercise1-8" />
</p>

[Back to Main](../readme.md)
