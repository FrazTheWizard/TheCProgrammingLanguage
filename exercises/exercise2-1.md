# Exercise 2-1
> Write a program to determine the ranges of `char`, `short`, `int` and `long` variables,
> both `signed` and `unsigned`, by printing appropriate values from standard headers and 
> by direct computation. Harder if you compute them: determine the ranges of the various floating-point types.

## Process
This was an interesting exercise and took 2 days of mostly research and playing around

For the manual calculation I only made functions for `char` and `int` because my laptop was taking a while to calculate it.

I had three confusion points:
1. `signed` vs `unsigned` types
2. using `printf` with different types
3. the c preprocessor defining values

**1.  Signed vs Unsigned**

I knew the general distinction before this exercise, but had never used it and honestly still don't completely 
understand why it's necessary to have them. There was this [video](https://youtu.be/OyaZPEgM7bo) which uses voltage as the reason 
for different storage methods, but that seems to be for microcontrollers only. As always I found this 
[Ben Eater](https://www.youtube.com/watch?v=4qH4unVtJkE) video a great explanation of using it.

**2. printf types**

It seems that even though a variable is stored as a particular type, the printf function can also display that data as any type.
Which further adds to my confusion to why it needs to be stored as signed or unsigned if you need to define what you want to see it as later anyway...

These come from this Stackoverflow [answer](https://stackoverflow.com/questions/2844/how-do-you-format-an-unsigned-long-long-int-using-printf)

`%d`--> for int

`%u`--> for unsigned int

`%ld`--> for long int or long

`%lu`--> for unsigned long int or long unsigned int or unsigned long

`%lld`--> for long long int or long long

`%llu`--> for unsigned long long int or unsigned long long

**3. C Preprocessor**

In floating point definitions in `float.h`, the constants are defined using this syntax:
```c
#define FLT_MAX		__FLT_MAX__
#define DBL_MAX		__DBL_MAX__
#define LDBL_MAX	__LDBL_MAX__
```
This apparently means that it's coming from the C Preprocessor, so I will leave out trying to understand what that means
for now and wait until the book talks about it in a later chapter

## Final Code
```c
#include <stdio.h>
#include <limits.h>
#include <float.h>

void findRangeChar(void);
void findRangeInt(void);

int main(){
    printf("//---RANGES FROM STANDARD HEADERS---//");
    printf("\n---CHAR---\n");
    printf("Char size: %d\n\n",CHAR_BIT);
    
    printf("Signed Char Min: %d\n", SCHAR_MIN);
    printf("Signed Char Max: %d\n", SCHAR_MAX);
    printf("Unsigned Char Max: %d\n", UCHAR_MAX);
    
    printf("\n---SHORT---\n");
    printf("Short Int Min: %d\n",SHRT_MIN);
    printf("Short Int Max: %d\n",SHRT_MAX);
    printf("Unsigned Short Int Max: %d\n",USHRT_MAX);
    
    printf("\n---INT---\n");
    printf("Int Min: %d\n",INT_MIN);
    printf("Int Max: %d\n",INT_MAX);
    printf("Unsigned Int Max: %u\n",UINT_MAX);
    
    printf("\n---LONG---\n");
    printf("Long Int Min: %ld\n",LONG_MIN);
    printf("Long Int Max: %ld\n",LONG_MAX);
    printf("Unsigned Long Int Max: %lu\n",ULONG_MAX);
    
    printf("\n---LONGLONG---\n");
    printf("Long Long Int Min: %lld\n",LLONG_MIN);
    printf("Long Int Max: %lld\n",LLONG_MAX);
    printf("Unsigned Long Int Max: %llu\n",ULLONG_MAX);


    printf("\n\n//---FLOATING POINTS---//");
    printf("\n---DIGITS---\n");
    printf("Floating: %g\n", FLT_DIG);
    printf("Double: %g\n", DBL_DIG);
    printf("Long Double: %g\n", LDBL_DIG);
    
    printf("\n---MIN---\n");
    printf("Floating: %g\n", FLT_MIN);
    printf("Double: %g\n", DBL_MIN);
    printf("Long Double: %g\n", LDBL_MIN);
    
    printf("\n---MAX---\n");
    printf("Floating: %g\n", FLT_MAX);
    printf("Double: %g\n", DBL_MAX);
    printf("Long Double: %g\n", LDBL_MAX);
    
    printf("\n---EXPONENT MIN---\n");
    printf("Floating: %g\n", FLT_MIN_EXP);
    printf("Double: %g\n", DBL_MIN_EXP);
    printf("Long Double: %g\n", LDBL_MIN_EXP);
    
    printf("\n---EXPONENT MAX---\n");
    printf("Floating: %g\n", FLT_MAX_EXP);
    printf("Double: %g\n", DBL_MAX_EXP);
    printf("Long Double: %g\n", LDBL_MAX_EXP);
    

    printf("\n\n//---CALCULATED RANGES---//\n");
    printf("---CHAR---\n");
    findRangeChar();
    printf("\n---INT---\n");
    findRangeInt();
    
    return 0;
}

void findRangeChar(void){
    char x = 1;
    long long min = x;
    long long max = x;
    while(x != 0) {
        x++;
        if(x<min)
            min = x;
        if(x>max)
            max = x;
    }
    printf("char Min: %lld\n", min);
    printf("char Max: %lld\n", max);
}

void findRangeInt(void){
    int x = 1;
    long long min = x;
    long long max = x;
    while(x != 0) {
        x++;
        if(x<min)
            min = x;
        if(x>max)
            max = x;
    }
    printf("int Min: %lld\n", min);
    printf("int Max: %lld\n", max);
}
```

## Output
<p align="center">
    <image src="../assets/exercise2-1_output.jpg" alt="output for exercise2-1" />
</p>


[Back to Main](../readme.md)
