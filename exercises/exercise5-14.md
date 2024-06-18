# Exercise 5-14
> Modify the sort program to handle a `-r` flag, which indicates sorting in reverse (decreasing) order. Be sure that `-r` works with `-n`

## Process
I found this one was a bit challenging to understand. There were a bunch of new concepts to wrap my head around such as:
- function pointers
- function pointers as variables, parameters
- `void *`
- `void *` as a parameter
- type cast as `(void **)`
- casting a whole function type? `(int (*)(void*, void*))`
- not a concept, but why is `lineptr` void? should be char? I get it makes the function more generic, but seems like unnecessary confusion when learning other new concepts..

I played around with function pointers and all the new concepts a bit before I attempted the exercise.
As with solving previous examples I used a combination of:
- YouTube videos
- stepping through the code with gdb
- understanding error messages and code with ChatGPT
- reading previous examples in the book for `processargs`

The videos I found most helpful were [Understanding and Using Function Pointers in C](https://www.youtube.com/watch?v=axngwDJ79GY), 
[Function Pointers](https://www.youtube.com/watch?v=f_uWOWViYc0) and [Demystifying Pointers](https://www.youtube.com/watch?v=yHWmGk3r-ho)

Also, as with previous exercises there is no smart error checking; if any arguments fail, the program exits

The way I actually solved the exercise was store a `int reverse` variable as 1 or -1, which is then passed to the qsort function and multipled with the comparison function

## Code
```c
#include <stdio.h>
#include <string.h> /* strcmp */
#include <stdlib.h> /* atof */
    
#define MAXLINES 5000 /* max #lines to be sorted */

char *lineptr[MAXLINES]; /* pointers to text lines */

int processargs(int argc, char *argv[], int *n, int *r);
int readlines(char *lineptr[], int maxlines);
void writelines(char *lineptr[], int nlines);
void qsort_kandr(void *lineptr[], int left, int right, int (*comp)(void *, void *), int reverse);
int numcmp(char *, char *);

/* main: sort input lines */
int main(int argc, char *argv[])
{
    int nlines;         /* number of input lines read */
    int numeric = 0;    /* 1 if numeric sort */
    int reverse = 1;    /* -1 if reverse order */
    
    int input = 0;
    if(processargs(argc, argv, &numeric, &reverse))
        return 1;
    
    if ((nlines = readlines(lineptr, MAXLINES)) >= 0) {
        qsort_kandr((void **) lineptr, 0, nlines-1,
        (int (*)(void*, void*))(numeric ? numcmp : strcmp),
        reverse);
        writelines(lineptr, nlines);
        return 0;
    } else {
        printf("error: input too big to sort\n");
        return 1;
    }
    
    return 0;
}

/* processargs: collect numeric and reverse input arguments */
int processargs(int argc, char *argv[], int *n, int *r)
{
    int c;
    while (--argc > 0 && (*++argv)[0] == '-') {
        while (c = *++argv[0])
            switch (c) {
                case 'n':
                    *n = 1;
                    break;
                case 'r':
                    *r = -1;
                    break;
                default:
                    printf("illegal character found: %c\n", c);
                    return 1;
                    break;
            }
    }
    return 0;
}

/* numcmp: compare s1 and s2 numerically */
int numcmp(char *s1, char *s2)
{
    double v1, v2;
    v1 = atof(s1);
    v2 = atof(s2);
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
           int (*comp)(void *, void *), int reverse)
{
    int i, last;
    void swap(void *v[], int i, int j);
    
    if (left >= right)  /* do nothing if array contains */
        return;         /* fewer than two elements */
    swap(v, left, (left + right)/2);
    last = left;
    for (i = left+1; i <= right; i++)
        if ( ((*comp)(v[i], v[left]) * reverse) < 0)
            swap(v, ++last, i);
    swap(v, left, last);
    qsort_kandr(v, left, last-1, comp, reverse);
    qsort_kandr(v, last+1, right, comp, reverse);
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
  <image src="../assets/exercise5-14_data.jpg" alt="output for exercise 5-14_data" />
</p>
<p align="center">
  <image src="../assets/exercise5-14_result.jpg" alt="output for exercise 5-14_result" />
</p>

[Back to Main](../readme.md)
