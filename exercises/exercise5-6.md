# Exercise 5-6

> Rewrite appropriate programs from earlier chapters and exercises with pointers instead of array indexing. Good possibilities include getline
> (Chapters 1 and 4), atoi, itoa, and their variants (Chapters 2, 3 and 4), reverse (Chapter 3), and strindex and getop (Chapter 4).

## Process
This one took me a number of sessions to complete. 

Turned out I struggled with some specifics of how pointers worked, so this was a good exercise and I'm feeling more confident about them now

### getline
I struggled with the `getline` function because I was trying to do it all with pointers, but wasn't able to find a way to check for EOF and store the charater in the `char` string.

After making a working solution just storing getchar in the string, I ended up just copying the first answer from [clc-wiki](https://clc-wiki.net/wiki/K%26R2_solutions:Chapter_5:Exercise_6)

### order of operations
Another thing that tripped me up was order of operations concerning combinations of the `++` post increment with a `*` dereference operator.
For instance, this conditional statement:

```c
if((*s++ < i) && *s > i)
```

I didn't know if the post increment happened 
- after `*s`was evaluated
- after `*s < i`
- after the whole line

Now I know it increments the pointer `s` after `*s < i`



## Code
```c
#include <stdio.h>

#include <string.h>

int p_getline(char * line, int limit);
int p_atoi(char * s);
int p_htoi(char * s);
void p_itoa(int n, char * s);
void p_reverse(char * s);
int p_strindex(char * source, char * searchfor);
int p_getop(char[]);
#define NUMBER '0' /* signal that a number was found */

int main() {
    // GETLINE
    printf("GETLINE:\n");
    char line[25];
    while (p_getline(line, 10))
        printf("%s", line);
    printf("\n\n");

    // ATOI
    char x1[] = "345";
    int x2 = p_atoi(x1);
    printf("ATOI\nstring: %s to int: %d\n\n", x1, x2);

    // ITOA
    int y1 = 8372;
    char y2[10];
    p_itoa(y1, y2);
    printf("ITOA\nint: %d to string: %s\n\n", y1, y2);

    // HTOI
    char z1[] = "0x34Ab";
    int z2 = p_htoi(z1);
    printf("HTOI\nstring: %s to int: %d\n\n", z1, z2);

    // STRINDEX
    char source[] = "Source string with target inside";
    char target[] = "target";
    int result = p_strindex(source, target);
    printf("Index of '%s' found in '%s' is %d\n\n", target, source, result);

    // GETOP
    int type;
    char s[10];
    printf("GETOP:\n");
    while ((type = p_getop(s)) != EOF)
        switch (type) {
        case NUMBER:
            printf("GETOP type: %c s: %s\n", type, s);
            break;
        case '\n':
            break;
        case '+':
            printf("GETOP type: %c s: %s\n", type, s);
            break;
        default:
            printf("return: %s\n", s, s);
            break;
        }
}

/* p_getline: read a line into s, return length, using pointers */
int p_getline(char * line, int limit) {
    int len = 0;
    for (;
        (( * line = getchar()) != EOF) && ( * line != '\n') && (len < limit); len++)
        line++;
    if ( * line != EOF)
        line++, len++;
    * line = '\0';
    return len;
}

/* p_atoi: convert s to integer using pointers */
int p_atoi(char * s) {
    int n = 0;
    for (;* s >= '0' && * s <= '9';)
        n = 10 * n + ( * s++ - '0');
    return n;
}

/* p_itoa: convert n to characters in s using pointers */
void p_itoa(int n, char * s) {
    int sign;
    char * p = s;

    if ((sign = n) < 0) /* record sign */
        n = -n; /* make n positive */

    do {
        /* generate digits in reverse order */
        * p++ = (unsigned) n % 10 + '0'; /* get next digit */
    } while ((n = (unsigned) n / 10) > 0); /* delete it */
    if (sign < 0)
        *
        p++ = '-';
    * p = '\0';
    p_reverse(s);
}

/* p_reverse: reverse string s in place using pointers*/
void p_reverse(char * s) {
    /* find last char of s */
    char * s_end = s + strlen(s) - 1;
    char c;

    /* swap end characters, continuing towards centre of string */
    for (; s < s_end; s++, s_end--) {
        c = * s;
        * s = * s_end;
        * s_end = c;
    }
}

/* p_htoi: convert n character as hex to integer using pointers */
int p_htoi(char * s) {
    int i, n;
    n = 0;

    /* ignore initial 0x or 0X */
    if ( * s == '0' && ( * (s + 1) == 'x' || * (s + 1) == 'X'))
        s += 2;

    /* ignore characters outside legal hex values */
    for (i;
        ( * s >= '0' && * s <= '9') ||
        ( * s >= 'A' && * s <= 'F') ||
        ( * s >= 'a' && * s <= 'f'); s++
    ) {
        /* add value to total integer */
        if ( * s >= '0' && * s <= '9')
            n = 16 * n + ( * s - '0');

        if ( * s >= 'A' && * s <= 'F')
            n = 16 * n + ( * s - 'A') + 10; /* add 10 because A is 10 */

        if ( * s >= 'a' && * s <= 'f')
            n = 16 * n + ( * s - 'a') + 10; /* add 10 because a is 10 */
    }
    return n;
}

/* p_strindex: return index of t in s, -1 if none using pointers*/
int p_strindex(char * s, char * t) {
    char * src, * search, * tar;
    src = search = s;
    tar = t;

    /* go through all chars in the string */
    for (;* src != '\0'; src++) {

        /* reset and find continous matching chars */
        search = src, tar = t;
        while ( * tar != '\0' && * search == * tar)
            search++, tar++;

        /* found complete string match */
        if (tar != t && * tar == '\0')

            /* return index */
            return src - s;
    }

    return -1;
}

#include <ctype.h>

int getch(void);
void ungetch(int);

/* p_getop: get next operator or numeric operand using pointers */
int p_getop(char * s) {
    int i, c;

    while (( * s = c = getch()) == ' ' || c == '\t')
    ;
    *(s + 1) = '\0';
    if (!isdigit( * s) && * s != '.') {
        if ( * s != '-')
            return * s; /* not a number, minus or negative */

        if (!isdigit( * ++s = getch())) /* if next char is not digit */ {
            ungetch( * s);
            * s = '\0';
            return '-'; /* minus sign */
        }
    }

    if (isdigit( * s)) /* collect integer part */
        while (isdigit( * ++s = getch()))
    ;
    if ( * s == '.') /* collect fraction part */
        while (isdigit( * ++s = getch()))
    ;
    if ( * s != EOF)
        ungetch( * s);
    * s = '\0';
    return NUMBER;
}

#define BUFSIZE 100

char buf[BUFSIZE]; /* buffer for ungetch */
int bufp = 0; /* next free position in buf */

int getch(void) /* get a (possibly pushed back) character */ {
    return (bufp > 0) ? buf[--bufp] : getchar();
}

void ungetch(int c) /* push character back on input */ {
    if (bufp >= BUFSIZE)
        printf("ungetch: too many characters\n");
    else
        buf[bufp++] = c;
}
```

## Output
<p align="center">
  <image src="../assets/exercise5-6.jpg" alt="output for exercise5-6" />
</p>

[Back to Main](../readme.md)
