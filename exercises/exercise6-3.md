# Exercise 6-3

> Write a cross-referencer that prints a list of all words in a document, and, for each word, a list of the line numbers on which it occurs. 
> Remove noise words like "the", "and", and so on.

## Process
This exercise took me about a week of one hour morning sessions. 

### Decisions

**1. Storing line numbers:**
  
Line numbers are stored in an array within the `tnode` structure. I chose this approach for simplicity, but it limits the number of lines to 100. I considered using `malloc` for dynamic storage but will wait until I cover that topic.
  
**2. Combining line number and `getword` functions:**

I combined the line-numbering and word-counting into a single function. This keeps `main` simple and allows readers to infer functionality from the function name, `getwordandlinenum`. 
However, in a larger project, this could be bad practice, as it reduces reusability and makes future modifications harder.

_(Following this line of thought I should also put the exlude word check in this function as well, so it seems I am inconsistent in my logic..)_

**3. Lowercasing words:**

All words are converted to lowercase to ensure consistent word counting and case-insensitive exclusion.

**4. Linear search for exclude list:**

The exclude list is currently searched linearly. Ideally, it should be sorted, and a binary search should be used for faster lookups.

**5. Excluding words:**

Words from the exclude list are not stored, as there's no need to keep them if they won’t be used.

## Code
```c
#include <stdio.h>
#include <ctype.h>
#include <string.h>
#include <stdlib.h>

#define MAXWORD     100
#define MAXLINENUM  100

struct tnode {          /* the tree node: */
    char *word;                 /* points to the text */
    int count;                  /* number of occurrences */
    int linenums[MAXLINENUM];   /* line numbers */
    struct tnode *left;         /* left child */
    struct tnode *right;        /* right child */
};


const char excludewords[][MAXWORD] = {
    "the","and","of","to","with","it","a","at","as","are","all","is","its","in","we"
};
int numwords = sizeof(excludewords) / sizeof(excludewords[0]);

int getwordandlinenum(char *word, int *linenum, int lim);
int excludeword(char *word, int numwords);
struct tnode *addtree(struct tnode *p, char *w, int linenum);
void treeprint(struct tnode *p);

/* word cross referencer */
int main(int argc, char **argv)
{
    struct tnode *root;
    char word[MAXWORD];
    int linenum = 1;
    
    root = NULL;
    while(getwordandlinenum(word, &linenum, MAXWORD) != EOF)
        if (isalpha(word[0])) {
            if (excludeword(word, numwords))
                continue;
            root = addtree(root, word, linenum);
        }    
    treeprint(root);
    return 0;
}

/* getwordandlinenum: get next word or character from input and linenumber */
int getwordandlinenum(char *word, int *linenum, int lim)
{
    int c, getch(void);
    void ungetch(int c);
    char *w = word;
    
    while((c = getch()) != EOF) {
        if (c == ' ' || c == '\t') 
            continue;
        
        if (c == '\n') {
            (*linenum)++;
            continue;
        }
        
        if (!isalpha(c)) {
            *w++ = c;
            *w = '\0';
            return c;
        }
        
        *w++ = tolower(c);
        
        for ( ; --lim > 0; w++)
            if (!isalnum(c = getch())) {
                ungetch(c);
                break;
            } else {
                *w = tolower(c);
            }
        *w = '\0';
        return word[0];
    }
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

int excludeword(char *word, int numwords)
{
    for(int i = 0; i < numwords; i++)
        if (strcmp(word, excludewords[i]) == 0)
            return 1;
    return 0;
}

struct tnode *talloc(void);
char *my_strdup(char *);

/* addtree: add a node with w, at or below p */
struct tnode *addtree(struct tnode *p, char *w, int linenum)
{
    int cond;
    
    if (p == NULL) {    /* a new word has arrived */
        p = talloc();   /* make a new node */
        p->word = my_strdup(w);
        p->count = 1;
        p->linenums[p->count-1] = linenum;
        p->left = p->right = NULL;
    } else if ((cond = strcmp(w, p->word)) == 0) {
        p->count++;     /* repeated word */
        p->linenums[p->count-1] = linenum;
    }
    else if (cond < 0)  /* less than into left subtree */
        p->left = addtree(p->left, w, linenum);
    else            /* greater than into right subtree */
        p->right = addtree(p->right, w, linenum);
    return p;
}

/* treeprint: in-order print of tree p */
void treeprint(struct tnode *p)
{
    
    if (p != NULL) {
        treeprint(p->left);
        printf("%s ", p->word);
        for (int i = 0; i < p->count; i++) {
            printf("%d", p->linenums[i]);
            if(i < (p->count-1))
                printf(", ");
        }
        printf("\n");
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
  <image src="../assets/exercise6-3_cmds.jpg" alt="output for exercise 6-3_cmds" />
</p>
<p align="center">
  <image src="../assets/exercise6-3_data_and_result.jpg" alt="output for exercise 6-3_data_and_result" />
</p>

[Back to Main](../readme.md)
