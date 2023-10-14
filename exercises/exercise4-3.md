# Exercise 4-3

> Given the basic framework, it's straightforward to extend the calculator. Add the modulus (%) operator and provisions for negative numbers.

## Process
Although the code is quite simple and could have bashed out a solution, I decided to spent some time learning the things I didnt understand.

My first misunderstanding came when I tried to input data into the provided calculator program.
My journey to understanding of what a terminal is lead me to a few interesting resources.
- Difference between [console, terminal and shell](https://www.hanselman.com/blog/whats-the-difference-between-a-console-a-terminal-and-a-shell)
- windows cmd [background](https://devblogs.microsoft.com/commandline/windows-command-line-backgrounder/)
- reference of what windows cmd is [doing](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/windows-commands)

Also I tried to clarify my understanding of [what EOF is](https://stackoverflow.com/questions/3061135/can-we-write-an-eof-character-ourselves)
but to be honest I'm still a little fuzzy on that concept

I also went into the [FAT file system](https://averstak.tripod.com/fatdox/fatintro.htm) 
when trying to send input to the calculator with `file < data.txt`

I also learned that when debuging with gdb, you can set input from file using `start < input.file`

Also, to help my understanding of program structure, I created a simple diagram:
<p align="center">
    <image src="../assets/exercise4-3.png" alt="info for exercise4-3" />
</p>

I found its not the way I would have set up this program on my own. The idea of multiple buffers is interesting. 
So I found it a little non-intuitive to work with and visually setting it out helped.

## Code
```c
#include <stdio.h>
#include <stdlib.h> /* for atof() */

#define MAXOP   100 /* max size of operand or operator */ 
#define NUMBER  '0' /* signal that a number was found */

int getop(char []);
void push(double);
double pop(void);

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
    <image src="../assets/exercise4-3_output.png" alt="output for exercise4-3" />

</p>

[Back to Main](../readme.md)
