# Exercise 1-14
> Write a program to print a histogram of the frequencies of different characters of input.

## Process
Just followed on the from previous exercise

## Final Code
```c
#include <stdio.h>

int main(){
	int c, i, j, unknown;

	unknown = 0;

	int lowercharcount[26];
	int uppercharcount[26];

	for (i = 0; i < 26; i++){
		lowercharcount[i] = 0;
		uppercharcount[i] = 0;
	}

        /* input */
	while((c = getchar()) != EOF){
		if (c == ' ' || c == '\t' || c == '\n')
			;
		else if (c >= 'a' && c <= 'z')
			++lowercharcount[c - 'a'];
		else if (c >= 'A' && c <= 'Z')
			++uppercharcount[c - 'A'];
		else
			++unknown;
	}

        /* output */ 
	printf("letters occurance histogram");

	for (i=0;i<26;i++) {
		printf("\n%c: %4d ", 'a' + i, lowercharcount[i]);
		for ( j = 0; j < lowercharcount[i]; j++) 
			putchar('-');
	}

        /* Uncomment for uppercase
	for ( i= 0; i < 26; i++)
		printf("\n%c: %4d",  'A' + i, uppercharcount[i]);
		for (j = 0; j < uppercharcount[i]; j++)
			putchar('-');
        */
    return 0;
}
```

## Output
<p align="center">
    <image src="../assets/exercise1-14_output1.jpg" alt="output for exercise1-14" />
</p>
<p align="center">
    <image src="../assets/exercise1-14_output2.jpg" alt="output for exercise1-14" />
</p>

[Back to Main](../readme.md)
