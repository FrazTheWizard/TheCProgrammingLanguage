# Exercise 5-8

> There is no error-checking in `day_of_year` or `month_day`. Remedy this defect.

## Process
Pretty straight forward, checking that the input year, month, day etc are within a valid range. Also I researched leap years as I didn't know about the not divided by 100 and but divided by 400 part.

After I wrote my code and checked other [solutions](https://clc-wiki.net/wiki/K%26R2_solutions:Chapter_5:Exercise_8) I rewrote one of my conditional lines:
```c
if (yearday <= 0 || (leap == 0 && (yearday > 365)) || (leap == 1 && (yearday > 366)))
```
to be the much superior line:
```c
if ((yearday < 1) || (leap && yearday > 366) || (!leap && yearday > 365))
```

The year 1752 came up in those solutions, as it was the year the calendar was changed to Gregorian calendar with leap years, so years should be after that. I chose not to check for that as this is just an exercise.

Another thing that could be improved here would be testing the error-checking works. I've just written a couple of test cases, but maybe there would be some way to test if all error checking is working. 
I'm not sure what the best way to do that would be though...

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
	
	/* non valid year */
	if (year <= 0)
		return -1;
	
	/* non valid month */
	if (month <= 0 || month > 12)
			return -1;
	
	leap = year%4 == 0 && year%100 != 0 || year%400 == 0;
	
	/* non valid day */
	if(day <= 0 || day > daytab[leap][month])
		return -1;
	
	for (i = 1; i < month; i++)
		day += daytab[leap][i];
	
	return day;
}

/* month_day: set month, day from day of year */
int month_day(int year, int yearday, int *pmonth, int *pday)
{
	int i, leap;
	
	/* non valid year */
	if (year <= 0)
		return -1;
	
	leap = year%4 == 0 && year%100 != 0 || year%400 == 0;
	
	/* non valid yearday */
	if ((yearday < 1) || (leap && yearday > 366) || (!leap && yearday > 365))
		return -1;
	
	for(i = 1; yearday > daytab[leap][i]; i++)
		yearday -= daytab[leap][i];
	
	*pmonth = i;
	*pday = yearday;
	
	return 0;
}


int main()
{
	int dayofyear;
	dayofyear = day_of_year(2024, 12, 32);
	if(dayofyear != -1)
		printf("Today is number %d day of the year\n", dayofyear);
	else
		printf("Non valid input for 'day_of_year'\n");
	
	dayofyear = day_of_year(2024, 4, 5);
	if(dayofyear != -1)
		printf("Today is number %d day of the year\n", dayofyear);
	else
		printf("Non valid input for 'day_of_year'\n");
	
	int month, day, *pmonth, *pday;
	pmonth = &month;
	pday = &day;
	
	if(month_day(2020, 368, pmonth, pday) != -1)
		printf("Today is day number %d of month number %d\n", day, month);
	else
		printf("Non valid input for 'month_day'\n");
	
	if(month_day(2020, 12, pmonth, pday) != -1)
		printf("Today is day number %d of month number %d\n", day, month);
	else
		printf("Non valid input for 'month_day'\n");
		
	return 0;
}
```

## Output
<p align="center">
  <image src="../assets/exercise5-8.jpg" alt="output for exercise5-8" />
</p>

[Back to Main](../readme.md)
