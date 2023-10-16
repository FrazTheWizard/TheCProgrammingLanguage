# Exercise 4-3

> Add commands to print the top element of the stack without popping, to duplicate it, and to swap the top two elements. Add a command to clear the stack.

## Process
Pretty straightforward. Added the commands in the switch statement and added extra functions that access the `val[]` and `sp` along with `push` and `pop`

I decided to couple the duplicate and print functions, to have one less function, but perhaps thats not greatest future maintainability design.

I am a little confused by this program and why the newline has to pop the stack. I feel like it shouldn't? The print function seems redundant. Maybe it will be addressed later...

## Code
```c
/*
Add commands to print the top element of the stack without popping, 
to duplicate it, and to swap the top two elements. 
Add a command to clear the stack.
*/

#include <stdio.h>
#include <stdlib.h> /* for atof() */

#define MAXOP   100 /* max size of operand or operator */ 
#define NUMBER  '0' /* signal that a number was found */

int getop(char []);
void push(double);
double pop(void);
double top(void);
void swap(void);
void clear(void);

/* reverse Polish calculator */
int main()
{
    int type;
    double op2;
    char s[MAXOP];
    
    while((type = getop(s)) != EOF) {
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
            case 't':   /* print the top character */
                printf("\t%.g\n", top());
                break;
            case 'd':   /* duplicate the top character */
                push(top());
                break;
            case 's':   /* swap the first two characters on stack */
                swap();
                break;
            case 'c':   /* clear the stack */
                clear();
                break;
            case '\n':
                printf("\t%.8g\n", pop());
                break;
            default:
                printf("error: unknown command %s\n", s);
                break;
        }
    }
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

double top(void)
{
    if (sp > 0)
        return val[sp - 1];
    else {
        printf("error: stack empty\n");
        return 0.0;
    }
}

void swap(void)
{
    double swapval;
    if(sp >= 1)
    {
        swapval = val[sp - 1];
        val[sp - 1] = val[sp - 2];
        val[sp - 2] = swapval;    
    }
    else {
        printf("error: stack doesn't have enough values to swap");
    }
    
}

void clear(void)
{
    sp = 0;
}

#include <ctype.h>

int getch(void);
void ungetch(int);

/* getop: get next operator or numeric operand */
int getop(char s[])
{
    int i, c;
    
    while ((s[0] = c = getch()) == ' ' || c == '\t')
        ;
    s[1] = '\0';
    if (!isdigit(c) && c != '.')
    {
        if (c != '-')
            return c;               /* not a number, minus or negative */
        
        if(!isdigit(c = getch()))   /* if next char is not digit */
        {
             ungetch(c);
             return '-';            /* minus sign */
        }
        
        s[i = 1] = c;
    } else {i = 0;}
    
    if (isdigit(c)) /* collect integer part */
        while (isdigit(s[++i] = c = getch()))
            ;
    if (c == '.')   /* collect fraction part */
        while (isdigit(s[++i] = c = getch()))
            ;
    s[i] = '\0';
    if (c != EOF)
        ungetch(c);
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
    Don't feel I need an image here. Pretty straightforward output.
</p>

[Back to Main](../readme.md)
