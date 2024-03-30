# Exercise 5-7

> Rewrite readlines to store lines in an array supplied by main, rather than calling alloc to maintain storage. How much faster is the program? 

## Process
Rewrote the function and commented out the original code for reference. 

Didn't notice any noticable speed difference when updating the function with a storage array,
I guess it made more of a difference on older machines when the book was written?

Other solutions online suggest that its a very minor difference, probably only relevant in code that
needs to be highly optimised

### Global Changes
1. introduced new macro MAXSTORAGE
2. created new `readlines` function
```c 
/* 1. */
#define MAXSTORAGE 10000

/* 2. */
int readlines_2(char *lineptr[], char *storage, int maxlines, int maxstorage)
```

### Changes to `main`
3. created array storage array with length of `MAXSTORAGE`
4. replaced call to new `realines_2` and pass in MAXSTORAGE
```c
/* .3 */
char storage[MAXSTORAGE];

/* .4 */
if ((nlines = readlines_2(lineptr, storage, MAXLINES, MAXSTORAGE)) >= 0) {
```

### New function `readlines_2`
ref in code below..


## Code
```c
#include <stdio.h>
#include <string.h>
#include "getline.c" /* getline from previous chapters stored in separate file */

/* max #lines to be sorted*/
#define MAXLINES 5000
#define MAXSTORAGE 10000

char *lineptr[MAXLINES];	/* pointers to text lines */

// int readlines(char *lineptr[], int maxlines);
int readlines_2(char *lineptr[], char *storage, int maxlines, int maxstorage);
void writelines(char *lineptr[], int nlines);

void qsort(char *lineptr[], int left, int right);

/* sort input lines */
int main()
{
	int nlines;	/* number of input lines read */
	char storage[MAXSTORAGE];
	
	// if ((nlines = readlines(lineptr, MAXLINES)) >= 0) {
	if ((nlines = readlines_2(lineptr, storage, MAXLINES, MAXSTORAGE)) >= 0) {		
		qsort(lineptr, 0, nlines-1);
		writelines(lineptr, nlines);
		return 0;
	} else {
		printf("error: input too big to sort\n");
		return 1;
	}
	
	return 0;
}

#define MAXLEN 1000 /* max length of any input line */
int getline(char *, int);
// char *alloc(int);

/* readlines: read input lines */
// int readlines(char *lineptr[], int maxlines)
// {
	// int len, nlines;
	// char *p, line[MAXLEN];
	
	// nlines = 0;
	// while ((len = getline(line, MAXLEN)) > 0)
		// p = array;
		// if (nlines >= maxlines || (p = alloc(len)) == NULL)
			// return -1;
		// else {
			// line[len-1] = '\0'; /* delete newline */
			// strcpy(p, line);
			// lineptr[nlines++] = p;
		// }
		// return nlines;
// }

/* readlines: read input lines into array */
int readlines_2(char *lineptr[], char *storage, int maxlines, int maxstorage)
{
	int len, nlines;
	char line[MAXLEN];
	nlines = 0;
	
	while ((len = getline(line, MAXLEN)) > 0) {
		
		if(nlines >= maxlines) /* not enough space to store another pointer */
			return -1;
		
		if((maxstorage - len) <= 0) /* not enough space to store line in array */
			return -1;
		
		if(line[len-1] == '\n') /* delete newline if exists at end of the line */
			line[len-1] = '\0';
		
		strcpy(storage, line);        /* copy line to storage*/
		lineptr[nlines++] = storage;  /* add pointer to line */
		storage += len;               /* move storage pointer */
		maxstorage -= len - 1;        /* update storage counter */
	}
	return nlines;
}


/* writelines: write output lines */
void writelines(char *lineptr[], int nlines)
{
	int i;
	
	for (i = 0; i < nlines; i++)
		printf("%s\n", lineptr[i]);
}

/* qsort: sort v[left]...v[right] into inreasing order */
void qsort(char *v[], int left, int right) 
{
	int i, last;
	void swap(char *v[], int i, int j);
	
	if(left >= right)	/* do nothing if array contains */
		return;					/* fewer than two elements */
	swap(v, left, (left + right)/2);
	last = left;
	for (i = left+1; i <= right; i++)
		if(strcmp(v[i], v[left]) < 0)
			swap(v, ++last, i);
		swap(v, left, last);
		qsort(v, left, last-1);
		qsort(v, last+1, right);
}

/* swap: interchange v[i] and v[j] */
void swap(char *v[], int i, int j)
{
	char *temp;
	
	temp = v[i];
	v[i] = v[j];
	v[j] = temp;
}
```

## Output
<p align="center">
  <image src="../assets/exercise5-7_a.jpg" alt="output for exercise5-7" />
</p>
<p align="center">
  <image src="../assets/exercise5-7_b.jpg" alt="output for exercise5-7" />
</p>

[Back to Main](../readme.md)
