# Exercise 1-19
> Write a function reverse(s) that reverses the character string s. Use it to write a program that reverses its input a line at a time. 

## Process
Pretty straight forward exercise. 

As the exercise didn't state to pass the length of the string, I made a way to find the length internal to the reverse function

Basic process I used was first find the end of the string. Then use two counters from beginning and end, swapping each character until they meet in the middle.

Also I think it's obvious, but I'm not swapping the `'\n'` and `'\0'` characters, because that seems impractical

## Final Code
```c
#include <stdio.h>
#define MAX 80

void reverse(char s[]);
int getline(char line[], int limit);

int main()
{
	char s[MAX];
	
	while(getline(s, MAX) != 0) {
		reverse(s);
		printf("%s", s);
	}
		
	return 0;
}

/* reverse character string s */
void reverse(char s[]) {
	int i, j;
	i = j = 0;
	char temp;
	
	/* find index of either '\n' or '\0', whichever is first */
	while(s[j] != '\n' && s[j] != '\0')
		j++;
	
	/* if the length of string is over 1 */
	if(j > 1) {
		/* turn length into index of last char in string */
		j--;
		
		/*	starting at beginning and ending of the string, work towards the 
			middle, swapping characters until reaching the centre of the string
		*/
		for (i=0; i < j; i++, j--) {
			temp = s[i];
			s[i] = s[j];
			s[j] = temp;
		}
	}
}

/* getline: read a line into s, return length */
int getline(char line[], int limit)
{
	int c, i;
	
	for (i=0; i<limit-1 && (c=getchar()) != EOF && c!='\n'; ++i)
		line[i] = c;
	if (c == '\n') {
		line[i] = c;
		++i;
	}
	line[i] = '\0';
	return i;
}
```

## Output
<p align="center">
    <image src="../assets/exercise1-19_a.jpg" alt="output for exercise1-19" />
</p>
        
<p align="center">
<image src="../assets/exercise1-19_b.jpg" alt="output for exercise1-19" />
</p>



[Back to Main](../readme.md)
