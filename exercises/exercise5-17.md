# Exercise 5-17
> Add a field-handling capability, so sorting may be done on fields within lines, each field sorted according to an independent set of options.
> (The index for this book was sorted with `-df` for the index category and `-n` for the page numbers.) 

## Process
This one took me a couple of weeks of 1 hour morning sessions and I found it quite challenging

I found it hard to wrap my head around the whole exercise, this is the largest c program I have written so far. 
The exercise is kind of vague, which I get is a challenge to fill the gaps in yourself.
So I went with xml for the input data, as a fun challenge and to potentially allow for named fields

I tried to keep the program intact from previous exercises with it's qsort, comparison, line collection and general structure

As usual, I've skipped any real error checking as it's a learning exercise.

I also decided to lean on global variables, as I think it's fine for a program of this size, which is unlikely going to go anywhere after this
and makes it easier to read in my opinion.

### Usage
The program can be used like previous exercises, with the addition of field sort and ouput

- `sort -nr` sort lines numerically and in reverse order

- `sort 0-` separate lines by fields, sort by the first field alphabetically

- `sort 1-dr 3-n` separate lines by fields, sort second field in reverse, directory order, then fourth field in numeric order



## Code
```c
#include <stdio.h>  /* getchar, printf*/
#include <stdlib.h> /* atof */
#include <ctype.h>  /* isdigit, isalnum, isspace, tolower */
#include <string.h> /* strcpy */

#define TRUE            1
#define FALSE           0
#define MAX_INPUT_LINES 5000
#define MAX_STR_LEN     1000
#define MAX_FIELDS      9
#define MAX_SORT_TYPES  4
#define SORT_NUMERIC    0
#define SORT_REVERSE    1
#define SORT_FOLD       2
#define SORT_DIRECTORY  3

int process_args(int argc, char *argv[]);
int process_arg_line(char *argv);
int process_arg_field(char *argv, int field_order_index);
int readlines(char *lineptr[], int maxlines);
int getline(char s[], int lim);
char *alloc(int n);
int find_field(char *s1, int field_num, char str[]);
int (*comp)(void *, void *);
int cmpstr(char *s, char *t);
int cmpnum(char *s, char *t);
int cmpfield(char *s, char *t);
void qsort_kandr(void *lineptr[], int left, int right, int (*comp)(void *, void *));
void (*output)(char *lineptr[], int nlines);
void writelines(char *lineptr[], int nlines);
void writefields(char *lineptr[], int nlines);
int calc_field_lengths(char *lineptr[], int nlines);

int field_sort[MAX_FIELDS][MAX_SORT_TYPES];
int field_sort_order[MAX_FIELDS];
int field_lengths[MAX_FIELDS];
int field_sort_count = 0;
int reverse     = FALSE;
int numeric     = FALSE;
int directory   = FALSE;
int fold        = FALSE;
char *lineptr[MAX_INPUT_LINES]; /* pointers to text lines */

/* main: sort input lines */
int main(int argc, char *argv[])
{
    int nlines; /* number of input lines read */
    
    if(process_args(argc, argv) == -1) 
        return 1;
    
    if ((nlines = readlines(lineptr, MAX_INPUT_LINES)) >= 0) {
        qsort_kandr((void **) lineptr, 0, nlines-1, comp);
        output(lineptr, nlines);
        return 0;
    } else {
        printf("error: input too big to sort\n");
        return 1;
    }
    
    return 0;
}

/* process_args: process cmdline arguments */
int process_args(int argc, char *argv[])
{
    int is_line, is_field, field_order_index, is_another_arg;
    
    comp = (int (*)(void *, void *)) cmpstr; /* default comparison */
    output = writelines;
    
    if (argc == 1) return 0;
    
    argc--, argv++; /* skip program filepath */
    
    field_order_index = 0;
    is_another_arg = argc > 0;
    
    while (is_another_arg) {
        is_line = (argv[0][0] == '-');
        is_field = isdigit(argv[0][0]);
        
        /* argument not valid */
        if (!is_line && !is_field) {
            printf("error: argument doesn't begin with '-' or number\n");
            return -1;
            
        /* argument */
        } else if (is_line && field_order_index == 0) {
            if (process_arg_line(*argv) == -1) {
                printf("error: unable to parse sort arguments\n");
                return -1;
            }
            if (numeric == 1) {
                comp = (int (*)(void *, void *)) cmpnum;
            }
        
        /* argument field */
        } else if (is_field) {
            if (process_arg_field(*argv, field_order_index) == -1) {
                printf("error: unable to parse sort field arguments\n");
                return -1;
            }
            field_order_index++;
            comp = (int (*)(void *, void *)) cmpfield;
            output = writefields;
        }
        
        argc--, argv++;
        is_another_arg = argc > 0;
    }
        
    return 0;
}

/* process_arg_line: capture sort flags */
int process_arg_line(char *argv)
{
    int c;
    while (c = *++argv) /* skip '-' char */
        switch (c) {
            case 'n':
                numeric = TRUE;
                break;
            case 'r':
                reverse = TRUE;
                break;
            case 'f':
                fold = TRUE;
                break;
            case 'd':
                directory = TRUE;
                break;
            default:
                printf("error: illegal character found: %c\n", c);
                return -1;
                break;
        }
    
    return 0;
}

/* process_arg_field: capture sort field flags in sort order index */
int process_arg_field(char *argv, int field_order_index)
{
        int is_char1_number, is_char2_dash; 
        int field_index, i, c;
        
        is_char1_number = isdigit(argv[0]);
        is_char2_dash   = argv[1] != '-';
        
        if(!is_char1_number || is_char2_dash) {
            printf("error: field syntax incorrect fieldnumber-sortoptions\n");
            return -1;
        } else {
            i = 2, field_index = (int) argv[0] - '0';
        }
        
        while (c = argv[i++]) {
            switch (c) {
                case 'n':
                    field_sort[field_index][SORT_NUMERIC] = TRUE;
                    break;
                case 'r':
                    field_sort[field_index][SORT_REVERSE] = TRUE;
                    break;
                case 'f':
                    field_sort[field_index][SORT_FOLD] = TRUE;
                    break;
                case 'd':
                    field_sort[field_index][SORT_DIRECTORY] = TRUE;
                    break;
                default:
                    printf("error: illegal character found: %c\n", c);
                    return -1;
                    break;
            }
        }
        
        field_sort_order[field_order_index] = field_index;
        field_sort_count++;
        
        return 0;
}

/* find_field: find field of index in line */
int find_field(char *s, int field_index, char *s_out)
{
    int tag, in_start_tag, found;
    
    tag = 0;
    in_start_tag = FALSE;
    found = FALSE;
    
    while (*s && !found) {
        if (*s == '<' && *(s+1) != '/')
            in_start_tag = TRUE;
        
        if (*s == '>') {
            if(!in_start_tag) {
                tag++;
            } else {
                in_start_tag = FALSE;
                if(tag == field_index) found = TRUE;
            }
        }
        s++;
    }
    
    while (*s != '<' && *s)
        *s_out++ = *s++;
    
    *s_out = '\0';
    return found;
}

/* cmpfield: compare s and t based on argument inputs */
int cmpfield(char *s, char *t)
{
    char field_a[MAX_STR_LEN], field_b[MAX_STR_LEN];
    int i, field_index, cmpval;
    int (*compare) (void *, void *);
    
    i = 0, cmpval = 0;
    
    do {
        field_index = field_sort_order[i++];
        
        numeric     = field_sort[field_index][SORT_NUMERIC];
        reverse     = field_sort[field_index][SORT_REVERSE];
        fold        = field_sort[field_index][SORT_FOLD];
        directory   = field_sort[field_index][SORT_DIRECTORY];
           
        find_field(s, field_index, field_a);
        find_field(t, field_index, field_b);
        
        if(numeric)
            cmpval = cmpnum(field_a, field_b);
        else        
            cmpval = cmpstr(field_a, field_b);
        
    } while(i < field_sort_count && !cmpval);
    
    return cmpval;
}

/* cmpnum: compare s1 and s2 numerically */
int cmpnum(char *s, char *t)
{
    double v1, v2;
    v1 = atof(s);
    v2 = atof(t);
    
    if (reverse) {
      double temp = v1;
      v1 = v2;
      v2 = temp;
    }
    
    if (v1 < v2)
        return -1;
    else if (v1 > v2)
        return 1;
    else
        return 0;
}

/* cmpstr: compare string s and t  */
int cmpstr(char *s, char *t)
{
    char s1, t1;
    do {
      if (directory) {
          while(!isalnum(*s) && !isspace(*s) && *s != '\0')
              s++;
          while(!isalnum(*t) && !isspace(*t) && *t != '\0')
              t++;
      }
      
      if (fold)
          s1 = tolower(*s), t1 = tolower(*t);
      else
          s1 = *s, t1 = *t;
      
      if (reverse) {
          char temp = s1;
          s1 = t1;
          t1 = temp;
      }
      
      s++, t++;
    } while (s1 != '\0' && s1 == t1);
    
    return s1 - t1;
}

/* readlines: read input lines */
int readlines(char *lineptr[], int maxlines)
{
	int len, nlines;
	char *p, line[MAX_STR_LEN];
	
	nlines = 0;
	while ((len = getline(line, MAX_STR_LEN)) > 0)
		if (nlines >= maxlines || (p = alloc(len)) == NULL)
			return -1;
		else {
            if (line[len-1] == '\n')
                line[len-1] = '\0'; /* delete newline */
			strcpy(p, line);
			lineptr[nlines++] = p;
		}
		return nlines;
}

/* getline: read a line into s, return length */
int getline(char s[], int lim)
{
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

/* qsort_kandr: sort v[left]...v[right] into increasing order */
void qsort_kandr(void *v[], int left, int right,
           int (*comp)(void *, void *))
{
    int i, last;
    void swap(void *v[], int i, int j);
    
    if (left >= right)  /* do nothing if array contains */
        return;         /* fewer than two elements */
    swap(v, left, (left + right)/2);
    last = left;
    for (i = left+1; i <= right; i++)
        if ( ((*comp)(v[i], v[left])) < 0)
            swap(v, ++last, i);
    swap(v, left, last);
    qsort_kandr(v, left, last-1, comp);
    qsort_kandr(v, last+1, right, comp);
}

/* swap: interchange v[i] and v[j] */
void swap(void *v[], int i, int j)
{
    char *temp;
    
    temp = v[i];
    v[i] = v[j];
    v[j] = temp;
}

/* writelines: write last n lines */
void writelines(char *lineptr[], int nlines)
{
    int i;
	
    for (i = 0; i < nlines; i++)
		printf("%s\n", lineptr[i]);
}

/* writelines: write last n lines */
void writefields(char *lineptr[], int nlines)
{
    int i, j, k, fields;
    char s[MAX_STR_LEN];
    
    fields = calc_field_lengths(lineptr, nlines);
	
    for (i = 0; i < nlines; i++) {
        for (j = 0; j <= fields; j++) {
            find_field(lineptr[i], j, s);
            for (k = 0; s[k]; k++) {
                printf("%c", s[k]);
            }
            while (k++ < field_lengths[j])
                printf(" ");
        }
        printf("\n");
    }
}


/* calc_field_lengths: find all field lengths for whitespace alignment */
int calc_field_lengths(char *lineptr[], int nlines)
{
    int field_index = 0, count_char = 0, count_field = 0;
    int fields_in_line = 0;
    char s[MAX_STR_LEN];
    
    for (int i = 0; i < nlines; i++) {
        while(find_field(lineptr[i], field_index, s)) {
            while(s[count_char++])
                ;
            
            if (count_char > field_lengths[field_index])
                field_lengths[field_index] = count_char;
            
            count_field++;
            count_char = 0, field_index++;
        }
        
        if(count_field > fields_in_line)
            fields_in_line = count_field;
        
        field_index = 0, count_field = 0;
    }
    
    return fields_in_line;
}

#define ALLOCSIZE 10000	/* size of available space */

static char allocbuf[ALLOCSIZE];	/* storage for alloc */
static char *allocp = allocbuf;		/* next free position */

/* alloc: return pointer to n characters */
char *alloc(int n)
{
	if (allocbuf + ALLOCSIZE - allocp >= n) { /* it fits */
		allocp += n;
		return allocp - n; /* old p */
	} else	/* not enough room */
		return 0;
}
```

## Output
<p align="center">
  <image src="../assets/exercise5-17_cmds.jpg" alt="output for exercise 5-17_cmds" />
</p>
<p align="center">
  <image src="../assets/exercise5-17_data_and_result.jpg" alt="output for exercise 5-17_data_and_result" />
</p>

[Back to Main](../readme.md)
