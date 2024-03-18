# Exercise 5-1

> As written, getint treats a + or - not followed by a digit as a valid representation of zero. Fix it to push such a character back on the input.

## Process
Found this question a little confusing at first. I think the question sort of assumes I know how the rest of the program goes together and what kind of input the program will take, without specifying it. 
Maybe back in Unix days this was an obvious program, but it's not obvious to me.

Like if the program receives an input character that isn't `isspace`, `isdigit`, `+`, `-` or `EOF` the program goes into an infinte loop of `getch` <-> `ungetch` till the end of the array. 
So I guess we are assuming the input is going to be clean?

My assumption is that the program is handling input, maybe piped in from another file, delimited by unknown amount of spaces, with some numbers prefixed with `-` or `+`.

Something like this
```
33 55    77  -22   +66 + 22 88 - 789
```

I have decided to just solve the problem as stated and not handle any cases such as `+ ++3` which would still create a `0` array character before the `3`.

## Code
```c
#include <stdio.h>
#include <ctype.h>
#define SIZE 8

int getint(int *pn);

int main()
{
        int n, array[SIZE], getint(int *);
        
        for (n = 0; n < SIZE && getint(&array[n]) != EOF; n++)
            ;
        
        for (int i = 0; i < SIZE; i++)
            printf("%d: %d\n", i,  array[i]);
}

int getch(void);
void ungetch(int);

/* getint: get next integer from input into *pn */
int getint(int *pn)
{
    int c, sign;
    
    while (isspace(c = getch())) /* skip white space */
        ;
    
    if(c == '+') {
        if (isdigit(c = getch())) { /* next character is digit */
            ungetch(c);
            c = '+';
        } else {
            c = getch();
        }
    }
    
    if(c == '-') {
        if (isdigit(c = getch())) { /* next character is digit */
            ungetch(c);
            c = '-';
        } else {
            c = getch();
        }
    }
        
    if (!isdigit(c) && c != EOF && c != '+' && c != '-') {
        ungetch(c); /* it's not a number */
        return 0;
    }
    sign = (c == '-') ? -1 : 1;
    if (c == '+' || c == '-')
        c = getch();
    for (*pn = 0; isdigit(c); c = getch())
        *pn = 10 * *pn + (c - '0');
    *pn *= sign;
    if(c!= EOF)
        ungetch(c);
    return c;
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
      <image src="../assets/exercise5-1.jpg" alt="output for exercise5-1" />
</p>

[Back to Main](../readme.md)
