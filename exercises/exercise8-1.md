# Exercise 8-1

> Rewrite the program **cat** from Chapter 7 using **read**, **write**, **open** and **close** instead of their standard library equivalents.
> Perform experiments to determine the relative speeds of the two versions.

## Process
The exercise itself was fairly straight forward, but this took me on a little bit of a journey understanding some underlying Windows things...

### no `syscalls.h` on Windows
Microsoft took a non UNIX approach to system calls on Windows, so there is no `syscalls.h`. 
Instead, Windows provides its own system calls like `CreateFile` and `CloseHandle`.

However, you can still use UNIX-style functions  `read`, `write`, `open`, `creat`, 
`close`, and `unlink` on Windows through `<fcntl.h>` and `<unistd.h>`. 
These headers `#include <io.h>`, which maps UNIX functions to their Windows equivalents, creating a POSIX compatibility layer.

Some videos I found helpful
- [Jacob Sorber - fopen vs open](https://www.youtube.com/watch?v=BQJBe4IbsvQ)
- [Jacob Sorber - syscall vs libcall](https://www.youtube.com/watch?v=2AmP7Pse4U0)

### PowerShell
It seemed PowerShell had an easy way to time program execution, so I left cmd.exe and went over there and was surprised to find 
a lot of the Unix style commands didn't work as I expected. Seems PowerShell is developed for Windows, which has a different model. 
Reading from an [interview](https://evrone.com/blog/jeffrey-snover-interview) with Jeffrey Snover (created Powershow):

> There is a core architectural difference between Unix and Windows.
> Linux is a file-oriented OS and Windows is an API-oriented OS.
> Everything was behind an API which returned a structured object.

A 2000's interview that was quite informative actually:
- [Jeffrey Snover - Monads Explained](https://www.youtube.com/watch?v=d0joo5iHCxs)

### UTF-16 LE BOM
I also found outputing text files from PowerShell inserted wierd characters. 
Seems PowerShell and indeed Windows itself is built on UTF-16, when that was the new thing and too much code rely's on it, 
so they can't update to UTF-8 which is the standarad now and also works nicely with ASCII

Helpful Articles:
- [Raymond Chen - windows unicode UTF-16 LE BOM](https://devblogs.microsoft.com/oldnewthing/20190830-00/?p=102823)
- [David Varghese - Character Encoding Explored](https://blog.davidvarghese.net/posts/character-encoding-part-2/)

Helpful Videos:
- [Em Nudge - UTF Explained](https://www.youtube.com/watch?v=uTJoJtNYcaQ)
- [Scott Hanselman - UTF Explained](https://www.youtube.com/watch?v=jeIBNn5Y5fI)

### Timings in PowerShell
Here is the timings script in PowerShell. 
The three text files had about 2000 characters each.
```powershell
Write-Output "i : Milliseconds"
Write-Output "---------------------"
$avg
for ($i = 0; $i -lt 10; $i++) {
    $time = Measure-Command { .\exercise8-1.exe .\data1.txt .\data2.txt .\data3.txt }
    $avg += $time.TotalMilliseconds
    Write-Output "$i : $($time.TotalMilliseconds)"
}
Write-Output ""
Write-Output "---------------------"
Write-Output "average $($avg/10)"
Write-Output ""
```

### Other Solutions - CLC Wiki and C Answer Book
After looking at other solutions, I don't understand why I restructured the program at all, maybe I was tired...

I should have just copied the format of the original and just replaced the standard library calls with system calls...

## Code
```c
#include <stdio.h>
#include <stdlib.h>
#include <fcntl.h>
#include <stdarg.h>
#include <string.h>

void fstdout(const char *s);
void error(char *fmt, ...);

/* cat: concatenate files, version 2 */
int main(int argc, char *argv[])
{
    if (argc == 1) /* no args; copy standard input */
        fstdout("stdin");
    else 
        while (--argc >= 1)
            fstdout(*(++argv));
}

/* fstdout: write stdin or file to stdout */
void fstdout(const char *s)
{
    int fd = 0, n;
    char buf[BUFSIZ];
    
    if (strcmp(s, "stdin") != 0)
        if ((fd = open(s, O_RDONLY)) == -1)
            error("opening %s", s);
    
    while (((n = read(fd, buf, BUFSIZ)) > 0))
        if (write(1, &buf, n) != n) 
            error("writing to %s", s);
    
    if (n == -1) 
        error("reading %s", s);
    
    if (close(fd) == -1) 
        error("closing %s", s);
}

/* error: print an error message and die */
/* from page 174 in K&R */
void error(char *fmt, ...)
{
    va_list args;
    
    va_start(args, fmt);
    fprintf(stderr, "error: ");
    vfprintf(stderr, fmt, args);
    fprintf(stderr, "\n");
    va_end(args);
    exit(1);
}
```

## Output
<p align="center">
  <image src="../assets/exercise8-1_cmds.jpg" alt="output for exercise8-1 cmds" />
</p>

<p align="center">
  <image src="../assets/exercise8-1_result.jpg" alt="output for exercise8-1 result" />
</p>

[Back to Main](../readme.md)
