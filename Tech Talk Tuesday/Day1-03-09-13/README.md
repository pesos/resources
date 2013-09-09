Tech Talk Tuesday - Cool C Programming
===================================================

**Speaker**: Sathyam M Vellal  
**Date**: 03-09-2013

---

## The C Preprocessor

### What you should know about the preprocessor
In the C Program build process, there are three main phases.

* Preprocessing
* Compiling
* Linking

The C Preprocessor is dumb! It does not anything about a C program. Its mainly used to manage the program's text according to our needs before the compiler begins its task. Managing involves including of header files, conditional inclusion of pieces of code, expanding macros, etc.  
If you want to see the output of the preprocessor, you can use the ```-E``` flag for the compiler; ie compile the program as ```gcc -E file.c```. This dumps the preprocessed output of file.c onto your terminal.

There are three standard input/output buffers 

* ```stdin``` The input buffer
* ```stdout``` The default output buffer
* ```stderr``` The output buffer for error/log messages

### Conditional Syntax - ```#ifdef``` and ```#ifndef```

The preprocessor directive ```#ifdef``` is used to define conditional groups of code at the preprocessor level. Based on a condition, a piece of code may or may not be included in the program. The body of this directive is usually termed *controlled text*

```c
#ifdef SOME_MACRO
//some piece of code
#endif
```

In the above example, the body of ```#ifdef``` is included only if ```SOME_MACRO``` is defined. The ```#define``` directive is used to define MACROs.

#### Example 

Here's a sample program.

```c
#ifdef SOME_MACRO
void foo ()
{
}
#endif

int main()
{
    return 0;
}
```
###### Fig(1.1)

Observe that ```SOME_MACRO``` is not defined. And hence the controlled text should not be included. And here's the output of the preprocessor (compiled with ```-E``` option)

```
# 1 "1.c"
# 1 "<command-line>"
# 1 "/usr/include/stdc-predef.h" 1 3 4
# 1 "<command-line>" 2
# 1 "1.c"






int main()
{
    return 0;
}
```
###### Fig(1.2)

Let me now define the MACRO ```#ifdef``` using ```#define```.

```c
#define SOME_MACRO

#ifdef SOME_MACRO
void foo ()
{
}
#endif

int main()
{
    return 0;
}

```
###### Fig(1.3)

And this is the output of the preprocessor now.

```
# 1 "1.c"
# 1 "<command-line>"
# 1 "/usr/include/stdc-predef.h" 1 3 4
# 1 "<command-line>" 2
# 1 "1.c"



void foo ()
{
}


int main()
{
    return 0;
}

```
###### Fig(1.4)

You can observe that when the macro is defined, the controlled text is included. Note again, that the preprocessor does not know that the controlled text is a function or any of the C constructs. All it sees is some text.  
You can also define MACROs when invoking the compiler with the ```-D``` option. Compiling the code in Fig(1.1) as ```gcc -E -DSOME_MACRO 1.c``` gives me the same output as Fig(1.4).

MACROs also can be object-like MACROs and function-like MACROs.

```c
#define BAR 5
#define FOO(X) printf("%d", X)

int main()
{
    //FOO is a function-like MACRO and BAR is an object-like MACRO.
    FOO(BAR);
    return 0;
}
```
###### Fig(1.5)

Here's the preprocessed output of the code in Fig(1.5)

```
# 1 "1.c"
# 1 "<command-line>"
# 1 "/usr/include/stdc-predef.h" 1 3 4
# 1 "<command-line>" 2
# 1 "1.c"



int main()
{

    printf("%d", 5);
    return 0;
}

```
###### Fig(1.6)

```BAR``` is an object-like MACRO which the preprocessor substitutes as 5, according to its definition. Therefore, the MACRO call ```FOO(BAR)``` is now ```FOO(5)```. It then expands the function-like MACRO ```FOO``` with its parameter as 5. You therefore see ```printf("%d", 5);```. Note that the ```printf``` of ```FOO``` does not end with a semicolon. But the MACRO is used with a semicolon ```FOO(...);```. This ensures that printf ends with a semicolon. Why this approach? Its a standard convention to end every statement of a C program with a semicolon and it would be very odd to have some line not ending with it.

### And now for a few tricks

#### Creating debug MACROs

