# Exercise 5-5

> Write versions of the library functions strncpy, strncat, and strncmp, which operate on at most the first n characters of their argument strings.
> For example, strncpy(c, t, n) copies at most n characters of t to s. Full descriptions are in Appendix B.

## Process
Honestly, I thought this was going to be harder than it was. 
I guess I assumed these built in functions would be more complicated, but they are fairly simple.

One thing I defintely learned in this exercise was memory overriding. I was trying to concatenate
to multiple strings with 0 length and discovered I was getting super weird results. I guess this
is what they mean when they say you have to manage your own memory in c.

I have also included my test code

## Function Descriptions
```
char *strncpy(s,ct,n)   copy at most n characters of string ct to s; return s.
                        Pad with '\0's if ct has fewer than n characters.
                        
char *strncat(s,ct,n)   concatenate at most n characters of string ct to 
                        string s, terminate s with '\0'; return s.
                        
int strncmp(cs,ct,n)    compare at most n characters of string cs to string ct; 
                        return <0 if cs<ct, 0 if cs==ct, or >0 if cs>ct.
```

## Code
```c
#include <stdio.h>

char *f_strncpy(char *s, char *ct, int n);
char *f_strncat(char *s, char *ct, int n);
int f_strncmp(char *s, char *ct, int n);
void test_strncpy();
void test_strncat();
void test_strncmp();

int main()
{
    test_strncpy();
    test_strncat();
    test_strncmp();
    return 0;
}

char *f_strncpy(char *s, char *ct, int n)
{
    // 1. copy at most n characters of string ct to s
    while(*ct && n--)
        *s++ = *ct++; // (unary operators associate right to left)
    
    // 2. pad with '\0's if ct has fewer than n characters
    if(!n)
        return s;
    
    while(*s)
        *s++ = '\0';
    
    return s;
}

char *f_strncat(char *s, char *ct, int n)
{
    // 1. go to end of s string
    while(*s)
        *s++;
    
    // 2. copy ct character into s until finished
    while(n-- && *ct)
        *s++ = *ct++;
    
    // 3. make sure it is closed with null
    *s = '\0';
    
    return s;
}

int f_strncmp(char *s, char *ct, int n)
{
    while(*s && n--)
    {
        if(*s == *ct)
            s++, ct++;
        if(*s < *ct)
            return 1;            
        if(*s > *ct)
            return -1;
    }
    return 0;
}

void test_strncpy()
{
    printf("\nSTRING COPY\n");
    char ct[] = "zyx";
    char s1[] = "abcdef";
    f_strncpy(s1, ct, 3);
    printf("f_strncpy('abcdef', 'zyx', 3) : %s\n", s1);
    
    char s2[] = "abcdef";
    strncpy(s2, ct, 4);
    printf("f_strncpy('abcdef', 'zyx', 4) : %s\n", s2);
}

void test_strncat()
{
    // be careful because it overrides memory...
    printf("\nSTRING CONCATENATE\n");
    char t[] = "world";
    char s[] = "hello";
    f_strncat(s, t, 7);
    printf("f_strncat('hello', 'world', 7) : %s\n", s);
}


void test_strncmp()
{
    printf("\nSTRING COMPARE\n");
    int x;
    x = f_strncmp("hello", "hello", 4);
    printf("f_strncmp('hello', 'hello', 4) : %d\n", x);
    x = f_strncmp("hello", "apple", 4);
    printf("f_strncmp('hello', 'apple', 4) : %d\n", x);
    x = f_strncmp("hello", "world", 4);
    printf("f_strncmp('hello', 'world', 4) : %d\n", x);
}


```

## Output
<p align="center">
  <image src="../assets/exercise5-5.jpg" alt="output for exercise5-5" />
</p>

[Back to Main](../readme.md)
