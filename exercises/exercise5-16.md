# Exercise 5-16
> Add the -d ("directory order") option, which makes comparisons only on letters, numbers and blanks. Make sure it works in conjunction with -f 

## Process
This was surprisingly very straightforward. Took code from previous exercise and did the following:
1. added new global variable 
```c 
int directory = 0;
``` 
2. added new case in `processargs`
```c 
case 'd':
  directoryorder = 1;
  break;
```
3. added a few lines to the beginning of `cmpstr`
```c
// directory order
if (directory) {
    while(!isalnum(*s) && *s != ' ' && *s != '\0')
        s++;
    while(!isalnum(*t) && *t != ' ' && *t != '\0')
        t++;
}   
```
## Code
```c
#include <stdio.h>
#include <string.h> /* strcpy */
#include <stdlib.h> /* atof */
#include <ctype.h>  /* tolower */
    
#define MAXLINES 5000 /* max #lines to be sorted */

char *lineptr[MAXLINES]; /* pointers to text lines */

int processargs(int argc, char *argv[]);
int readlines(char *lineptr[], int maxlines);
void writelines(char *lineptr[], int nlines);
void qsort_kandr(void *lineptr[], int left, int right, 
                 int (*comp)(void *, void *));
int cmpstr(char *, char *);
int numcmp(char *, char *);

int numeric = 0;    /* numeric sort */
int reverse = 0;    /* reverse sorting order */
int fold = 0;       /* fold upper and lowercase together */
int directory = 0;  /* only compare letters, numbers and blanks */

/* main: sort input lines */
int main(int argc, char *argv[])
{
    int nlines;     /* number of input lines read */
    
    if(processargs(argc, argv))
        return 1;
     
     /* comparison function for qsort */
    int (*comp)(void *, void *);
    
    if (numeric)
        comp = (int (*)(void *, void *)) numcmp;
    else
        comp = (int (*)(void *, void *)) cmpstr;
    
    /* read all lines */
    if ((nlines = readlines(lineptr, MAXLINES)) >= 0) {
        
        /* qsort */
        qsort_kandr((void **) lineptr, 0, nlines-1, comp);
        
        /* writelines */
        writelines(lineptr, nlines);
        
        return 0;
    } else {
        printf("error: input too big to sort\n");
        return 1;
    }
    
    return 0;
}

/* processargs: process arguments -f -n -r */
int processargs(int argc, char *argv[])
{
    int c;
    while (--argc > 0 && (*++argv)[0] == '-') {
        while (c = *++argv[0])
            switch (c) {
                case 'n':
                    numeric = 1;
                    break;
                case 'r':
                    reverse = 1;
                    break;
                case 'f':
                    fold = 1;
                    break;
                case 'd':
                    directory = 1;
                    break;
                default:
                    printf("illegal character found: %c\n", c);
                    return 1;
                    break;
            }
    }
    return 0;
}

/* strcmp: compare string s and t  */
int cmpstr(char *s, char *t)
{
    char s1, t1;
    do {
        
      // directory order
      if (directory) {
          while(!isalnum(*s) && *s != ' ' && *s != '\0')
              s++;
          while(!isalnum(*t) && *t != ' ' && *t != '\0')
              t++;
      }      
      
      // fold
      s1 = fold ? tolower(*s) : *s;
      t1 = fold ? tolower(*t) : *t;
      
      // reverse
      if (reverse) {
          char temp = s1;
          s1 = t1;
          t1 = temp;
      }
      
      s++, t++;
    } while (s1 != '\0' && s1 == t1);
    
    return s1 - t1;
}

/* numcmp: compare s1 and s2 numerically */
int numcmp(char *s1, char *s2)
{
    double v1, v2;
    v1 = atof(s1);
    v2 = atof(s2);
    
    // reverse
    if (reverse) {
      double temp = v1;
      v1 = v2;
      v2 = temp;
    }
    
    if (v1 < v2)
        return -1;
    else if (v1 > v2)
        return 1;
    else
        return 0;
}

#define MAXLEN 1000     /* max length of any input line */
int getline(char s[], int lim); /* read a line into s, return length */
char *alloc(int n); /* return pointer to n characters */

/* readlines: read input lines */
int readlines(char *lineptr[], int maxlines)
{
	int len, nlines;
	char *p, line[MAXLEN];
	
	nlines = 0;
	while ((len = getline(line, MAXLEN)) > 0)
		if (nlines >= maxlines || (p = alloc(len)) == NULL)
			return -1;
		else {
            if (line[len-1] == '\n')
                line[len-1] = '\0'; /* delete newline */
			strcpy(p, line);
			lineptr[nlines++] = p;
		}
		return nlines;
}

/* writelines: write last n output lines */
void writelines(char *lineptr[], int nlines)
{
	int i;
	
	for (i = 0; i < nlines; i++)
		printf("%s\n", lineptr[i]);
}

/* getline: read a line into s, return length */
int getline(char s[], int lim)
{
    int c, i;
    
    for (i=0; i<lim-1 && (c=getchar())!=EOF && c!='\n'; i++)
        s[i] = c;
    
    if (c == '\n') {
        s[i] = c;
        ++i;
    }
    
    s[i] = '\0';
    return i;
}

/* qsort_kandr: sort v[left]...v[right] into increasing order */
void qsort_kandr(void *v[], int left, int right,
           int (*comp)(void *, void *))
{
    int i, last;
    void swap(void *v[], int i, int j);
    
    if (left >= right)  /* do nothing if array contains */
        return;         /* fewer than two elements */
    swap(v, left, (left + right)/2);
    last = left;
    for (i = left+1; i <= right; i++)
        if ( ((*comp)(v[i], v[left])) < 0)
            swap(v, ++last, i);
    swap(v, left, last);
    qsort_kandr(v, left, last-1, comp);
    qsort_kandr(v, last+1, right, comp);
}

/* swap: interchange v[i] and v[j] */
void swap(void *v[], int i, int j)
{
    char *temp;
    
    temp = v[i];
    v[i] = v[j];
    v[j] = temp;
}

#define ALLOCSIZE 10000	/* size of available space */

static char allocbuf[ALLOCSIZE];	/* storage for alloc */
static char *allocp = allocbuf;		/* next free position */

/* alloc: return pointer to n characters */
char *alloc(int n)
{
	if (allocbuf + ALLOCSIZE - allocp >= n) { /* it fits */
		allocp += n;
		return allocp - n; /* old p */
	} else	/* not enough room */
		return 0;
}
```

## Output
<p align="center">
  <image src="../assets/exercise5-16_cmds.jpg" alt="output for exercise 5-16_cmds" />
</p>
<p align="center">
  <image src="../assets/exercise5-16_data_and_result.jpg" alt="output for exercise 5-16_data_and_result" />
</p>

[Back to Main](../readme.md)
