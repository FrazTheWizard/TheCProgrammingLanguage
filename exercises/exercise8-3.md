# Exercise 8-3

> Design and write _flushbuf, fflush and fclose

## Process
This one took me on a bit of a journey. Turns out I had the wrong understanding of how IO works.

I pieced together what I need to do from the existing code in the book and my own research. 
I assume `_flushbuf` does the allocation and reseting like `_fillbuf`, then I assume `fflush` calls `_flushbuf` and then finally `fclose` calls `fflush`. 
I made a simple example of the code working how I think it should, but honestly I am little confused on what exactly these functions should do.

### Research

A simple program with `putchar()` prints a character and stepping through individual `putchar()`s in `gdb` prints each character, so I assumed it just makes a sys call each time to write the character.
But it doesn't, as far as I [understand](https://www.gnu.org/software/libc/manual/html_node/Flushing-Buffers.html), the C runtime flushes all buffers when the program exits successfully. So a call to `putchar` just puts the character into the stdout buffer and then it's flushed
only when the buffer is full or at the end of the program. `gdb` doesn't buffer output, so thats why it prints each character.

Also, there are 3 types of [buffering](https://www.gnu.org/software/libc/manual/html_node/Buffering-Concepts.html): unbuffered, line buffered and full buffering. 
Line buffering is used for terminals. Full buffering is more for files.

I read more into [standard streams](https://pubs.opengroup.org/onlinepubs/9699919799/functions/V2_chap02.html#tag_15_05),
[crt0](https://en.wikipedia.org/wiki/Crt0) and [POSIX streams](https://pubs.opengroup.org/onlinepubs/9699919799/functions/V2_chap02.html#tag_15_05).

According to the [SEI CERT C Coding standards](https://wiki.sei.cmu.edu/confluence/display/c/FIO23-C.+Do+not+exit+with+unflushed+data+in+stdout+or+stderr), its necessary to explicitly flush buffers before
exiting and you can't rely on the c runtime to do it for you.
I also found this [Comp Cert C Compiler](https://en.wikipedia.org/wiki/CompCert) which is meant to be formally correct mathematically. Maybe I will use it in the future.

I played around with `objdump -d` on executables to see all the startup functions as assembly in action.
Found this program [x64dbg](https://x64dbg.com/) which seems good for debuging assembly, as well as [crackmes.one](https://crackmes.one/) which has practice executables to crack.
Watched this video by Jacob Sorber on [-g3 -ggdb](https://www.youtube.com/watch?v=dh1mil1ehvE) which helped debugging macros with gdb.

Was reading into microsoft's rewriting of the [C runtime](https://devblogs.microsoft.com/cppblog/the-great-c-runtime-crt-refactoring/) and
kind of lost interest in the rabbit hole [here](https://learn.microsoft.com/en-us/previous-versions/6wdz5232(v=vs.140))

Maybe it's being on windows, or maybe its just C runtimes themselves, but I find this lower level very confusing and hard to know where to find the correct information on what I need to know or do.
I feel a bit unsatisfied with this exercise, but I have a working solution and I think I've spent enough time on it trying to understand.

### Other Solutions - CLC Wiki
Again [Anonymous](https://clc-wiki.net/wiki/K%26R2_solutions:Chapter_8:Exercise_3) having the most robust descriptive answer. Although I don't understand their logic of `_flushbuf` calling `fflush` though, seems backwards to me.

### Other Solutions - C Answer Book
Again these answers are so streamlined. It seems so obvious to simple and easy to write it like this after reading it.

I liked the pointer arithmetic comparison in `_flushbuf` and `fflush` to see if the file pointer is in the array. Way better than my `for` loop comparison.
```c
if (fp < _iob || fp >= _iob + OPEN_MAX)
    return EOF;
```
Although I noticed they didn't handle the case of `fflush` being given `NULL`, in which case it should flush all buffers.

## Code
```c
#define NULL        0
#define EOF         (-1)
#define BUFSIZ      1024
#define OPEN_MAX    20  /* max #files open at once */

typedef struct _iobuf {
    int   cnt;          /* characters left */
    char *ptr;          /* next character position */
    char *base;         /* location of buffer */
    int   flag;         /* mode of file access */
    int   fd;           /* file descriptor */
} FILE;
extern FILE _iob[OPEN_MAX];

#define stdin  (&_iob[0])
#define stdout (&_iob[1])
#define stderr (&_iob[2])

enum _flags {
    _READ   = 01,       /* file open for reading */
    _WRITE  = 02,       /* file open for writing */
    _UNBUF  = 04,       /* file is unbuffered */
    _EOF    = 010,      /* EOF has occurred on this file */
    _ERR    = 020       /* error occurrred on this file */
};

int _fillbuf(FILE *);
int _flushbuf(int, FILE *);

#define feof(p)     (((p)->flag & _EOF) != 0)
#define ferror(p)   (((p)->flag & _ERR) != 0)
#define fileno(p)   ((p)->fd)

#define getc(p) (--(p)->cnt >= 0 ? (unsigned char) *(p)->ptr++ : _fillbuf(p))
#define putc(x,p) (--(p)->cnt >= 0 ? *(p)->ptr++ = (x) : _flushbuf((x),p))
                
#define getchar()     getc(stdin)
#define putchar(x)    putc((x), stdout)

#include <fcntl.h>
#define PERMS 0666 /* RW for owner, group, others */

#include <stdlib.h>

/* _fillbuf: allocate and fill input buffer */
int _fillbuf(FILE *fp)
{
    int bufsize;
    
    if ((fp->flag&(_READ|_EOF|_ERR)) != _READ)
        return EOF;
    bufsize = (fp->flag & _UNBUF) ? 1 : BUFSIZ;
    if (fp->base == NULL)   /* no buffer yet */
        if ((fp->base = (char *) malloc(bufsize)) == NULL)
            return EOF;     /* can't get buffer */
    fp->ptr = fp->base;
    fp->cnt = read(fp->fd, fp->ptr, bufsize);
    if (--fp->cnt < 0) {
        if (fp->cnt == -1)
            fp->flag |= _EOF;
        else
            fp->flag |= _ERR;
        fp->cnt = 0;
        return EOF;
    }
    return (unsigned char) *fp->ptr++;
}

/* _flushbuf: flush file buffer and reset it */
int _flushbuf(int x, FILE *fp)
{
    int bufsize, n;
    
    if ((fp->flag & _ERR) == _ERR)
        return EOF;
    if ((fp->flag & _WRITE) != _WRITE)
        return EOF;
    bufsize = (fp->flag & _UNBUF) ? 1 : BUFSIZ;
    if (fp->base == NULL) { /* no buffer yet */
        if ((fp->base = (char *) malloc(bufsize)) == NULL)
            return EOF;     /* can't get buffer */
        fp->ptr = fp->base;
        if ((fp->flag & _UNBUF) != _UNBUF) {
            *(fp->ptr++) = x;
            fp->cnt = bufsize-1;
            return fp->cnt;
        }
    }
    
    n = write(fp->fd, fp->base, bufsize - fp->cnt);
    if (n != (bufsize - fp->cnt))
        return EOF;
    fp->ptr = fp->base;
    fp->cnt = bufsize;    
    return fp->cnt;
}

/* from page 242: On an output stream, fflush causes any buffered but unwritten data to be written;
on an input stream the effect is undefined. It returns EOF for a write error, and
zero otherwise. fflush (NULL) flushes all output streams */
int fflush(FILE *fp)
{
    int i;
    
    if (fp == NULL) {
        for (i = 0; i < OPEN_MAX; i++) {
            if (_iob[i].flag & _WRITE && !(_iob[i].flag & _UNBUF)) {
                if (_flushbuf(0, &_iob[i]) == EOF) {
                    return EOF;
                }
            }
        }
    } else {
        if (_flushbuf(0, fp) == EOF)
            return EOF;
    }
    return 0;
    
}

/* from page 242: fclose flushes any unwritten data for stream, discards any unread buffered input,
frees any automatically allocated buffer, then closes the stream. It returns EOF if
any errors occured, and zero otherwise. */
int fclose(FILE *fp)
{
    int fd;
    if (fp == NULL)
        return EOF;
    if (fflush(fp) == EOF)
        return EOF;
    if (fp->base != NULL)
        free(fp->base);
    fp->ptr = NULL;
    fp->cnt = 0;
    fp->base = NULL;
    fp->flag = 0;
    fd = fp->fd;
    fp->fd = -1;
    return close(fd);
}


FILE _iob[OPEN_MAX] = {     /* stdin, stdout, stderr */
    {0, (char *) 0, (char *) 0, _READ,   0},
    {0, (char *) 0, (char *) 0, _WRITE,  1},
    {0, (char *) 0, (char *) 0, _WRITE | _UNBUF,  2},
};

int main()
{
    putc('a', stdout);
    putchar('b');
    _flushbuf(0, stdout);
    putc('c', stdout);
    putchar('d');
    fclose(stdout);
    putchar('e');
}
```

## Output
<p align="center">
  <image src="../assets/exercise8-3_cmds_and_result.jpg" alt="output for exercise8-3 cmds and result" />
</p>

[Back to Main](../readme.md)
