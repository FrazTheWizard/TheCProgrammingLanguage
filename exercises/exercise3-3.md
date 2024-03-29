# Exercise 3-3

> Write a function expand(s1,s2) that expands shorthand notations 
 like a-z in the string s1 into the equivalent complete list 
 abc...xyz in s2. Allow for letters of either case and digits, 
 and be prepared to handle cases like a-b-c and a-z0-9 
 and -a-z. Arrange that a leading or trailing - is taken literally.

## Process
Completed over a few days. The main functionality is fairly straightforward.

Found that writing the main functionality as a story to have helped:
```
if leading dash is found print dash
set current index to one more value

LOOKING FOR FROM
we are looking for 'from' candidate alphaALPHAnumber
if found 'from' candidate then looking for dash

LOOKING FOR DASH
if no dash, go back to looking for 'from' candidate
if dash looking for 'to' candidate alphaALPHAnumber
 
LOOKING FOR TO
(SPECIALCASE)if found EOF, print dash
if no candidate go back to looking for 'from' candidate
if found 'to' candidate 
if 'to' match casetype of 'from'
print expansion
set 'from' to value of 'to'
looking for dash
```
I found the other potential sidecases to be interesting.

After I completed the functionality, I refactored if statements to a switch statement with constant
values. 

I could keep going and make the code more readable, as well as handle 
more potential errors or potential undesirable behavior but I'm happy to leave it there.

Also I was confused as to what 'taken literally' means, so I just printed it

## Edit
This is a note after finishing my solution. I found this [video](https://youtu.be/o0Ouhlh1SfA) that shares a solution to the problem, 
with two things I didn't consider.

1. Using string manipulation to find parameters
   
   my solution is for a input stream of incoming characters, rather than a string that I could just move forward or backward in 
3. Using standard library comparisons
   
   I performed my comparisons by counting the exact ascii numbers
   
In hindsight, those two things would have made my code much more compact and readable. But I'm happy with my solution. Could be improved, but it works!

## Code
```c
#include <stdio.h>

#define LINE_LENGTH 999

#define FROM    0
#define DASH    1
#define TO      2

#define EX_OFF  0
#define EX_ON   1

#define LITERAL_DASH_OFF    0
#define LITERAL_DASH_ON     1

void expand(char s1[], char s2[]){
    int i, j, from, to, looking, extendeddash, index, literaldash;
    char c;
    i = 0, index = 0;
    literaldash = LITERAL_DASH_OFF;
    looking = FROM;
    extendeddash = EX_OFF;
    
    /* remove leading dash */
    if(s1[0]=='-'){
        s2[index++] ='-';
        i=1;
    }
    
    for(;(c = s1[i]) != EOF; i++){
        literaldash = LITERAL_DASH_OFF;
        switch(looking){
            case FROM:
                if(
                    (c >= 48 && c <= 57) ||
                    (c >= 65 && c <= 90) ||
                    (c >= 97 && c <= 122)
                ){
                    looking = DASH;
                    from = c;
                }
                if(c == '-'){
                    literaldash = LITERAL_DASH_ON;
                }
                break;
            
            case DASH:
                if(c == '-'){
                    looking = TO;
                    literaldash = LITERAL_DASH_ON;
                } else {
                    if(
                        (c >= 48 && c <= 57) ||
                        (c >= 65 && c <= 90) ||
                        (c >= 97 && c <= 122)
                    ){
                        from = c;
                        looking = DASH;
                        extendeddash = EX_OFF;
                    } else {
                        looking = FROM;
                    }
                }
                break;
            
            case TO:
                if(
                    (c >= 48 && c <= 57) ||
                    (c >= 65 && c <= 90) ||
                    (c >= 97 && c <= 122)
                ){
                    to = c;
                    /* Error Check - Value */
                    if(to <= from){
                        looking = FROM;
                        from = c;
                        extendeddash = EX_OFF;
                        break;
                    }
                    
                    /*skip 'from' if extended dash*/
                    if(extendeddash == EX_ON){
                        j = ++from;
                    } else {
                        j = from;
                    }
                    
                    /*print chars from - to*/
                    for(;j <= to; j++){
                        /*printf("%c", j);*/
                        s2[index++] = j;
                    }                    
                    looking = DASH;
                    extendeddash = EX_ON;
                    from = to;
                } else if (c == '-'){
                    literaldash = LITERAL_DASH_ON;
                    looking = FROM;
                } else {
                    looking = FROM;
                    from = c;
                }
                break;
        }
    }
    if(literaldash == LITERAL_DASH_ON){
        s2[index] = '-';
        s2[++index] = '\0';    
    } else {
        s2[index] = '\0';
    }
    
}

int main(){
    char s1[LINE_LENGTH]; char s2[LINE_LENGTH];
    int len=0;
    char c;

    while((c = getchar()) != EOF){
        s1[len] = c;
        len++;
    }
    
    s1[len++] = EOF;
    expand(s1,s2);
    
    int i = 0;
    while(s2[i] != '\0'){
        printf("%c", s2[i++]);
    }
}
```

## Output
<p align="center">
    <image src="../assets/exercise3-3_output1.jpg" alt="output for exercise3-3" />
</p>
<p align="center">
    <image src="../assets/exercise3-3_output2.jpg" alt="output for exercise3-3" />
</p>

[Back to Main](../readme.md)
