# Exercise 7-3

> Revise minprintf to handle more of the other facilities of printf

## Process
Pretty straight forward, just added some extra case's to the switch statement

Checked C answer book which had a nice solution that collected everything between `%` - `char` as the format for the printf

## Code
```c
#include <stdio.h>
#include <stdarg.h>

/* minprinf: minimal c with variable argument list */
void minprintf(char *fmt, ...)
{
    va_list ap;
    char *p, *sval;
    int ival;
    double dval;
    
    va_start(ap, fmt);
    for (p = fmt; *p; p++) {
        if (*p != '%') {
            putchar(*p);
            continue;
        }
        
        switch(*++p) {
            case 'i':
            case 'd':
                ival = va_arg(ap, int);
                printf("%d", ival);
                break;
            case 'f':
                dval = va_arg(ap, double);
                printf("%f", dval);
                break;
            case 's':
                for (sval = va_arg(ap, char *); *sval; sval++)
                    putchar(*sval);
                break;
            case 'x':
                ival = va_arg(ap, int);
                printf("%x", ival);
                break;
            case 'o':
                ival = va_arg(ap, int);
                printf("%o", ival);
                break;
            case 'c':
                ival = va_arg(ap, int);
                printf("%c", ival);
                break;
            default:
                putchar(*p);
                break;
        }
    }
    va_end(ap);
}

int main()
{
    minprintf("%i %f %s\n", 22, 33.0, "hello world");
    minprintf("%x %o %c", 255, 255, 97);
    return 0;
}
```

## Output
<p align="center">
  <image src="../assets/exercise7-3_result.jpg" alt="output for exercise7-3 result" />
</p>

[Back to Main](../readme.md)
