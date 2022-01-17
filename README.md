## Return Oriented Programming-ROP

Return Oriented Programming (or ROP) is the idea of chaining together small snippets of assembly with stack control to cause the program to do more complex things.

As we saw in buffer overflows, having stack control can be very powerful since it allows us to overwrite saved instruction pointers, giving us control over what the program does next. Most programs don't have a convenient give_shell function however, so we need to find a way to manually invoke system or another exec function to get us our shell.

## 32 bit

Imagine we have a program similar to the following:

```C
#include <stdio.h>
#include <stdlib.h>

char name[32];

int main() {
    printf("What's your name? ");
    read(0, name, 32);

    printf("Hi %s\n", name);

    printf("The time is currently ");
    system("/bin/date");

    char echo[100];
    printf("What do you want me to echo back? ");
    read(0, echo, 1000);
    puts(echo);

    return 0;
}
```
