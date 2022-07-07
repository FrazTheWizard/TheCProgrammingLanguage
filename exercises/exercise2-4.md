# Exercise 2-4
> Write an alternate version of `squeeze(s1,s2)` that deletes each character in `s1` that matches any character in the *string* `s2`

## Process

This took one session and was fairly straight forward. I liked that I was again able to visualise a solution without much thought.

The solution is basically, for each character in the first array `s1`

Go through every character in the second array `s2` and stop if you find:
1. matching character `s1[i] != s2[j]`
2. the string termination `s2[j] != '\0'`

If you found the termination `'\0'`, then copy the character

## Final Code
```c
#include <stdio.h>

void squeeze(char s1[], char s2[]);
void printarray(char s[]);

int main(void) {
    char sx[3]; char sy[3];
    
    /* no overlapping chars */
    sx[0] = 'a'; sx[1] = 'b'; sx[2] = '\0';
    sy[0] = 'c'; sy[1] = 'd'; sy[2] = '\0';
    
    squeeze(sx, sy);
    printf("No overlapping: ");
    printarray(sx);
    
    /* 1 overlapping char */
    sx[0] = 'a'; sx[1] = 'b'; sx[2] = '\0';
    sy[0] = 'c'; sy[1] = 'a'; sy[2] = '\0';
    squeeze(sx, sy);
    printf("\n\'a\' overlapping: "); 
    printarray(sx);
    
    return 0;
}

/* delete all characters in s2 from s1 */
void squeeze(char s1[], char s2[]) {
    int i, j, k;
    i = j = k = 0;
    for(i; s1[i] != '\0'; i++) {
        for(j = 0; (s1[i] != s2[j]) && (s2[j] != '\0'); j++)
            ;
        if(s2[j] == '\0')   
            s1[k++] = s1[i];
    }
    s1[k] = '\0';
}

void printarray(char s[]) {
    int i = 0;
    while(s[i] != '\0')
        putchar(s[i++]);
}
```
## Output


<p align="center">
    <image src="../assets/exercise2-4_output.jpg" alt="output for exercise2-4" />
</p>

[Back to Main](../readme.md)
