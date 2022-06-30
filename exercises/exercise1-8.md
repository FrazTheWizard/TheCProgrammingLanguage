# Exercise1-8
> Write a program to count blanks, tabs, and newlines.

## Process
\-

## Final Code
```c
#include <stdio.h>

int main() {
	int c;

	double newlineCount;
	double tabCount;
	double blankCount;

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