Lot of programmers are used to debug by logging the program's execution. This can be as simple as using ```printf``` all over the code to print messages (on the ```stderr``` buffer) and use this information to debug. Further more, you may want to write pieces of code to mainly check the correctness of the program (only while you're developing it). When its time for you to release your code, you'll have to remove all the logging statements, blocks of code which was used to check correctness, etc. This brings in a lot of work the developer. Additionally, if at some later point of time, the user had to improve his code, he might want to insert all such blocks again (its literally a headache!).  
Now, the preprocessor turns out to be very helpful!

The following program demonstrates three features widely used in C Programs. We'll go through each one by one

```c
#include <stdio.h>
#include <assert.h>

#ifdef CHECK_ENABLED
#define CHECK(X, Y) \
do { printf("Performing Assertion\n"); assert((X - Y) != 0); printf("Assertion passed\n"); } while (0)
#else
#define CHECK(X, Y)
#endif

void foo (int x, int y)
{
    CHECK(x, y);
    printf ("%d: %d\n", __LINE__, (x+y)/(x-y));
}

int main ()
{
    foo (4, 3);
    foo (4, 4);
    return 0;
}


```
###### Fig(1.7)

Let this be a file, *debug.c* .
What is this program trying to do? This program calls the function ```foo()``` with 2 arguments, ```x``` and ```y```.; and foo wants perform the operation ```(x + y) / (x - y)```. It also uses a function-like MACRO ```CHECK``` which expands to an assert statement (forget about the ```do {...} while(0)``` for now). ```assert``` is a library function declared in ```assert.h``` which aborts the program if the given assertion fails (in this example ```(x - y) != 0```). 

According to our expectations, The program and must execute the first ```foo(4, 3)``` call and should fail when trying execute the second ```foo(4, 4)``` call because of division by zero (Forget about the ```__LINE__``` in the printf). Now let me compile it without defining the ```CHECK_ENABLED``` MACRO. Compile with ```gcc debug.c``` and run ```./a.out```; the following is the output.

```
14: 7
Floating point exception (core dumped)

```
###### Fig(1.8)

It worked as expected. The program printed "7" and failed when executing the second ```foo(4, 4)``` call because of the division with zero. Let me now compile with also defining the ```CHECK_ENABLED``` MACRO. Compile with ```gcc debug.c DCHECK_ENABLED``` and run ```./a.out``` and here's the output. 

```
Performing Assertion
Assertion passed
14: 7
Performing Assertion
a.out: 1_preprocessor.c:13: foo: Assertion `(x - y) != 0' failed.
Aborted (core dumped)

