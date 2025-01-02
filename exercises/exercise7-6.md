# Exercise 7-6

> Write a program to compare two files, printing the first line where they differ

## Process
Very happy to finally be handling files in C!

Pretty straightforward exercise, just wrote what kind of came naturally. Happy to see it matched the solutions I looked at.
Skipped any error checking for this exercise.

### Other Answers - CLC Wiki
As usual I liked codybartfast's solutions the best. I liked the compactness of the non error checking solution

### Other Answers - The C Answer Book
An even more compact solution as an if-else block. I can see how that would help readability, where you can clearly see the program paths as opposed to having to read through the program to see the exit points.
    
## Code
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define MAX 100

int main(int argc, char *argv[])
{
    FILE *f1, *f2;
    char s1[MAX], s2[MAX], *n1, *n2, *prog = argv[0];
    
    if (argc != 3) {
        fprintf(stderr, "%s: requires two files to compare\n", prog);
        exit(1);
    } else {
        n1 = argv[1];
        n2 = argv[2];
    }
    
    if ((f1 = fopen(argv[1], "r")) == NULL) {
        fprintf(stderr, "error reading file: %s\n", argv[1]);
        exit(1);
    }
    
    if ((f2 = fopen(argv[2], "r")) == NULL) {
        fprintf(stderr, "error reading file: %s\n", argv[2]);
        exit(1);
    }
    
    while ((fgets(s1, MAX, f1) != NULL) && (fgets(s2, MAX, f2) != NULL)) {
        if (strcmp(s1, s2) != 0) {
            fprintf(stdout, "%s:\n%s\n%s:\n%s", n1, s1, n2, s2);
            exit(0);
        }
    }
    
    fclose(f1);
    fclose(f2);
    exit(0);
}
```

## Output
<p align="center">
  <image src="../assets/exercise7-6_cmds.jpg" alt="output for exercise7-6 cmds" />
</p>

<p align="center">
  <image src="../assets/exercise7-6_data_and_results.jpg" alt="output for exercise7-6 data and result" />
</p>

[Back to Main](../readme.md)
