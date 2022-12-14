# lesson-10

## Review of Previous Concepts:
### MYSTRSTR & GREP
```c
#include <stdio.h>

char *mystrstr(char *haystack, char *needle);

int main() {

    char haystack[100] = "In CS102, Janina is getting an A.";
    char needle[10] = "Janina";
    char *ret;

    ret = mystrstr(haystack, needle);

    printf("The substring is: %s\n", ret);
    printf("The substring is: %c\n", *ret);

    return 0;
}


char *mystrstr(char *haystack, char *needle) {
    char *nstart = needle;

    while (*needle != '\0' && *haystack != '\0') {
        printf("%c %c\n", *haystack, *needle);
        if (*haystack == *needle) {
            haystack++;
            needle++;
        }
        else {
            needle = nstart;
            haystack++;
        }
    }

    return haystack - (needle - nstart);
}
```
Code explanation (draw a diagram to help understand the code):
- ```nstart``` saves the beginning address of ```needle```. We increment ```haystack``` every iteration of the loop. When ```haystack == needle```, we also increment ```needle``` by 1. When needle points to the end of Janina, ```needle``` points to ‘\0’, and the loop iteration stops. 
  - The return statement: ```(nstart-needle)``` gives the length of the word in needle. Haystack is at the end of the needle word in the haystack array so, ```(haystack - (nstart-needle))``` is the start of where needle is in the ```haystack```.


## KEYWORD-COUNTING PROGRAM: Review

We discussed the following program last class.

```c
#include <stdio.h>
#include <ctype.h>
#include <string.h>
#define MAXWORD 100
struct key {
    char *word;
    int count;
} keytab[] = {
        "auto", 0,
        "break", 0,
        "case", 0,
        "char", 0,
        "const", 0,
        "continue", 0,
        "default", 0,
        /* ... */
        "unsigned", 0,
        "void", 0,
        "volatile", 0,
        "while", 0
};
#define NKEYS (sizeof keytab / sizeof keytab[0])
int getword(char *, int);
int binsearch(char *, struct key *, int);
int getch(void);
void ungetch(int);
// K&R Pg. 134-136
// count C keywords
int main()
{
    int n;
    char word[MAXWORD];
    while (getword(word, MAXWORD) != EOF)
        if (isalpha(word[0]))
            if ((n = binsearch(word, keytab, NKEYS)) >= 0)
                keytab[n].count++;
    for (n = 0; n < NKEYS; n++)
        if (keytab[n].count > 0)
            printf("%4d %s\n",
                keytab[n].count, keytab[n].word);
    return 0;
}
// binserach: find word in tab[0]...tab[n-1
int binsearch(char *word, struct key tab[], int n)
{
    int cond;
    int low, high, mid;
    low = 0;
    high = n - 1;
    while (low <= high) {
        mid = (low + high) / 2;
        if ((cond = strcmp(word, tab[mid].word)) < 0)
            high = mid - 1;
        else if (cond > 0)
            low = mid + 1;
        else
            return mid;
    }
    return -1;
}
// getword:get next word or character from input
int getword(char *word, int lim)
{
    int c, getch(void);
    void ungetch(int);
    char *w = word;
    while (isspace(c = getch()))
        ;
    if (c != EOF)
        *w++ = c;
    if (!isalpha(c)) {
        *w = '\0';
        return c;
    }
    for ( ; --lim > 0; w++)
        if (!isalnum(*w = getch())) {
            ungetch(*w);
            break;
        }
    *w = '\0';
    return word[0];
}
// K&R pg. 79
#define BUFSIZE 100
char buf[BUFSIZE]; // buffer for ungetch
int bufp = 0; // next free position in buf
int getch(void) // get a (possibly pushed back) character
{
    return (bufp > 0) ? buf[--bufp] : getchar();
}
void ungetch(int c) // push character back on input
{
    if (bufp >= BUFSIZE)
        printf("ungetch: too many caracters\n");
    else
        buf[bufp++] = c;
}
```


## KEYWORD-COUNTING PROGRAM: above program using pointers to and arrays of structures
- Using pointers instead of array indices: ```./a.exe < keywordCount.c```

#### NOTES
- The return type of binsearch is a pointer to the key structure.
- The addition of 2 pointers is illegal, but subtraction is legal.
- Note that we also avoid illegal pointers.

