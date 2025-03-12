# Exercise 8-5

> Modify the fsize program to print the other information contained in the inode entry.

## Process
This exercise took me 3 weeks and was a wild rabbithole into Windows and POSIX differences.

My development environment GCC 8.1.0 built with mingw-w64 on Windows 10 has served me well up until now, 
but it seems it doesn't cover the POSIX-specific functionality I needed for this exercise.

I knew I could probably find a solution because I got `fsize` working by using mingw-w64 `<dirent.h>`:
- replaced the custom `opendir`, `readdir` and `closedir` functions
- replaced `Dirent *dp` with `struct dirent *dp;`
- used `dp->d_name` instead of `dp->name`

So began a 3 week deep dive reading articles, header files, watching videos and much chatgpt-ing...

### TLDR: 
- rewrote `opendir`, `readdir` and `closedir` with MSVCRT functions `_findfirst`, `_findnext` and `_findclose` 
- used values from `stat()`

## Research 1 - POSIX on Windows
Firstly I had another pass at trying to understand the boundaries between WSL, Cygwin, MSYS, MinGW and MinGW-W64.

My understanding is:
```
Cygwin – Full POSIX-like environment on Windows; heavy, complex, and not native.
MinGW – GNU build tools for Windows; compiles native programs.
MinGW-w64 – Updated MinGW with 32-bit & 64-bit support; now the standard.
MSYS – Minimal Cygwin-based environment for MinGW-w64 and C development on Windows.
MSYS2 – Improved Cygwin-based version with a package manager; now the standard.
WSL1 – Full Linux OS using the Windows kernel; slower but no VM overhead.
WSL2 – Full Linux OS running in a lightweight Hyper-V VM; now the standard.
```

