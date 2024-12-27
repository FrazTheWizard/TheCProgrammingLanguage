# Exercise 7-1

> Write a program that converts upper case to lower or lowercase to upper, depending on the name it is invoked with, as found in argv[0]. 

## Process
Very straightforward exercise, I was expecting something more complicated ha. Kind of a fun idea to change functionality based on the name of the program. 
Could be a cool hidden feature in a game.

Being on windows, the program checks for the name and for `.exe` extension. 

Used a function pointer because it felt like an appropriate use case.

Used symbolic links to change `argv[0]`.

Found the answers on [clc-wiki](https://clc-wiki.net/wiki/K%26R2_solutions:Chapter_7) to be maybe too complicated? 
I liked the simplicity of the answer in the C Answers Book.

## Code
```c
#include <stdio.h>
#include <ctype.h>
#include <string.h>

int main(int argc, char *argv[])
{
    int c, (*convert)(int c) = NULL;
    
    if ((strcmp(*argv,"lower") == 0) || (strcmp(*argv,"lower.exe") == 0))
        convert = tolower;
    else if ((strcmp(*argv,"upper") == 0) || (strcmp(*argv,"upper.exe") == 0))
        convert = toupper;
    
    if (convert == NULL) {
        while ((c = getchar()) != EOF)
            putchar(c);
    } else {
        while ((c = getchar()) != EOF)
            putchar(convert(c));
    }
    
    return 0;
}
```

## Output
<p align="center">
  <image src="../assets/exercise7-1_cmds.jpg" alt="output for exercise 7-1_cmds" />
</p>
<p align="center">
  <image src="../assets/exercise7-1_data_and_result.jpg" alt="output for exercise7-1 data and result" />
</p>

[Back to Main](../readme.md)
