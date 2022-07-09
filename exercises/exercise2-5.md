# Exercise 2-5

> Write the function `any(s1,s2)`, which returns the first location in the string `s1` where any character from the string `s2` occurs, or 
`-1` if `s1` contains no characters from `s2`. (The standard library function strpbrk does the same job but returns a pointer to the location)

## Process
Pretty straight forward exercise. Very similar to the last. I think I really get it now that `\0` ends a string in C


## Final Code
```c
#include <stdio.h>

int any(char s1[], char s2[]);

int main(void) {
    
    char s1[] = "hello world\0";
    char s2[] = "abc\0";
    char s3[] = "w\0";
    
    printf("%d\n", any(s1, s2));
    printf("%d\n", any(s1, s3));
    
    return 0;
}

int any(char s1[], char s2[]) {
    int i, j;
    
    for (i = 0; s1[i] != '\0'; i++) {
        for (j = 0; s2[j] != '\0'; j++) {
            if (s1[i] == s2[j]) {
                return i;
            }
        }
    }
    
    return -1;
}
```


## Output

<p align="center">
    <image src="../assets/exercise2-5_output.jpg" alt="output for exercise2-5" />
</p>

[Back to Main](../readme.md)      
