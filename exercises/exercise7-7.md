# Exercise 7-7

> Modify the pattern finding program of Chapter 5 to take its input from a set of named files or, if no files are named as arguments,
> from the standard input. Should the file name be printed when a matching line is found? 

## Process
Took about 2 days, wrote the most straightforward solution I could think of.

Trying to be less perfectionist with these exercises, so once I had a working solution, I left it.

Learnt a lot after looking at other solutions to the exercise.

### My Modifications
- add input argument `-f` for adding files
- input argument files should come before the pattern
- add global struct `params` to store parameters
- add multidimensional array `filenames[][]` for storing input argument filenames
- pass `argc` and `argv` as pointers to functions to progress through the arguments

My main development focus was to keep `main` small and readable.

I wouldn't progress `argc` and `argv` through external functions again, pointer indirection is too messy.

I wrote separate functions for dealing with files and stdin, but that logic could definitely be combined.

### Other Solutions - CLC Wiki
Looked at the 4 submitted solutions. It's interesting that such a simple exercise can have different ways of solving it. Some solutions were easy to follow and others were not.

After looking at those, I realised:
- I could improve my usage instructions using `[optional]` and `[file1 file2 ...]` to clearly show users optional input arguments
- I didn't need to use the `-f` command and should have put filenames at the end of the argument list so they can infinitely expand

### Other Solutions - C Answer Book
This is such an amazing solution
- So straightforward and easy to read
- No dynamic memory allocation
- Great combination of handling files or stdin
- Nice way to slice the abstraction with `fpat`
    
## Code
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define MAXFILES 10
#define MAXFILENAME 100
#define MAXLINE 100

struct {
    int except;
    int number;
    int files;
} params = {0, 0, 0};

char filenames[MAXFILES][MAXFILENAME];
void getargs(int *argc, char ***argv);
int getfilenames(int *argc, char ***argv);
void matchfiles(int n, char *pattern);
void matchstdin(int argc, char *pattern);
int getline(char line[], int limit);

/* find: print all lines that match pattern in stdin or files */
int main(int argc, char *argv[])
{
    int found = 0;
    int n = 0;
    
    getargs(&argc, &argv);
    if (params.files) {
        n = getfilenames(&argc, &argv);
        matchfiles(n, *argv);
    } else {
        matchstdin(argc, *argv);
    }
}

/* matchstdin: read all lines in stdin and find matching lines */
void matchstdin(int argc, char *pattern)
{
    int lineno = 0, found = 0;
    char line[MAXLINE];
    
    if (argc < 1) {
        printf("Usage: find -x -n -f pattern|files\n");
        exit(0);
    }
    while (getline(line, MAXLINE) > 0) {
        lineno++;
        if ((strstr(line, pattern) != NULL) != params.except) {
            if (params.number)
                printf("%ld:", lineno);
            printf("%s", line);
            found++;
        }
    }
}

/* matchfiles: open all files and find matching lines */
void matchfiles(int n, char *pattern)
{
    FILE *fp;
    char line[MAXLINE];
    int lineno = 0, found = 0;
    
    for (int i = 0; i < n; i++) {
        if ((fp = fopen(filenames[i], "r")) == NULL) {
            fprintf(stderr, "Error: opening file %s\n", filenames[i]);
            exit(1);
        }
        
        while (fgets(line, MAXLINE, fp) != NULL) {
            lineno++;
            if ((strstr(line, pattern) != NULL) != params.except) {
                if (params.number)
                    printf("%ld:", lineno);
                printf("%s", line);
                found++;
            }
        }
        printf("Found: %d lines in file %s\n\n", found, filenames[i]);
        found = 0;
        fclose(fp);
    }
}

/* getfilenames: get  and store filenames from arguments */
int getfilenames(int *argc, char ***argv)
{
    int n;
    
    if (*argc < 2) { /* require at least one file and pattern */
        fprintf(stderr, "Usage: find -x -n -f files pattern\n");
        exit(1);
    }
    while ((*argc > 1) && (n < MAXFILES)) {
        if (strlen(**argv) > MAXFILENAME) {
            fprintf(stderr, "filename too large: %s", **argv);
            exit(1);
        }
        strcpy(filenames[n], **argv);
        n++, (*argv)++, (*argc)--;
    }
    return n;
}

/* getargs: get arguments and assign to params */
void getargs(int *argc, char ***argv)
{
    int c;
    
    while (--*argc > 0 && **(++*argv) == '-') {
        while (c = *(++**argv)) {
            switch (c) {
                case 'x':
                    params.except = 1;
                    break;
                case 'n':
                    params.number = 1;
                    break;
                case 'f':
                    params.files = 1;
                    break;
                default:
                    fprintf(stderr, "find: illegal option %c\n", c);
                    exit(1);
                    break;
            }
        }
    }    
}

/* getline: read a line into s, return length */
int getline(char line[], int limit)
{
    int c, i;
    
    for (i=0; i<limit-1 && (c=getchar()) != EOF && c!='\n'; ++i)
        line[i] = c;
    if (c == '\n') {
        line[i] = c;
        ++i;
    }
    line[i] = '\0';
    return i;
}
```

## Output
<p align="center">
  <image src="../assets/exercise7-7_cmds.jpg" alt="output for exercise7-7 cmds" />
</p>

<p align="center">
  <image src="../assets/exercise7-7_data_and_result.jpg" alt="output for exercise7-7 data and result" />
</p>

[Back to Main](../readme.md)
