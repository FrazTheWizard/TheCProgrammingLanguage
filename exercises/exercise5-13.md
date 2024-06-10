# Exercise 5-13

> Write the program **tail**, which prints the last n lines of its input. By default, _n_ is 10, let us say, but it can be changed by an optional argument so that

>    `tail -n`
> 
> prints the last _n_ lines. The program should behave rationally no matter how unreasonable the input of the value of _n_.
> Write the program so it makes the best use of available storage; lines should be stored as in the sorting program of Section 5.6, not in a two-dimensional array of fixed size.

## Process
This exercise was a fairly straightforward process of copying functions from previous exercises and examples and linking them up together. 
Kind of cool just being able to take chunks of code already written and making something new

I took the process input function from last exercise and the readlines/writelines alloc/afree come from the previous example mentioned in the exercise description

I also modified the readlines function to allow for a final line that doesn't end with a newline character, otherwise it removes the last character

## Code
```c
#include <stdio.h>
#include <ctype.h>
#include <stdlib.h>
#include <string.h>

#define DEFAULT_N 10    /* default lines to be printed */
#define MAXLINES 5000   /* max #lines to be stored */
#define ALLOCSIZE 10000	/* size of available space */
#define MAXLEN 1000     /* max length of any input line */

int processinput(int argc, char *argv[]);
int isdigits(char *s);
int readlines(char *lineptr[], int maxlines);
void writelines(char *lineptr[], int n, int nlines);
int getline(char s[], int lim);
char *alloc(int n);
void afree(char*p);

static char allocbuf[ALLOCSIZE];	/* storage for alloc */
static char *allocp = allocbuf;		/* next free position */
char *lineptr[MAXLINES];          /* pointers to text lines */

int main(int argc, char *argv[])
{
    int nlines; /* number of input lines read */
    int tailindex; /* index to start printing lines from
    
    /* amount of lines to print */
    int n = processinput(argc-1, argv+1);
    
    /* read lines into buffer */
    if ((nlines = readlines(lineptr, MAXLINES)) >= 0) {
        
        /* if requested lines is bigger than input lines, start index is 0 */
        tailindex = nlines - n < 0 ? 0 : nlines - n;
        
        /* print n lines */
        writelines(lineptr, tailindex, nlines);

        return 0;
    } else {
        printf("error reading input");
    }
    return 0;
}

/* read args, return input if valid, else default n */
int processinput(int argc, char *argv[])
{
    /* no input argument, just go with default */
    if (argc == 0)
        return DEFAULT_N;

    char firstchar, *pnumber;
    firstchar = *argv[0];
    pnumber = (*argv)+1;    
    
    /* if valid input, return number*/
    if (firstchar == '-' && isdigits(pnumber))
        return atoi(pnumber);
    
    /* if invalid input, just go with default */
    return DEFAULT_N;
}

/* returns 1 if all chars are digits, else 0 */
int isdigits(char *s) 
{
    while (*s) 
        if(!isdigit(*s++)) 
            return 0;
    return 1;
}

/* readlines: read input lines */
int readlines(char *lineptr[], int maxlines)
{
	int len, nlines;
	char *p, line[MAXLEN];
	
	nlines = 0;
	while ((len = getline(line, MAXLEN)) > 0)
		if (nlines >= maxlines || (p = alloc(len)) == NULL)
			return -1;
		else {
            if (line[len-1] == '\n')
                line[len-1] = '\0'; /* delete newline */
			strcpy(p, line);
			lineptr[nlines++] = p;
		}
		return nlines;
}

/* read a line into s, return length */
int getline(char s[], int lim)
{
    int c, i;
    
    for (i=0; i<lim-1 && (c=getchar())!=EOF && c!='\n'; i++)
        s[i] = c;
    
    if (c == '\n') {
        s[i] = c;
        ++i;
    }
    
    s[i] = '\0';
    return i;
}

/* writelines: write last n output lines */
void writelines(char *lineptr[], int n, int nlines)
{
	int i;
	
	for (i = n; i < nlines; i++)
		printf("%s\n", lineptr[i]);
}

char *alloc(int n) /* return pointer to n characters */
{
	if (allocbuf + ALLOCSIZE - allocp >= n) { /* it fits */
		allocp += n;
		return allocp - n; /* old p */
	} else	/* not enough room */
		return 0;
}

void afree(char*p) /* free storage pointed to by p */
{
	if (p>= allocbuf && p < allocbuf + ALLOCSIZE)
		allocp = p;
}
```

## Output
<p align="center">
  <image src="../assets/exercise5-13_a.jpg" alt="output for exercise 5-13_a" />
</p>
<p align="center">
  <image src="../assets/exercise5-13_b.jpg" alt="output for exercise 5-13_b" />
</p>

[Back to Main](../readme.md)
