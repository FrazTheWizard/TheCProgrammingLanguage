# Exercise 3-1

> Our binary search makes two tests inside the loop, when one would suffice (at the price of more tests outside).
> Write a version with only one test inside the loop and measure the difference in run-time.


## Process
This took me two days. I didn't quite understand the question. 
It seems to me the loop is still doing the same amount of checks, just in the while check rather than in the loop itself.
I was trying to find some faster way of doing it. So I check online and people have the same answer that I was working with.

**Side Note** I found the powershell command `Measure-Command` which measures execution time of programs.
Also the `Out-Default` allows the Measure-Command to not consume the output
```powershell
Measure-Command{.\exercise3-1.exe | Out-Default}
```


## Final Code

```c
#include <stdio.h>

int binsearch(int x, int v[], int n);
int binsearch2(int x, int v[], int n);

int main(void) {
    int v[] = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
    
    int res = binsearch2(9, v, 9);
    
    printf("%d", res);
    return 0;
}

int binsearch(int x, int v[], int n) 
{
    int low, mid, high;
    low = 0;
    high = n - 1;
    while(low <= high) {
        mid = (low + high) / 2;
        if (x < v[mid])
            high = mid - 1;
        else if (x > v[mid])
            low = mid + 1;
        else /* match found */
            return mid;
    }
    return -1; /* no match */   
}

int binsearch2(int x, int v[], int n) 
{
    int low, mid, high;
    low = 0;
    high = n - 1;
    mid = (low + high) / 2;
    
    while((low <= high) && (v[mid] != x)) {
        if (x < v[mid])
            high = mid - 1;
        else
            low = mid + 1;
        mid = (low + high) / 2;
    }
    
    return v[mid] == x ? mid : -1;
}

```


## Output

<p align="center">
    <image src="../assets/exercise3-1.jpg" alt="output for exercise3-1" />
</p>

[Back to Main](../readme.md)
