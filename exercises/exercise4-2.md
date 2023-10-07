# Exercise 4-2

> Extend atof to handle scientific notation of the form 123.45e-6 where a floating-point number may be followed by e or E and an optionally signed exponent.

## Process
Firstly took a bit of time learning how floating point is stored in a computer.
Found this [computerphile](https://youtu.be/f4ekifyijIg?si=qI1xKPT2dnaNLldi) video helpful.
Started becoming quite complicated, floating point hadn't really been covered in the book in any depth and also I realised you didnt need a huge understanding of
floating point to get the solution.

I also implemented the power function from earlier in the book.

This [solution](https://youtu.be/jguAkjeMMkA?si=m2T8gEzKx8Ulfct_) video explained things quite nicely

## Code
```c
#include <ctype.h>
#include <stdio.h>

double atof(char s[]);
double pow2(int base, int n, int sign);


int main(void)
{
    double x1 = atof("123.45e-6");
    printf("%f\n", x1);
}

/* atof: convert string s to double */
double atof(char s[])
{
    double val, power, baseNumber, exponentialValue;
    int i, sign, e, esign;
    
    for (i = 0; isspace(s[i]); i++) /* skip white space */
        ;
    sign = (s[i] == '-') ? -1 : 1;
    if (s[i] == '+' || s[i] == '-')
        i++;
    
    for (val = 0.0; isdigit(s[i]); i++)
        val = 10.0 * val + (s[i] - '0');
    
    if (s[i] == '.')
        i++;
    
    for (power = 1.0; isdigit(s[i]); i++) {
        val = 10.0 * val + (s[i] - '0');
        power *= 10.0;
    }
    
    if(s[i] == 'e' || s[i] == 'E')
        i++;
    
    esign = (s[i] == '-') ? -1 : 1;
    if (s[i] == '+' || s[i] == '-')
        i++;
    
    for (e = 0; isdigit(s[i]); i++)
        e = 10 * e + (s[i] - '0');
    
    baseNumber = sign * (val / power);
    exponentialValue  =  pow2(10, e, esign);

    return baseNumber * exponentialValue ;
}

/* power: raise base to n-th power; n >= 0 */
double pow2(int base, int n, int sign)
{
    int i;
    double p;
    
    p = 1;
    for (i = 1; i <= n; ++i)
    {
        if(sign == 1)
            p = p * base;
        else if (sign == -1)
            p = p / base;
    }
        
    return p;
}
```

## Output
<p align="center">
    Returns 0.000123
</p>

[Back to Main](../readme.md)
