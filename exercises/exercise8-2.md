# Exercise 8-2

> Rewrite fopen and _fillbuf with fields instead of explicit bit operations. Compare code size and execution speed.

## Process
This was a fairly straightforward exercise, rewriting `fopen` and `_fillbuf` using bitfields with a manual search and replace.
I also spent a while doing research on things I didn't understand.

I'm assuming by `fields` this question refers to `bit-fields` as the index on page 266
says `field see bit-field`.

```c
// Quick reminder of a bit-field
struct {
  unsigned int x : 1; // 1 bit
  unsigned int y : 1; // 1 bit
} flags;
// access bit-fields as structures eg. flags.x = 1
```

In direct answer to the exercise question, I didn't notice any significant difference between execution speed or code size.
The bit-fields method used slightly more lines of code, ran 0.5 of a millisecond faster (after caching), and would use slightly less memory, so maybe that's better for caching or tight embedded system.

Maybe bit-fields would be faster in a hot loop, however due to the completely implementation dependant operation of bit-fields, 
I think using a normal structure would be less error prone and modern computing speeds and memory might negate any possible benefit that bit-fields would give.

### Testing
After much research (see below), I ended up using a similar test to the anonymous answer in CLC-Wiki, creating a random file full of text, then reading it in and 
timing it using the same method in PowerShell that I used for the previous exercise.

`createdata.c`
```c
#include <stdio.h>

int main()
{
    for (int i = 0; i < 100000; i++)
        printf("Lorem ipsum dolor sit amet, consectetur adipiscing elit. Sed non risus. Suspendisse lectus tortor, dignissim sit amet, adipiscing nec, ultricies sed, dolor. Cras elementum ultrices diam. Maecenas ligula massa, varius a, semper congue, euismod non, mi. Proin porttitor, orci nec nonummy molestie, enim est eleifend mi, non fermentum diam nisl sit amet erat. Duis semper. Duis arcu massa, scelerisque vitae, consequat in, pretium a, enim. Pellentesque congue. Ut in risus volutpat libero pharetra tincidunt. Cras  dapibus. Vivamus elementum semper nisi. Aenean vulputate eleifend tellus. Aenean leo ligula, porttitor eu, consequat vitae, eleifend ac, enim.");
}
```
`exercise8-2.ps1`
```powershell
Write-Output "i : Milliseconds"
Write-Output "---------------------"
$avg
for ($i = 0; $i -lt 10; $i++) {
    $time = Measure-Command { .\exercise8-2.exe }
    $avg += $time.TotalMilliseconds
    Write-Output "$i : $($time.TotalMilliseconds)"
}
Write-Output ""
Write-Output "---------------------"
Write-Output "average $($avg/10)"
Write-Output ""
```

