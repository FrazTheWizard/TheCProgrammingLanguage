# Exercise 5-11

> Modify the programs entab and detab (written as exercises in Chapter 1) to accept a list of tab stops as arguments. Use the default tab settings if there are no arguments. 

## Process
This one was a little challenging and took me about a week of 1 hour morning sessions.

### Research
I think part of the problem was I don't really use tab stops much, so I had to play around with them to understand them more.

While trying to get variable tabstops working in Notepad++ and not finding a way, I discovered the whole thing of
[Elastic Tabstops](https://nick-gravgaard.com/elastic-tabstops/), of which there is a plugin for notepad++. 
It makes sense and seems like a good idea, but after trying it, I didn't find it very intuitive so left it. Good to know about though.

Played around with variable tabstops in WordPad, which was ok, but I needed to see nonprintable characters to understand what was going on.
Then I used Google Docs, which has variable tabstops and can show non printing characters, but I didn't want to upload text files to see what results I was getting.

What I was really looking for was a way to visualise the variable tab stops as I was working this exercise. 
So I just ended up using Notepad++, showing the non printable characters and visualising in the tabstops in my head.

### Decisions
1. I chose to make the code straightforward processwise for my brain instead of trying to find the most condense and efficent way of writing / executing it. 
There would probably be a way to calculate the spaces - tabs needed using a nice one line math formula, but I'm ok with just a longer, but working solution. 

2. I also chose to ignore input error checking as well, to keep it just about the exercise topic.

3. I also played around with where to put the dividing line between abstractions. I've been listening to Casey Muratori's talks about [designing apis](https://www.youtube.com/watch?v=ZQ5_u8Lgvyk) 
and [splitting code](https://www.youtube.com/watch?v=5IUj1EZwpJY).
I chose to divide between
- MAIN - handle input, count spaces and call tabstop function
- FUNCTION - printing default tabstops
- FUNCTION - printing variable tabstops
- FUNCTION - getline

I left the decision of which tabstop function to call in main to give slightly more control to main, without it knowing the implementation

Potentially the variable or default tabstop functions might be changed in future exercises, so having them ready as separated tabstop logic is helpful for modification

## Code - entab
```c
#include <stdio.h>
#include <stdlib.h> /* for atoi() */
#define MAXLINE 1000
#define DEFAULT_TABSTOP 4

int getline(char s[], int lim);
void tabstops_variable(int start, int end, int tsCount, char *tsValues[]);
void tabstops_default(int start, int end);

int main(int argc, char *argv[])
{
    char s[MAXLINE];
    int len;
    int start = -1; 
    
    while((len = getline(s, MAXLINE)) > 0) {
        for (int i = 0; i < len; i++) {
            
            /* was not counting blanks */
            if ((s[i] != ' ') && (start == -1))
                putchar(s[i]);

            /* begin blank sequence */
            else if ((s[i] == ' ') && (start == -1))
                start = i;

            /* end of blank sequence */
            else if ((s[i] != ' ') && (start != -1)) {
                if (argc == 1)
                    tabstops_default(start, i);
                else
                    tabstops_variable(start, i, argc-1, argv+1);
                putchar(s[i]);
                start = -1; /* reset sequence of blanks */
            }
        }
    }   
    
    return 0;
}

void tabstops_default(int start, int end) {
    int tsTarget = 0;
    
    /* find target tabstop */
    do {
        tsTarget += DEFAULT_TABSTOP;
    } while (start >= tsTarget);
    
    /* print tabs, if any */
    while (end >= tsTarget) {
        putchar('\t');
        start = tsTarget;
        tsTarget += DEFAULT_TABSTOP;
    }
    
    /* print remaining blanks, if any */
    while (++start <= end)
        putchar(' ');
    
    return;
}

/* convert blank sequence into tabstops */
void tabstops_variable(int start, int end, int tsCount, char *tsValues[]) {
    int i = 0, tsTarget = 0;

    /* find target tabstop */
    do {
        tsTarget += atoi(tsValues[i]);
    } while ((start >= tsTarget) && (++i < tsCount));
    
    /* no target tabstop found or blank sequence too short */
    if (i == tsCount || end < tsTarget) {
        while(start++ < end) putchar(' ');
        return;
    }

    /* reached last tabstop */
    if ((i + 1) == tsCount) {
        putchar('\t');
        while(tsTarget++ < end) putchar(' ');
        return;
    }
    
    /* call recursive, start from target tabstop */
    putchar('\t');
    tabstops_variable(tsTarget, end, tsCount, tsValues);

    return;
}

/* read a line into s, return length */
int getline(char s[], int lim) {
    int c, i;

    for (i=0; i<lim-1 && (c=getchar())!=EOF && c!='\n'; i++)
        s[i] = c;
    
    if (c == '\n') {
        s[i] = c;
        ++i;
    }

    s[i] = '\0';
    return i;
}
```

## Code - detab
```c
#include <stdio.h>
#include <stdlib.h> /* for atoi() */
#define MAXLINE 1000
#define DEFAULT_TABSTOP 4

int getline(char s[], int lim);
int tabstops_variable(int tabpos, int tsCount, char *tsValues[]);
int tabstops_default(int tabpos);

int main(int argc, char *argv[])
{
    char s[MAXLINE];
    int len, blanks = 0, offset = 0;
    
    while((len = getline(s, MAXLINE)) > 0) {
        for (int i = 0; i < len; i++) {
            switch(s[i]) {
                case '\t':  
                    if (argc == 1)
                        blanks = tabstops_default(i + offset);
                    else 
                        blanks = tabstops_variable(i + offset, argc-1, argv+1);
                    offset += (blanks-1);
                    while (blanks--) putchar(' ');
                    break;
                    
                case '\n':
                    offset = 0;
                    putchar(s[i]);
                    break;

                default:
                    putchar(s[i]);
                    break;
            }
        }
    }
    return 0;
}

int tabstops_default(int tabpos) {
    int tsTarget = 0;
    
    /* find target tabstop */
    do {
        tsTarget += DEFAULT_TABSTOP;
    } while (tabpos >= tsTarget);
    
    return tsTarget - tabpos;
}

int tabstops_variable(int tabpos, int tsCount, char *tsValues[]) {
    int i = 0, tsTarget = 0;
    
    /* find target tabstop */
    do {
        tsTarget += atoi(tsValues[i]);
    } while ((tabpos >= tsTarget) && (++i < tsCount));
    
    return tsTarget - tabpos;
}

/* read a line into s, return length */
int getline(char s[], int lim) {
    int c, i;
    
    for (i=0; i<lim-1 && (c=getchar())!=EOF && c!='\n'; i++)
        s[i] = c;
    
    if (c == '\n') {
        s[i] = c;
        ++i;
    }
    
    s[i] = '\0';
    return i;
}
```

## Output
<p align="center">
  <image src="../assets/exercise5-11_a.jpg" alt="output for exercise 5-11_a" />
</p>
<p align="center">
  <image src="../assets/exercise5-11_b.jpg" alt="output for exercise 5-11_b" />
</p>

[Back to Main](../readme.md)
