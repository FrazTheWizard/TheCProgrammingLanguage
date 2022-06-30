# Exercise1-10
> Write a program to copy its input to its output, 
> replacing each tab by *\t*, each backspace by *\b* and each backslash by *//*. 
> This makes tabs and backspaces visible in an unambiguous way.

## Process

Fairly simple task

It seems the concept of 'backspace' is different than I assumed. I wasn't able to input it directly to the program 
as it gets consumed by the console as a direct command

Small test to illustrate how it works as output from a C program:
```c
printf("abc");
printf("\b\b\b");
printf("\n");
printf("123");
    
printf("\n----\n");

printf("abc");
printf("\b\b\b");
printf("123");
```
<p align="center">
    <image src="../assets/exercise1-10_interesting.jpg" alt="interesting for exercise1-10" />
</p>
The backspace command moves the cursor back a step, but doesn't replace it with a blank, which was my assumption...

## Final Code
```c
#include <stdio.h>

int main(){
	int c;
	while ((c = getchar()) != EOF) {
		int escaped = 0;

		if (c == '\t') {
			putchar('\\');
			putchar('t');
			escaped = 1;
		}

		if (c == '\b') {
			putchar('\\');
			putchar('b');
			escaped = 1;
		}

		if (c == '\\') {
			putchar('\\');
			putchar('\\');
			escaped = 1;
		}

		if (escaped == 0) {
			putchar(c);
		}
		escaped = 0;
	}
	return 0;
}
```
## Output

In order to get the backspace command into the program, I used another C program for the input
```c
#include <stdio.h>

int main(){
    printf("This is some sample data\n");
    
    putchar('\t'); /* tab */
    putchar('\n');
    
    putchar('\b'); /* backspace */
    putchar('\n');
    
    putchar('/');  /* backslash */
    
    return 0;
}
```

<p align="center">
    <image src="../assets/exercise1-10_output.jpg" alt="output for exercise1-10" />
</p>

[Back to Main](../readme.md)
