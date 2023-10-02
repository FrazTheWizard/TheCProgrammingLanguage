# Exercise 3-6

> Write a version of itoa that accepts three arguments instead of two.
> The third argument is a minimum field width; the converted number must be padded
> with blanks on the left if necessary to make it wide enough.

## Process
This one took not even half an hour, just two lines of code. 
`while(i < w)
        s[i++] = 0x20;`  
I guess the question is to make sure you have an understanding of the concept. Which I believe I do now.

I was a little confused about the word blank. I initially thought it meant a `0` character but after [checking](https://youtu.be/U7RNrPNjt4w?si=2WOpTESSoz5qrr3G)
my solution I realised it meant a blank space ` `

## Code

```c
#include <stdio.h>
#include <string.h>
#define size 30

void itoa(int, char[], int);
void reverse(char s[]);

int main(void)
{
    char s[size];
    for (int w = 0; w < 20; w = w + 5)
    {
        itoa(50, s, w);
        printf("padding of %2d:%s\n", w, s);    
    }
}

/* itoa: convert n to characters in s with minimum field width */
void itoa(int n, char s[], int w)
{
    int i, sign;

    if ((sign = n) < 0) /* record sign */ 
        n = -n;         /* make n positive */
        
    i = 0;
    do {        /* generate digits in reverse order */
        s[i++] = (unsigned) n % 10 + '0';  /* get next digit */
    } while ((n = (unsigned) n / 10) > 0);    /* delete it */
    
    if (sign < 0)
        s[i++] = '-';
    
    while(i < w)
        s[i++] = 0x20;    
    
    s[i] = '\0';
    reverse(s);
}

/* reverse: reverse string s in place */
void reverse(char s[])
{
    int c ,i ,j;
    
    for (i = 0, j = strlen(s) -1; i < j; i++, j--) {
        c = s[i];
        s[i] = s[j];
        s[j] = c;
    }
}
```
## Output
<p align="center">
    <image src="../assets/exercise3-6.jpg" alt="output for exercise3-6" />
</p>

[Back to Main](../readme.md)