### Other Solutions - CLC-Wiki
Checking the [answers](https://clc-wiki.net/wiki/K%26R2_solutions:Chapter_8:Exercise_2) on CLC-Wiki, I was surprised to see that the 
first two solutions used structures instead of bit-fields.

The final anonymous solution pointed that out and had a pretty good overview of the solution. They also embeded 
the bitfield structure into `_iobuf`, which I didn't think of. I also liked their method of testing the program by reading
a large file, so I copied that for my test.

### Other Solutions - C Answer Book
Interesting to see that they explicitly set fields off, whereas I was relying on fields being defaulted to 0, which is error prone.
Also they added another field of `is_buf` to complement `is_unbuf`, I suppose to be more explicit.

### Futher Research
I was satisfied with my solution at that point, but I wanted to actually check if my code was doing what I 
thought it was. 
I was confused specifically with:
- did `stdlib.h`, `fcntl.h` or gcc bring in external functionality without me knowing?
- this line `extern FILE _iob[OPEN_MAX];` made me think it was refering to an OS external references
- how does the code call the Unix system calls on Windows if it's not Unix? Through MinGW somehow?
- is the code calling the c runtime and how would I know?

From here I went on a long rabbithole about gcc, profiling and windows user mode architecture:

1. gcc

Read documentation and tried different switches. It seems I was interested in the linking step of compilation, but wasn't able to get the info I was looking for.

2. C profiling, gcov, gprof

Watched this [video](https://www.youtube.com/watch?v=M6RCUiZzl8Y) by Jacob Sorber
about profiling function calls. Also, I tried some of the gnu profiling tools which while interesting, didn't seem like
they gave me the information I was looking for.

3. Very Sleepy

Looking at this awesome profilers [list](https://github.com/fenbf/AwesomePerfCpp) and found this profiling tool for Windows called [Very Sleepy](http://www.codersnotes.com/sleepy/)
which basically links your source code to dll system calls. I had some trouble with it until I realised that due to the way it records time, 
if a program ran too fast, it wouldn't show anything.

4. extern

I intially thought the line `extern FILE _iob[OPEN_MAX];` was referencing code in another translation unit, but then after looking up `extern`
I realised its just delaying the definition to later in the same file. I found this definition on page 80:
> "...if an external variable is to be referred to before it is defined,
> or if it is defined in a different source file from the one where it is being used,
> then an extern declaration is mandatory"
  
5. windows error codes

Found this official [defintion](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-erref/87fba13e-bf06-450e-83b1-9241dc81e781) of how to read windows error codes.
While running programs and checking the `%errorlevel%` in cmd, I didn't know how to read them, but things make a bit more sense now.

6. Windows Assessment and Deployment Kit

Had a look at the Windows Performance recorder (WPR) as part of the offical profilers, but it's 
such a large toolset and maybe overkill for learning about a program specifically.

7. process monitor & process explorer

Two tools from sysinternals, which I learned is a separate group of people that created Windows tools that then became part of microsoft but still retain some independance.
They were cool and I might use them in the future.
Some videos I watched: [video 1](https://www.youtube.com/watch?v=tR22u6H8E5w) [video 2](https://www.youtube.com/watch?v=HdV9QuvgS_w)

9. api monitor

[api monitor](http://www.rohitab.com/apimonitor) was exactly the tool I was looking for. You launch your program and it shows all the system calls your program makes
and which dll they are tied to. Unfortunely the timings are way off because I guess it's interrupting every process to log things.

11. pe explorer

Watched this talk [Pavel Yosifovich: Native Applications: What, Why, and How?](https://www.youtube.com/watch?v=EKBvLTuI2Mo) and he used a program he made called pe explorer
which was also really cool to just see what dll's the program has linked.

13. crt versions

Spent a while trying to understand how the C runtime works on windows. Apparently there a 3 different 'types':
- `msvcrt.dll` is the older version, which is available everywhere, gcc uses for my programs, but will be phased out
- `C++ Redistributables` are patches over time to the C runtime that windows have been making over time
- `ucrt.dll` is the new "Universal C Runtime" for Windows 10 onwards. Its less UNIX like and aims to be the new standard on Windows

## Code
```c
#define NULL        0
#define EOF         (-1)
#define BUFSIZ      1024
#define OPEN_MAX    20  /* max #files open at once */

struct _flags {
    unsigned int _READ  : 1; /* file open for reading */
    unsigned int _WRITE : 1; /* file open for writing */
    unsigned int _UNBUF : 1; /* file is unbuffered */
    unsigned int _EOF   : 1; /* EOF has occurred on this file */
    unsigned int _ERR   : 1; /* error occurrred on this file */
};

typedef struct _iobuf {
    int   cnt;          /* characters left */
    char *ptr;          /* next character position */
    char *base;         /* location of buffer */   
    struct _flags flag; /* mode of file access */
    int   fd;           /* file descriptor */
} FILE;
extern FILE _iob[OPEN_MAX];

#define stdin  (&_iob[0])
#define stdout (&_iob[1])
#define stderr (&_iob[2])

int _fillbuf(FILE *);
int _flushbuf(int, FILE *);

#define feof(p)     (((p)->flag._EOF) != 0)
#define ferror(p)   (((p)->flag._ERR) != 0)
#define fileno(p)   ((p)->fd)

#define getc(p) (--(p)->cnt >= 0 \
                ? (unsigned char) *(p)->ptr++ : _fillbuf(p))
#define putc(x,p) (--(p)->cnt >= 0 \
                ? *(p)->ptr++ = (x) : _flushbuf((x),p))

#define getchar()     getc(stdin)
#define putchar(x)    putc((x), stdout)

#include <fcntl.h>
#define PERMS 0666 /* RW for owner, group, others */

/* fopen: open file, return file ptr */
FILE *fopen(char *name, char *mode)
{
    int fd;
    FILE *fp;
    
    if (*mode != 'r' && *mode != 'w' & *mode != 'a')
        return NULL;
    for (fp = _iob; fp < _iob + OPEN_MAX; fp++)
        if(fp->flag._READ == 0 && fp->flag._WRITE == 0)
            break; /* found free slot */
    if (fp >= _iob + OPEN_MAX)  /* no free slots */
        return NULL;
        
    if (*mode == 'w')
        fd = creat(name, PERMS);
    else if (*mode == 'a') {
        if ((fd = open(name, O_WRONLY, 0)) == -1)
            fd = creat(name, PERMS);
        lseek(fd, 0L, 2);
    } else
        fd = open(name, O_RDONLY, 0);
    if (fd == -1)   /* couldn't access name */
        return NULL;
    fp->fd = fd;
    fp->cnt = 0;
    fp->base = NULL;
    if ((*mode == 'r'))
        fp->flag._READ = 1;
    else
        fp->flag._WRITE = 1;
    return fp;
}

#include <stdlib.h>

/* _fillbuf: allocate and fill input buffer */
int _fillbuf(FILE *fp)
{
    int bufsize;
    
    if ((fp->flag._READ == 0) || (fp->flag._EOF == 1) || (fp->flag._ERR == 1))
        return EOF;
    bufsize = fp->flag._UNBUF == 1 ? 1 : BUFSIZ;
    if (fp->base == NULL)   /* no buffer yet */
        if ((fp->base = (char *) malloc(bufsize)) == NULL)
            return EOF;     /* can't get buffer */
    fp->ptr = fp->base;
    fp->cnt = read(fp->fd, fp->ptr, bufsize);
    if (--fp->cnt < 0) {
        if (fp->cnt == -1)
            fp->flag._EOF = 1;
        else
            fp->flag._ERR = 1;
        fp->cnt = 0;
        return EOF;
    }
    return (unsigned char) *fp->ptr++;
}

FILE _iob[OPEN_MAX] = {     /* stdin, stdout, stderr */
    {0, (char *) 0, (char *) 0, {._READ = 1 }, 0},
    {0, (char *) 0, (char *) 0, {._WRITE = 1}, 1},
    {0, (char *) 0, (char *) 0, {._WRITE = 1, ._UNBUF = 1},  2},
};

int main()
{
    FILE *file;
    char c;
    
    file = fopen("data.txt", "r");
    while ((c = getc(file)) != EOF)
        ;
}
```

## Output
<p align="center">
  <image src="../assets/exercise8-2_cmds.jpg" alt="output for exercise8-2 cmds" />
</p>

<p align="center">
  <image src="../assets/exercise8-2_result.jpg" alt="output for exercise8-2 result" />
</p>

[Back to Main](../readme.md)
