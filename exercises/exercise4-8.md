# Exercise 4-8

> Suppose that there will never be more than one character of pushback. Modify getch and ungetch accordingly.

## Process
Just removed the array and updated the functions.

Not sure needed to go as far as [zer0325](https://www.youtube.com/watch?v=k3PSQO1yL1s) in renaming the variables. 
I wanted it to be a similar to the original as possible.

Note: I discovered you can add `,` between the ternary `?:` statement. It also returns the most right expression in a chain of `,` expressions.

## Code
```c
char buf;       /* buffer for ungetch */
int bufp = 0;   /* next free position in buf */

int getch(void) /* get a (possibly pushed back) character */
{
    return (bufp == 1) ? bufp = 0, buf : getchar();
}

void ungetch(int c) /* push character back on input */
{
    buf = c, bufp = 1;
}
```

## Output
<p align="center">
    No Output
</p>

[Back to Main](../readme.md)
