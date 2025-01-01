# Exercise 7-5

> Rewrite the postfix calculator of Chapter 4 to use scanf and/or sscanf to do the input and number conversion

## Process
This one was challenging, took about a day. Took a while to wrap my head around the edgecases of how `scanf` works.

### Attempt 1
First I tried to solve the problem by reading in numbers using `scanf("%f", &n)` but as you encounter a `+` or `-` character it could either be:
- a sign for a number `+22 -22`
- plus `+` or minus `-`

By the time you have found out which it is, you have consumed the input, so you can't recover from the error.

### Attempt 2
I realised that `scanf` and `sscanf` would be a good combo to read a string of input and then parse that string, so you can recover from that error. 

I built up my solution using a static buffer and static pointer to keep track of where the program has parsed so far between calls to `getop()`.

One functionality I wanted to retain from the original calculator was the abilty to have consecutive characters so:
- `1 1 + 2 -`
- `1 1+2-`

should produce the same result.

In order to deal with not finding a way to read `\n`, I added a `p` case to print whatever is on the stack instead.

My solution works, but it does feel a bit clunky and I wonder if some execution paths might produce errors...

### Other Answers - CLC Wiki
I really like the simplicity of their solutions using `while (scanf("%s%c", s, &e) == 2)` instead of a `getop()` function.

You can use the character after the input to see what to do next and if it fails, you much be at the end of input.

However that doesn't allow for the clumped input that I wanted to implement eg. `1 1+2-`.

### Other Answers - The C Answer Book
This solution was way simplier than I had thought. They just replace `getchar()` with `scanf(%c, &c)` in `getop()`, but they did say it doesn't improve the solution.
    
## Code
```c
#include <stdio.h>
#include <ctype.h>

#define MAXOP 100
#define MAXBUF 100
#define MAXVAL 100 /* maximum depth of val stack */
#define NUMBER 0

void skipnum(char **sp);
int getop(double *num);
void push(double);
double pop(void);

int sp = 0;         /* next free stack position */
double val[MAXVAL]; /* values stack */

int main()
{
    int type;
    double num, op2;
    while((type = getop(&num)) != EOF) {
        switch (type) {
            case NUMBER:
                push(num);
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
            case 'p':
                printf("\t%.8g\n", pop());
                break;
            default:
                printf("error: unknown command %s\n", num);
                break;  
        }
    }
}

/* getop: get next operator or numeric operand */
int getop(double *num)
{
    static char buf[MAXBUF];
    static char *bufp = buf;
    int ret;
    double f;
    char c;
    
    /* at start or end of buffer, read more input */
    if (bufp == buf || *bufp == '\0') {
        ret = scanf("%s", buf);
        
        if (ret == EOF || ret == 0)
            return EOF;
        else
            bufp = buf;
    }
        
    ret = sscanf(bufp, "%lf", &f);
    if (ret > 0) {
        skipnum(&bufp);
        *num = f;
        return NUMBER;
    }
    
    ret = sscanf(bufp, " %c", &c);
    if (ret > 0) {
        bufp++;
        return c;
    }
}

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

/* skipnum: move pointer past a floating point number */
void skipnum(char **bufp)
{
    if (**bufp == '-' || **bufp == '+')
        (*bufp)++;
    while (isdigit(**bufp))
        (*bufp)++;
    if (**bufp == '.')
        (*bufp)++;
    while (isdigit(**bufp))
        (*bufp)++;
}
```

## Output
<p align="center">
  <image src="../assets/exercise7-5_cmds.jpg" alt="output for exercise7-5 cmds" />
</p>

<p align="center">
  <image src="../assets/exercise7-5_data_and_result.jpg" alt="output for exercise7-5 data and result" />
</p>

[Back to Main](../readme.md)
