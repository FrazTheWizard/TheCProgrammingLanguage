# Exercise 6-6

> Implement a simple version of the #define processor (i.e., no arguments) suitable for use with C programs, based on the routines of this section.
> You may also find getch and ungetch helpful.

## Process
This exercise was a good exercise in exploring internal boundaries for a program which collects input

Things I struggled with:
- where to separate direct input collection, reading `#define` keyword and handling error messages and recovery during collection
- should the program collect input as tokens, tokens with meaning (lexing), words, or gather whole line and process line
- should `getch` and `ungetch` only be in one place, or can they be spread out among the functions like in `installdefine()`

Further research about parsers, tokenizers, or interpreters, lexers:
 - A tokenizer breaks a stream of text into tokens, usually by looking for whitespace (tabs, spaces, new lines).
 - A lexer is basically a tokenizer, but it usually attaches extra context to the tokens -- this token is a number, that token is a string literal, this other token is an equality operator.
 - A parser takes the stream of tokens from the lexer and turns it into an abstract syntax tree representing the (usually) program represented by the original text.

After I finished my solution for the exercise, I looked at answers to the exercise on `clc-wiki.net` (as I usually do), 
then I found this "The C Answer Book", which had a great solution that also handled `undef` and seemed much less complicated. 
I think I will look at this book's answers for comparison to my own solutions for the remaining exercises.
I was also happy to find my solution resembled the structure in the book.

## Code
```c
#include <stdio.h>
#include <string.h>
#include <ctype.h>
#include <stdlib.h>

#define BUFSIZE 100
#define SIZE 100
#define HASHSIZE 2

enum {OUTPUT, ERROR, DEFINE};
char *macroword = "define";

struct nlist {      /* table entry */
    struct nlist *next; /* next entry in chain */
    char *name;         /* defined name */
    char *defn;         /* replacement text */
};

int lexer(char *token, int lim);
int installdefine();
unsigned hash(char *s);
struct nlist *lookup(char *s);
struct nlist *install(char *name, char *defn);
int getch(void);
void ungetch(int c);

char buf[BUFSIZE];  /* buffer for ungetch */
int bufp = 0;       /* next free position in buf */
static struct nlist *hashtab[HASHSIZE]; /* pointer table */

int main()
{
    int tokentype;
    char token[SIZE];
    
    while((tokentype = lexer(token, SIZE)) != EOF) {
        switch (tokentype) {
            case DEFINE:
                if (installdefine() == ERROR)
                    printf("ERROR installing define statement\n");
                break;
            case OUTPUT:
                printf("%s", token);
                break;
        }   
    }
    return 0;
}

/* lexer: returns OUTPUT, DEFINE or EOF. OUTPUT stored in token */
int lexer(char *token, int lim)
{
    int c;
    char *w = token, *m = macroword;
        
    c = getch();
    
    if (c == EOF)
        return EOF;
    
    *w++ = c;
    
    if (c == '#') {
        while(((c = getch()) == *m) && (*m++ != '\0'))
            *w++ = c;
        ungetch(c);
        if (*m == '\0'){
            *w = '\0';
            return DEFINE;            
        }
    }
        
    if (!isalnum(c)) {
        *w = '\0';
    } else {
        for ( ; --lim > 0; w++)
            if (!isalnum(*w = getch())) {
                ungetch(*w);
                break;
            }
        *w = '\0';        
    }
    
    struct nlist *np = lookup(token);
    if (np != NULL)
        strcpy(token, np->defn);
    
    return OUTPUT;
}

/* installdefine: add name and defn to hashtable */
int installdefine(void) {
    int tokentype;
    char token[SIZE];
    char name[SIZE], defn[SIZE];
    
    tokentype = lexer(token, SIZE);
    if (tokentype != OUTPUT || (strcmp(token, " ") != 0)) return ERROR;
    
    tokentype = lexer(token, SIZE);
    if (tokentype != OUTPUT) return ERROR;
    strcpy(name, token);
    
    tokentype = lexer(token, SIZE);
    if (tokentype != OUTPUT || (strcmp(token, " ") != 0)) return ERROR;
    
    tokentype = lexer(token, SIZE);
    if (tokentype != OUTPUT) return ERROR;
    strcpy(defn, token);
    
    struct nlist *installed = install(name, defn);
    
    if(installed == NULL)
        return ERROR;
    
    return DEFINE;
}


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

/* hash: form hash value for string s */
unsigned hash(char *s)
{
    unsigned hashval;
    
    for (hashval = 0; *s != '\0'; s++)
        hashval = *s + 31 * hashval;
    return hashval % HASHSIZE;
}

/* lookup: look for s in hashtab */
struct nlist *lookup(char *s)
{
    struct nlist *np;
    
    for (np = hashtab[hash(s)]; np != NULL; np = np->next)
        if (strcmp(s, np->name) == 0)
            return np;  /* found */
        return NULL;    /* not found */
}

/* install: put (name, defn) in hashtab */
struct nlist *install(char *name, char *defn)
{
    struct nlist *np;
    unsigned hashval;
    
    if ((np = lookup(name)) == NULL) {  /* not found */
        np = (struct nlist *) malloc(sizeof(*np));
        
        if (np == NULL || (np->name = strdup(name)) == NULL)
            return NULL;
        hashval = hash(name);
        np->next = hashtab[hashval];
        hashtab[hashval] = np;
    } else      /* already there */
        free ((void*) np->defn); /* free previous defn */
    if ((np->defn = strdup(defn)) == NULL)
        return NULL;
    return np;
}
```

## Output
<p align="center">
  <image src="../assets/exercise6-6_cmds.jpg" alt="output for exercise 6-6_cmds" />
</p>
<p align="center">
  <image src="../assets/exercise6-6_data_and_result.jpg" alt="output for exercise6-6 data and result" />
</p>

[Back to Main](../readme.md)