```
###### Fig(1.9)

Wow! That's magic! The program did not even try to perform the division because the assertion ```(4 - 4) != 0``` failed and also printed a lot of debugging messages!

Let us now analyze what happened.  
In Fig(1.9), the program was compiled with the ```CHECK_ENABLED``` MACRO defined which included the following controlled text in the program.

```c
#define CHECK(X, Y) \
do { printf("Performing Assertion\n"); assert((X - Y) != 0); printf("Assertion passed\n"); } while (0)
```
###### Fig(1.10) 

This defines a function-like MACRO ```CHECK(X, Y)``` to be ```do { printf("Performing Assertion\n"); assert((X - Y) != 0); printf("Assertion passed\n"); } while (0)```. Hence this function-like MACRO ```CHECK(X, Y);``` expands to ```do { printf("Performing Assertion\n"); assert((X - Y) != 0); printf("Assertion passed\n"); } while (0)```. With ```foo(4, 3);``` this assertion succeeds and prints "7"; with ```foo(4, 4");``` this assertion fails and the program aborts.

When the program is compiled without defining ```CHECK_ENABLED``` MACRO [refer Fig(1.8)], the controlled text in the else portion of the conditional group gets included in the program. ie

```c
#define CHECK(X, Y)
```
###### Fig(1.11)

This defines a function-like MACRO ```CHECK(X, Y)``` to nothing! So the function-like MACRO expands to nothing! ie the assertion is not in the code any more and so the program prints "7" and gets a floating point exception when trying a division with zero.

So what just happened? In Fig(1.9) the program was executed in a debugging mode; the debugging mode in the above example was enabled by defining the ```CHECK_ENABLED``` MACRO; and in this debugging mode, the assertion statement was included in the code and the program aborted stating where it failed and why. The debugging mode also included some debugging messages which I printed so that I'd know what went wrong and where.  
When the debugging mode wasn't enabled (by not defining the ```CHECK_ENABLED``` MACRO), the assertion wasn't included in the code by the preprocessor as ```CHECK(X, Y)``` expands to nothing. Now the program crashes.  
Look at the error that comes up - ```Floating point exception (core dumped)```. You cannot make out what happened where! The runtime threw an error message and exited the program. Whereas when the debugging is enabled, the assertion gives a better error message - ```a.out: 1_preprocessor.c:13: foo: Assertion `(x - y) != 0' failed.```. It failed in line 13. That's where the function-like MACRO ```CHECK(x, y)``` expands.  

**Feature/Trick #1** : In short, you can now run your program in debug mode and release mode by just defining / not-defining a debug MACRO. Normally this debug MACRO is ```DEBUG```. I've used ```CHECK_ENABLED``` in the example to include some checks. You must also understand now that in the release mode, we only concentrate on execution of the program and not error checking because they can make the program slower. Imagine calling the ```foo(4, 3)``` a million times in the debug mode. The assert function is called a million times but your intention is to check the correctness of the code and not the execution speed. In the release mode, the assertion is not even there and so your program executes much faster and is assumed to be correct.  
Nice trick huh? Such an amazing work around to solve such an important problem and that too with just using the C Preprocessor!

I mentioned long back that there are three features the above program [Fig(1.7)] tries to show. The first was the debug/release switch which we have covered in detail. The second is this very odd statement -

```c
do { printf("Performing Assertion\n"); assert((X - Y) != 0); printf("Assertion passed\n"); } while (0)

```
###### Fig(1.12)

**Feature/Trick #2** : Why the ```do { ... } while(0)``` ??! Why can't I just write a set of statements? Maybe because I wanted to group them. Then why couldn't I just use ```{ ... }``` instead of the do while zero?  
Consider this situation

```c
#define FOO(X, Y) printf("%s\n", X); printf("%s\n", Y)

int main ()
{
    if (/*some condition*/)
        FOO("YES", "TRUE");
}

```
###### Fig(1.13)

Remember that its a standard convention not to end the code within a function-like MACRO with a semicolon (because the MACRO is used with a semicolon at the end)? So the second ```printf``` doesn't end with a semicolon, its taken care of later.  
An if condition is associated only with the statement/block following it. After the function-like MACRO ```FOO(X, Y)``` expands, we have something like the following -

```c
if (/*some condition*/)
        printf("%s\n", "YES"); printf("%s\n", "TRUE");
```
###### Fig(1.14)

Note that the first printf gets associated with the if condition and not the second! So "TRUE" is always printed. What a bug! So to avoid this let me rewrite the program as follows -

```c
#define FOO(X, Y) { printf("%s\n", X); printf("%s\n", Y) }

int main ()
{
    if (/*some condition*/)
        FOO("YES", "TRUE");
    else
        FOO("NO", "FALSE");
}

```
###### Fig(1.15)

Notice that I've now included flower brackets to group the two printfs into a block and I also wanted an else portion which I've included. This sounds fine. Lets now observe what happens when the function-like MACRO ```FOO(X, Y)``` expands.

```c
if (/*some condition*/)
    { printf("%s\n", "YES"); printf("%s\n", "TRUE") };
else
    { printf("%s\n", "NO"); printf("%s\n", "FALSE") };
```
###### Fig(1.16)

Observe that the second printf doesn't end with a semicolon but the blocks end with a semicolon, because the semicolon was present after the MACRO. This will through a bunch of errors because printf doesn't have a semicolon to start with and there's a semicolon after the if block (which ends the if block) so the else portion becomes stray code without a corresponding if! Uff! This is crazy! Well, no. Here is why do while zero is used. Lets rewrite the program using do while zero

```c
#define FOO(X, Y) do { printf("%s\n", X); printf("%s\n", Y); } while(0)

int main ()
{
    if (/*some condition*/)
        FOO("YES", "TRUE");
    else
        FOO("NO", "FALSE");
}

```
###### Fig(1.17)

When the MACRO expands, we get it like this -

```c
if (/*some condition*/)
    do { printf("%s\n", "YES"); printf("%s\n", "TRUE"); } while(0);
else
    do { printf("%s\n", "NO"); printf("%s\n", "FALSE"); } while(0);
```
###### Fig(1.18)

You can now see that the if block contains one statement which is a loop. All the statements are placed in that loop and this loop executes only once because the condition fails. Isn't that amazing?!

**Feature/Trick #3** : And lastly, the third feature the program uses is ```printf("%d", __LINE__);```. ```__LINE__``` is a preprocessor MACRO that expands to current line number in the source file, as an integer. ```__LINE__``` is useful when generating log statements, debug messages, etc. There's also ```__FILE__``` which expands to the current file name.

**Feature/Trick #4** : Consider the following header file (say *file.h*)

```c
struct foo {
    int a;
    char b;
};

```
###### Fig(1.19)

Now if your project contains a lot of files and you include this header file in many of them and compile, you'll encounter errors of redeclaration! And so header files begin with the following construct 

```c
#ifndef FILE_H
#define FILE_H

struct foo {
    int a;
    char b;
};

#endif
```
###### Fig(1.20)

So what's the change now? When some .c file which includes this header is being compiled, it sees the directive ```#ifndef FILE_H ... #endif```; which basically means include the controlled text if the MACRO ```FILE_H``` is not defined. But wait, what is this present in the controlled text? ```#define FILE_H```. What I'm doing is defining the MACRO ```FILE_H``` and including the header contents if ```FILE_H``` was previously not declared. Now when another file is being compiled which includes the same header, the compiler sees that the MACRO ```FILE_H```  is defined now and skips including the controlled text of the conditional group avoiding redeclaration of structures and other entities of a header file. 

**Feature/Trick 5** : This is the last preprocessor feature I'll be discussing! The C preprocessor gives two directives ```#warning``` and ```#error```. I expect you might have guessed their uses. Consider the following code snippet - 

```c
#ifdef USE_OLD_FOO
#warning "Using old foo(). This is deprecated! You are getting this message because USE_OLD_FOO is defined"
void foo()
{
    //This implementation is no longer recommended for use. 
    /*
    Some implementation
    */
}
#else
void foo()
{
    //A better foo implementation
    /*
    Some implementation
    */
}
#endif

```
###### Fig(1.21)

The above snippet includes the old version of the function ```foo()``` if ```USE_OLD_FOO``` is defined, else by default the new version would be used. One may need to use old versions of codes for legacy support (say). But when using the old version, the developer would like to warn the user about it. In such cases, you can use the ```#warning``` directive. The ```#errror``` is similar, but throws an error

---
## Swapping two variables

**Feature/Trick 6** : This is more a programming trick than a trick in C. To swap two variables without using additional space or arithmetic operators, you can simply use the xor operator; like so -

```c
a = a ^ b;
b = a ^ b;
a = a ^ b;
```
###### Fig(2.1)

---
## Pointers in C 

### Arrays and Pointers not entirely the same!

**Feature/Trick 7** : One of the heavily misunderstood concepts of C is that pointers and arrays are the same. They are not. Pointers are merely variables holding the address of some location where as an array is conceptualized as a sequence of memory locations of a type. At compile time, an array is an array. Only during runtime, an array degenerates to a pointer. To prove this fact, let me show you an example -

```c
int a[10] = { 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 };
int *b = a;

printf("%d\n%d\n", sizeof(a), sizeof(b));
```
###### Fig(3.1)

And the output is (assuming size of int is 4 bytes and address size is 8 bytes) - 

```
40
8
```
###### Fig(3.2)

See what happens? At compile time, the compiler has information regarding the array. It drops all the information in the end and so at runtime, an array acts like a constant pointer.

### ```v[index] and index[v]``` are the same

**Feature/Trick 8** : Yes arrays and pointers are not the same, but are interpreted in the same way; by dereferencing them to get the value. The indexing operation ```v[index]``` where ```v``` is some array and ```index``` is some index is internally converted to ```*(v + index)```. The funny thing now is that if you use this as ```index[v]```, its converted to ```*(index + v)``` and both mean the same!

---
## Standard I/O

**Feature/Trick 9** :You are no doubt familiar with ```scanf``` and ```printf``` functions. You also be familiar with ```fscanf``` and ```fprintf``` if you have worked on file handling. 
For example - 

```c
FILE *fptr = fopen("some_file.txt", "r");
char* first_name; int age;

fscanf(fptr, "%s %d", first_name, &age);
printf("Name: %s\nAge: %d\n\n", first_name, age);

fclose(fptr);
```
###### Fig(4.1)

The above snippet demonstrates reading from a file using ```fscanf```.  
How about this? You have a string containing two integers and you'd like to read from that. Its not a file name, its a character array. C provides reading and writing to character arrays (or C strings) with the use of ```sscanf``` and ```sprintf```.  
Let me demonstrate this - 

```c
char str[20] = "10 20";
int a, b;

sscanf(str, "%d %d", &a, &b);
sprintf(str, "%d %d", a*a, b*b);
printf("%s", str);
```
###### Fig(4.2)

In the above snippet, a string contains two numbers 10 and 20 (as characters of the string). Using sscanf you can read these to variables and also write back to the C string!

**Feature/Trick 10** : ```scanf``` is probably one of the most magical function in C. To know more about the tricks in ```scanf```, visit [my answer in quora](http://qr.ae/IHQTN) about the same.

**Feature/Trick 11** : One of the many problems with C strings is reading them; C strings can easily overflow if a string longer than size of the character array is entered. ```scanf``` is useless and so is ```gets```. The best function to use when reading strings is ```fgets```. ```fgets``` takes the string, the length to be read and the file stream where the string is to be read from as arguments. If you want to read from the console, you can use ```stdin``` as your input file stream. The function call would then go like this - 

```c
int n; //some length; maybe known later.
char str[n]; //works from c99

fgets(str, n, stdin); //reads max n characters to 'str' from stdin.
```
###### Fig(4.3)

You can also use the new ```gets_s``` function; but ```fgets``` is pretty much the standard convention.


---
## C99, C11 and misc

You've been using the old fashioned way of initializing C structures right? Like so -

```c
struct foo
{
    int a;
    int b;
};

int main()
{
    struct foo f = {100, 123};
    printf("%d %d\n", f.a, f.b);

    return 0;
}
```
###### Fig(5.1)

**Feature/Trick 12** : C99 provides a new way of doing the same with more ease. Like so -

```c
struct foo
{
    int a;
    int b;
};

int main()
{
    struct foo f = {.b = 123, .a = 100}; // C99 style
    printf("%d %d\n", f.a, f.b);

    return 0;
}
```
###### Fig(5.2)

**Feature/Trick 13** : C99 also provides a mechanism to initialize only certain indices of an array. 

```c
int a[100] = {1, [50] = 1}; //initializes a[0] and a[50] to 1 and the rest 0
printf("%d %d %d %d\n", a[0], a[1], a[50], a[51]);
```
###### Fig(5.3)

**Feature/Trick 14** : Did you know about the ```atexit``` function? This function can be used to register functions that are to be called when the program finishes its execution! For example - 

```c
#include <stdio.h>
#include <stdlib.h>

void foo(void)
{
    printf("Goodbye Foo!\n");
}

void bar(void)
{
    printf("Goodbye Bar!\n");
}

int main(int argc, wchar_t* argv[])
{
    atexit(bar);
    atexit(foo);
    return 0;
}

```
###### Fig(5.4)

Notice that ```foo``` and ```bar``` functions haven't been called but are registered to be called when the program exits. Such functions should not return anything nor accept any arguments. You can register upto 32 such functions. They'll be called in the LIFO order.

**Feature/Trick 15** : I found this somewhere in stackoverflow. It seems that a game the developer had to initialize a huge two dimensional float array with lots of values. And he employed this to do so - 

```c
double array[SIZE][SIZE] = {
    #include "float_values.txt"
}
```
###### Fig(5.6)

The file *float_values.txt* contains all the values to be initialized in the array. This way, the values are not inlined in the file containing the code. The preprocessor will have included that file. Nice idea!

**Feature/Trick 16** : C11 has introduced something great! Heard of templates in C++ where you could write type independent code? Well, that's possible in a slightly different way in C11 now! Want to see how? Scroll down a little.

```c
#include <stdio.h>
#include <stdlib.h>

void printchar(char X) { printf("char: %c\n", X); }
void printint(int X) { printf("int: %d\n", X); }
void printfloat(float X) { printf("float: %f\n", X); }
void printdouble(double X) { printf("double: %lf\n", X); }
#define print(X) _Generic((X), \
                               char: printchar, \
                               int: printint, \
                               float: printfloat, \
                               double: printdouble)(X)

int main()
{
    print (1);
    print (1.2);
    print ((char)'a');
    return 0;
}

```
###### Fig(5.7)

The above program illustrates a generic ```print``` function which adapts to the type of the argument sent! How is this achieved? C11 introduced the ```_Generic``` keyword. This maps the type and the corresponding function that should be called for the type. You can abstract this to a function-like MACRO. In the above program, a function-like macro ```print(X)``` is defined to expand to the ```_Generic``` construct. This takes in as its first argument ```_Generic((X), ...```as the parameter and the rest map the type to the function that needs to be called ```_Generic((X), char: printchar, int: printint, ...```; in the end this construct is invoked like a function by passing the value ```...double: printdouble)(X)```. The abstract function-like MACRO can be used to call this type-generic function and that's cool!

That's it folks! Hope you enjoyed reading this!
