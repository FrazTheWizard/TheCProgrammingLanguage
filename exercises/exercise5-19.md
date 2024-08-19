# Exercise 5-19
> Modify **undcl** so that it does not add redundant parentheses to declarations.

## Process
This exercise was a lot easier than the previous exercise due to being familar with the concept 
and general process now. Again, I tried to take a similar approach, make as little code change 
as possible and stay within the spirit of the existing program.

My modification only handles the program and input data as defined in the book, but no doubt
would need to be rethought if this program were to be extended.

### Changes
1. Added true and false values for clarity
```c
enum {TRUE, FALSE};
```

2. Removed printing of parentheses when finding `*` token and just record that we have
seen it: `collectpointers = TRUE;`
```c
else if (type == '*') {
  sprintf(temp, "*%s", out);
  strcpy(out, temp);
  collectpointers = TRUE;
}
```

3. At the top of the gettoken loop, handle the parentheses printing
```c
if (type != '*' && collectpointers == TRUE) {
  if (type != NAME) {
    sprintf(temp, "(%s)", out);
    strcpy(out, temp);
  }
  collectpointers = FALSE;
}
```

## Code
```c

#include <stdio.h>
#include <string.h>
#include <ctype.h>

#define MAXTOKEN 100

enum {NAME, PARENS, BRACKETS};
enum {TRUE, FALSE};

int  gettoken(void);        /* type of last token */
int  tokentype;             /* last token string */
char token[MAXTOKEN];       /* identifier name */
char out[1000];

/* undcl: convert word description to declaration */
int main()
{
    int type, collectpointers = FALSE;
    char temp[MAXTOKEN];
    
    while(gettoken() != EOF) {
        strcpy(out, token);
        while((type = gettoken()) != '\n') {
            
            if (type != '*' && collectpointers == TRUE) {
                if (type != NAME) {
                    sprintf(temp, "(%s)", out);
                    strcpy(out, temp);
                }
                collectpointers = FALSE;
            }
            
            if (type == PARENS || type == BRACKETS)
                strcat(out, token);
            else if (type == '*') {
                sprintf(temp, "*%s", out);
                strcpy(out, temp);
                collectpointers = TRUE;
            } else if (type == NAME) {
                sprintf(temp, "%s %s", token, out);
                strcpy(out, temp);
            } else
                printf("invalid input at %s\n", token);
        }
        printf("%s\n", out);
    }
    return 0;
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
  <image src="../assets/exercise5-19_cmds.jpg" alt="output for exercise 5-19_cmds" />
</p>
<p align="center">
  <image src="../assets/exercise5-19_data_and_result.jpg" alt="output for exercise 5-19_data_and_result" />
</p>

[Back to Main](../readme.md)
