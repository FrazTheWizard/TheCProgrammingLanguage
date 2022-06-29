# Exercise 1-22
> Write a program to "fold" long input lines into two or more shorter lines 
> after the last non-blank character that occurs before the *n*-th column of input. 
> Make sure your program does something intelligent with very long lines, 
> and if there are no blanks or tabs before the specified column.

## Process
This took two days. I spent the first day noodling around with the problem and thinking of possible solutions. 
The second day I sat down and wrote the basic idea of the progam on paper, then actually wrote the program and fixed the little problems.

About the 'intelligent' way of dealing with long lines, I figured the program would allow 5 Max size columns then just spit out an error message

## Final Code
```c
#include <stdio.h>
#define COLSIZE 20
#define MAXCOLS 5

int main() {
	int c, index, spaceindex, colcounter;
	char s[COLSIZE];
	index = 0;
	spaceindex = -1;
	colcounter = 0;
	
	while((c=getchar()) != EOF) {
		s[index] = c;
		
		if(index == (COLSIZE-1)) { /* end of column */
		
			if(spaceindex == -1) {
				for(int i=0; i<COLSIZE; i++)
					putchar(s[i]);
				putchar('\n');
				index = 0;
				colcounter++;
				
				if(colcounter==MAXCOLS) {
					printf("Error: Line too long");
					return 0;
				}	
			}
			else {
				for(int i=0; i<spaceindex; i++)
					putchar(s[i]);
				putchar('\n');
				for(int i=spaceindex+1; i<COLSIZE; i++)
					putchar(s[i]);
				index = 0;
				spaceindex = -1;
				colcounter = 0;
			}
		}
		else {
			if(c == '\n') {
				for(int i=0; i<=index; i++)
					putchar(s[i]);
				index = 0;
				spaceindex = -1;
				colcounter = 0;
			}
			else {
				if(c==' ' || c=='\t')
					spaceindex = index;	
				index++;
			}
		}		
	}
  
  /* 
  this last part is to handle if there are any characters left over on the last line 
  because the while loop terminates on EOF without printing the characters stored in the array
  I feel there has to be a better way to handle this, but not sure how...  
  */
	if(index==0 && c=='\n') { 
		/* no characters left over */
	} else {
		for(int i=0; i<index;i++)
			putchar(s[i]);	
	}
	
	return 0;
}
```

## Output
<p align="center">
    <image src="../assets/exercise1-22_outputone.jpg.jpg" alt="output for exercise1-22" />
</p>

<p align="center">
    <image src="../assets/exercise1-22_output2.jpg" alt="output for exercise1-22" />
</p>

[Back to Main](../readme.md)
