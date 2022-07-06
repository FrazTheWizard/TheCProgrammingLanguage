# Exercise 2-3
> Write the function `htoi(s)`, which converts a string of hexadecimal digits
    (including an optional `0x` or `0X`) ito its equivalent integer value. 
    The allowable digits are `0` through `9`, `a` through `f`, and `A` through `F`.
  
## Process
This took me a session. Figuring out how this one worked was kind of cool and maybe slightly annoying...

I just felt an intuitive answer and was able to write the code, without consciously knowing where I was going with it yet.
Thats pretty cool, but hard to explain how I arrived at the solution or replicate the process later on. 
I suppose it happened because I've been working on these exercises everyday and could just feel the answer.

I added length checking to the function because it was continuing past the string, as it only terminates on non valid characters

---
I also stumbled accross two **strange properties of C**:
1. You can add more characters to an array than the limit that you set

[Apparently](https://stackoverflow.com/questions/51841051/why-can-we-insert-more-elements-in-an-array-than-what-it-can-hold) its called 'undefined behavior' and C allows for it due to the old days when people understood the hardware 


2. The first time running C program is slightly slower then the other times

[Apparently](https://stackoverflow.com/questions/13365914/why-the-first-time-c-program-runs-it-runs-10x-slower) this is because it needs to read from disk the first time, then it is cached



## Final Code
```c
#include <stdio.h>

int htoi(char s[], int len);
    
int main(void) {
    char s1[2];
    s1[0] = 'f';
    s1[1] = 'a';
    
    char s2[4];
    s2[0] = '0';
    s2[1] = 'x';
    s2[2] = '5';
    s2[3] = 'b';
        
    printf("\'fa\': %d\n", htoi(s1, 2));
    printf("\'0x5b\': %d", htoi(s2, 4));
    return 0;
}

int htoi(char s[], int len){
    int i, n;
    n = 0;
    
    /* ignore initial 0x or 0X */
    if(s[0] == '0' && (s[1] =='x' || s[1]=='X'))
        i = 2;
    else
        i = 0;
    
    /* ignore characters outside legal hex values */
    for(i; 
        (s[i] >= '0' && s[i] <= '9') || 
        (s[i] >= 'A' && s[i] <= 'F') ||
        (s[i] >= 'a' && s[i] <= 'f') &&
        i < len;
        i++
        )
    {
        /* add value to total integer */
        if(s[i] >= '0' && s[i] <= '9')
            n = 16 * n + (s[i] - '0');   
        
        if(s[i] >= 'A' && s[i] <= 'F')
            n = 16 * n + (s[i] - 'A') + 10; /* add 10 because A is 10 */
        
        if(s[i] >= 'a' && s[i] <= 'f')
            n = 16 * n + (s[i] - 'a') + 10; /* add 10 because a is 10 */
    }
    return n;
}
```
## Output
<p align="center">
    <image src="../assets/exercise2-3_output.jpg" alt="output for exercise2-3" />
</p>


[Back to Main](../readme.md)
