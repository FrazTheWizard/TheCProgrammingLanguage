# Exercise 1-13
> Write a program to print a histogram of the lengths of words in its input.
> It is easy to draw the histogram with the bars horizontal; a vertical orientation is more challenging.

## Process
Did this a while ago, so can't remember my thought process exactly, but I don't remember any terribly difficult puzzles

After the horizontal version, I did the suggested vertical version, which was more challenging, but again, don't remember exactly

## Final Code
Horizontal
```c
#include <stdio.h>

#define OUT 0
#define IN  1

int main(){
	int c, i, j, ccount, state;

	ccount = 0;
	state = OUT;
	
	int nc[20];

	for (i = 0; i < 20; i++)
		nc[i] = 0;

	while((c = getchar()) != EOF){
		
		if (c == ' ' || c == '\t' || c == '\n') {
			if (state == IN){
				++nc[ccount];
				ccount = 0;
				state = OUT;
			}
		} else if (state == OUT){
			state = IN;
			++ccount;
		} else
			++ccount;
	}

	printf("word length histogram\n");
	for(i = 0; i < 20; i++){
		printf("letters %2d: ", i);
		for (j = 0; j < nc[i]; j++)
			putchar('-');
		putchar('\n');
	}
    return 0;
}
```

Vertical
```c
#include <stdio.h>

#define OUT 0
#define IN  1
#define COL '#'

int main(){
	int c, i, j, ccount, state;

	ccount = 0;
	state = OUT;
	
	int nc[20];

	for (i = 0; i < 20; i++)
		nc[i] = 0;

	while((c = getchar()) != EOF){
		
		if (c == ' ' || c == '\t' || c == '\n') {
			if (state == IN){
				++nc[ccount];
				ccount = 0;
				state = OUT;
			}
		} else if (state == OUT){
			state = IN;
			++ccount;
		} else
			++ccount;
	}


	printf("\nword length histogram vertical\n");
	
	/* get biggest value in counted in the array */
	int max, cur;
	max = nc[0];	

	for (i = 1; i < 20; i++){
		if (nc[i] > max)
			max = nc[i];
	}

	/* decrement through each value from max and print space or line */
	cur = max;
	for (i = max; i > 0; i--){
		printf("%3d", i);
		for (j = 0; j < 20; j++){
			if (nc[j] >= cur){
				printf("  %c ", COL);
			} else {
				printf("    ");
			}
		}
		putchar('\n');
		--cur;
	}

	printf("   ");
	for ( i = 0; i < 20; i++) printf(" %2d ", i);
    
    return 0;
}
```

<p align="center">
    <image src="../assets/exercise1-13_output1.jpg" alt="output for exercise1-13" />
</p>
<p align="center">
    <image src="../assets/exercise1-13_output2.jpg" alt="output for exercise1-13" />
</p>
<p align="center">
    <image src="../assets/exercise1-13_output3.jpg" alt="output for exercise1-13" />
</p>

[Back to Main](../readme.md)
