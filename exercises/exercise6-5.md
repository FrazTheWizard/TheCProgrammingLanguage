# Exercise 6-5

> Write a function undef that will remove a name and definition from the table maintained by lookup and install.

## Process
This one was fairly straight forward and probably took about a week for me to figure out.

Mostly I was looking into Hash tables, which was a new concept for me. 
Also freeing memory is kind of new for me too, I was trying to find a good way to 
confirm that the memory has been freed on windows, but didn't find any easy to use tools, so left that for a later time.

I reduced the hashtable size to 2 to make the visual example smaller and grouped the printing functions.

## Code
```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <ctype.h>

#define HASHSIZE 2
#define BUFSIZE 2
#define SIZE 100

struct nlist {      /* table entry */
    struct nlist *next; /* next entry in chain */
    char *name;         /* defined name */
    char *defn;         /* replacement text */
};

int getch(void);
void ungetch(int c);
int getword(char *word, int lim);
unsigned hash(char *s);
struct nlist *lookup(char *s);
struct nlist *install(char *name, char *defn);
char *my_strdup(char *);
void undef(char *name);
void printtab(void);
void removeandprint(char *name);

static struct nlist *hashtab[HASHSIZE]; /* pointer table */
char buf[BUFSIZE];  /* buffer for ungetch */
int bufp = 0;       /* next free position in buf */

int main()
{
    install("name1", "definition1");
    install("name2", "definition2");
    install("name3", "definition3");
    install("name4", "definition4");
    
    printtab();
    
    removeandprint("name1");
    removeandprint("name2");
    removeandprint("name4");
    removeandprint("name3");
    
    return 0;
}

void removeandprint(char *name)
{
    undef(name);
    printf("\nremoved '%s'\n\n", name);
    printtab();
}

void printtab(void)
{
    struct nlist *ptr;

    for(int i = 0; i < HASHSIZE; i++) { 
        if(hashtab[i] == NULL)
            printf("%d: NULL\n",i);
        else {
            printf("%d: ", i);
            for (ptr = hashtab[i]; ptr != NULL; ptr = ptr->next) {
                printf("%s-%s -> ", ptr->name, ptr->defn);
            }
            printf("NULL\n");
        }
    }
}

/* undef: remove a name and definition from the table  */
void undef(char *name) 
{    
    unsigned index = hash(name);
    struct nlist *np = hashtab[index];
    struct nlist *prev = NULL;
    
    for (; np != NULL; prev = np, np = np->next)
        if(strcmp(np->name, name) == 0)
            break;
    
    if (np == NULL) return; /* no matching entry found */

    if (prev == NULL)
        hashtab[index] = np->next; /* remove first entry at index */
    else
        prev->next = np->next;

    free(np->name);
    free(np->defn);
    free(np);
    np = NULL;
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

/* my_strdup: make a duplicate of s */
char *my_strdup(char *s)
{
    char *p;
    
    p = (char *) malloc(strlen(s) + 1); /* +1 for '\0' */
    if (p != NULL)
        strcpy(p, s);
    return p;
}

/* getword: get next word or character from input */
int getword(char *word, int lim)
{
    int c, getch(void);
    void ungetch(int);
    char *w = word;
    
    while (isspace(c = getch()))
        ;
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
  <image src="../assets/exercise6-5_cmds.jpg" alt="output for exercise 6-5_cmds" />
</p>

[Back to Main](../readme.md)
