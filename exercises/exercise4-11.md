# Exercise 4-11

> Modify getop so that it doesn't need to use ungetch. Hint: use an internal static variable. 

## Process
Honestly, didn't understand this exercise so had to look at a [solution](https://www.youtube.com/watch?v=cy86MparUY8) and now I understand.

### TLDR
Replace the buffer and functions `getch` and `ungetch` with a static variable that holds the carry over state between function calls and `getchar` from `<stdio.h>`.

### Solution
The `c` variable serves multiple purposes. 
1. It is first used by `getop` to hold the initial new character,
2. Then it is used by `getop` as a temporary holder as it processes the value of each input character

And now:

3. it statically holds the last character seen, ready for the next `getop` call

### Confusion
I think my confusion came from thinking I needed to retain the buffer functionality, but its ok to just drop it. 

I find these questions confuse me by not indicating how much I am expected to modify the orginal code and what starting point of the calculator I am working from.

## Code
```c
/* getop: get next operator or numeric operand */
int getop(char s[])
{
    int i;
    static int c;
    
    while ((s[0] = c = getchar()) == ' ' || c == '\t')
        ;
    s[1] = '\0';
    if (!isdigit(c) && c != '.')
        return c;   /* not a number */
    i = 0;
    if (isdigit(c)) /* collect integer part */
        while (isdigit(s[++i] = c = getchar()))
            ;
    if (c == '.')   /* collect fraction part */
        while (isdigit(s[++i] = c = getchar()))
            ;
    s[i] = '\0';
    printf("%d: %c\n", c);
    return NUMBER;
}
```

## Output
<p align="center">
    No Output
</p>

[Back to Main](../readme.md)
