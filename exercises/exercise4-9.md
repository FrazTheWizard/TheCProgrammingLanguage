# Exercise 4-9

> Our getch and ungetch do not handle a pushed-back EOF correctly. Decide what their properties ought to be if an EOF is pushed back, then implement your design.

## Process
This one sent me on a rabbit hole!

TLDR - the buffer holding input characters `char buf[BUFSIZE];` should be an `int`.

I realised a bug to terminate the program would be send the character that was equal to the value of EOF (-1). But I didn't find it easy to do.

As I understand now, a lot of the "why?" confusion came from character encodings and displaying characters through different tools.
Values would dispay differently on every tool I was using:
- Windows 10 cmd
- gdb (on Windows 10 cmd)
- notepad++

### Things I found helpful in my understanding
1. character values 0 - 127 (*ascii range*) display symbols always mean the same thing
2. character values 127 - 255 (*extended ascii range*) display symbols depending on the active codepage in Windows cmd 
3. `chcp` shows the active code page on Windows cmd (default is 850)
4. `chcp 720` changes codepage to [code page 720](https://en.wikipedia.org/wiki/Code_page_720) or [Windows-1256](https://en.wikipedia.org/wiki/Windows-1256)
5. As this great [article](https://www.joelonsoftware.com/2003/10/08/the-absolute-minimum-every-software-developer-absolutely-positively-must-know-about-unicode-and-character-sets-no-excuses/) explains,
there is a difference between ascii, extended ascii encodings and unicode encodings (Sidenote - guy cofounded stack overflow).
6. As this [video](https://www.youtube.com/watch?v=D7OvYJRb5IQ) explains, `EOF` is not actually a value stored anywhere in a file and it is not any value coming from an input. It is just a C macro that is returned by `getchar` when it finds the end of input

### So char should be int?
Basically `getchar` returns an `int` so you can capture `EOF`

Errors if you store the result in a char are as this [site](https://c-faq.com/stdio/getcharc.html) and this [site](https://en.wikibooks.org/wiki/C_Programming/stdio.h/getchar#Common_mistake) explains
> 1. If type char is signed, and if EOF is defined (as is usual) as -1, the character with the decimal value 255 ('\377' or '\xff' in C) will be sign-extended and will compare equal to EOF, prematurely terminating the input.
> 2. If type char is unsigned, an actual EOF value will be truncated (by having its higher-order bits discarded, probably resulting in 255 or 0xff) and will not be recognized as EOF, resulting in effectively infinite input.

Also as explained in previous section 1.5.1 of the book:
> The problem is distinguishing the end of the input from valid data. The solution is that *getchar*
> returns a distinctive value when there is no more input a value that cannot be confused with any real character.
> This value is called **EOF**, for "end of file". We must declare c to be a type big enough to hold any value that *getchar* returns.
> We can't use **char** since c must be big enough to hold **EOF** in addition to any possible **char**. Therefore we use **int**.



## Code
```c
#define BUFSIZE 100

int buf[BUFSIZE];  /* buffer for ungetch */
int bufp = 0;       /* next free position in buf */

int getch(void) /* get a (possibly pushed back) character */
{
    return (bufp > 0) ? buf[--bufp] : getchar();
}

void ungetch(int c) /* push character back on input */
{
    if (bufp >= BUFSIZE)
        printf("ungetch: too many characters\n");
    else
        buf[bufp++] = c;
}
```

## Output
<p align="center">
    No Output
</p>

[Back to Main](../readme.md)
