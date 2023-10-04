# Exercise 4-1

> Write the function strrindex(s,t), which returns the position of the rightmost occurence of t in s, or -1 if there is none.

## Process
Nothing I really want to say about this one. Pretty straightforward

## Code
```c
#include <stdio.h>

int strrindex(char s[], char t[]);

char s[] = "ab ab";
char pattern[] = "ab";
int result;
    
int main()
{
    result = strindexright(s, pattern);
    printf("%d", result);
}

/*strindex: return rightmost index of t in s, -1 if none */
int strrindex(char s[], char t[])
{
    int i, j, k, lastfound;
    lastfound = -1;
    
    for (i = 0; s[i] != '\0'; i++)
    {
        for (j=i, k=0; t[k]!='\0' && s[j]==t[k]; j++, k++)
            ;
        if (k > 0 && t[k] == '\0')
            lastfound = i;
    }
    return lastfound;
}
```

## Output
<p align="center">
    <image src="../assets/exercise4-1.jpg" alt="output for exercise4-1" />
</p>

[Back to Main](../readme.md)
