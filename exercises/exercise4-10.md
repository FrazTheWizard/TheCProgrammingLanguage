# Exercise 4-10

> An alternate organization uses getline to read an entire input line; this makes getch and ungetch unnecessary. Revise the calculator to use this approach.

## Process
This one proved to be a little bit of thinking and work

Initally, I had a solution just like [zer0325](https://www.youtube.com/watch?v=fhqk3rZfqg0)'s solution which used a `while(getline() > 0){rest of program}` structure

However:
- I want to avoid the double while loop, as I find it makes reading code harder
- in the spirit of the chapter being about external variables and trying to keep to the original code structure,
I opted to put the **getline** function in the **getop** function, so **main** doesn't need to know about **getline**.

The program seems to work based on the minimal testing I did, but I still feel uneasy about it. However I'm happy to call the exercise done.

Also, I used the base calculator program, to cut down on code to read

## Code
```c
#include <stdio.h>
#include <stdlib.h> /* for atof() */

#define MAXOP   100 /* max size of operand or operator */ 
#define NUMBER  '0' /* signal that a number was found */

int getopfromline(char []);
void push(double);
double pop(void);

/* reverse Polish calculator */
int main()
{
    int type;
    double op2;
    char s[MAXOP];

    while((type = getopfromline(s)) != EOF) {
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
#define BUFSIZE 100 /* max line length */

char inputstring[BUFSIZE];
int inputindex, inputlen = 0;
int getline(char s[0], int lim);

/* getopfromline: get next operator or numeric operand from line */
int getopfromline(char s[])
{
    /*
        read in new line when:
        - inital line load:     inputlen == 0
        - reached end of line:  inputindex >= inputlen
    */
    if( inputlen == 0 || inputindex >= inputlen)
    {
        if((inputlen = getline(inputstring, BUFSIZE)) > 0)
            inputindex = 0; /* start at begining of new line */
        else 
            return EOF;
    }
    
    int i, c;
    
    while ((s[0] = c = inputstring[inputindex++]) == ' ' || c == '\t')
        ;
    s[1] = '\0';
    
    if (!isdigit(c) && c != '.')
            return c; /* not a number */
    
    i = 0;
    if (isdigit(c)) /* collect integer part */
        while (isdigit(s[++i] = c = inputstring[inputindex++]))
            ;
    if (c == '.')   /* collect fraction part */
        while (isdigit(s[++i] = c = inputstring[inputindex++]))
            ;
    s[i] = '\0';
    if (c != EOF)
        --inputindex;
    return NUMBER;
}

/* getline: get line into s, return length */
int getline(char s[], int lim)
{
    int c, i;
    
    i = 0;
    while (--lim > 0 && (c=getchar()) != EOF && c != '\n')
        s[i++] = c;
    if(c == '\n')
        s[i++] = c;
    s[i] = '\0';
    return i;
}
```

## Output
<p align="center">
    No Output
</p>

[Back to Main](../readme.md)
