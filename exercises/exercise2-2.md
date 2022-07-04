# Exercise 2-2 

> Write a loop equivalent to the `for` loop above without using `&&` or `||`

## Process
Fairly straight forward task. Found a little confusion in wrapping my head around the order of things.
I found this exercise helped me to understand C string storage a little more thoroughly.

## Final Code
```c
#include <stdio.h>
#define MAXLINE 10 /* maximum input line size */

int getlinewhile(char line[], int maxline);

int main(void) {
    
    char line[MAXLINE];
    int len;
    while((len = getlinewhile(line, MAXLINE)) > 0) {
        for(int i = 0; i < len; i++)
                putchar(line[i]);    
    }
    return 0;
}

int getlinewhile(char line[], int maxline) {
    int c;
    int i = 0;
    int run = 1;
    
    while(run != 0) {
        if(i < MAXLINE-1){
            c = getchar();
            if(c != EOF){
                if(c != '\n'){
                    line[i] = c;
                    i++;
                } else {
                    run = 0;
                }
            } else {
                run = 0;
            }
        } else {
            run = 0;
        }
    }
    
    if (c == '\n') {
        line[i] = c;
		i++;
    }
    
    line[i] = '\0';
    
    return i;
}
```

## Output
<p align="center">
    <image src="../assets/exercise2-2_output1.jpg" alt="output for exercise2-2" />
</p>
<p align="center">
    <image src="../assets/exercise2-2_output2.jpg" alt="output for exercise2-2" />
</p>

[Back to Main](../readme.md)
