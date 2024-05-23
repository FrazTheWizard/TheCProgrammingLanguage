# Exercise 5-10

> Rewrite the program **expr**, which evaluates a reverse Polish expression from the command line, where each operator or operand is a separate argument. For example,
> `expr 2 3 4 + *`

## Process
Pretty straight forward exercise

I was having weird problems with input until I realised that `*` on the windows command line gets you all files in the current directory as input

Apparently this is something to do with MinGW and the way it operates with Windows environment. 
If you add the flag to your program, it tells the operating system not to do that:

```c 
extern int _CRT_glob = 0;
```

## Code
```c
#include <stdio.h>
#include <stdlib.h> /* for atof() */

#define MAXOP   100 /* max size of operand or operator */ 
#define NUMBER  '0' /* signal that a number was found */

int getop(char [], char *);
void push(double);
double pop(void);

/* 
 * disables MinGW globbing that is added to this program. 
 * this is so the windows command line doesn't interpret
 * the '*' symbol as 'grab every file in the current directory'.
 * this flag tells the operating system not to do that please.
 */
extern int _CRT_glob = 0; 

/* reverse Polish calculator */
int main(int argc, char *argv[])
{
    int type;
    double op2;
    char s[MAXOP];

    int i = 1;
		
    while(i < argc) {
      type = getop(s, argv[i++]);
        switch (type) {
            case NUMBER:
                push(atof(s));
                break;
            case '+':
                push(pop() + pop());
                break;
            case '*':
                push(pop() * pop());
                break;
            case '-':
                op2 = pop();
                push(pop() - op2);
                break;
            case '/':
                op2 = pop();
                if (op2 != 0.0)
                    push(pop() / op2);
                else
                    printf("error: zero divisor\n");
                break;
            case '%':
                op2 = pop();
                if (op2 != 0.0)
                    push((int) pop() % (int) op2);
                else
                    printf("error: zero divisor\n");                
                break;
            case '\n':
                printf("\t%.8g\n", pop());
                break;
            default:
                printf("error: unknown command %s\n", s);
                break;
        }
    }
		
    printf("%.8g\n", pop());
		
    return 0;
}


#define MAXVAL 100 /*maximum depth of val stack */

int sp = 0;         /* next free stack position */
double val[MAXVAL]; /* values stack */

/* push: push f onto value stack */
void push(double f)
{
    if (sp < MAXVAL)
        val[sp++] = f;
    
    else
        printf("error: stack full, can't push %g\n", f);
}

/* pop: pop and return top value from stack */
double pop(void)
{
    if (sp > 0)
        return val[--sp];
    else {
        printf("error: stack empty\n");
        return 0.0;
        
    }
}

#include <ctype.h>

int getch(void);
void ungetch(int);

/* getop: get next operator or numeric operand */
int getop(char s[], char *input)
{
    int i, c;
    
    while ((s[0] = c = *input++) == ' ' || c == '\t')
        ;
    s[1] = '\0';
    if (!isdigit(c) && c != '.')
    {
        if (c != '-')
            return c;               /* not a number, minus or negative */
        
        if(!isdigit(c = *input++))   /* if next char is not digit */
        {
             ungetch(c);
             return '-';            /* minus sign */
        }
        
        s[i = 1] = c;
    } else {i = 0;}
    
    if (isdigit(c)) /* collect integer part */
        while (isdigit(s[++i] = c = *input++))
            ;
    if (c == '.')   /* collect fraction part */
        while (isdigit(s[++i] = c = *input++))
            ;
    s[i] = '\0';
    // if (c != EOF)
        // ungetch(c);
    return NUMBER;
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
  <image src="../assets/exercise5-10.jpg" alt="output for exercise 5-10" />
</p>

[Back to Main](../readme.md)
