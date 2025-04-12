# Exercise 8-7

> `malloc` accepts a size request without checking its plausability; `free` belives that the block it is asked to free contains a valid `size` field.
> Improve these routines so they take more pains with error checking. 

## Process
I found this exercise hard to grasp so I ended up following the excellent explanation from a CLC Wiki solution which I've basically copied here.

Also, I wanted to try testing tools for this exercise, but _AddressSanitizer_, _Valgrind_, and the fuzzers I found don’t work on Windows.

### Confusion

I didn't quite understand what `plausability` and `valid` meant in this context. 

I think part of what threw me off was how the program relies on the user to return correct addresses instead of tracking allocated blocks. 
I'd have used a separate linked list of allocated blocks, but maybe it's better to avoid unnecessary references and logic, I guess I'm not used to thinking of a program in this way

Initially, I thought the exercise involved fixing memory corruption of the allocated blocks, but that seems out of scope because there would be many other problems to take care of.

I knew that the exercise wasn't about fixing an invalid value passed to `unsigned nbytes` and `unsigned size`, because they would just overflow:

```
C89/C90 Specification
6.1.2.5 - Types
Computation involving unsigned operands can never overflow because a result 
that cannot be represented by the resulting unsigned integer type is reduced 
modulo the number that is one greater than the largest value
```

## Other Solutions - [CLC-Wiki](https://clc-wiki.net/wiki/K%26R2_solutions:Chapter_8:Exercise_7)
The anonymous answer here was amazing. I've basically just copied their answer.

I now realise the exercise is about handling incorrect user input to the functions and what happens at the variable size limits.

The main issues, as I understand, are:
- `malloc(0)` creates a Header only block of size 1 and returns a pointer to invalid memory
- `malloc(UINT_MAX)` creates a memory allocation too big to also include the Header
- `free(NULL)` causes segmentation fault when trying to dereference allocation
- `free(*ptr)` where `*ptr` has size 0, it has to be an incorrect allocation, due to first issue
- `free(*ptr)` trying to join two large blocks, could create a block with size so big it overflows the `unsigned size` field

So the fixes would be:
- `malloc`: check `nbytes` isn't 0 
- `malloc`: check max allocation is `< UINT_MAX - sizeof(Header)`
- `free`: check `*ptr` isn't `NULL`
- `free`: check `*ptr` size isn't 0
- `free`: check that merged blocks size won't become `> UINT_MAX` before merging

## Other Solutions - C Answer Book
It suggests two things:
- create an _'arbitrary constant MAXBYTES'_ _'...that works best for your system.'_
- create a _'static variable maxalloc that remembers the size of the largest block used so far'_ to check when freeing

Doesn't explain how to determine MAXBYTES, not sure I like this as a solution.

Also, I feel like there could be a more robust solution than just remembering the largest block. 

Personally I feel like keeping a list of allocated blocks, so you can return it to freelist would be slower, but less errors?

## Code - changes to `malloc`
```c
/* check nbytes isn't 0 */
if (nbytes == 0) {
  printf("Failed to allocate 0 bytes");
  return NULL;
}

/* check max allocation is < UINT_MAX - sizeof(Header) */
if (nbytes > (UINT_MAX - sizeof(Header))) {
  printf("Failed to allocate UINT_MAX - sizeof(Header) bytes");
  return NULL;
}
```

## Code - changes to `free`
```c
/* check ptr isn't NULL */
if (ap == NULL) {
  printf("Failed to free NULL");
  return;
}

/* check ptr size isn't 0 */
if (bp->s.size == 0) {
  printf("Failed to free allocated block of size 0");
  return;

/* check that merged blocks size won't become > UINT_MAX before merging */
if ((bp + bp->s.size == p->s.ptr) && ((bp->s.size + p->s.ptr->s.size) < UINT_MAX)) {
  /* upper join code */
}
if ((p + p->s.size == bp) && ((bp->s.size + p->s.size) < UINT_MAX)) {
  /* lower join code */
}
```

