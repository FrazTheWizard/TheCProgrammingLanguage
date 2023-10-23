# Exercise 4-7

> Write a routine ungets(s) that will push back an entire string onto the input. Should ungets know about buf and bufp, or should it just use ungetch?

## Process
This is pretty straightforward. It can just use ungetch for every char in a string.

As zer0325 [points out](https://www.youtube.com/watch?v=R3C6tLguJVc) ungetch does the same thing a routine using buf and bufp would be doing anyway.
## Code
```c
void ungets(char s[])
{
    int i = 0;
    do {
        ungetch(s[i]);
    } while(s[i++] != '\0');       
}
```

## Output
<p align="center">
    No Output
</p>

[Back to Main](../readme.md)