```c
#include <stdio.h>
#include <ctype.h>
#include <string.h>
#define MAXWORD 100
struct key {
    char *word;
    int count;
} keytab[] = {
        "auto", 0,
        "break", 0,
        "case", 0,
        "char", 0,
        "const", 0,
        "continue", 0,
        "default", 0,
        /* ... */
        "unsigned", 0,
        "void", 0,
        "volatile", 0,
        "while", 0
};
#define NKEYS (sizeof keytab / sizeof keytab[0])
int getword(char *, int);
struct key *binsearch(char *, struct key *, int);
int getch(void);
void ungetch(int);
// K&R Pg. 137
// count C keywords; pointer version
int main()
{
    char word[MAXWORD];
    struct key *p;
    while (getword(word, MAXWORD) != EOF)
        if (isalpha(word[0]))
            if ((p=binsearch(word, keytab, NKEYS)) != NULL)
                p->count++;
    for (p = keytab; p < keytab + NKEYS; p++)
        if (p->count > 0)
            printf("%4d %s\n",
                p->count, p->word);
    return 0;
}
// binserach: find word in tab[0]...tab[n-1]
struct key *binsearch(char *word, struct key *tab, int n)
{
    int cond;
    struct key *low = &tab[0];
    struct key *high = &tab[n];
    struct key *mid;
    while (low < high) {
        mid = low + (high-low) / 2;
        if ((cond = strcmp(word, mid->word)) < 0)
            high = mid;
        else if (cond > 0)
            low = mid + 1;
        else
            return mid;
    }
    return NULL;
}
// getword:get next word or character from input
int getword(char *word, int lim)
{
    int c, getch(void);
    void ungetch(int);
    char *w = word;
    while (isspace(c = getch()))
        ;
    if (c != EOF)
        *w++ = c;
    if (!isalpha(c)) {
        *w = '\0';
        return c;
    }
    for ( ; --lim > 0; w++)
        if (!isalnum(*w = getch())) {
            ungetch(*w);
            break;
        }
    *w = '\0';
    return word[0];
}
// K&R pg. 79
#define BUFSIZE 100
char buf[BUFSIZE]; // buffer for ungetch
int bufp = 0; // next free position in buf
int getch(void) // get a (possibly pushed back) character
{
    return (bufp > 0) ? buf[--bufp] : getchar();
}
void ungetch(int c) // push character back on input
{
    if (bufp >= BUFSIZE)
        printf("ungetch: too many caracters\n");
    else
        buf[bufp++] = c;
}
```



## SELF-REFERENTIAL STRUCTURES
#### TREE EXAMPLE

```c
#include <stdio.h>
#include <stdlib.h>
#include <ctype.h>
#include <string.h>
// K&R pg. 140-143
#define MAXWORD 100
struct tnode {           // the tree node:
    char *word;          // points to the text
    int count;           // number of occurrences
    struct tnode *left;  // left child
    struct tnode *right; // right child
};
struct tnode *addtree(struct tnode *, char *);
void treeprint(struct tnode *);
int getword(char *, int);
// word frequency count
int main()
{
    struct tnode *root;
    char word[MAXWORD];
    root = NULL;
    while (getword(word, MAXWORD) != EOF)
        if (isalpha(word[0]))
            root = addtree(root, word);
    treeprint(root);
    return 0;
}
struct tnode *talloc(void);
char *mystrdup(char *);
// addtree: add a node with w, at or below p
struct tnode *addtree(struct tnode *p, char *w)
{
    int cond;
    if (p == NULL) {    // a new word has arrived
        p = talloc();   // make a new node
        p->word = mystrdup(w);
        p->count = 1;
        p->left = p->right = NULL;
    } else if ((cond = strcmp(w, p->word)) == 0)
        p->count++;     // repeated word
    else if (cond < 0)  // less than into left subtree
        p->left = addtree(p->left, w);
    else                // greater than into right subtree
        p->right = addtree(p->right, w);
    return p;
}
// treeprint: in-order print of tree p
void treeprint(struct tnode *p)
{
    if (p != NULL) {
        treeprint(p->left);
        printf("%4d %s\n", p->count, p->word);
        treeprint(p->right);
    }
}
// talloc: make a tnode
struct tnode *talloc(void)
{
    return (struct tnode *) malloc(sizeof(struct tnode));
}
char *mystrdup(char *s) // make a duplicate of s
{
    char *p;
    p = (char *) malloc(strlen(s)+1); // +1 for '\0'
    if (p != NULL)
        strcpy(p, s);
    return p;
}
int getword(char *word, int lim)
{
    int c, getch(void);
    void ungetch(int);
    char *w = word;
    while (isspace(c = getch()))
        ;
    if (c != EOF)
        *w++ = c;
    if (!isalpha(c)) {
        *w = '\0';
        return c;
    }
    for ( ; --lim > 0; w++)
        if (!isalnum(*w = getch())) {
            ungetch(*w);
            break;
        }
    *w = '\0';
    return word[0];
}
// K&R pg. 79
#define BUFSIZE 100
char buf[BUFSIZE]; // buffer for ungetch
int bufp = 0; // next free position in buf
int getch(void) // get a (possibly pushed back) character
{
    return (bufp > 0) ? buf[--bufp] : getchar();
}
void ungetch(int c) // push character back on input
{
    if (bufp >= BUFSIZE)
        printf("ungetch: too many caracters\n");
    else
        buf[bufp++] = c;
}
```