## Code - Full
```c
#include <stdio.h>
#include <string.h>
#include <limits.h>
#include "win32_stuff.h" /* from previous exercise */

typedef long Align;

union header {
    struct {
        union header *ptr;
        unsigned size;
    } s;
    Align x;
};

typedef union header Header;

Header base;
Header *freep = NULL;

void *mmalloc(unsigned nbytes)
{
    Header *p, *prevp;
    void *allocatemorespace(unsigned);
    unsigned nunits;
    
    if (nbytes == 0) {
        printf("Cannot allocate 0 bytes\n");
        return 0;
    }
    
    if (nbytes > (UINT_MAX - sizeof(Header))) {
      printf("Failed to allocate UINT_MAX - sizeof(Header) bytes\n");
      return NULL;
    }
    
    nunits = (nbytes + sizeof(Header) - 1) / sizeof(Header) + 1;
    if ((prevp = freep) == NULL) {
        prevp = freep = &base;
        base.s.ptr = &base;
        base.s.size = 0;
    }
    
    for (p = prevp->s.ptr; ; prevp = p, p = p->s.ptr) {
        if (p->s.size >= nunits) {
            if (p->s.size == nunits) {
                prevp->s.ptr = p->s.ptr;  
            } else {
                p->s.size -= nunits;
                p += p->s.size;
                p->s.size = nunits;
            }
            freep = prevp;
            return (void *)(p + 1);
        }        
        
        if (p == freep) {
            if ((p = allocatemorespace(nunits)) == NULL)
                return NULL;
        }
    }
}

void *allocatemorespace(unsigned nunits)
{
    void *cp, *commitMemory(unsigned);
    Header *up;
    void ffree(void *ap);
    static unsigned pageSize = 0;
    unsigned getPageSize(void);
    
    if (pageSize == 0) {
        pageSize = getPageSize();
    }
    
    if (nunits < (pageSize / sizeof(Header))) {
        nunits = pageSize / sizeof(Header);
    }
    
    cp = commitMemory(nunits * sizeof(Header));
    if (cp == NULL)
        return NULL;
    up = (Header *) cp;
    up->s.size = nunits;
    ffree(up + 1);
    return freep;
}

void ffree(void *ap)
{
    Header *bp, *p;
    
    bp = (Header *) ap - 1;
    
    if (ap == NULL) {
        printf("Failed to free memory, at NULL\n");
        return;
    }
    
    if (bp->s.size == 0) {
        printf("Failed to free memory, block size 0\n");
        return;
    }
    
    for (p = freep; !(bp > p && bp < p->s.ptr); p = p->s.ptr)
        if (p >= p->s.ptr && (bp > p || bp < p->s.ptr))
            break;
    
    if ((bp + bp->s.size == p->s.ptr) && ((bp->s.size + p->s.ptr->s.size) < UINT_MAX)) {
        bp->s.size += p->s.ptr->s.size;
        bp->s.ptr = p->s.ptr->s.ptr;
    } else {
        if ((bp->s.size + p->s.ptr->s.size) < UINT_MAX)
            printf("Free blocks too large to merge upper\n");
        bp->s.ptr = p->s.ptr;
    }
    
    if ((p + p->s.size == bp) && ((bp->s.size + p->s.size) < UINT_MAX)) {
        p->s.size += bp->s.size;
        p->s.ptr = bp->s.ptr;
    } else
        if ((bp->s.size + p->s.size) < UINT_MAX)
            printf("Free blocks too large to merge lower\n");
        p->s.ptr = bp;
    freep = p;
}

int main()
{
    printf("Test 1\n");
    mmalloc(0);
    printf("\n");
    
    printf("Test 2\n");
    mmalloc(UINT_MAX);
    printf("\n");
 
    printf("Test 3\n");
    ffree(NULL);
    printf("\n");
    
    printf("Test 4\n");
    Header h = {.s = {.ptr=NULL, .size=0}};
    ffree(&h+1);
    printf("\n");
    
    
    printf("Test 6\n");
    Header h1;
    Header h2;
    
    base.s.ptr = &h2;
    base.s.size = 0;
    h2.s.ptr = &base;
    h2.s.size = UINT_MAX / 2;
    freep = &base;
    
    ffree(&h1+1);
}
```

## Output
<p align="center">
  <image src="../assets/exercise8-7_cmds.jpg" alt="output for exercise8-7 cmds" />
</p>

[Back to Main](../readme.md)
