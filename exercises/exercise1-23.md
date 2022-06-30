# Exercise 1-23
> Write a program to remove all comments from a C program. 
> Don't forget to handle quoted strings and character constants properly.
> C comments do not nest.

## Process
This took about half an hour. I wrote the logic on paper, then wrote the code and it worked first time.
Not sure if I understand the question properly, I feel like it should have been harder, 
based on the difficulty I had with the previous questions...

But maybe it was just an easy question and I should feel good about it's simplicity 😀

The only thing I might add to improve this program would be to remove the blank line left
after removing a multicomment

## Final Code
```c
#include <stdio.h>

int main() {
	int c;
	int multiline, singleline, foundasterix, foundslash;
	while((c=getchar())!=EOF) {
		if(multiline==1) {
			if(c=='*')
				foundasterix=1;
			else {
				if(c=='/' && foundasterix==1)
					multiline=0;
				else
					foundasterix=0;
			}
		}
		else {
			if(singleline==1) {
				if(c=='\n')
					singleline=0;
			}
			else {
				if(c=='/') {
					if(foundslash==1) {
						singleline=1;
						foundslash=0;
					}
					else
						foundslash=1;
				}
				else {
					if(c=='*') {
						if(foundslash==1){
							multiline=1;
							foundslash=0;
						}
					}
					else {
						putchar(c);
					}
				}
			}
		}
	}
	return 0;
}
```

## Output
<p align="center">
    <image src="../assets/exercise1-23_output1.jpg" alt="output for exercise1-23" />
</p>

<p align="center">
    <image src="../assets/exercise1-23_output2.jpg" alt="output for exercise1-23" />
</p>

[Back to Main](../readme.md)
