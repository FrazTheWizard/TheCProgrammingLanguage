# Exercise 1-12
> Write a program that prints its input one word per line

## Process
I think this is pretty straight forward

## Final Code

```c
#include <stdio.h>

#define	IN 1
#define OUT 0

int main(){
    int c, state;
    state = OUT;

	while((c = getchar()) != EOF){
		if (c == '\n' || c == '\t' || c == ' '){
			if (state == IN){
				putchar('\n');
				state = OUT;
			} else 
				; /* let character drop */
		} else {
			state = IN;
			putchar(c);	
		}
	}
    return 0;
}
```

<p align="center">
    <image src="../assets/exercise1-12_output1.jpg" alt="output for exercise1-12" />
</p>
<p align="center">
    <image src="../assets/exercise1-12_output2.jpg" alt="output for exercise1-12" />
</p>

[Back to Main](../readme.md)
