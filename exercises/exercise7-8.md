# Exercise 7-8

> Write a program to print a set of files, starting each new one on a new page, with a title and a running page count for each file.

## Process
This was a fun exercise and did it in a morning.

I interpreted this exercise as creating output that could be sent to a printer and printed as pages on paper as opposed to terminal printing. 

Researched the formfeed control `\f` or ASCII control character 12 and experimented printing with Windows.
I decided it would be most practical to use the `Microsoft Print to PDF` 'printer' to print to a PDF file to prove it works. 

I settled on using the inbuilt WordPad program on Windows 10 that can interpret the formfeed control and creates a new page. The program
outputs the file as a `*.doc`, because I thought that tricked WordPad into using the formfeed character, but I realised later that it 
still reads the formfeed character even if it's a text file, so it's not necessary.

The settings I used in WordPad were
```
page            A4 page
margin          1 inch / 25.4 mm
font            courier new
size            12pt
line spacing    1.0
```

Those settings created these limits which I used in the program:
```
line length     62
lines per page  54
```

### Other Solutions - CLC Wiki
Looked at the three solutions, which all interpreted the exercise as printing to the terminal.

### Other Solutions - C Answer Book
Again it's another amazingly concise answer from this book, kind of wish I'd discovered it earlier going through these exercises.

This answer used the formfeed method for 'printing', and as it lists Brian Kernighan as a contributer, I'm going to assume this is the correct
interpretation of this exercise.

### Afterthought
After looking at those solutions and using the program I would change some things:
- if no files provided as arguments, take from stdin. Better flexibility
- just output to stdout and let the user decide how to write the file through the cmdline. Better flexibility
- use `fgets` with a `MAXLINE`, instead of processing each character and manually tracking line length. Simplier Logic

Im happy with
- doing a `isprint()` on characters before printing, all other solution I tried didn't have this
- general logic of meeting line restrictions and making some 'smart' descisions


## Code
```c
#include <stdio.h>
#include <stdlib.h>
#include <ctype.h>

#define MAXLINELEN 62
#define MAXLINESNUM 54
#define OUTPUTFILENAME "file.doc"

int main(int argc, char *argv[])
{
    FILE *wfile = NULL, *rfile;
    int filesread = 0;
    void writefile(FILE *wfile, FILE *rfile, char *filename);
    
    if (argc == 1) {
        printf("Usage: printfiles file1 [file2 ...]\n");
        exit(0);
    }
    
    while (--argc > 0 && ++argv != NULL) {
        /* open read file */
        if ((rfile = fopen(*argv, "r")) == NULL) {
            fprintf(stderr, "Error: cannot open file %s\n", *argv);
        } else {
            filesread++;    
            /* open write file if not already open */
            if (wfile == NULL) {
                if ((wfile = fopen(OUTPUTFILENAME, "w")) == NULL) {
                    fprintf(stderr, "Error: cannot open writefile\n");
                    exit(-1);
                }
            }
            printf("reading file - %s\n", *argv);
            writefile(wfile, rfile, *argv);
            fclose(rfile);
        }
    }
    if (wfile != NULL) fclose(wfile);
    if (filesread == 0) {
        fprintf(stderr, "Error: unable to read any files\n");
        exit(-1);
    }
    printf( "%d file%s printed to %s\n",
            filesread, filesread > 1 ? "s" : "", OUTPUTFILENAME);
}           

void writefile(FILE *wfile, FILE *rfile, char *filename)
{
    int c, linelen = 0, linenum = 0, pagenum = 1;
    
    while ((c = fgetc(rfile)) != EOF) {

        /* check line length */
        if (linelen >= MAXLINELEN && c != '\n') {
            linenum++;
            linelen = 0;
            fputc('\n', wfile);
        }
        
        /* check page size */
        if (linenum >= MAXLINESNUM) {
            pagenum++;
            linenum = 0;
            linelen = 0;
            fputc('\f', wfile);
        }
        
        /* print page title */
        if (linenum == 0) {
            fprintf(wfile, "%s - page %d", filename, pagenum);
            fprintf(wfile, "\n\n");
            linenum += 3;
        }
        
        /* check newline */
        if (c == '\n') {
            linenum++; 
            linelen = 0;
            fputc('\n', wfile);
        } 
        
        /* print char */
        if (isprint(c)) {
            linelen++;
            fputc(c, wfile);
        }
    }
    
    fputc('\f', wfile);
}
```

## Output
<p align="center">
  <image src="../assets/exercise7-8_cmds.jpg" alt="output for exercise7-8 cmds" />
</p>

<p align="center">
  <image src="../assets/exercise7-8_result1.jpg" alt="output for exercise7-8 result 1" />
</p>

<p align="center">
  <image src="../assets/exercise7-8_result2.jpg" alt="output for exercise7-8 result 2" />
</p>

[Back to Main](../readme.md)
