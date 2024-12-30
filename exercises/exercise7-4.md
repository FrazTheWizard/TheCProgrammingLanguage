# Exercise 7-4

> Write a private version of `scanf` analogous to `minprintf` from the previous section

## Process
Found this exercise more challenging than the previous one.
I think partly that's because I have no experience with `scanf`, so needed to play around with it for a while before I understood what it does.

I implemented a similar method to the previous answer in the C Answers book by collecting a format string and passing it to `scanf`.
`minscanf` supports `%d`, `%i`, `%o`, `%u`, `%x`, `%c`, `%s`, `%f`.
It also supports some other feature like spaces, words but not support for things like (*) suppression character.

## Code
```c
#include <stdio.h>
#include <stdarg.h>
#include <ctype.h>

void minscanf(char *fmt, ...)
{
    va_list ap;
    char *p;
    
    int *dval;
    int *ival;
    int *oval;
    unsigned int *uval;
    unsigned int *xval;
    char *cval;
    char *sval;
    float *fval;
    
    char fmtstr[100], *pfmtstr = fmtstr;
    
    va_start(ap, fmt);

    for (p = fmt; *p; p++) {
        *pfmtstr++ = *p;

        if (*p != '%')
            continue;
        else
            p++;
        
        while (!isalpha(*p)) {
            *pfmtstr++ = *p++;
        }
        
        *pfmtstr++ = *p;
        *pfmtstr = '\0';
        
        switch(*p) {
            case 'd':
                dval = va_arg(ap, int *);
                scanf(fmtstr, dval);
                break;
            case 'i':
                ival = va_arg(ap, int *);
                scanf(fmtstr, ival);
                break;
            case 'o':
                oval = va_arg(ap, int *);
                scanf(fmtstr, oval);
                break;
            case 'u':
                uval = va_arg(ap, unsigned int *);
                scanf(fmtstr, uval);
                break;
            case 'x':
                xval = va_arg(ap, unsigned int *);
                scanf(fmtstr, xval);
                break;
            case 'c':
                cval = va_arg(ap, char *);
                scanf(fmtstr, cval);
                break;
            case 's':
                sval = va_arg(ap, char *);
                scanf(fmtstr, sval);
                break;
            case 'e':
            case 'f':
            case 'g':
                fval = va_arg(ap, float *);
                scanf(fmtstr, fval);
                break;
        }
        pfmtstr = fmtstr; /* reset format string */
    }
    va_end(ap);
}

int main()
{
    int d;
    int i;
    int o;
    unsigned int u;
    unsigned int x;
    char c;
    char s[100];
    float f;
    
    minscanf("randomword %d", &d);
    minscanf("%i", &i);
    minscanf("%o", &o);
    minscanf("%u", &u);
    minscanf("%x", &x);
    minscanf("%c %c", &c, &c); /* newline, space and char */
    minscanf("%s", s);
    minscanf("%f", &f);
    
    printf("d: %d\n", d);
    printf("i: %u\n", i);
    printf("o: %u\n", o);
    printf("u: %u\n", u);
    printf("x: %u\n", x);
    printf("c: %c\n", c);
    printf("s: %s\n", s);
    printf("f: %f\n", f);
}
```

## Output
<p align="center">
  <image src="../assets/exercise7-4_cmds.jpg" alt="output for exercise7-4 cmds" />
</p>

<p align="center">
  <image src="../assets/exercise7-4_data_and_result.jpg" alt="output for exercise7-4 data and result" />
</p>

[Back to Main](../readme.md)