## TABLE LOOKUP
#### HASH TABLE

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

// K&R Pg. 144-145

struct nlist {              // table entry:
    struct nlist *next;     // next entry in chain
    char *name;             // defined name
    char *defn;             // replacement text
};

#define HASHSIZE 101
static struct nlist *hashtab[HASHSIZE]; // pointer table

// hash: form hash value for string
unsigned hash(char *s)
{
    unsigned hashval;

    for (hashval = 0; *s != '\0'; s++)
        hashval = *s + 31 * hashval;
    return hashval % HASHSIZE;
}

// lookup:look for s in hashtab
struct nlist *lookup(char *s)
{
    struct nlist *np;

    // loop through linked list
    for (np = hashtab[hash(s)]; np != NULL; np = np->next)
        if (strcmp(s, np->name) == 0)
            return np; // found

    return NULL;       // not found
}

struct nlist *lookup(char *);

// install:put (name, defn) in hashtab
struct nlist *install(char *name, char *defn)
{
    struct nlist *np;
    unsigned hashval;

    if ((np = lookup(name)) == NULL) { // not found
        np = (struct nlist *) malloc(sizeof(*np));
        if (np == NULL || (np->name = strdup(name)) == NULL)
            return NULL;
        hashval = hash(name);
        np->next = hashtab[hashval];
        hashtab[hashval] = np;
    } else // already there
        free((void *) np->defn); // free previous defn
    if ((np->defn = strdup(defn)) == NULL)
        return NULL;

    return np;
}


int main()
{
    // Example of using a hash table
    install("A", "65");
    install("B", "66");

    struct nlist *n = lookup("B");

    printf("%s", n->defn);

    return 0;
}
```

## TYPEDEF
The C programming language provides a keyword called typedef, which you can use to give a type a new name. Following is an example to define a term BYTE for one-byte numbers − https://www.tutorialspoint.com/cprogramming/c_typedef.htm

The following typedef:
```c
typedef unsigned char byte;
````
can be used to write this: 
```c
unsigned char b1, b2;
```
like this:
```
byte  b1, b2;
```

#### BASIC TYPEDEF

```c
typedef int Length;

Length len, maxlen;
Length *lengths[];

typedef char *String;
String p, ineptr[MAXLINES], alloc(int);
int strcmp(String, String);
p = (String) malloc(100);
```

### STRUCT TYPEDEF
```c
typedef struct tnode *Treeptr;

typedef struct tnode{   // the tree node:
    char *word;             // points to the text
    int count;              // number of occurrences
    Treeptr left;           // left child
    Treeptr right;          // right child
} Treenode;

Treeptr talloc(void)
{
    return (Treeptr) malloc(sizeof(Treenode));
}
```

## Debugging
In class we will go over how to use debugging in Replit/VSCode. Gdb is another way we can debug using the command line. The following section contains instructions on how to use gdb.

## GDB
If you are using cygwin, this should have been installed when you downloaded cygwin. Otherwise, you can experiment with gdb on Replit.

#### WHAT IS GDB?
- A debugger for C and C++ (along with Pascal, Objective-C, and Fortran).
- Does 4 main things:
    - Start your program, specifying anything that might affect its behavior
    - Make your program stop on specified conditions
    - Examine what has happened when your program stopped
    - Change things in your program so you can experiment with correcting the effects of one bug

Resource on GDB:
https://web.eecs.umich.edu/~sugih/pointers/gdbQS.html#:~:text=Setting%20breakpoints%20A%20breakpoint%20is,%5Bfilename%5D%3A%5Blinenumber%5D%22.

### SAMPLE SEGMENTATION FAULT PROGRAM
```c
#include <stdio.h>

int main()
{
    int a, b, c;
    char d, e, f;
    a = 1;
    b = 2;
    d = 'D';
    e = 'E';

    int *a_ptr, *b_ptr, *c_ptr;
    char *d_ptr, *e_ptr, *f_ptr;

    a_ptr = &a;
    b_ptr = a_ptr;
    c_ptr = b_ptr;

    d_ptr = &d;
    e_ptr = d_ptr;
    f_ptr = d_ptr;

    char *ptr = NULL; // Null pointer
    printf("%c", *ptr);

    return 0;
}
```

