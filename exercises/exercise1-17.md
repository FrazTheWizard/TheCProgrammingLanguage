# Exercise 1-17
>  Write a program to print all input lines that are longer than 80 characters. 

## Process
This was an interesting one

In my solution I opted to go for slightly more verbose and, in my opinion, straightforward readable process. As the main validation process seemed very related to the last exercise
I also decided to keep the `getline` function as it seemed fitting for an exercise from this chapter as well as I think it adds to the idea of readability, re-using a function that the reader has likely seen before

Some other decisions:
- code follows a chronological use case with a switch to unambiguously show we are only concerned with `overmaxlen` state
- `FALSE` and `TRUE` are less ambiguous to read intention then `0` and `1`
- program should theoretically print infinitely long lines, by printing in chunks

I liked ideas in two other [solutions](https://clc-wiki.net/wiki/K%26R2_solutions:Chapter_1:Exercise_17)

**Solution by arnuld**
- just modified the previous exercise slightly, which means less new concepts for the reader

**Solution by lgmlnvsk**
- didn't contain any other functions, sort of minimal solution
- commented every 'obvious' line intention

Lastly there is a check after the while loop to see if the file ended without a newline character but still has not printed the final char count

## Final Code
```c
#include <stdio.h>

#define MAXCHARLENGTH 80 /* not including newline and null terminator */
#define FALSE   0
#define TRUE    1

int getline(char line[], int limit);

int main()
{
    char line[MAXCHARLENGTH + 2]; /* +2 for newline and null terminator */
    int len, totallen = 0, overmaxlen = FALSE;
    
    while((len = getline(line, MAXCHARLENGTH + 2)) != 0){
        switch (overmaxlen) {
            case FALSE:
                /* if line is max length and last char is not newline */
                if(len == (MAXCHARLENGTH+1) && (line[len-1] != '\n')){
                    printf("%s", line); /* print line chunk */
                    totallen = len;     /* begin running total */
                    overmaxlen = TRUE;  /* next line will be part of this line */
                }
                break;
            case TRUE:
                printf("%s", line); /* print line chunk */
                totallen += len;    /* add to running total */  
                if(line[len-1] == '\n'){ /* reached end of extended line */
                    overmaxlen = FALSE; /* end line gathering */
                    printf("line length: %d\n", totallen-1); /* printout */
                }            
                break;
        }
    }
	
	/* if last line did not have a '\n' as the last character */
	if (len == 0 && overmaxlen == TRUE)
		printf("\nline length: %d\n", totallen); /* printout */
        
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
    <image src="../assets/exercise1-17_a.jpg" alt="output for exercise1-17" />
</p>
        
<p align="center">
<image src="../assets/exercise1-17_b.jpg" alt="output for exercise1-17" />
</p>



[Back to Main](../readme.md)
