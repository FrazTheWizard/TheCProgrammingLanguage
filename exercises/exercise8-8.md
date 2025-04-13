# Exercise 8-8

> Write a routine `bfree(p,n)` that will free an arbitrary block `p` of `n` characters into the free list maintained by malloc and free.
> By using bfree, a user can add a static or external array to the free list at any time. 

## Process
Well, this took way less time than expected, was able to do it in a day.

I think the large amount of time I spent playing around trying to understand how the program works, meant I had the knowledge
necessary to implement a solution straightaway.

My Solution:
1. check recieved memory is valid (not `NULL`)
2. check memory can hold at least one `Header` and one free block
3. if malloc things aren't initialised, initalise them  (`freep`, `base`)
4. free the memory to add it to the circularly linked list

I like the idea of this function, taking different pieces of memory and adding them together for a free list to allocate to.

Being the last exercise in the book, I thought it would be more complex, but I'm happy to end on something straightforward!

## Other Solutions - CLC Wiki
Some interesting solutions, some were kinda complex and buggy, and then Anonymous coming in with a great solution again.

Anonymous points out that if memory isn't bigger than two `Header` blocks then isn't not really useful, so shouldn't be allowed.

## Other Solutions - C Answer Book
A nice short solution as always.

I really like their decision to make `bfree` return the size allocated. This allows a way to determine if the free process worked and to
so see how much wasted space there was due to `Header` alignment.

I was playing around with returning a `void *` with my solution so I could return `NULL` if it failed, but didn't feel `ffree` should return a pointer.
Returning the size a a much better solution.

## Code 
```c
#include <stdio.h>
#include <string.h>
#include "malloc.h" /* from previous exercise */

void bfree(void *p, unsigned n)
{
    Header *bp;
    unsigned nunits;
    
    if (n == NULL) {
        printf("bfree error: *p can't be NULL\n");
        return;
    }
    
    if (n < sizeof(Header)) {
        printf("bfree error: n must be > sizeof(Header)\n");
        return;
    }
    
    bp = (Header *) p;
    bp->s.size = n / sizeof(Header);

    if (freep == NULL) {
        freep = &base;
        base.s.ptr = &base;
        base.s.size = 0;        
    }        

    ffree(bp + 1);
}

#define LEN (sizeof(Header) * 2)
static char arr[LEN];

int main()
{
    char *p;
    
    bfree(&arr, LEN);
    p = (char *) mmalloc(LEN / 2);
    strncpy(p, "hello world", 12);
    
    printf("\nString stored:\n%s\n\n", p);
    printf("%u Byte Array:\n", LEN);    
    printf("%32s%32s\n", "End of Header 1 ->|", "End of Header 2 ->|");
    for (int i = 0; i < LEN; i++)
        printf("%c|", arr[i]);
    printf("\n");
}
```

## Output
<p align="center">
  <image src="../assets/exercise8-8_cmds.jpg" alt="output for exercise8-8 cmds" />
</p>

[Back to Main](../readme.md)
