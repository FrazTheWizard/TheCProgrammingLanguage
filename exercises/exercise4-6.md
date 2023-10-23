# Exercise 4-6

> Add commands for handling variables. (It's easy to provide twenty-six variables with single-letter names.) Add a variable for the most recently printed value.

## Process
Had a bit of trouble fitting in variables into the existing framework and tried a few different ways that didn't work

Eventually settled on a way to set and get variables using this syntax:
- `v[a-z]` : save variable
- `g[a-z]` : get variable

I guess I based it on the windows calculator `Memory Recall` and and `Memory Store` functions that I'm familiar with.

I also used the get syntax to get the recently printed value:
- `g!` : recently printed value

Lastly I added a help function because I forget what they are sometimes...

After finishing my code, I checked zer0325's [solution](https://www.youtube.com/watch?v=_jRhgKvYQI0)

I really like his solution, its simplier to understand and elegant. He did however change from the structure in the book a bit more than I've been trying to do. 
But maybe the point is to develop your own structure...

Things I liked about his code:
- switch between math operations, commands and variables
- using uppercase vs lowercase letters for variables vs commands
- doing the check after processing the input number in `getop()` seems a better way to structure that function

Things I didn't like:
- separating the variable name and the '=' to set the variable. Maybe it's just different to my design, but I like being able to set or get in 1 command
- I don't think I agree with most recently printed character variable always being at the top of the stack. For instance, you may want the recently printed variable
in the previous calculation to be included at the end of a long calculation you are doing, where the last printed value will no longer be at the top of the stack..

## Code
```c
#include <stdio.h>
#include <stdlib.h> /* for atof() */
#include <math.h>

#define MAXOP   100 /* max size of operand or operator */ 
#define NUMBER  '0' /* signal that a number was found */
#define SETVARIABLE 'v'
#define GETVARIABLE 'g'
#define VARSIZE 26

int getop(char[]);
void push(double);
double pop(void);
double top(void);
void swap(void);
void clear(void);
void help(void);

double vars[VARSIZE];
double recentlyprinted;

/* reverse Polish calculator */
int main()
{
    int type;
    double op2;
    char s[MAXOP];
    
    help();
    
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
            case '%': /* modulo */
                op2 = pop();
                if (op2 != 0.0)
                    push((int) pop() % (int) op2);
                else
                    printf("error: zero divisor\n");                
                break;
            case 's': /* sin(x) */
                push(sin(pop()));
                break;
            case 'e': /* exp(x) */
                push(exp(pop()));
                break;
            case 'p': /* pow(x,y) */
                op2 = pop();
                push(pow(pop(), op2));
                break;
            case 't':   /* print the top character */
                recentlyprinted = top();
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
            case 'v':
                if(s[0] == 'E')
                    printf("error: unknown variable %c\n", s[1]);
                else {
                    vars[s[0] - 'a'] = top();
                    printf("stored variable '%c' : %g\n", s[0], top());    
                }
                break;
            case 'g':
                if(s[0] == 'E')
                    printf("error: unknown variable '%c'\n", s[1]);
                else if(s[0] == '!') {
                    printf("get recently printed variable : %g\n", recentlyprinted);
                    push(recentlyprinted);
                }
                else {
                    printf("get variable '%c': %g\n", s[0], vars[s[0] - 'a']);
                    push(vars[s[0] - 'a']);
                }
                break;
            case 'h': /* help */
                help();
                break;
            case '\n':
                recentlyprinted = top();
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
    i = 0;
    
    while ((s[0] = c = getch()) == ' ' || c == '\t')
        ;
    s[1] = '\0';
    if (!isdigit(c) && c != '.') {
        switch(c) {
            case '-': 
                if(isdigit(c = getch())) {
                    s[i = 1] = c;   /* begin negative number */    
                } else {    
                    ungetch(c); 
                    return '-';     /* minus sign */
                }
                break;
            case 'v': /* set variable */
                if(isalpha(c = getch()))
                    s[0] = tolower(c);
                else {
                    s[0] = 'E';
                    s[1] = c;
                }
                return SETVARIABLE;
                break;
            case 'g': /* get variable */
                if(isalpha(c = getch()))
                    s[0] = tolower(c);
                else if (c == '!')
                    s[0] = '!';
                else {
                    s[0] = 'E';
                    s[1] = c;
                }
                return GETVARIABLE;
                break;
            default:
                return c;
                break;
        }
    }
    
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

void help(void)
{
    printf("reverse polish calculator\n\n");
    printf("%% modulo\ns sin(x)\ne exp(x)\np pow(x, y)\n");
    printf("t print top character on stack\nd duplicate top character on stack\n");
    printf("w swap top two characters on stack\nc clear stack\n");
    printf("h help\n\n");
}
```

## Output
<p align="center">
    <image src="../assets/exercise4-6.jpg" alt="output for exercise4-6" />
</p>

[Back to Main](../readme.md)
