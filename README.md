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
We obviously have a stack buffer overflow on the echo variable which can give us EIP control when main returns. But we don't have a give_shell function! So what can we do?

We can call system with an argument we control! Since arguments are passed in on the stack in 32-bit Linux programs (see calling conventions), if we have stack control, we have argument control.

When main returns, we want our stack to look like something had normally called system. Recall what is on the stack after a function has been called:

```cmd
        ...                                 // More arguments
        0xffff0008: 0x00000002              // Argument 2
        0xffff0004: 0x00000001              // Argument 1
ESP ->  0xffff0000: 0x080484d0              // Return address
```

So `main's` stack frame needs to look like this:

```cmd
        0xffff0008: 0xdeadbeef              // system argument 1
        0xffff0004: 0xdeadbeef              // return address for system
ESP ->  0xffff0000: 0x08048450              // return address for main (system's PLT entry)
```

Then when main returns, it will jump into system's PLT entry and the stack will appear just like system had been called normally for the first time.

Note: we don't care about the return address system will return to because we will have already gotten our shell by then!

Arguments
This is a good start, but we need to pass an argument to system for anything to happen. As mentioned in the page on ASLR, the stack and dynamic libraries "move around" each time a program is run, which means we can't easily use data on the stack or a string in libc for our argument. In this case however, we have a very convenient name global which will be at a known location in the binary (in the BSS segment).

Putting it together
Our exploit will need to do the following:

Enter "sh" or another command to run as name
Fill the stack with
Garbage up to the saved EIP
The address of system's PLT entry
A fake return address for system to jump to when it's done
The address of the name global to act as the first argument to system