Some other readings:
- I found [video](https://www.youtube.com/watch?v=nbCB8bJU5eg) and [video](https://www.youtube.com/watch?v=KHWlz1GnFJU) helpful on understanding MSYS.
- [article](https://medium.com/@evilsapphire_s/how-is-a-hello-world-compiled-in-mingw-gcc-part-1-preprocessing-3e3898e3c11d) explaining how gcc compiles hello world with MinGW.
- Stackexhange [answer](https://unix.stackexchange.com/questions/11983/what-exactly-is-posix) on what POSIX is
- Windows doesn't follow the '[Everything is a file](https://en.wikipedia.org/wiki/Everything_is_a_file)' POSIX approach
- Some old [chat](https://mingw-users.narkive.com/JpEGojiV/linux-dirent-d-type-windows-equivalent) about `dirent.h` with `mingw`
- (unrelated but cool) Super cool [timeline diagram](https://www.levenez.com/unix/) of the evolution of Unix. (Also for [windows](https://www.levenez.com/windows) and [languages](https://www.levenez.com/lang))
- (unrelated but cool) Transcribed audio [interviews](https://mirror.math.princeton.edu/pub/unixarchive/Documentation/OralHistory/) with people who developed UNIX. 

## Research 2 - Partial POSIX Functionality
Some functions in MinGW seem to be implemented for compatibility rather than full functionality and don’t work as expected.

### `open`, `read`, `close`
MinGW maps `open`, `read` and `close` to msvcrt.dll Windows calls, which works with files but not directories.

For instance if I run:
```c
#include <stdio.h>
#include <io.h>
#include <fcntl.h>
#include <errno.h>

int main()
{
    int fd = open("some_directory", O_RDONLY);
    printf("%d, errno: %d, strerror: %s", fd, errno, strerror(errno));
}
```
The result is:
```
-1, errno: 13, strerror: Permission denied
```

### `opendir`, `readdir`, `closedir`
MinGW provides POSIX-like functions `opendir`, `readdir` and `closedir`, but they only partially work. 

The dirent structure exists, but certain values are always empty (from MinGW-w64 
[dirent.h](https://github.com/mirror/mingw-w64/blob/93f3505a758fe70e56678f00e753af3bc4f640bb/mingw-w64-headers/crt/dirent.h#L25)):
```c
struct dirent
{
	long		d_ino;		/* Always zero. */
	unsigned short	d_reclen;	/* Always zero. */
	unsigned short	d_namlen;	/* Length of name in d_name. */
	char		d_name[260]; /* [FILENAME_MAX] */ /* File name. */
};
```
From [Microsoft Docs (MSDN)](https://learn.microsoft.com/en-us/cpp/c-runtime-library/reference/stat-functions?view=msvc-170) on `_stat()`
```
"The inode, and therefore st_ino, has no meaning in the FAT, HPFS, or NTFS file systems."
```

## Research 3 - Documentation Stack
Documentation for standard C is pretty straightforward, but when you start looking into UNIX-like functions, it gets kind of murky if they exist or how they work. 

I had trouble finding solid documentation, I guess its some sort of combination of reading all the layers and then a bit of trial and error...

1. ISO C - C Standard Library - [cppreference](https://en.cppreference.com/w/c)
2. POSIX - OpenGROUP Docs [SuSv2](https://pubs.opengroup.org/onlinepubs/7990989775/), [SuSv3](https://pubs.opengroup.org/onlinepubs/009695399/)
3. MinGW-w64 - No centralized documentation, source code is best reference
4. GCC - [GNU Compiler Docs](https://gcc.gnu.org/onlinedocs/)
5. Windows CRT [MSVCRT/UCRT Docs](https://learn.microsoft.com/en-us/cpp/c-runtime-library/c-run-time-library-reference?view=msvc-170)
6. Windows DLLs - [Microsoft Docs (MSDN)](https://learn.microsoft.com/en-us/windows/win32/api/)
7. WindowsNT DLLs - Undocumented

This got me on to trying to see what the complilation process is actually doing, specifically at the linker stage

## Research 4 - GCC
I had a deeper look into how gcc works, trying to understand what the macros end up creating, which headers are being included and which dll's are being linked.

I found these commands quite useful:
```
gcc -M              Headers
gcc -H              Headers with paths
gcc -E              Preprocess
gcc -dD -E          Preprocess + dump defines
gcc -v              Verbose output of what gcc is doing
gcc -dumpspecs      baked-in defaults gcc follows if you gave it no options.
gcc -Wl, --verbose  print out what the linker is doing

nm a.exe            prints names embedded in a compiled executable
ar                  archive, honestly don't quite understand this tool yet but seems useful
```

Found this semicomplete beginner friendly [guide](https://gcc-newbies-guide.readthedocs.io/en/latest/) for working with gcc, seems the guy actually 
[works](https://developers.redhat.com/articles/2024/04/03/improvements-static-analysis-gcc-14-compiler#) on gcc now.

Also found this summarisation of an [article](https://en.wikibooks.org/wiki/GNU_C_Compiler_Internals/GNU_C_Compiler_Architecture) on GCC Internals pretty straightforward to follow along with

My understanding of how GCC works:
```
FROM CHATGPT:
1. Preprocessor (cpp - part of cc1.exe) → Expands macros & includes headers.
2. Compiler (cc1.exe for C, cc1plus.exe for C++) → Translates C/C++ code into assembly.
3. Assembler (as.exe) → Converts assembly into machine code (.o files).
4. Linker (ld/collect2.exe) → Combines .o files and libraries into an executable.
```
If you want to find where these internal components are you can use a command like:
`gcc -print-prog-name=cc1`

## Research 5 - `dirent.c`
I found two implementations of `dirent.c` for Windows which I used as reference:
- MinGW version of [dirent.c](https://github.com/mingw-w64/mingw-w64/blob/master/mingw-w64-crt/misc/dirent.c )
- [stackoverflow answer](https://stackoverflow.com/questions/23607994/where-is-the-code-for-dirent-h-opendir-readdir-closedir-undefined-symbol) that provided link
to old implementation of [dirent.c](https://web.archive.org/web/20190717230650/http://www.two-sdg.demon.co.uk/curbralan/code/dirent/dirent.c)

The MinGW version maps functions to probably the correct Win32 API combinations:
```
opendir - GetFileAttributes
readdir - FindFirst, FindNext
closedir - FindClose
```
But I couldn't access to `GetFileAttributes()` from MinGW, so I went the with the other implementation mapping

```
opendir - FindFirst
readdir - FindNext
closedir - FindClose
```

MinGW provides access to the Win32 API functions through `_findfirst`, `_findnext` and `_findclose` through `io.h`, which use a `struct _finddata_t`

Win32 API also seems to require you to add `/*` to the end of the filename if you want a directory, so `some_dir` becomes `some_dir/*`

## Research Final Thoughts
1. C was designed to be OS independant language, but POSIX is not so independant
2. Straying from purely C Standard library gets murky on documentation and portability on Windows
3. GCC and GNU Tools were built on POSIX, theres only so much MinGW can do on Windows
4. Might be better to work with a different compilier if doing a lot of Windows FileSystem stuff..

## Other Solutions - CLC-Wiki
Seems like there are even some differenes in Linux Distros on how these directory filesystem operations work

## Other Solutions - C Answer Book
Not relevant to me as I'm not using Unix

## Code `dirent.h`
```c
#define NAME_MAX 259     /* longest filename component */
                            /* system dependant */

typedef struct {    /* portable directory entry: */
    long ino;               /* inode number */
    char name[NAME_MAX+1];  /* name + '\0' terminator */
} Dirent;

typedef struct {    /* minimal DIR: no buffering, etc */
    int fd;              /* file descriptor for directory */
    Dirent d;           /* the directory entry */
} DIR;

DIR *opendir(char *dirname);
Dirent *readdir(DIR *dfd);
void closedir(DIR *dfd);

```
## Code `exercise8-5.c`
```c
#include <stdio.h>
#include <fcntl.h>
#include <stdlib.h>
#include <sys/stat.h>
#include <time.h>
#include "dirent.h"

void fsize(char *);

/* print file sizes */
int main(int argc, char **argv)
{
    if (argc == 1)  /* default: current directory */
        fsize(".");
    else
        while (--argc > 0)
            fsize(*++argv);
    return 0;
}

void dirwalk(char *, void (*fcn)(char *));

/* fsize: print size of file "name" */
void fsize(char *name)
{
    struct stat stbuf;
    
    if (stat(name, &stbuf) == -1) {
        fprintf(stderr, "fsize: can't access %s\n", name);
        return;
    }
    
    if ((stbuf.st_mode & S_IFMT) == S_IFDIR)
        dirwalk(name, fsize);
    
    struct tm *c_time = localtime(&stbuf.st_ctime);
    char c_time_buf[100];
    strftime(c_time_buf, sizeof(c_time_buf), "%Y-%m-%d %H:%M %p", c_time);
    printf("%20s", c_time_buf);
    
    struct tm *m_time = localtime(&stbuf.st_mtime);
    char m_time_buf[100];
    strftime(m_time_buf, sizeof(m_time_buf), "%Y-%m-%d %H:%M %p", m_time);
    printf("%20s", m_time_buf);
    
    struct tm *a_time = localtime(&stbuf.st_atime);
    char a_time_buf[100];
    strftime(a_time_buf, sizeof(a_time_buf), "%Y-%m-%d %H:%M %p", a_time);
    printf("%20s", a_time_buf);
    
    printf("%8ld %s\n", stbuf.st_size, name);
}

#define MAX_PATH 1024

/* dirwalk: apply fcn to all files in dir */
void dirwalk(char *dir, void (*fcn)(char *))
{
    char name[MAX_PATH];
    Dirent *dp;
    DIR *dfd;
    
    if ((dfd = opendir(dir)) == NULL) {
        fprintf(stderr, "dirwalk: can't open %s\n", dir);
        return;
    }
    while ((dp = readdir(dfd)) != NULL) {
        if (strcmp(dp->name, ".") == 0 ||
            strcmp(dp->name, "..") == 0)
            continue; /* skip self and parent */
        if (strlen(dir)+strlen(dp->name)+2 > sizeof(name))
            fprintf(stderr, "dirwalk: name %s/%s too long\n", dir, dp->name);
        else {
            sprintf(name, "%s/%s", dir, dp->name);
            (*fcn)(name);
        }
    }
    closedir(dfd);
}

/* my_opendir: open a directory for my_readdir calls */
DIR *opendir(char *dirname)
{
    int fd;
    struct stat stbuf;
    struct _finddata_t info;
    DIR *dp;
    char dirname_w32[MAX_PATH+3]; /* windows dir ext "/*" with null '\0' */
    
    strcpy(dirname_w32, dirname);
    strcat(dirname_w32, "/*");
     
    if ((fd = _findfirst(dirname_w32, &info)) == -1)
        return NULL;
    
    if ((dp = (DIR *) malloc(sizeof(DIR))) == NULL)
        return NULL;
    
    dp->fd = fd;
    return dp;
}

/* readdir: read directory entries in sequence */
Dirent *readdir(DIR *dfd)
{
    int fd;
    struct _finddata_t dirbuf;
    static Dirent d;
    
    while ((fd = _findnext(dfd->fd, &dirbuf)) != -1) {
        strncpy(d.name, dirbuf.name, NAME_MAX-1);
        d.name[NAME_MAX] = '\0'; /* ensure termination */
        return &d;
    }
    
    return NULL;
}

void closedir(DIR *dp)
{
    if (dp) {
        _findclose(dp->fd);
        free(dp);
    }
}
```

## Output
<p align="center">
  <image src="../assets/exercise8-5_cmds.jpg" alt="output for exercise8-5 cmds" />
</p>

[Back to Main](../readme.md)
