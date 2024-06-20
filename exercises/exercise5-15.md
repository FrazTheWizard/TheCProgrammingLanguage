# Exercise 5-14
> Add the option `-f` to fold upper and lower case together, so that case distinctions are not made during sorting; for example, `a` and `A` compare equal.

## Process
Found this one less challenging then that last exercise, now that I know the basic idea and how to restructure the program

### Changes
After looking at some [solutions](https://clc-wiki.net/wiki/K%26R2_solutions:Chapter_5:Exercise_14) for the previous exercise, I modified some things:
- removed `strcmp` from `string.h` and added a `strcmp` function from previous examples in the book to have more control over how the comparison is done
- moved reverse logic out of `qsort` into the comparison functions `numcmp` and `strcmp`, as it seems more appropriate place for that logic
- changed storing state from passing function parameters to using global variables, as this is a small program and I would prefer easier to read functions
- stored the comparison function pointer as a temp variable in main to make intention clearer
- cast comparison function pointer to avoid compiler error, which I didn't quite understand previous exercise

### Control Flow
I started to realise that you only really have a few choices when it comes to storing and passing state around, which have different tradeoffs.
1. passing as function parameters
    - seem to be better for bigger programs, when following state around is harder
    - keeps flow between functions clear, but leads to long function declarations and more messy code

3. using global variables
    - seem to be good for small programs such as this one, but might be hard as program grows..
    - keeps code and functions clearer, but following flow between functions becomes hard

5. function pointers
    - this way is new to me, but seems it would be good for large programs, swapping big paths of logic
    - seems to break code into simplier paths, but perhaps a little more complex idea to keep in your head, with casting and types etc

## Code
```c
#include <stdio.h>
#include <string.h> /* strcpy */
#include <stdlib.h> /* atof */
#include <ctype.h>  /* tolower */
    
#define MAXLINES 5000 /* max #lines to be sorted */

int processargs(int argc, char *argv[]);
int readlines(char *lineptr[], int maxlines);
void writelines(char *lineptr[], int nlines);
void qsort_kandr(void *lineptr[], int left, int right, int (*comp)(void *, void *));
int cmpstr(char *, char *);
int numcmp(char *, char *);

int numeric = 0;         /* numeric sort */
int reverse = 0;         /* reverse sorting order */
int fold = 0;            /* fold upper and lowercase together */
char *lineptr[MAXLINES]; /* pointers to text lines */

/* main: sort input lines */
int main(int argc, char *argv[])
{
    int nlines;     /* number of input lines read */

    /* process args -n -r -f */
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
                default:
                    printf("illegal character found: %c\n", c);
                    return 1;
                    break;
            }
    }
    return 0;
}

/* strcmp: compare string s and t */
int cmpstr(char *s, char *t)
{
    char s1, t1;
    do {
      
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
  <image src="../assets/exercise5-14_cmds.jpg" alt="output for exercise 5-14_cmds" />
</p>
<p align="center">
  <image src="../assets/exercise5-14_data.jpg" alt="output for exercise 5-14_data_and_result" />
</p>

[Back to Main](../readme.md)
