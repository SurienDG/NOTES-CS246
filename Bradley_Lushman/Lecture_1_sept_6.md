# CS246 - Lecture 1 - Sept 6, 2018

## Module 1: Linux Shell

shell: interface to the OS - gte OS to run program/manage files/etc

- graphical {click/touch}
- command-line-type commands at a prompt
- more versatile

This course: bash (command line)

Make sure you\re running bash

login and type `echo $0` should say bash

if not:
www.student.cs.uwaterloo.ca/password

### Files, Input, Output

`cat` - displays the contents of a file

![catFile Diagram](Images/cs246_Sept6_catFile.jpg "catFile")

In linux a directory is just a special kind of file.

In Linux, every line of a valid test file must end with a newline character, INCLUDING THE LAST ONE!

Marmoset checks for this!

^C (i.e. CTRL + c) to stop (kills the program)

`ls` - lists files in the current directory (non-hidden files)

`ls - a` **all** files (incl. hidden)

### Hidden files
- name starts with a dot (.)

`pwd` - prints current directory

What if you just type `cat`? 
- Nothing happens
- `cat` is waiting for input - type something

Now, it prints every you type

**Usefule?** Maybe, if we can capture the output in a file ...

Observe:
 ```Bash
 cat > myfile.txt
 some words
 some more words
 ```

To stop: CTRL - D (^D) at the beginning of a line send a end of file (EOF) signal to cat.

Now `cat myfile.txt` outputs what we typed

In general:
```Bash
command arg 

