# Exercise1-24
> Write a program to check a C program for rudimentary syntax errors like
> unbalanced parentheses, brackets and braces. Don't forget about quotes, both single and double, 
> escape sequences and comments. (This program is hard if you do it in full generality.)

## Process
Wrote this one over a day and kept a small scope. 

It checks for:
1. matching parentheses, brackets and braces
2. matching single and double quotes
3. matching comments
4. escape sequences within single or double quotes

I set a hierarchy so that it doesn't check for parentheses, brackets or braces if there is a char, string or comment being set, 
which could result in strange error messages further down the line

This code is quite verbose and I'm wondering if theres a simple way to cut down on a lot of repetitive conditional statements, 
but overall I'm happy it works. I can see you could really go down a rabbit hole creating all the ins and outs of syntax validation


## Final Code
```c
#include <stdio.h>

int main() {
    int c;
    int foundError = 0;
    
    int singleQuote = 0;
    int doubleQuote = 0;
    int escapeSeq = 0;
    int comment = 0;
    
    int braceBegin = 0;
    int braceEnd = 0;
    int bracketBegin = 0;
    int bracketEnd = 0;
    int parenthesesBegin = 0;
    int parenthesesEnd = 0;
    
    /* 
        COMMENT STATUS
        0 no comment
        1 found '/'
        2 single line comment
        3 multiline comment
        4 found '*'
    */
    
    while((c = getchar()) != EOF) {
        
        /* single quote mode */
        if(c=='\'' && doubleQuote==0 && escapeSeq==0 && comment==0){
            if(singleQuote==1)
                singleQuote=0;
            else
                singleQuote=1;
        }
        
        /* double quote mode */
        if(c=='\"' && singleQuote==0 && escapeSeq==0 && comment==0){
            if(doubleQuote==1)
                doubleQuote=0;
            else
                doubleQuote=1;
        }
        
        
        /* comment begin mode */
        if(c=='/' && singleQuote==0 && doubleQuote==0 && escapeSeq==0 && comment!=2){

            if(comment==1)
                comment=2; /* single line comment begin */
            else {
                if(comment==4)
                    comment=0; /* found end of multiline comment*/
                else
                    comment=1;
            }
        }
        
        /* comment single line mode */
        if(c=='\n' && doubleQuote==0 && comment==2)
            comment=0; /* turn off single line comment */
        
        /* comment start mode */
        if(c=='*' && singleQuote==0 && doubleQuote==0 && escapeSeq==0 && comment!=0 && comment!=2){
            if(comment==1){
                comment = 3; /* multiline comment begin */
            }
            else {
                if(comment==3){
                    comment = 4; /* maybe multiline comment end */
                }
            }
        }
        
        
        /* bracket begin */
        if(c=='[' && singleQuote==0 && doubleQuote==0 && escapeSeq==0 && comment==0)
            bracketBegin++;
        
        /* bracket end */
        if(c==']' && singleQuote==0 && doubleQuote==0 && escapeSeq==0 && comment==0)
            bracketEnd++;
        
        /* brace begin */
        if(c=='{' && singleQuote==0 && doubleQuote==0 && escapeSeq==0 && comment==0)
            braceBegin++;
        
        /* brace end */
        if(c=='}' && singleQuote==0 && doubleQuote==0 && escapeSeq==0 && comment==0)
            braceEnd++;
        
        /* parentheses begin */
        if(c=='(' && singleQuote==0 && doubleQuote==0 && escapeSeq==0 && comment==0)
            parenthesesBegin++;
        
        /* parentheses end */
        if(c==')' && singleQuote==0 && doubleQuote==0 && escapeSeq==0 && comment==0)
            parenthesesEnd++;
        
        /* any character with escape sequence mode on*/
        if(escapeSeq==1){
            if(c=='n' || c=='\\' || c=='t'){
                escapeSeq = 0;
            } else {
                printf("Error with escape sequence\n");
                return 0;
            }
        }
        
        /* escape sequence mode */
        if(c=='\\' && (singleQuote==1 || doubleQuote==1) &&escapeSeq == 0){
                escapeSeq = 1;
        }
    }
    
    if(singleQuote==1) {
        foundError++;
        printf("Error! ");
        printf("Single quote not closed\n");
    }
  
    if(doubleQuote==1){
        foundError++;
        printf("Error! ");
        printf("Double quote not closed\n");
    }
        
    if(escapeSeq==1){
        foundError++;
        printf("Error! ");
        printf("Escape sequence not closed\n");
    }

    if(comment==1 || comment ==3){
        foundError++;
        printf("Error! ");
        printf("Comment not closed\n");
    }
        
    if(comment==2){
        foundError++;
        printf("Error! ");
        printf("Singleline comment not closed\n");
    }
        
    
    if(bracketBegin!=bracketEnd){
        foundError++;
        printf("Error! ");
        printf("Found bracket not closed\n");
    }
        
    if(braceBegin!=braceEnd){
        foundError++;
        printf("Error! ");
        printf("Found brace not closed\n");
    }
        
    if(parenthesesBegin!=parenthesesEnd){
        foundError++;
        printf("Error! ");
        printf("Found parentheses not closed\n");
    }
    
    if(foundError==0)
        printf("No Errors Found");
    else     
        printf("Found %d error types", foundError);
    
    return 0;
}
```

## Output
Without syntax errors
<p align="center">
    <image src="../assets/exercise1-24_output1.jpg" alt="output for exercise1-24" />
</p>

<p align="center">
    <image src="../assets/exercise1-24_output2.jpg" alt="output for exercise1-24" />
</p>

With syntax errors

<p align="center">
    <image src="../assets/exercise1-24_output3.jpg" alt="output for exercise1-24" />
</p>

<p align="center">
    <image src="../assets/exercise1-24_output4.jpg" alt="output for exercise1-24" />
</p>

[Back to Main](../readme.md)
