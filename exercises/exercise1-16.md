# Exercise 1-16
>  Revise the main routine of the longest-line program so it will correctly print the length of arbitrarily long input lines, and as much as possible of the text.

## Process
This was a challenging one

First of all I found the question confusing. I think it wants me to rewrite `main` so that that it can handles lines longer than `MAXLINE`

After attempting to figure it out myself, I gained a bit of clarity looking at some [solutions](https://clc-wiki.net/wiki/K%26R2_solutions:Chapter_1:Exercise_16) that point at a previous paragraph just before the question in the book:

> It is worth mentioning in passing that even a program as small as this one presents some sticky
> design problems. For example, what should main do if it encounters a line which is bigger than
> its limit? getline works safely, in that it stops collecting when the array is full, even if no
> newline has been seen. **By testing the length and the last character returned, main can
> determine whether the line was too long, and then cope as it wishes.** In the interests of brevity,
> we have ignored this issue.

My Solution:
1. Check each sentence to see if it's longer than a set maximum length (`MAXLINE`) and doesn't end with a new line character
2. If a sentence fits the criteria in step 1, add its length to a running total called "prevlen." Keep doing this for consecutive sentences that also meet the criteria
3. When you reach a sentence that is shorter than `MAXLINE` or ends with a new line character, stop adding lengths and calculate the maximum length ("max") for that group of sentences

I also modified the program to only print the beginning of lines longer than `MAXLINE`, because I think that would be more useful if you were actually using this program

I'm not 100% sure I've covered every input case, need to find a better way to test programs...

## Final Code
```c
#include <stdio.h>
#define MAXLINE 5 /* maximum input line size */

int getline(char line[], int maxline);
void copy(char to[], char from[]);

/* K&R exercise 1-16: print longest input line */
int main()
{
	int len; /* current line length */
	int max = 0; /* maximum length seen so far */
	char line[MAXLINE]; /* current input line */
	char longest[MAXLINE]; /* longest line */

    /* -- added variables -- */
    int prevlen = 0;        /* previous line length that went over max */
    char linebegin[MAXLINE];/* holds start of line longer than MAXLINE */
	
    while ((len = getline(line, MAXLINE)) > 0) {
        
        if(len == (MAXLINE-1)) { /* reached MAXLINE */
            if(line[MAXLINE - 2] == '\n') { /* reached end of line */
                len = prevlen + len;
                prevlen = 0;
            } else {    /* start of line longer than MAXLINE */
                prevlen = len + prevlen;
                if(prevlen == (MAXLINE-1))
                    copy(linebegin, line);
                continue;
            }
        } else {
            if(prevlen > 0) { /* if any amount of prevlines went over MAXLINE */
                len = prevlen + len;
                prevlen = 0;
            }
        }
        
        if (len > max) {
			max = len;
            if(len >= (MAXLINE-1))
                copy(longest, linebegin);
            else
                copy(longest, line);
        }
    }
    
    if (max > 0) /* there was a line */
        if(max > MAXLINE)
            printf("MAX %d: %s..", max, longest);
        else    
            printf("MAX %d: %s", max, longest);
}

/* getline: read a line into s, return length **/
int getline(char s[], int lim)
{
	int c, i;

	for (i=0; i<lim-1 && (c = getchar())!=EOF && c!='\n'; ++i)
		s[i] = c;
	if (c == '\n') {
		s[i] = c;
		++i;
	}
	s[i] = '\0';
	return i;
}

/* copy: copy 'from' into 'to'; assume 'to' is big enough */
void copy(char to[], char from[])
{
	int i;

	i = 0;
	while ((to[i] = from[i]) != '\0')
		++i;
}
```

## Output
<p align="center">
    <image src="../assets/exercise1-16_a.jpg" alt="output for exercise1-16" />
</p>
        
<p align="center">
<image src="../assets/exercise1-16_b.jpg" alt="output for exercise1-16" />
</p>



[Back to Main](../readme.md)
