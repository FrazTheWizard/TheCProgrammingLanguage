# Exercise 5-7

> Rewrite the routines day_of_year and month_day with pointers instead of indexing.

## Process
Nice to go over transforming indexing into pointers again, slightly different way to think about working on memory and storing things, which I think I'm coming to prefer...

Had a working solution after not too much time. After looking at other [solutions](https://clc-wiki.net/wiki/K%26R2_solutions:Chapter_5:Exercise_9) I was able to make the code a bit more concise.

I liked using `while` loops over `for` loops for these pointer based calculations because it's less code, but maybe a for loop indicates going over consecutive operations more clearly?

Found an interesting way of calculating the month in the `month_day` function which uses pointer difference,
I didn't think to use that:
```c
/* taken from solution by voidboy */
*pmonth = 1 + p - &daytab[leap][1];
```

## Code
```c
#include <stdio.h>

static char daytab[2][13] = {
	{0, 31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31},
	{0, 31, 29, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31}
};

/* day_of_year : set day of year from month & day */
int day_of_year(int year, int month, int day)
{
	int i, leap;	
	
	leap = year%4 == 0 && year%100 != 0 || year%400 == 0;

	char *pdaytab = &daytab[leap][1];

	while (--month)
		day += *pdaytab++;
	
	return day;
}

/* month_day: set month, day from day of year */
void month_day(int year, int yearday, int *pmonth, int *pday)
{
	int leap;
	char *pdaytab;
	*pmonth = 1;
	
	leap = year%4 == 0 && year%100 != 0 || year%400 == 0;
	pdaytab = &daytab[leap][1];
	
	while (yearday >= *pdaytab) {
		yearday -= *pdaytab++;
		*pmonth += 1;
	}
	
	*pday = yearday;
}

int main()
{
	int dayofyear = day_of_year(2024, 5, 3);
	printf("03/05/2024 is the %dth day of year\n", dayofyear);
	
	int *pmonth, *pday, month, day;
	pmonth = &month, pday = &day;
	month_day(2024, 124, pmonth, pday);
	printf("the 124th day of 2024 is month: %d and day: %d", month, day);
	
	return 0;
}
```

## Output
<p align="center">
  <image src="../assets/exercise5-9.jpg" alt="output for exercise 5-9" />
</p>

[Back to Main](../readme.md)
