# Exercise 8-4

> the standard library function
> 
> ``` int fseek(FILE *fp, long offset, int origin) ```
> 
> is identical to `lseek` except that `fp` is a file pointer instead of a file descriptor and the return value is an `int` status, not a position. Write `fseek`.
> Make sure that your `fseek` coordinates properly with the buffering done for the other functions of the library.

## Process
This one took me a little while to familiarise myself with how fseek is supposed to work. 


I can see it being a fast, useful way to manipulate files but I found it a bit of a bizarre function to work with, but I guess it's because I'm not used to its functionality. 

I found the 'lazy' style execution a bit strange. `lseek` moves the file pointer within the OS but you dont see anything in the `FILE` until you read. 
Also sometimes it works on the `FILE` buffer as opposed to making an `lseek` call. I just made my solution always call `lseek`.

I read that switching between read and write should execute or requires a flush, but I ignored that.

I stopped after my solution matches how the standard library `fseek` works for my test case, as I didn't want to go down the endless rabbit hole
of accounting for all possible edgecases, because I felt I got the gist of the exercise.

I ran into a problem when the `FILE` reached `EOF` then fseek would have no effect. Eventually I realised that i needed to switch off the `_EOF` flag by setting `cnt = 0` after using `fseek`.

### Research
I was confused how the Unix `fseek` connects to the underlying Windows `fseek` through. I assume it happens through mingw but didn't find the exact way that happens.

- `gcc -H file.c` was good way to see all included headers in a file and where they come from
- `objdump -S` was good for outputing and connecting the assembly to c source when I was looking to see what was going on.

### Other Solutions - CLC-Wiki
The CLC solutions made me realise that focusing on READ/WRITE instead of `origin` is a better abstraction point for the `fseek`.

The solution by anonymous had very verbose error checking, which actually caught an error in my code that the standard library fseek didn't catch.
I guess the standard library fseek just lets you do whatever you want.

### Other Solutions - C Answer Book
As usual, a super simple solution.

## Code `min_stdio.h`
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

/* fopen: open file, return file ptr */
FILE *fopen(char *name, char *mode)
{
    int fd;
    FILE *fp;
    
    if (*mode != 'r' && *mode != 'w' & *mode != 'a')
        return NULL;
    for (fp = _iob; fp < _iob + OPEN_MAX; fp++)
        if ((fp->flag & (_READ | _WRITE)) == 0)
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
    fp->flag = (*mode == 'r') ? _READ : _WRITE;
    return fp;
}

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
            return EOF;     /* cant get buffer */
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

/* On an output stream, fflush causes any buffered but unwritten data to be written;
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

/* fclose flushes any unwritten data for stream, discards any unread buffered input,
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
```
## Code `exercise8-4.c`
```c
#include "min_stdio.h"

int fseek(FILE *fp, long int offset, int origin)
{
    int r;
    if (fp->base == NULL) {
        if (lseek(fileno(fp), offset, origin) == -1)
            return -1;
    } else {
        fp->flag &= ~_EOF;
        switch (origin) {
            case 0:
                fp->cnt = 0; /* force _fillbuf to read */
                if (lseek(fileno(fp), offset, origin) == -1);
                    return -1;
                break;
            case 1:
                if (lseek(fileno(fp), offset, origin) == -1);
                    return -1;
                break;
            case 2:
                fp->cnt = 0; /* force _fillbuf to read */
                if (lseek(fileno(fp), offset, origin) == -1)
                    return -1;
                break;
            default:
                return -1;
        }
    }
    return 0;
}

int main()
{
    FILE *fp;
    int c;
    
    fp = fopen("data.txt", "r");
    
    /* move 6 chars from beginning - "world" */
    fseek(fp, 6, 0);
    while ((c = getc(fp)) != EOF)
        putchar(c);
    putchar('\n');
    
    /* return to beginning and move 3 chars - "lo world" */
    fseek(fp, 0, 0);
    fseek(fp, 3, 1);
    while ((c = getc(fp)) != EOF)
        putchar(c);
    putchar('\n');
    
    /* last 3 chars - "rld" */
    fseek(fp, -3, 2);
    while ((c = getc(fp)) != EOF)
        putchar(c);
    
    fclose(fp);
    fflush(stdout);
}
```

## Output
<p align="center">
  <image src="../assets/exercise8-4_cmds.jpg" alt="output for exercise8-4 cmds" />
</p>

[Back to Main](../readme.md)
