# Exercise 5-2

> Write **getfloat**, the floating-point analog of **getint**. What type does **getfloat** return as its function value?

## Process
In direct answer to the question, I think `getfloat` should also return an `int` (same as `getint`). 
The value `getfloat` returns won't be meaningful, just as a way to capture `EOF`, so I have modified it to always return `0` unless it finds `EOF`.


I tried to stay within the `getint` style of program structure, but didn't find any way that I found worked. Decided to restructure the function to something that makes sense to me.

### New `getfloat` function structure
1. loop over each input character until found something we want to process
2. if found EOF return from function early
3. loop over each input character until we are happy that we have a valid chunk of input
4. convert chunk of input to a float using `atof` from `stdlib.h` as shown in previous Reverse Polish Calculator exercises
5. put float in array

### Note
Carried over idea from previous exercise so that `getfloat` does not consider `'+'` or `'-'` as a `0`
Also, included that `getfloat` can now discard random input characters (as I have shown in output section), which was bugging me

## Code
```c
#include <stdio.h>
#include <ctype.h> /* for isspace(), isdigit() */
#include <stdlib.h> /* for atof() */
#define SIZE 6

int getfloat(double *);

int main()
{
        int n, getfloat(double *);
        double array[SIZE];
        
        for (n = 0; n < SIZE && getfloat(&array[n]) != EOF; n++)
            ;
        
        for (int i = 0; i < SIZE; i++)
            printf("%d: %.3f\n", i,  array[i]);
}

int getch(void);
void ungetch(int);
#define MAXNUM 100

int getfloat(double *pn)
{
    int i = 0, c;
    char s[MAXNUM];
    
    int found = 0;
    while(!found) /* input validation loop */
    {
        s[0] = c = getch();             /* get next input character */
        
        if (isdigit(c)) {               /* found digit */
            found = 1;
        }
        else if (c == '-') {            /* found '-' */
            if(isdigit(c = getch())) {  /* is next char digit */
                s[i = 1] = c;
                found = 1;
            } else {
                ungetch(c);
            }
        }
        else if (c == EOF) {            /* found end of file */
           found = 1;
        }
    }
    
    if (c == EOF) /* exit early */
        return c;
    
    while (isdigit(s[++i] = c = getch()))      /* collect integer part */
            ;
    if (c == '.')
        while (isdigit(s[++i] = c = getch()))  /* collect fraction part */
            ;
    s[i] = '\0';
    
    *pn = atof(s);
    
    ungetch(c);
    return 0;
}

#define BUFSIZE 100

char buf[BUFSIZE];  /* buffer for ungetch */
int bufp = 0;       /* next free position in buf */

int getch(void) /* get a (possibly pushed back) character */
{
    return (bufp > 0) ? buf[--bufp] : getchar();
}

void ungetch(int c) /* push character back on input */
{
    if (bufp >= BUFSIZE)
        printf("ungetch: too many characters\n");
    else
        buf[bufp++] = c;
}
```

## Output
<p align="center">
      <image src="../assets/exercise5-2_working.jpg" alt="output for exercise5-2" />
</p>

[Back to Main](../readme.md)
