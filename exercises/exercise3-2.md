# Exercise 3-2

> Write a function `escape(s,t)` that converts characters like newline and tab into visible escape sequences like 
> `\n` and `\t` as it copies the string `t` to `s`. Use a `switch`. Write a function for the other direction as well, 
> converting escape sequences into the real characters.


## Process
Took two days. Had a bit of trouble with syntax trying to work out initialisation and declarations.

Also the escape2 function feels kind of dodgy, not sure how to make it better though...


## Final Code

```c
#include <stdio.h>

#define STRING_LENGTH 50

void escape(char s[], char t[]);
void escape2(char s[], char t[]);

int main(void) {
    
    char s[STRING_LENGTH];
    char t[STRING_LENGTH] = {'A', 'b', '\n', 'c', '\t', '\0'};
    
    escape(s,t);
    
    printf("String to Chars\n");
    int i = 0;
    while(s[i] != '\0') {
        putchar(s[i++]);
    }
    putchar('\n');
    
    char x[STRING_LENGTH] = {};
    char y[STRING_LENGTH] = {'x', 'y','\\', '\\', 'z', '\\', 't', 'o', '\\', 'n', 'a', '\0'};
    
    escape2(x,y);
    
    printf("\nChars to String\n");
    i = 0;
    while(x[i] != '\0') {
        putchar(x[i++]);
    }
    
    return 0;
}

void escape(char s[], char t[])
{
    int i, j;
    i = j = 0;
    while (t[j] != '\0'){
        switch(t[j]) {
            case '\t':
                s[i++] = '\\';
                s[i++] = 't';
                break;
            case '\n':
                s[i++] = '\\';
                s[i++] = 'n';
                break;
            default:
                s[i++] = t[j];
                break;
         }
         j++;
    }
}

void escape2(char s[], char t[])
{
    int i, j, esc;
    i = j = esc = 0;
    
    while(t[j] != '\0'){
        switch(t[j]) {
            case '\\':
                if (esc) {
                    esc = 0;
                    s[i++] = '\\';
                } else 
                    esc = 1;
                break;
            default:
                if(esc) {
                    switch(t[j]) {
                        case 't':
                            s[i++] = '\t';
                            break;
                        case 'n':
                            s[i++] = '\n';
                            break;
                        default:
                            break;
                    }
                    esc = 0;
                    break; /* escape the if statement */
                }
                else
                    s[i++] = t[j];
                break;
        }
        j++;
    }    
}
```


## Output

<p align="center">
    <image src="../assets/exercise3-2.jpg" alt="output for exercise3-2" />
</p>

[Back to Main](../readme.md)
