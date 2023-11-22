# Exercise 5-3

> Write a pointer version of the function **strcat** that we showed in Chapter 2: **strcat(s,t)** copies the string **t** to the end of **s**.

## Process
Pretty straightforward code wise

I was a little stumped on how the array concatenation actually worked because up until now, I had understood that arrays were fixed size, but then here we are changing the size of it

I noticed that using `sizeof()` on one of these modified arrays gives the original size, but the printing functions (which just look for null terminattion) go over the size into other memory

This [stackoverflow](https://stackoverflow.com/questions/1239938/accessing-an-array-out-of-bounds-gives-no-error-why) answer explains that going out of bounds in undefined behaviour. 
So I guess the book is showing that you can do these things, but seems this is not a safe way to do it?

## Code
```c
#include <stdio.h>

void f_strcat(char s[], char t[]);

int main()
{
    char s1[] = {"hello "};
    char s2[] = {"world"};
    f_strcat(s1, s2);
    printf("%s\n", s1);
    return 0;
}

/* strcat: concatenate t to end of s; s must be big enough */
void f_strcat(char *s, char *t)
{
    int i = 0, j = 0;
    
    // get end index of s
    while(*(s + i) != '\0')
        i++;
    
    // append t to s
    while((*(s + i) = *(t + j)) != '\0')
        i++, j++;    
}
```

## Code 2 : Even simplier version! from [zer0325](https://github.com/zer0325/KnR-Exercise-Solution/blob/master/Chapter/05/Exercise/5-03.c)
```c
void strcat(char *s, char *t)
{
	while(*s)
		++s;

	while(*s++ = *t++)
		;
}
```

## Output
<p align="center">
      Not needed, just prints "hello world"
</p>

[Back to Main](../readme.md)
