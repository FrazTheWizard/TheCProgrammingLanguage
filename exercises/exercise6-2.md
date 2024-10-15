# Exercise 6-2

> Write a program that reads a C program and prints in alphabetical order each group of variable names that are identical in the first 6 characters, but different somewhere thereafter.
> Don't count words within strings and comments. Make 6 a parameter that can be set from the command line.

## Process
Surprisingly this one was pretty straightforward and took me two mornings of one hour sessions. I'm also surprised the previous exercise and this exercise aren't more about structures.

I added two functions to handle the compare size `parseargs` and `strtrunc`:
- `parseargs` checks for arguments matching '-int' and ignores all else.
- `strtrunc` truncates each word by adding a `'\0'` null in each found word at the compare size.

Took my logic from previous exercise in skipping string constants and comments then condensed and improved it to handle concurrent comments or strings and to fit within `getword`.

## Code
```c
#include <stdio.h>
#include <ctype.h>
#include <string.h>
#include <stdlib.h>

#define MAXWORD 100

struct tnode {          /* the tree node: */
    char *word;             /* points to the text */
    int count;              /* number of occurrences */
    struct tnode *left;     /* left child */
    struct tnode *right;    /* right child */
};

struct tnode *addtree(struct tnode *, char *);
void treeprint(struct tnode *);
int getword(char *, int);
int parseargs(int , char **);
void strtrunc(char *, int);

/* word frequency count */
int main(int argc, char **argv)
{
    struct tnode *root;
    char word[MAXWORD];
    
    int cmpsize = parseargs(argc, argv);
    printf("compare word size: %d\n", cmpsize ? cmpsize : MAXWORD);
    
    root = NULL;
    while (getword(word, MAXWORD) != EOF)
        if (isalpha(word[0])) {
            if (cmpsize > 0)
                strtrunc(word, cmpsize);
            root = addtree(root, word);
        }
    treeprint(root);
    return 0;
}

int parseargs(int argc, char **argv)
{
    if (argc == 1 || argc > 2) return 0;
    
    argc--, argv++; /* skip program filepath */
    if (*(*argv)++ == '-') {
       return atoi(*argv); 
    } else
        return 0;
}

/* getword: get next word or character from input */
int getword(char *word, int lim)
{
    int c, getch(void);
    void ungetch(int);
    char *w = word;
    int prevchar = '\0';
    
    while (isspace(c = getch()))
        ;
    
    /* skip string constants */
    while (c == '"' && c != EOF) {
        while((c=getch()) != '"' && c != EOF)
            ;
        if (c == '"')
            c = getch();
    }
    
    /* skip comments */
    while (c == '/') {
        if ((c = getch()) == '/' || c == '*') {
            if (c == '/') {
                while ((c = getch()) != '\n' && c != EOF)
                    ;
            } else if (c == '*') {            
                while ((c = getchar()) != EOF) {
                    if (prevchar == '*' && c == '/'){
                        c = getch();
                        break;
                    }
                    prevchar = c;
                }
            }
        } else {
            ungetch(c);
            c = '/';
            break;
        }
    }
    
    if (c != EOF)
        *w++ = c;
    if (!isalpha(c)) {
        *w = '\0';
        return c;
    }
    for ( ; --lim > 0; w++)
        if (!isalnum(*w = getch())) {
            ungetch(*w);
            break;
        }
    *w = '\0';
    return word[0];
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

void strtrunc (char *word, int n)
{
    if (strlen(word) > n)
        word[n] = '\0';
}

struct tnode *talloc(void);
char *my_strdup(char *);

/* addtree: add a node with w, at or below p */
struct tnode *addtree(struct tnode *p, char *w)
{
    int cond;
    
    if (p == NULL) {    /* a new word has arrived */
        p = talloc();   /* make a new node */
        p->word = my_strdup(w);
        p->count = 1;
        p->left = p->right = NULL;
    } else if ((cond = strcmp(w, p->word)) == 0)
        p->count++;     /* repeated word */
    else if (cond < 0)  /* less than into left subtree */
        p->left = addtree(p->left, w);
    else            /* greater than into right subtree */
        p->right = addtree(p->right, w);
    return p;
}

/* treeprint: in-order print of tree p */
void treeprint(struct tnode *p)
{
    if (p != NULL) {
        treeprint(p->left);
        printf("%4d %s\n", p->count, p->word);
        treeprint(p->right);
    }
}

/* talloc: make a tnode */
struct tnode *talloc(void)
{
    return (struct tnode *) malloc(sizeof(struct tnode));
}

/* my_strdup: make a duplicate of s */
char *my_strdup(char *s)
{
    char *p;
    
    p = (char *) malloc(strlen(s) + 1); /* +1 for '\0' */
    if (p != NULL)
        strcpy(p, s);
    return p;
}
```

## Output
<p align="center">
  <image src="../assets/exercise6-2_cmds.jpg" alt="output for exercise 6-2_cmds" />
</p>
<p align="center">
  <image src="../assets/exercise6-2_data_and_result.jpg" alt="output for exercise 6-2_data_and_result" />
</p>

[Back to Main](../readme.md)
