# Exercise 1-20
> Write a program that replaces tabs in the input with the proper number of blanks to space to the next tab stop. 
> Assume a fixed set of tab stops, say every *n* columns. Should n be a variable or a symbolic parameter?

## Process
This one took me just over a week. Part of the reason for taking so long was spending only 1 hour a day on it, 
I was sick and was trying to find what I thought was an elegant solution.

My initial idea was to make a separate function called *detab* that would actually add the blanks in the array before printing.
I thought this would be cool because it would separate the work from the input and output functions, for a nice clean solution.
But it turned out that was quite complicated and I think unnessacarily complicated
all in the name of making a 'cleaner' solution.

So instead, I integrated the work into the output print function


Two things I stuggled with are:
1. Index's and Lens are 1 digit apart, either starting at 0 or 1
2. Multiple tabs per line


**Solution Thinking in Steps**

a. `index % n`

find how many chars we are past the last tab column

b. `n - (index % n)`

find how many more to go

c. `shift = n - ((i + tabAmount) % n)`

keep a memory of how many previous spaces I had added to the line already, incase of previous tabs, 

d. `tabAmount = tabAmount + shift - 1`

update amount of spaces moved
 

## Final Code
```c
# include <stdio.h>
# define MAXLINE 1000

int getline(char s[], int lim);
void printlinetab(char s[], int len);

char s[MAXLINE];
int n;

int main() {
	int len;
	extern int n;
	n = 4; /* define tab stop amount */
	
	while((len = getline(s, MAXLINE)) > 0) {
		printlinetab(s,len);
	}	
	return 0;
}

/* print line with spaces for tabs */
void printlinetab(char s[], int len) {
	extern int n;
	int tabAmount = 0;
	int shift = 0;
	
	for(int i=0; i<len; i++)
		if(s[i]=='\t') {
			shift = n - ((i + tabAmount) % n);
			tabAmount = tabAmount + shift - 1;
			
			for(int j=0;j<shift;j++)
				putchar(' ');
		}
		else {
			putchar(s[i]);
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
    <image src="../assets/exercise1-20_output1.jpg" alt="output for exercise1-20" />
</p>

<p align="center">
    <image src="../assets/exercise1-20_output2.jpg" alt="output for exercise1-20" />
</p>

[Back to Main](../readme.md)
