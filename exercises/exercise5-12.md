# Exercise 5-12

> Extend entab and detab to accept the shorthand `entab -m +n` to mean tab stops every n columns, starting at column m. Choose convenient (for the user) default behavior. 

## Process
I found this as challenging as the first part of the exercise and took about a week of 1 hour morning sessions

These programs have enough overlapping concepts to be rewritten to be more condense and have some shared functions between them, 
but I went with a **pragmatic approach** to this extension by adding in new functions without rewriting the whole thing. 
This is an approach I have been doing more recently in my software dev job, gets things done faster, but leaves more tech debt

A main focus was not trying to be too smart with logic, names etc. Just keep everything obvious, linear and obvious function names.
I've also added comments for my intention. Perhaps I could have pushed it further in this direction, more self documenting code etc, but happy with the result

With the behavior of the program, I've opted for a 'compiler' like behavior. Either it works or it doesn't. 
The program only uses the first `-m` and `+n` if they are valid, otherwise doesn't try to use them.
Following that it will try the variable tabstops or finally default tabstops. Again, not trying to be smart

I found the input part challenging, so separated it out into a separate function

As with the previous exercise, I have not put in any real error checking, seemed out of scope of the exercise and I'm sure will learn more about later in the book

I also learnt about [Sentinel Values](https://en.wikipedia.org/wiki/Sentinel_value) vs [Flags](https://en.wikipedia.org/wiki/Flag_(programming)) for distinguising an unset variable from a set variable.
I went with sentinel values eg. `int m = -1` and `int n = -1`, because it's one less variable to have in your mind and it seems related and encapsulated to what you are using the variable for

### Decisions
- merge tabstop functions - no
- extract shared logic - no
- error checking - not really
- handles negative number input - no
- extract input parsing into function - yes
- only take first valid `-m` and `+n`
- if `-m` or `+n` are invalid, don't use either

## Code - entab
```c
#include <stdio.h>
#include <ctype.h> /* isdigit */
#include <stdlib.h> /* atoi */
#define MAXLINE 1000
#define DEFAULT_TABSTOP 4

int getline(char s[], int lim);
int processinput(int *m, int *n, char *argv[]);
int isdigits(char *s);

void tabstops_variable(int start, int end, int tsCount, char *tsValues[]);
void tabstops_default(int start, int end);
void tabstops_fixed(int start, int end, int m, int n);

int main(int argc, char *argv[])
{
    char s[MAXLINE];
    int len, start=-1; 

    int m=-1, n=-1, validInput;
    validInput = processinput(&m, &n, argv);

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
                if (validInput) {
                    tabstops_fixed(start, i, m, n);
                } else {
                    if (argc == 1)
                        tabstops_default(start, i);
                    else
                        tabstops_variable(start, i, argc-1, argv+1);
                }
                putchar(s[i]);
                start = -1; /* reset sequence of blanks */
            }
        }
    }   
    
    return 0;
}

/* returns 1 and sets m and n if valid, else returns 0 */
int processinput(int *m, int *n, char *argv[]) {
    char firstChar, *pNumber;
    
    /* go through all arguments */
    while (*++argv) {
        firstChar = *argv[0];
        pNumber = (*argv)+1;
        
        switch(firstChar) {
            /* capture m */
            case '-':
                if (isdigits(pNumber)) 
                    *m = atoi(pNumber);
                else return 0;
                break;
            
            /* capture n */         
            case '+':
                if (isdigits(pNumber)) 
                    *n = atoi(pNumber);
                else return 0;
                break;
                
            default: break;
        }
        
        /* return true if both m and n have been set */
        if (*m != -1 && *n != -1)
            return 1;
    }
    
    return 0;
}

/* returns 1 if characters in a string are digits otherwise returns 0 */
int isdigits(char *s) {
    while (*s) 
        if(!isdigit(*s++)) 
            return 0;
    return 1;
}

void tabstops_fixed(int start, int end, int m, int n) {
    
    /* whole sequence is before m - no tabs */
    if (end < m) {
        while (start++ < end)
            putchar(' ');
        return;
    }
    
    /* print preceding blanks. bring start to m or greater */
    if (start < m) {
        while (start < m){
            putchar(' ');
            ++start;
        }
    }
    
    /* start target tabstop at m */
    int tsTarget = m;
    
    /* find target tabstop */
    do {
        tsTarget += n;
    } while (start >= tsTarget);    
    
    /* print tabs, if any */
    while (end >= tsTarget) {
        putchar('\t');
        start = tsTarget;
        tsTarget += n;
    }
    
    /* print remaining blanks, if any */
    while (++start <= end) {
        putchar(' ');
    }
    
    return;
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
#include <ctype.h> /* isdigit */
#include <stdlib.h> /* atoi */

#define MAXLINE 1000
#define DEFAULT_TABSTOP 4

int getline(char s[], int lim);
int processinput(int *m, int *n, char *argv[]);
int isdigits(char *s);

int tabstops_variable(int tabpos, int tsCount, char *tsValues[]);
int tabstops_default(int tabpos);
int tabstops_fixed(int tabpos, int m, int n);

int main(int argc, char *argv[])
{
    char s[MAXLINE];
    int len, blanks = 0, offset = 0;
    
    int m=-1, n=-1, validInput;
    validInput = processinput(&m, &n, argv);
    
    while((len = getline(s, MAXLINE)) > 0) {
        for (int i = 0; i < len; i++) {
            switch(s[i]) {
                case '\t':  
                    if (validInput) {
                        blanks = tabstops_fixed(i + offset, m, n);
                    } else {
                        if (argc == 1)
                            blanks = tabstops_default(i + offset);
                        else 
                            blanks = tabstops_variable(i + offset, argc-1, argv+1);
                    }
                    /* update offset to index */
                    offset += (blanks-1);
                    
                    /* print blanks */
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

/* returns 1 and sets m and n if valid, else returns 0 */
int processinput(int *m, int *n, char *argv[]) {
    char firstChar, *pNumber;
    
    /* go through all arguments */
    while (*++argv) {
        firstChar = *argv[0];
        pNumber = (*argv)+1;
        
        switch(firstChar) {
            /* capture m */
            case '-':
                if (isdigits(pNumber)) 
                    *m = atoi(pNumber);
                else return 0;
                break;
            
            /* capture n */         
            case '+':
                if (isdigits(pNumber)) 
                    *n = atoi(pNumber);
                else return 0;
                break;
                
            default: break;
        }
        
        /* return true if both m and n have been set */
        if (*m != -1 && *n != -1)
            return 1;
    }
    
    return 0;
}

/* returns 1 if characters in a string are digits otherwise returns 0 */
int isdigits(char *s) {
    while (*s) 
        if(!isdigit(*s++)) 
            return 0;
    return 1;
}

int tabstops_fixed(int tabpos, int m, int n) {
    int tsTarget = m;
    
    /* find target tabstop */
    do {
        tsTarget += n;
    } while (tabpos >= tsTarget);
    
    return tsTarget - tabpos;
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
  <image src="../assets/exercise5-12_a.jpg" alt="output for exercise 5-12_a" />
</p>
<p align="center">
  <image src="../assets/exercise5-12_b.jpg" alt="output for exercise 5-12_b" />
</p>

[Back to Main](../readme.md)
