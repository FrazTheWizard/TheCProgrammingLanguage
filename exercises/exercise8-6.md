# Exercise 8-6

> The standard library function calloc(n,size) returns a pointer to n objects of size size, with the storage initialized to zero. Write calloc, by calling malloc or by modifying it. 

## Process
This took me about three weeks of research and two minutes of writing the solution to the exercise.

Just like last exercise, I spent a lot of time on windows specifics, and called win32 functions to build up `morecore`.

I decided to use the `memset` for the solution, but a for loop is probably the proper answer for this exercise.

## Research 1 - `sbrk`
Had a look to see how `sbrk` is supposed to work to get the general idea. Watched [video](https://www.youtube.com/watch?v=NDKArv9WAlQ) and [video](https://www.youtube.com/watch?v=vEXRpiI4Dhk).

## Research 2 - Windows Memory
MSDN had an [example](https://learn.microsoft.com/en-us/windows/win32/Memory/reserving-and-committing-memory) on how to allocate memory which is referred to as "Reserving" and "Committing" memory.

I looked at the functions I thought I would need for equivalant functionality to `sbrk` in windows:
- [VirtualAlloc](https://learn.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-virtualalloc) - to allocate memory
- [VirutalFree](https://learn.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-virtualfree) - to free allocated memory (didn't use as it's not in example)
- [getSystemInfo](https://learn.microsoft.com/en-us/windows/win32/api/sysinfoapi/nf-sysinfoapi-getsysteminfo) - information for allocating memory
- [SYSTEM_INFO struct](https://learn.microsoft.com/en-us/windows/win32/api/sysinfoapi/ns-sysinfoapi-system_info) - the struct returned by `getSystemInfo()`

## Research 3 - Virtual Memory
Had a look more into how the virtual memory works on Windows.
- Random [article](https://arvid.io/2018/04/02/memory-mapping-on-windows/) on windows memory mapping.
- Old 2011 [video](https://www.youtube.com/watch?v=X1qwl9ANpxM) explaining VirtualAlloc usage using Windows Performance Analyzer
- [Video](https://www.youtube.com/watch?v=MPnsPlDJbhI) tracing VirtualAlloc with xdbg with a focus on Malware analysis focus
- An old 2008 informative offical [blog](https://learn.microsoft.com/en-us/archive/blogs/markrussinovich/pushing-the-limits-of-windows-virtual-memory) by Mark Russinovich (Win Sys Internals). The first blog wasn't linked properly on MSDN, but I found it [published](http://m.blog.chinaunix.net/uid-10073362-id-248411.html) somewhere else
- stackoverlow [answer](https://stackoverflow.com/questions/872072/whats-the-differences-between-virtualalloc-and-heapalloc) on difference between `VirtualAlloc`, `HeapAlloc`, `malloc`, and `new`.

Some relevant Windows quirks explained by Raymond Chen "The Old New Thing" articles:
- [article](https://devblogs.microsoft.com/oldnewthing/20151008-00/?p=91411) about commit vs reserve
- [article](https://devblogs.microsoft.com/oldnewthing/20141003-00/?p=43923) why 4000000 is the first memory address for an exe file
- [article](https://devblogs.microsoft.com/oldnewthing/20031008-00/?p=42223) why allocation granularity is 64KB
- [article](https://devblogs.microsoft.com/oldnewthing/20090611-00/?p=17933) why microsoft used KB instead of KiB
- [article](https://devblogs.microsoft.com/oldnewthing/20171227-00/?p=97656) why use PAGE_NOACCESS when reserving memory

## Research 4 - General Memory Allocation
Memory allocation is kind of a new topic for me, so I wanted to know about it a bit more before I try to rewrite malloc with different windows functions.

As far as I could tell, the book shows a _"tail end circular linked list memory allocator"_, so I looked up those keywords to find more information

- Series of [articles](https://www.gingerbill.org/article/2021/11/30/memory-allocation-strategies-005/) by GingerBill (the creator of Odin Programming Language) about memory allocation strategies

- [Article](https://www.rfleury.com/p/untangling-lifetimes-the-arena-allocator) by Ryan Fleury about arena style memory allocation in C
> This view follows quite naturally from another aspect of modern programming 
thinking (and education), which claims many problems are gross and complex, and 
thus we need abstraction to make them appear simpler. It’s not, advocates of 
this philosophy claim, that the problems themselves should be simplified—they 
are, on the other hand, intrinsically complex, and it is the job of tooling to 
make them appear less complex.

- from stackoverflow [answer](https://stackoverflow.com/questions/12825148/what-is-the-meaning-of-the-term-arena-in-relation-to-memory)
> An arena is just a large, contiguous piece of memory that you allocate once and then use to manage memory manually by handing out parts of that memory 
- blog [post](https://gist.github.com/attractivechaos/862fb6e58147b47c9d16bf2d9e12445f) on why it's called an "arena".
- nullprogram [post](https://nullprogram.com/blog/2023/09/27/) on Arena Allocator tips and tricks

## Research 5 - Specific Code
Pulled apart this line to understand it:
```c
nunits = (nbytes + sizeof(Header) - 1) / sizeof(Header) + 1;
```
- The `-1` is to round up the calculation because integer division truncates
- The `+1` is to add space for the header itself

I found the whole Header process kind of confusing, so I ended up writing out the code by hand using pen and paper, to get a better understanding
of how it works. That helped alot.

## Research 6 - Unrelated but interesting
Interface and API design:
- old 2006 [article](http://www.catb.org/~esr/writings/luxury-part-deux.html) on bad complex interface and api design that seems to have caused a stir
- [article](https://www.joelonsoftware.com/2001/10/24/user-interface-design-for-programmers/) by Joel Spolsky on interface design for programmers
- John Carmark on [inlined code](http://number-none.com/blow/blog/programming/2014/09/26/carmack-on-inlined-code.html) on Jonathon Blows blog
- meta parsing [language](https://metadesk.handmade.network/)
- [outliner](https://indigrid.handmade.network/) for organising thoughts

## Other Solutions - CLC-Wiki
Nothing to say

## Other Solutions - C Answer Book
Again such a compact way of writing the solution

## Code 1 - `win23_stuff.h`
```c
#include <stddef.h> // NULL, size_t

#define MEM_COMMIT  0x1000
#define MEM_RESERVE 0x2000
#define MEM_RELEASE 0x8000

#define PAGE_NOACCESS 0x01
#define PAGE_READWRITE 0x04

typedef unsigned long DWORD;
typedef unsigned short WORD;
typedef void* LPVOID;
typedef unsigned long long DWORD_PTR;

struct SYSTEM_INFO {
    union {
        DWORD dwOemId;
        struct {
            WORD wProcessorArchitecture;
            WORD wReserved;
        } dum_st;
    } dum_un;

    DWORD     dwPageSize;
    LPVOID    lpMinimumApplicationAddress;
    LPVOID    lpMaximumApplicationAddress;
    DWORD_PTR dwActiveProcessorMask;
    DWORD     dwNumberOfProcessors;
    DWORD     dwProcessorType;
    DWORD     dwAllocationGranularity;
    WORD      wProcessorLevel;
    WORD      wProcessorRevision;
};

struct SYSTEM_INFO sysinfo;
unsigned isSysInfoRetrieved = 0;
void *mem_reserve_ptr = NULL;
void *mem_ptr = NULL;

void GetSystemInfo(struct SYSTEM_INFO *s);
void* VirtualAlloc( LPVOID lpAddress, size_t dwSize, 
                    DWORD flAllocationType, DWORD flProtect);

/* retrieveSystemInfo: gets singleton system info */
void retrieveSystemInfo(void)
{
    if (isSysInfoRetrieved == 0) {
        GetSystemInfo(&sysinfo);
        isSysInfoRetrieved = 1;
    }
}

/* getPageSize: returns system page size */
unsigned getPageSize(void)
{
    retrieveSystemInfo();
    return sysinfo.dwPageSize;
}

/* reserveMemory: reserves singleton contiguous memory block */
void *reserveMemory(void)
{
    void *m;
    
    retrieveSystemInfo();
    
    if (mem_reserve_ptr != NULL) /* set single reserve block limit */
        return mem_reserve_ptr;
    
    m = VirtualAlloc(NULL, sysinfo.dwAllocationGranularity, MEM_RESERVE, PAGE_NOACCESS);
    if (m == NULL) {
        return NULL; /* reserve memory failed */
    }
    
    return mem_reserve_ptr = mem_ptr = m;
}

/* commitMemory: commits pageSize memory blocks up to single reserve block */
void *commitMemory(unsigned bytes)
{
    void *m, *requested, *max;
    
    if ((mem_reserve_ptr = reserveMemory()) == NULL) {
        return NULL; /* reserve memory failed */
    }
    
    requested = (mem_ptr + bytes);
    max = mem_reserve_ptr + sysinfo.dwAllocationGranularity;
    
    if (requested > max) {
        return NULL;
    }
    
    m = VirtualAlloc(mem_ptr, bytes, MEM_COMMIT, PAGE_READWRITE);
    if (m != NULL) {
        mem_ptr += bytes;
    }
    
    return m;
}
```

## Code 2 - `malloc.h`
```c
#include "win32_stuff.h"

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
    
    nunits = (nbytes + sizeof(Header) - 1) / sizeof(Header) + 1;
    if (freep == NULL) {
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
    
    for (p = freep; !(bp > p && bp < p->s.ptr); p = p->s.ptr)
        if (p >= p->s.ptr && (bp > p || bp < p->s.ptr))
            break;
    
    if (bp + bp->s.size == p->s.ptr) {
        bp->s.size += p->s.ptr->s.size;
        bp->s.ptr = p->s.ptr->s.ptr;
    } else
        bp->s.ptr = p->s.ptr;
    
    if (p + p->s.size == bp) {
        p->s.size += bp->s.size;
        p->s.ptr = bp->s.ptr;
    } else
        p->s.ptr = bp;
    freep = p;
}
```

## Code 3 - `exercise8-6.c`
```c
#include <stdio.h>
#include <string.h>
#include "malloc.h"
#define COUNT 5

void *ccalloc(size_t nmemb, size_t size);
void debug_memory(void *mem, size_t size);

struct node {
    int vala;
    int valb;
};

int main() {
    struct node *nodes;

    nodes = (struct node *) ccalloc(sizeof(struct node), COUNT);
    
    if (nodes == NULL) {
        printf("memory allocation failed\n", sizeof(struct node) * COUNT);
        return 1;
    }
    
    nodes[0].vala = 11;
    nodes[0].valb = 12;
    nodes[1].vala = 21;
    nodes[1].valb = 22;
    nodes[2].vala = 31;
    nodes[2].valb = 32;
    nodes[3].vala = 41;
    nodes[3].valb = 42;
    nodes[4].vala = 51;
    nodes[4].valb = 52;
    
    printf("write the structs:\n");
    for (int i = 0; i < COUNT; i++) {
        printf("node %d: val_a: %d, val_b: %d\n", i, nodes[i].vala, nodes[i].valb);
    }
    printf("\n");
    
    
    printf("memory after writing structs:\n");
    debug_memory((void *) nodes, sizeof(struct node) * COUNT);
}

void *ccalloc(size_t nmemb, size_t size)
{
    void *mem;
    size_t n;
    
    n = nmemb * size;
    mem = mmalloc(n);
    
    if (mem == NULL) {
        return NULL;        
    }
    
    printf("memory initially:\n");
    debug_memory(mem, n);
    printf("\n");
    
    mem = memset(mem, 0, n);
    
    printf("memory after memset :\n");
    debug_memory(mem, n);
    printf("\n");
    
    return mem;
}

void debug_memory(void *mem, size_t size)
{
    int i;
    char *c;
    
    for (i = 0; i < size; i++) {
        c = ((char *) mem) + i;
        printf("%c", *c == '\0' ? '0' : *c);
    }
    printf("\n");
}
```

## Output
<p align="center">
  <image src="../assets/exercise8-6_cmds.jpg" alt="output for exercise8-6 cmds" />
</p>

[Back to Main](../readme.md)
