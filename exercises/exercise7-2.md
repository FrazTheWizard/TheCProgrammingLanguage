# Exercise 7-2

> Write a program that will print arbitrary input in a sensible way. As a minimum, it should print non-graphic characters in octal or hexadecimal according to local custom, and break long lines. 

## Process
Tried to keep my code readable with enums, defines and linear flow logic in the while loop. 
I found it easier to think about in terms of decimal codes, but this program should probably use `ctype.h` functions like `iscntrl`.
Also skipped any input error handling for this exercise.

Tested my solution by printing all ASCII characters from 0 - 255 _(skipping 26)_ for the program to decipher.

Had problems reading in ASCII control character 26 `SUB`, so skipping that. As far as I understand, the program would need to read the binary and not the 'text' of the characters, but I 
assume we will cover how to do that later in the chapter when looking at files.

After finishing my solution, checked out the C Answer Book and really liked the use of offloading the line splitting to a small function, makes the code more readable. 

## Code
```c
#include <stdio.h>

#define MAXLEN 80
#define GRAPHICSIZE 1
#define NONGRAPHICSIZE 5

void handleabritaryinput(char base);
enum Input {GRAPHIC, NONGRAPHIC, NONASCII};

int main(int argc, char *argv[])
{
    int base = 'x';
    if (argc == 2)
        base = argv[1][0]; /* assume correct input as it's just an exercise */
    handleabritaryinput(base);
}

/* handleabritaryinput: print arbitrary input in a sensible way */
void handleabritaryinput(char base)
{
    int c, pos = 0, shift = 0;
    enum Input inputtype;

    while ((c = getchar()) != EOF) {
        
        if ((c >= 32 && c <= 126) || c == '\n') { /* printable characters */
            inputtype = GRAPHIC;
            shift = GRAPHICSIZE;
        } else if (c >= 0 && c <= 255) {
            inputtype = NONGRAPHIC;
            shift = NONGRAPHICSIZE;
        } else {
            inputtype = NONASCII;
            shift = 0;
        }
    
        if (pos > (MAXLEN - shift)) { /* break long lines */
            putchar('\n');
            pos = 0;     
        }
        
        pos += shift; /* keep track of line length */
        
        switch (inputtype) {
            case GRAPHIC:
                if (c == '\n')
                    pos = 0;
                printf("%c", c);
                break;
            case NONGRAPHIC:
                if (base == 'o')
                    printf("0%03o ", c); /* zero padded octal 0000 - 0377 */
                else if (base == 'x')
                    printf("0x%02X ", c); /* zero padded hex 0x00 - 0xFF */
                break;
            case NONASCII:
                /* skip Unicode chars as I don't know enough about them yet */
                break;
        }     
    }    
}
```

## Output
<p align="center">
  <image src="../assets/exercise7-2_cmds.jpg" alt="output for exercise 7-2_cmds" />
</p>
<p align="center">
  <image src="../assets/exercise7-2_data_and_result.jpg" alt="output for exercise7-2 data and result" />
</p>

[Back to Main](../readme.md)
