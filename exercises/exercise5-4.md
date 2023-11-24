# Exercise 5-4

> Write a function **strend(s,t)**, which returns 1 if the string **t** occurs at the end of the string **s**, and zero otherwise.

## Process
This was a good exercise for me

I found some things helpful in initally building the function:
- writing down some test strings to see the data I'm working on
- sketching a bit of a flow chart of the process, using images or just pseudocode, to get my head around possible ways the function could be achieved

Then once I had a working function that processed the test strings correctly:
- try to break the function using different input strings. If it breaks, modify the function and add that input to the test strings

## Deep Thoughts
While looking at some other [solutions](https://clc-wiki.net/wiki/K%26R2_solutions:Chapter_5:Exercise_4) I can see more elegant and possibly more efficent solutions

I like my function because it doesn't use any external libraries, it's sort of self contained logic using atomic building blocks of c. 
Maybe this adds some more lines to read, but in the long run, I think this keeps the program simple for future modification. 
Maybe in the context of a larger program I would extract similar logic into a function, but here I like the non library approach.

I also find readability flow of the my function pretty good, it's one loop of the whole string processing each character.

I did like the solution by **Kirill Yatsenko**
```c
int strend(const char* s, const char* t)
{
    s += strlen(s) - strlen(t);

    while(*s && *s++ == *t++);

    return !*s;
}
```
Apart from being three lines of code, which is a small amount of code, I also liked the method of jumping to the position where `t` *should* be in `s` and starting there, because there's no use in looking elsewhere.
Seems kind of an obvious way to process the data now.

## Code
```c
#include <stdio.h>

int strend(char *s, char *t);

int main()
{
    char find[] = "rum";
    char s1[] = "red rum";
    char s2[] = "rum red";
    char s3[] = "rum";
    char s4[] = "ru";
    char s5[] = "";
    
    printf("'%s' at end of '%s' : %d\n", find, s1, strend(s1, find));
    printf("'%s' at end of '%s' : %d\n", find, s2, strend(s2, find));
    printf("'%s' at end of '%s' : %d\n", find, s3, strend(s3, find));
    printf("'%s' at end of '%s' : %d\n", find, s4, strend(s4, find));
    printf("'%s' at end of '%s' : %d\n", find, s5, strend(s5, find));
    
    return 0;
}

/* strend(*s, *t) : if string t is found at the end of string s */
int strend(char *s, char *t)
{
    int found = 0;
    char *originalt = t;
    
    while(*s)
    {
        if(*s == *t)                        /* matching char */
            found = 1, t++;
        else {
            if(found == 1)
                t = originalt, found = 0;   /* reset counter */
            else
                ;                           /* skip non-matching char */
        }
        s++;
    }
    
    if(*t != 0) /* reached end of s but not t */
        found = 0;
    
    return found;
}
```

## Output
<p align="center">
  <image src="../assets/exercise5-4.jpg" alt="output for exercise5-4" />
</p>

[Back to Main](../readme.md)
