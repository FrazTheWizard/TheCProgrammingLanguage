# Exercise 7-9

> Function like _isupper_ can be implemented to save space or to save time. Explore both possibilities.

## Process
Spent a few days on this and learnt some interesting things!

I didn't understand what the question wanted me to do. 
So I started by implementing the eleven `is` and two `to` ctype functions to see if any patterns emerged:

```c
#define punctrange  (c >= 33 && c <= 47) || \
                    (c >= 58 && c <= 64) || \
                    (c >= 91 && c <= 96) || \
                    (c >= 123 && c <= 126)

#define spacerange  c == '\n' || \
                    c == '\t' || \
                    c == '\v' || \
                    c == '\f' || \
                    c == '\r' || \
                    c == ' '

#define hexrange    myisdigit(c) || \
                    (c >= 'a' && c <= 'f') || \
                    (c >= 'A' && c <= 'F')

int myiscntrl(int c){return (c >= 0 && c <= 31) || c == 127 ? 1 : 0;}
int myisgraph(int c){return c >= 33 && c <= 126 ? c : 0;}
int myisprint(int c){return c >= 32 && c <= 126 ? c : 0;}
int myispunct(int c){return punctrange ? c : 0;}
int myisupper(int c){return c >= 'A' && c <= 'Z' ? c : 0;}
int myislower(int c){return c >= 'a' && c <= 'z' ? c : 0;}
int myisdigit(int c){return c >= '0' && c <= '9' ? c : 0;}
int myisspace(int c){return spacerange ? c : 0;}
int myisxdigit(int c){return hexrange ? c : 0;}
int myisalpha(int c){return myislower(c) || myisupper(c) ? c : 0;}
int myisalnum(int c){return myisalpha(c) || myisdigit(c) ? c : 0;}
int mytoupper(int c){return myislower(c) ? c - ('a' - 'A') : c;}
int mytolower(int c){return myisupper(c) ?  c + ('a' - 'A') : c;}
```

I was surprised that they were quite simple to implement, and I didn't see another way you could write them to speed them up or use less space. 
So I looked at other solutions to get an idea.

### Other Solutions - CLC Wiki
Some very interesting solutions by `Gregory Pietsch` and `codybartfast` that interpreted the exercise to be about a case of character encoding.

Apparently the C standard doesn't enforce a particular encoding, so they were suggesting to use a lookup table instead of the kind of code I implemented
in my `ctype` functions above. 

Apparently something like this following code will return different results with [EBCDIC](https://en.wikipedia.org/wiki/EBCDIC), which is an encoding I was not aware of up until this point:
```c 
int isupper(int c)
{
  return (c >= 'A' && c <= 'Z');
}
```


So I went down the rabbit hole of encoding, which I think is best summed up in the [C89-90 Standard](https://github.com/sys-research/c-standard-drafts/blob/main/C89%3AC90.pdf):
```
5.2.1 Character sets
Two sets of characters and their associated collating sequences shall be defined: the set in
which source files are written, and the set interpreted in the execution environment. The values
of the members of the execution character set are implementation-defined. any additional
members beyond those required by this subclause are locale-spec
```
Which I understand that to mean there are three separate parts:
1. **Guaranteed:** Source Character Set (the characters you write in your editor)
2. **Guaranteed:** Execution Character Set (what is represented in your compilied program and are sent to a console)
3. **Not Guaranteed:** Execution Character Encoding (how the character is actually encoded as a value eg. 'A' as 65.

Also, it only guarantees the first 127 characters and anything else is implementation dependant.

This is probably my third pass at understanding character encoding now and I think I mostly get it...

### Other Solutions - C Answer Book
Seems they took a different approach to the question and as Brian Kernighan is listed as contributer, I'm going to assume this is the intended interpretation of the question.

The answer in the book looks at the difference between `isupper` as a function and a macro.

- macro: saves time but uses more space
  - no extra setup like in a function
  - the statement is now everywhere
- function: saves space but uses more time
  - function overhead
  - only have to call a function

I guess this idea would be called an [inline function](https://en.wikipedia.org/wiki/Inline_function) today.

## Code
I feel like I cheated because I didn't come to this solution naturally and I'm taking from 'The C Answer Book'. 
But its a very short answer and more of a concept, so I'm including it as 'my' answer.
### Macro version
```c
#define macroisupper(c) (c >= 'A' && c <= 'Z' ? c : 0)

int main()
{
    int x = macroisupper('A');
}
```

### Function version
```c
int functionisupper(int c){
    return c >= 'A' && c <= 'Z' ? c : 0;
}

int main()
{
    int x = functionisupper('A');
}
```

## Output
None. Without digging into the assembly output, I don't feel like any output is necessary here.

[Back to Main](../readme.md)
