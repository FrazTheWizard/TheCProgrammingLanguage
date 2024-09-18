# Exercise 5-20
> Expand dcl to handle declarations with function argument types, qualifiers like const, and so on.

## Process
Wow, so this exercise took me a month and two days of morning hour sessions. 
It was quite challenging at times but I learnt a lot about C grammer and syntax, recursive functions and parsing.

I spent a while reading and playing around with the reference manual sections on declarations. 
It has actually been nice to see C formally laid out like this to see the limits of the grammer,
rather than a 'usually works but vague understanding' of what precisely is going on. 

I also feel like I have a better grasp of grammer parsing vs semantic meaning parsing, where the language
is allowed to grammatically express something without knowing how it all fits together yet. This is a way to
try to keep the grammer parsing simple and modular. I found out about the [lexer hack](https://en.wikipedia.org/wiki/Lexer_hack) 
which passes semantic meaning back to the grammer parsing to help parsing, which is considered bad flow.

This exercise also had me bumping up against my understanding of precedence order. One technique for figuring
it out is the [spiral rule](https://c-faq.com/decl/spiral.anderson.html), but I found [postfix > prefix](https://stackoverflow.com/questions/16260417/the-spiral-rule-about-declarations-when-is-it-in-error)
easier to think about when determining an evaluation order.

## Approach
My approach was try to keep to the original `dcl` program, add new syntax to `gettoken` and add additional 
recursive functions. 

The direct syntax parsing gets handled by `gettoken` which has become kind of large, but it's nice to have
just one function that handles all the tricky stuff, then the recursive parsing handles the logic.

Comma separators in a declaration, print out the declaration, but keep the specifiers for the next part.

I wanted function parameters to use the same arrays as declarations, so after the recursive functions capture the 
storage, qualifiers and types, the output is stored in `returnspec` until needed, which frees them up for possible 
function parameters. 

`dirdcl` determines between declaration `identifiers` and function `paramidentifiers` by assuming the first identifier 
in a declaration is for the declaration and all following identifiers must be for parameters, which feels a bit wonky and 
could maybe be improved, but works.

## Decisions
- lines need to terminate with `;` to be considered valid
- any characters between `=` and `;` are counted as an initialiser and just copied and printed as is.
- identifiers are alpha numeric and can have `_` underscores.
- each comma separated declaration is printed onto a new line.
- blank lines are skipped
- there is some error checking, where the program prints `error pasing line` and skips the line.

The grammer rules I decided to follow were:
```
declaration:
    declaration-specifiers init-declarator-list*opt* ;
    
declaration-specifiers:
    storage-class-specifier declaration-specifiers*opt*
    type-specifier declaration-specifiers*opt*
    type-qualifier declaration-specifiers*opt*
    
init-declarator-list:
    init-declarator
    init-declarator-list , init-declarator
    
init-declarator:
    declarator
    declarator = initialiser

declarator:
    pointer*opt* direct-declarator
    
direct-declarator:
    identifier
    ( declarator )
    direct-declarator [ constant-expression*opt* ]
    direct-declarator ( parameter-type-list )
    direct-declarator ( identifier-list*opt* )
    
pointer:
    * type-qualifier-list*opt*
    * type-qualifier-list*opt* pointer
    
type-qualifier-list:
    type-qualifier
    type-qualifier-list type-qualifier

parameter-type-list:
    parameter-list
    parameter-list , ... // didn't do this one
    
parameter-list:
    parameter-declaration,
    parameter-list , parameter-declaration
    
parameter-declaration:
    declaration-specifiers declarator
    declaration-specifiers abstract-declarator*opt* // didn't do this one
```





## Code
```c
#include <stdio.h>
#include <string.h>
#include <ctype.h>

#define MAX 1000
#define NTYPES 9
#define NSTORAGE 4
#define NQUALIFIERS 2

const char *TYPELIST[NTYPES] = {
    "void", "char", "short", "int", "long", 
    "float", "double", "signed", "unsigned"
};
const char *STORAGELIST[NSTORAGE] = {
    "auto", "register","static", "extern"
};
const char *QUALIFIERLIST[NQUALIFIERS] = {
    "const", "volatile"
};

enum {
    TSTORAGE, TTYPE, TQUALIFIER, TIDENTIFIER, TPOINTER, 
    TPARENSBEGIN, TPARENSEND, TNEWLINE, TSEMICOLON,
    TARRAY, TBRACKETSEND, TCOMMA, TINITIALISER
};

void dclspec(void);
void initdcllist(void);
void initdcl(void);
void dcl(void);
void pointer(char *);
void dirdcl(void);
void paramtypelist(void);
void parameterlist(void);
void parameterdcl(void);

int gettoken(void);
void printdcl(void);

char paramidentifier[MAX];
char initialiser[MAX];
char qualifiers[MAX];
char identifier[MAX];
char returnspec[MAX];
char storage[MAX];
char types[MAX];
char token[MAX];
char out[MAX];
int tokentype;

/* dcl: recursive descent parse c declarations */
int main()
{
    while (gettoken() != EOF)
    {
        if (tokentype == TNEWLINE) {
            continue;
        }
        
        initialiser[0] = '\0';
        identifier[0] = '\0';
        qualifiers[0] = '\0';
        storage[0] = '\0';
        types[0] = '\0';
        out[0] = '\0';
        returnspec[0] = '\0';
        
        dclspec();
        
        strcat(returnspec, storage);
        strcat(returnspec, qualifiers);
        strcat(returnspec, types);
        
        qualifiers[0] = '\0';
        storage[0] = '\0';
        types[0] = '\0';
        
        initdcllist();
        
        while (tokentype != TSEMICOLON && tokentype != TNEWLINE){
            gettoken();
        }
        
        if (tokentype == TNEWLINE) {
            printf("error parsing line\n");
        }
        
        if (tokentype == TSEMICOLON) {
            printdcl();
        }
    }
    
    return 0;
}

void dclspec(void)
{
    if(tokentype == TSTORAGE) {
        strcat(storage, " ");
        strcat(storage, token);
        gettoken();
        dclspec();
    }
    if(tokentype == TTYPE) {
        strcat(types, " ");
        strcat(types, token);
        gettoken();
        dclspec();
    }
    if(tokentype == TQUALIFIER) {
        strcat(qualifiers, " ");
        strcat(qualifiers, token);
        gettoken();
        dclspec();
    }
}

void initdcllist(void)
{
    initdcl();
    if(tokentype == TCOMMA) {
        printdcl();
        out[0] = '\0', identifier[0] = '\0', initialiser[0] = '\0';
        gettoken();
        initdcllist();
    }
}

void initdcl(void)
{
    dcl();
    if (tokentype == TINITIALISER) {
        strcat(initialiser, token);
        gettoken();
    }
}

void dcl(void)
{
    char pointerout[MAX] = {};
    
    pointer(pointerout);
    dirdcl();
    
    strcat(out, pointerout);
}

void pointer(char *pointerout)
{
    if (tokentype == TPOINTER) {
        gettoken();
        if (tokentype == TQUALIFIER) {
            strcat(pointerout, " ");
            strcat(pointerout, token);
            gettoken();
        }
        if (tokentype == TPOINTER) {
            pointer(pointerout);
        }
        
        strcat(pointerout, " pointer to");
    }
}

void dirdcl(void)
{
    if ( tokentype == TIDENTIFIER || tokentype == TPARENSBEGIN) 
    {
        switch(tokentype) {
            case TIDENTIFIER:
                if (identifier[0] == '\0') {
                    strcat(identifier, token);
                } else if (paramidentifier[0] == '\0'){
                    strcat(paramidentifier, " ");
                    strcat(paramidentifier, token);
                } else {
                    printf("error: unkown identifier %s\n", token);
                }
                gettoken();
                break;
            case TPARENSBEGIN:
                gettoken();
                dcl();
                if (tokentype == TPARENSEND) {
                    gettoken();
                } else {
                    printf("error: missing ')'\n");
                }
                break;
        }
        
        switch(tokentype) {
            case TARRAY:
                strcat(out, " array");
                strcat(out, token);
                strcat(out, " of");
                gettoken();
                break;
            case TPARENSBEGIN:
                strcat(out, " function(");
                gettoken();
                parameterlist();
                strcat(out, " ) returning");
                if (tokentype == TPARENSEND) {
                    gettoken();
                } else {
                    printf("error: missing ')'\n");
                }
                break;
        }
    }
}

void parameterlist(void)
{
    parameterdcl();
    strcat(out, storage);
    strcat(out, qualifiers);
    strcat(out, types);
    strcat(out, paramidentifier);
    storage[0] = '\0'; qualifiers[0] = '\0'; types[0] = '\0';
    paramidentifier[0] = '\0';
    if (tokentype == TCOMMA) {
        strcat(out, ",");
        gettoken();
        parameterlist();
    }
}

void parameterdcl(void)
{
    dclspec();
    dcl();
}


void printdcl(void) {
    printf("%s:%s%s", identifier, out, returnspec);
    if (initialiser[0] != '\0') {
        printf(" %s", initialiser);
    }
    printf("\n");
}

int gettoken(void)
{
    int c, getch(void);
    void ungetch(int);
    char *p = token; 
    int word, found = 0;
    
    while ((c = getch()) == ' ' || c == '\t')
        ;
    
    if (isalpha(c)) {
        for (*p++ = c; isalnum(c = getch()) || c == '_';)
            *p++ = c;
        *p = '\0';
        ungetch(c);
        for (word = 0; word < NSTORAGE && !found; word++) {
            if (strcmp(STORAGELIST[word], token) == 0) {
                found = 1;
                return tokentype = TSTORAGE;
            }
        }
        
        for (word = 0; word < NTYPES && !found; word++) {
            if (strcmp(TYPELIST[word], token) == 0) {
                found = 1;
                return tokentype = TTYPE;
            }
        }
        
        for (word = 0; word < NQUALIFIERS && !found; word++) {
            if (strcmp(QUALIFIERLIST[word], token) == 0) {
                found = 1;
                return tokentype = TQUALIFIER;
            }
        }
        
        return tokentype = TIDENTIFIER;
    }
    
    if (c == '*') {
        return tokentype = TPOINTER;
    }
    
    if (c == '(') {
        return tokentype = TPARENSBEGIN;
    }
    
    if (c == ')') {
        return tokentype = TPARENSEND;
    }
    
    if (c == '\n') {
        return tokentype = TNEWLINE;
    }
    
    if (c == ';') {
        return tokentype = TSEMICOLON;
    }
    
    if (c == '[') {
        
        while (c != ']' && c != ',' && c != ';' && c != '\n' && c != EOF) {
            *p++ = c;
            c = getch();
        }
        
        if (c == ']') {
            *p++ = c;
        } else {
            ungetch(c);
        }
        
        *p = '\0';
        return tokentype = TARRAY;
    }
    
    if (c == ',') {
        return tokentype = TCOMMA;
    }
    
    if (c == '=') {
        while(c != ',' && c != ';' && c !='\n' && c != EOF) {
            *p++ = c;
            c = getch();
        }
        *p = '\0';
        ungetch(c);
        return tokentype = TINITIALISER;
    }
    
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
  <image src="../assets/exercise5-20_cmds.jpg" alt="output for exercise 5-20_cmds" />
</p>
<p align="center">
  <image src="../assets/exercise5-20_data_and_result.jpg" alt="output for exercise 5-20_data_and_result" />
</p>

[Back to Main](../readme.md)
