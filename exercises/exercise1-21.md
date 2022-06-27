# Exercise 1-21
> Write a program *entab* which replaces strings of blanks by the minimum amount of tabs and blanks 
> to achieve the same spacing. Use the same tab stops as for *detab*. 
> When either a tab or a single blank would suffice to reach a tab stop, which should be given preference?

## Process

This exercise took me two days and was not as complicated as the previous exercise, as I had worked through it quite thoroughly and figured out how
this problem behaves

The complicated part of this problem was the calculation of how many tabs vs spaces to replace tabs by

**Solution Thinking in Steps**
There are kind of three parts: before, middle and end

a. before

if the sequence of blanks does not cross at least one tab stop, remain spaces

c. middle (optional)

the integer of the division of the rest of the sequence of blanks become tabs

b. end (optional)

the remainder of the division of the rest of the sequence of blanks become spaces

Also in direct answer to the question, I tink a single blank should be given preference, 
although I didn't implement it here because I wanted to move on to other questions

## Final Code
```c
#include <stdio.h>
#define MAXLINE 1000

int getline(char s[], int lim);
int printlineEntab(char s[], int len);

char s[MAXLINE];
int n; /* tab column size */

int main() {
	extern int n;
	int len;
	n = 4; /* set tab column size */

	while((len = getline(s, MAXLINE)) > 0) {
		printlineEntab(s,len);
	}
	return 0;
}

int printlineEntab(char s[], int len) {
	extern int n;
	int start, end;
	int seqlen, initMod, firstTabStop;
	int tabs, blanks;
	start = -1;
	
	for(int i=0; i<len; i++){
		if(s[i]==' '){
			if(start==-1) /* begin blank sequence */
				start = i;	
			else {} /* pass */			
		}
		else { /* found character */
			if(start==-1) /* was not counting blanks */
				putchar(s[i]);
			else {
				end = i;
				seqlen = end-start;
				initMod = n - (start % n);
				
				if(initMod > seqlen) { /* blanks not long enough to make a tab */
					for(int j=0; j<seqlen; j++)
						putchar(' ');
					putchar(s[i]);
					start = -1; /* reset sequence of blanks */
				}					
				else { /* blanks long enough to make a tab */
					firstTabStop = start + initMod;
					tabs = 1;
					
					tabs = tabs + ((end - firstTabStop) / n); /* subsequent tabs to add */
					blanks = (end - firstTabStop) % n; /* subsequent blanks to add (if any) */
					
					for(int j=0;j<tabs;j++)
						putchar('\t');
					
					for(int j=0;j<blanks;j++)
						putchar(' ');
					
					putchar(s[i]);
					start = -1; /* reset sequence of blanks */
				}
			}		
		}
	}
}

/* read a line into s, return length */
int getline(char s[], int lim) {
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
```

## Output
<p align="center">
    <image src="../assets/exercise1-21_output1.jpg" alt="output for exercise1-21" />
</p>

<p align="center">
    <image src="../assets/exercise1-21_output2.jpg" alt="output for exercise1-21" />
</p>

[Back to Main](../readme.md)