### COMPILING A PROGRAM FOR DEBUGGING
- gcc -g segfault.c
    - The -g flag enables built-in debugging support; note that the file size is slightly bigger and it takes more time
- gdb
    - Start the gdb interface
- (gdb) help
    - get the help menu

### RUNNING A PROGRAM IN GDB
- (gdb) file a.exe
    - loads the executable in gdb
- (gdb) run
    - runs the executable
- (gdb) continue
    - continues the program

### BREAKPOINTS
#### BREAKPOINTS INSTRUCTIONS
- Places in code to tell gdb to stop so you can analyze the code
- (gdb) break segfault.c:8
    - Note that it stops before an actionable line (initialization/assignment/function call)
    - Can also break at a specific function w/ the function name
- (gdb) run

#### CHECKING VARIABLE VALUES
- (gdb) step
    - steps through the program line by line; enter to repeat
- (gdb) next
    - goes inside a function; more on that later

#### CHECKING VARIABLE VALUES
- (gdb) print a
- (gdb) print a_ptr
- (gdb) print *a_ptr
- (gdb) print ptr - NULL Pointer
- (gdb) print *ptr - can't access!

### STEPPING INTO FUNCTIONS
#### STEPPING INTO FUNCTIONS SAMPLE

```c
#include <stdio.h>

void sample_function()
{
    int a, b, c;
    a = 101;
    b = 102;
    char d, e, f;
    d = 'd';
    e = 'e';

    return;
}

int main()
{
    sample_function();

    int a, b, c;
    char d, e, f;
    a = 1;
    b = 2;
    d = 'D';
    e = 'E';

    int *a_ptr, *b_ptr, *c_ptr;
    char *d_ptr, *e_ptr, *f_ptr;

    a_ptr = &a;
    b_ptr = a_ptr;
    c_ptr = b_ptr;

    d_ptr = &d;
    e_ptr = d_ptr;
    f_ptr = d_ptr;

    char *ptr = NULL; // Null pointer
    printf("%c", *ptr);

    return 0;
}
```

### STEPPING INTO A FUNCTION
- (gdb) break segfault_func.c:17
- (gdb) step
    - Note that this time, we actually entered the function
    - Also note that the variables are in the scope of the function, not main

### WATCHING VARIABLES

#### MODIFIED PROGRAM
```c
#include <stdio.h>

void sample_function()
{
    int a, b, c;
    a = 101;
    b = 102;
    char d, e, f;
    d = 'd';
    e = 'e';

    return;
}

int main()
{
    sample_function();

    int a, b, c;
    char d, e, f;
    a = 1;
    b = 2;
    d = 'D';
    e = 'E';
    a = 11;
    a = 111;

    int *a_ptr, *b_ptr, *c_ptr;
    char *d_ptr, *e_ptr, *f_ptr;

    a_ptr = &a;
    b_ptr = a_ptr;
    c_ptr = b_ptr;

    d_ptr = &d;
    e_ptr = d_ptr;
    f_ptr = d_ptr;

    char *ptr = NULL; // Null pointer
    printf("%c", *ptr);

    return 0;
}
```

#### GDB WATCH
- gdb
- (gdb) file a.exe
- (gdb) break segfault_watch.c:17
- (gdb) run
- (gdb) watch a - whenever a changes, it will print the old and new values and stop before the next line
- (gdb) next

### OTHER FEATURES

#### OTHER GDB COMMANDS
- breaktrace - stack trace leading to a seg fault
- where - same as backtrace, but in middle of program (e.g. in a function or in main)
- finish - runs until current function finishes
- info breakpoints - shows all declared breakpoints
- clear file:linenum - deletes breakpoint
- delete - deletes all breakpoints

#### CONDITIONAL BREAKPOINTS
- (gdb) break segfault_watch.c:27 if a >=100
    - stops at line 27 if the variable a is greater than 100

## GITHUB FORKS
#### FORK EXAMPLE
Original Repo: https://github.com/CS102-Classroom-Notes-and-Assignments/demo-repo

Forked Repo: https://github.com/nsubbaian/demo-repo-fork

Resource: https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks/about-forks

# Homework
- The C Programming Language, 2nd Edition - Kerninghan & Ritchie, Chapter 6.3-6.8
