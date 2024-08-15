# Exercise 5-18
> Make **dcl** recover from input errors

## Process
This one was big for me, a bunch of new concepts for my brain and felt a little overwhelming at first.

Recursive descent, which was a new idea for me, is a cool way to parse an unknown amount of related input text, so I played around stepping
through the code using gdb and examining the stack frames to get a better feel for how it worked. I didn't realise you could select different
stack frames, show that frames local variables and return early with an input value.

This brought me onto GNU C, which I didn't know about. As gcc is the compilier I am using, it
turns out it compiles it's own 'version' of C sometimes called [glibc](https://en.wikipedia.org/wiki/Glibc),
which I've heard of before but now I know what it is.

GNU C allows nested functions and some elements of closures, which really confused me, because I 
knew that C didn't allow that. This [documentation](https://gcc.gnu.org/onlinedocs/gcc-8.5.0/gcc/Nested-Functions.html#Nested-Functions) and 
[article](https://nullprogram.com/blog/2019/11/15/) explain a bit more about it.

After reading [deeper](https://gcc.gnu.org/onlinedocs/gcc/Standards.html) into the gcc usage with C standards, I came away with this:
```
ISO C: Allows function declarations but not nested functions.
GNU C: Allows nested functions as a GCC-specific extension.
Compiling on gcc with -std and -pedantic: enforces ISO C compliance
```

### Solving the problem
This exercise was another vague one, so I pieced together my own definition of the task

The book lists things that dcl isn't built to handle yet
```
1. It can only handle a simple data type like char or int
2. It does not handle argument types in functions, or qualifiers like const
3. Spurious blanks confuse it
4. Invalid declarations will also confuse it
```
So I took the exercise to mean the first 3 are part of the design and we are tasked with
changing the 4th item `Invalid declarations will also confuse it`

The scope of errors I decided to work with were
```
blank lines
extra alpha characters after datatype and name have been captured
no newline at the end of file
function parameter types in declaration
```

The way I decided to handle it was only do the minimal amount of code change and just add
some extra lines to the main function.

Added a check and continue to skip blank lines.
```c
if (tokentype == '\n')      
  continue; /* skip blank lines */
```

Added this sequence to print what we have and skip the line if the program encounters any errors.
```c
if (tokentype != '\n') {
  printf("syntax error\n");
  while((tokentype = gettoken()) != '\n' && tokentype != EOF) 
      ; /* skip this line which has some error */
  printf("ERROR %s: %s %s\n", name, out, datatype);
  if (tokentype == EOF) {
      printf("ERROR: missing newline at end of file");
      break;
  }
```

## Code
```c
#include <stdio.h>
#include <string.h>
#include <ctype.h>

#define MAXTOKEN 100

enum {NAME, PARENS, BRACKETS};

void dcl(void);
void dirdcl(void);

int  gettoken(void);        /* type of last token */
int  tokentype;             /* last token string */
char token[MAXTOKEN];       /* idetifier name */
char name[MAXTOKEN];        /* data type = char, int, etc. */
char datatype[MAXTOKEN];    /* output string */
char out[1000];

int main()  /* convert delaration to words */
{
    while (gettoken() != EOF) {     /* 1st token on line */
        if (tokentype == '\n')      
            continue;               /* skip blank lines */
        strcpy(datatype, token);    /* is the datatype */
        out[0] = '\0';
        dcl();  /* parse rest of line */
        if (tokentype != '\n') {
            printf("syntax error\n");
            while((tokentype = gettoken()) != '\n' && tokentype != EOF) 
                ; /* skip this line which has some error */
            printf("ERROR %s: %s %s\n", name, out, datatype);
            if (tokentype == EOF) {
                printf("ERROR: missing newline at end of file");
                break;
            }
        } else
            printf("%s: %s %s\n", name, out, datatype);
    }
    
    return 0;
}

/* dcl: parse a declarator */
void dcl(void)
{
    int ns;
    
    for (ns = 0; gettoken() == '*'; )   /* count *'s */
        ns++;
    dirdcl();
    while (ns-- > 0)
        strcat(out, " pointer to");
}

/* dirdcl: parse a direct declarator */
void dirdcl(void)
{
    int type;
    
    if (tokentype == '(') {             /* ( dcl ) */
        dcl();
        if (tokentype != ')')
            printf("error: missing )\n");
    } else if (tokentype == NAME)       /* variable name */
        strcpy(name, token);
    else
        printf("error: expected name or (dcl)\n");
    while((type=gettoken()) == PARENS || type == BRACKETS)
        if (type == PARENS)
            strcat(out, " function returning");
        else {
            strcat(out, " array");
            strcat(out, token);
            strcat(out, " of");
        }
}

int gettoken(void)  /* return next token */
{
    int c, getch(void);
    void ungetch(int);
    char *p = token;
    
    while ((c = getch()) == ' ' || c == '\t')
        ;
    if (c == '(') {
        if ((c = getch()) == ')') {
            strcpy(token, "()");
            return tokentype = PARENS;
        } else {
            ungetch(c);
            return tokentype = '(';
        }
    } else if (c == '[') {
        for (*p++ = c; (*p++ = getch()) != ']';)
            ;
        *p = '\0';
        return tokentype = BRACKETS;
    } else if (isalpha(c)) {
        for (*p++ = c; isalnum(c = getch());)
            *p++ = c;
        *p = '\0';
        ungetch(c);
        return tokentype = NAME;
    } else
        return tokentype = c;
}

#define BUFSIZE 100
char buf[BUFSIZE];  /* buffer for ungetch */
int bufp = 0;       /* next free position in buf */

int getch(void) /* get a (possibly pushed back) character */
{
    return (bufp > 0) ? buf[--bufp] : getchar();
}

void ungetch(int c) /* push character back on input */
{
    if (bufp >= BUFSIZE)
        printf("ungetch: too many characters\n");
    else
        buf[bufp++] = c;
}
```

## Output
<p align="center">
  <image src="../assets/exercise5-18_cmds.jpg" alt="output for exercise 5-18_cmds" />
</p>
<p align="center">
  <image src="../assets/exercise5-18_data_and_result.jpg" alt="output for exercise 5-18_data_and_result" />
</p>

[Back to Main](../readme.md)
