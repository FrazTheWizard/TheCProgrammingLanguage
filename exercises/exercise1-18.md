# Exercise 1-18
>  Write a program to remove trailing blanks and tabs from each line of input, and to delete entirely blank lines

## Process
Decisions about how to interpret this exercise and the constraints around the solution
- 'trailing' meaning blanks: `' '` and tabs: `'\t'` AFTER the last char in the line and BEFORE the end of the line
- 'blank lines' meaning lines consisting of only blanks and tabs and `'\n'` or lines consisting of only a `'\n'`
- the solution won't deal with lines longer than the buffer, which in this case is 80
- if the last line in the input does not end in a `'\n'`, the program will also not end in a `'\n'`

I decided to use the `getline` function from previous exercises as it seems fitting and a good way to structure the program.

There was some difficulty in cleaning up the order of operations so it reads logically.

I think a solution to handling infinitly long input lines would take a more complex approach similar to the previous exercise. 
After looking at some solutions online, the idea of the program doing multiple passes instead of trying to do it all in 1 chunk seems like a cool idea.

## Final Code
```c
#include <stdio.h>

#define MAX_LENGTH 80
#define FALSE	0
#define TRUE	1

int getline(char line[], int limit);

int main()
{
	char line[MAX_LENGTH];
	int len, i, endnewline = TRUE;
	
	while((len = getline(line, MAX_LENGTH)) != 0){
		
		/* does last input line end in newline */
		if(line[len - 1] == '\n')
			len = len - 1; /* skip newline char */
		else
			endnewline = FALSE;
		
		/* skip all trailing whitespace */
		while((line[len-1] == '\t' || line[len-1] == ' ') && (len > 0))
			len--;
		
		/* print output */
		if (len > 0) {
			for (i=0; i < len; i++)
				putchar(line[i]);
			
			if (endnewline == TRUE)
				putchar('\n');
		}
	}
	
	return 0;
}

/* getline: read a line into s, return length */
int getline(char line[], int limit)
{
    int c, i;
    
    for (i=0; i<limit-1 && (c=getchar()) != EOF && c!='\n'; ++i)
        line[i] = c;
    if (c == '\n') {
        line[i] = c;
        ++i;
    }
    line[i] = '\0';
    return i;
}
```

## Output
<p align="center">
    <image src="../assets/exercise1-18_a.jpg" alt="output for exercise1-18" />
</p>
        
<p align="center">
<image src="../assets/exercise1-18_b.jpg" alt="output for exercise1-18" />
</p>



[Back to Main](../readme.md)
