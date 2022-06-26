# Exercise 1-2

> Experiment to find out what happens when `printf`'s argument string contains `\c`, where *c* is some character not listed above.

## Process
Apart from the usual ones like tab, newline etc I found two interesting ones, *beep* and *backspace*. Couldn't get either to work properly unfortunately..

## Final Code
```c
#include <stdio.h>

main() {

    printf("C escape sequences\n");
    printf("\a"); /* makes a beep, but system sounds must be on */
    printf("\b"); /* move the cursor back a space */
}
```
[Back to Main](../readme.md)
