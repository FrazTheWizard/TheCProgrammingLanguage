# Exercise 6-1

> Our version of **getword** does not properly handle underscores, string constants, 
> comments, or preprocessor control lines. Write a better version. 

## Process
This exercise took me about two weeks of 1-hour morning sessions and got me reflecting 
on program structure.

After completing my version and reviewing other solutions, I noticed many added things
like an enum token system, processed a lot in main, or inserted a separate stream filter. 
(Interestingly, most didn’t handle multiline conditional preprocessor lines or 
`EOF` during character skipping.)

Besides solving the problem, I see the secondary goal as optimizing for 
future readers (which could be you). Clever token systems, extra logic in main or 
additional stream filters increase cognitive load, making future changes harder and 
I think should be avoided if possible.

I tried to keep my addition to `getword` "mentally collapsable". I think adding an 
'else if' type solution makes the code easier to parse in your head when reading 
than introducing an extra concept like tokens.

I've added three short functions to getword that I think fit within it's flow, 
which I could maybe simplify further with more time, but I'm happy with it.

My solution handles multiline conditional preprocessor statements and if they find `EOF` during
skipping part of the stream, they `ungetch` EOF, and leave it for the next pass.

I am assuming the question implys the following improvements:

- **Underscores are part of identifiers**  
  > Appendix A2.3: Identifiers  
  >  
  > An identifier is a sequence of letters and digits. The first character must be a letter, and the **underscore (`_`) counts as a letter**.

- **Ignore anything within string constants**  
  Example:  
  ```c
  "ignore this"
  ```

- **Ignore single-line and multi-line comments**
  ```c
  // ignore this  
  /* ignore this */
  ```

- **Ignore single-line and conditional multi-line preprocessor directives**
  ```c
  #directive ignore this

  #if ignore this #endif
  #ifdef ignore this #endif
  #ifndef ignore this #endif
  ```
## Code
```c
#include <stdio.h>
#include <ctype.h>
#include <string.h>

int getword(char *word, int lim);
int getch(void);
void ungetch(int);

int main()
{
    char word[100];
    while(getword(word, 100) != EOF)
        printf("%s\n", word);
    return 0;
}

/* getword: get next word or character from input */
int getword(char *word, int lim)
{
    void skipstringconstant(void);
    void skipcomment(int c);
    void skippreprocessor(int c, char *word, char *w);
    int c;
    char *w = word;
    
    while (isspace(c = getch()))
        ;
    if (c != EOF)
        *w++ = c;
    
    if (c == '"') {
        strcat(word, "\"");
        skipstringconstant();
        return '"';
    } 
    
    if (c == '/') {
        if ((c = getch()) == '/' || c == '*') {
            *w++ = c;
            *w = '\0';
            skipcomment(c);            
            return '/';
        }
    }
    
    if (c == '#') {
        skippreprocessor(c, word, w);
        return '#';        
    }
    
    if (!isalpha(c) && c != '_') {
        *w = '\0';
        return c;
    }
    for ( ; --lim > 0; w++)
        if (!isalnum(*w = getch()) && *w != '_') {
            ungetch(*w);
            break;
        }
    *w = '\0';
    return word[0];
}

void skipstringconstant(void)
{
    int c;
    while((c = getch()) != '"' && c != EOF)
        ;
    if (c == EOF)
        ungetch(c);
}

void skipcomment(int c)
{
    if (c == '/') {
        while ((c = getch()) != EOF && c != '\n')
            ;
    } else if (c == '*') {
        int prevchar = '\0';
        while ((c = getchar()) != EOF) {
            if (prevchar == '*' && c == '/')
                break;
            prevchar = c;
        }
    }
    if (c == EOF)
        ungetch(c);
}

void skippreprocessor(int c, char *word, char *w)
{
    while ((c = getch()) != EOF && !isspace(c))
        *w++ = c;
    *w = '\0';
    
    int conditional = strcmp(word, "#if") == 0 || 
                      strcmp(word, "#ifdef") == 0 || 
                      strcmp(word, "#ifndef") == 0;
        
    if (conditional) {
        char endword[1000], *e;
        int notfound = 1;
        
        while (notfound) {
            e = endword;
            while ((c = getch()) != EOF && !isspace(c))
                *e++ = c;
            *e = '\0';
            
            if (c == EOF) {
                notfound = 0;
                ungetch(EOF);
            } 
            else if(strcmp(endword, "#endif") == 0) {
                notfound = 0;
                ungetch(c);
            }
        }
    }
    
    while ((c=getch()) != EOF && c != '\n')
        ;
    if (c == EOF) 
        ungetch(c);
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
  <image src="../assets/exercise6-1_cmds.jpg" alt="output for exercise 6-1_cmds" />
</p>
<p align="center">
  <image src="../assets/exercise6-1_data_and_result.jpg" alt="output for exercise 6-1_data_and_result" />
</p>

[Back to Main](../readme.md)

