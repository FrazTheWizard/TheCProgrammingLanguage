# Exercise 6-4

> Write a program that prints the distinct words in its input sorted into decreasing order of frequency of occurrence. Precede each word by its count.

## Process
This one took me two weeks of morning sessions and I learnt a lot more about using pointers to structures and recursive functions.

**1. First Approach - sorting tree directly**

My first, naive attempt was trying to sort the binary tree in place. My thought being, if the data was already stored in the tree, 
I could sort it recursively by swapping nodes. After about a week building that solution, I realized that sorting the tree would need
too many full tree traversals to locate higher or lower values on other branches.

**2. Second Approach - array sort**

My second idea was to traverse the tree, building a sorted array and printing directly from it. However, I didn't want to use more memory.

**3. Third Approach - print tree by frequency without extra memory (selected solution)**

Repeatedly traverse the tree to find the largest frequency, print nodes matching that count, and repeat with decreasing counts until all nodes are printed.
I like this soution for not using extra memory and naturally keeping the words in alphabetical order.

### After Thought
After looking at solutions online, I found a better approach: 

- store pointers to each node in an array while building the tree
- quicksort the array by frequency
- print the array

I'm happy with my solution for not using extra memory but the memory it saves is so minimal compared to the loss in performance and in reality I would use the array quicksort method!

## Code
```c
#include <stdio.h>
#include <ctype.h>
#include <string.h>
#include <stdlib.h>

#define MAXWORD 100

struct tnode {          /* the tree node: */
    char *word;                 /* points to the text */
    int count;                  /* number of occurrences */
    struct tnode *left;         /* left child */
    struct tnode *right;        /* right child */
};

/* unchanged */
int getword(char *word, int lim);
void treeprint(struct tnode *p);

/* modified - added wordcount */
struct tnode *addtree(struct tnode *p, char *w, int *wordcount);

/* added */
void sortedtreeprint(struct tnode *p, int wordcount);
void findnextlargest(struct tnode *p, int *largest, int max);
void treeprintvalue(struct tnode *p, int target, int *wordcount);

/* word occurence counter */
int main(int argc, char **argv)
{
    struct tnode *root;
    char word[MAXWORD];
    int wordcount = 0;
    
    root = NULL;
    while(getword(word, MAXWORD) != EOF)
        if (isalpha(word[0]))
            root = addtree(root, word, &wordcount);
    
    if (wordcount > 0)
        sortedtreeprint(root, wordcount);
    
    return 0;
}

/* sortedtreeprint: print tree in reverse sorted order */
void sortedtreeprint(struct tnode *root, int wordcount)
{
    int max = 0;
    int largest = 1;
    while (wordcount) {
        findnextlargest(root, &largest, max);
        treeprintvalue(root, largest, &wordcount);
        max = largest;
        largest = 1;    
    }
}

/* findnextlargest: find largest count in tree up to max value. 0 is no max */
void findnextlargest(struct tnode *p, int *largest, int max)
{
    if (p != NULL) {
        findnextlargest(p->left, largest, max);
        if (max) {
            if ((p->count > *largest) && (p->count < max)) {
                *largest = p->count;    
            }
        } else if (p->count > *largest) {
            *largest = p->count;
        }
        findnextlargest(p->right, largest, max);
    }
}

/* treeprintvalue: print nodes that match value and decrease wordcount */
void treeprintvalue(struct tnode *p, int target, int *wordcount)
{
    if (p != NULL) {
        treeprintvalue(p->left, target, wordcount);
        if (p->count == target) {
            (*wordcount)--;
            printf("%d %s\n", p->count, p->word);
        }
        treeprintvalue(p->right, target, wordcount);
    }
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

struct tnode *talloc(void);
char *my_strdup(char *);

/* addtree: add a node with w, at or below p */
struct tnode *addtree(struct tnode *p, char *w, int *wordcount)
{
    int cond;
    
    if (p == NULL) {    /* a new word has arrived */
        p = talloc();   /* make a new node */
        p->word = my_strdup(w);
        p->count = 1;
        p->left = p->right = NULL;
        (*wordcount)++;
    } else if ((cond = strcmp(w, p->word)) == 0) {
        p->count++;     /* repeated word */
    } else if (cond < 0)  /* less than into left subtree */
        p->left = addtree(p->left, w, wordcount);
    else            /* greater than into right subtree */
        p->right = addtree(p->right, w, wordcount);
    return p;
}

/* treeprint: in-order print of tree p */
void treeprint(struct tnode *p)
{
    
    if (p != NULL) {
        treeprint(p->left);
        printf("%d,%s", p->count, p->word);
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
  <image src="../assets/exercise6-4_cmds.jpg" alt="output for exercise 6-4_cmds" />
</p>
<p align="center">
  <image src="../assets/exercise6-4_data_and_result.jpg" alt="output for exercise 6-4_data_and_result" />
</p>

[Back to Main](../readme.md)
