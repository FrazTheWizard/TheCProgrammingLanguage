# Exercise 1-2

> Experiment to find out what happens when `printf`'s argument string contains `\c`, where *c* is some character not listed above.

## Process
[Escape sequences in C](https://en.wikipedia.org/wiki/Escape_sequences_in_C) - wikipedia
> sequence of characters that does not represent itself when used inside a character or string literal, but is translated into another character or a sequence of characters that may be difficult or impossible to represent directly

Apart from the usual ones like tab, newline etc I found two interesting ones, *beep* and *backspace*. Couldn't get either to work properly unfortunately..

## Final Code
```c
#include <stdio.h>

main() {

    printf("\n");   /* newline */
    printf("\t");   /* tab */
    printf("\'");   /* single apostrophe */
    printf("\"");   /* double apostrophe */
    printf("\\");   /* escape character itself */
    
    /* different ways to write escape sequences     */
    printf("newline\n");         /* using character  */
    printf("newline%d",10);     /* using decimal    */
    printf("newline%d",0x0A);   /* using hex        */
    
    /* interesting ones */
    printf("\a");   /* makes a beep, but system sounds must be on */
    printf("\b");   /* move the cursor back a space */
}
```
[Back to Main](../readme.md)
