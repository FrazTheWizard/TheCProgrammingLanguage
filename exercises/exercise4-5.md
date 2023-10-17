# Exercise 4-5

> Add access to library functions like sin, exp, and pow. See <math.h> in Appendix B, Section 4.

## Process
Just added the math.h header and functions in main

Changed the swap function to be letter 'w', so that s could be 'sin'

Slight tangent, but starting to see that classic thing where features keep getting added to programs and it creates a need
for the program structure to be rethought. Some potential issues:
- how can the user know all the input codes? Maybe some sort of system to input letters
- should calculation be done in main? maybe a rule should be that all the code for computing the functions should be separated, to help maintainability.

(Noticed I made an error in the previous exercise with the print function, so fixed that)

## Code
```c
#include <stdio.h>
#include <stdlib.h> /* for atof() */
#include <math.h>

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
            case 's':
                push(sin(pop()));
                break;
            case 'e':
                push(exp(pop()));
                break;
            case 'p':
                op2 = pop();
                push(pow(pop(), op2));
                break;
            case 't':   /* print the top character */
                printf("\t%.8g\n", top());
                break;
            case 'd':   /* duplicate the top character */
                push(top());
                break;
            case 'w':   /* swap(w) the first two characters on stack */
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
    <image src="../assets/exercise4-5.jpg" alt="output for exercise4-5" />
</p>

[Back to Main](../readme.md)
