# CPSC/ECE 3200: Introduction to Operating System - Project #2

In this project, you will work on two programs that help you (1) to be familiar with the concepts of process management and system calls; (2) to understand the functionalities and internals of the shell; and (3) to gain hand-on experience of writing system programs.

# Task A: Concurrent Prime Sieve (30 Points)

<code>Write a concurrent version of prime sieve using pipes. The program shall run on a Linux machine 
in the School of Computing computer labs. </code>

## Overview

Your job for this task is to write a concurrent version of prime sieve program using pipes. In mathematics, the Sieve of Eratosthenes is an ancient algorithm for finding all prime numbers up to any given limit. A classical Sieve program finds all prime numbers up to a given integer *N*
by repeatedly dropping the multiples of newly founded prime numbers. If you are unfamiliar with the algorithm, you can read its wikipage at https://en.wikipedia.org/wiki/Sieve_of_Eratosthenes.

The idea of conncurent prime sieve is due to Doug McIlroy, inventor of Unix pipes. The following figure and text borrowed from the page (https://swtch.com/~rsc/thread/) created by Russ Cox, one of the XV6 authors.



A generating process can feed the numbers 2, 3, 4, ..., N into the left end of the pipeline: the first process in the line eliminates the multiples of 2, the second eliminates the multiples of 3, the third eliminates the multiples of 5, and so on:

![Illustration of Concurrent Sieve Using Pipes](./sieve.gif)

In your program, you are required to use pipe and fork to set up the pipeline. The length of the pipeline depends on N, which is passed into the program as its first argument. 
The first process feeds the numbers 2 through N into the pipeline. 
For each prime number, you will arrange to create one process that reads from its left neighbor over a pipe and writes to its right neighbor over another pipe. 

## Specification

1. You provide a program `primes.c` and put it into the folder `TaskA`.
1. Invoking the command `make` shall create a executable binary `primes`, which takes a single positive integer *N* which is larger than 2. 
1. You can assume the test will use a reasonable *N* such that your program will complete successfully on a system with a limited number of file descriptors and processes. 
1. The program `primes` shall output all primes from 2 through *N* with the following format:
```
pid=<pid of of the process that drops of multiples of 2> prime 2
pid=<pid of of the process that drops of multiples of 3> prime 3
...
pid=<pid of of the process that drops of multiples of p> prime p 
```
Here *p* is the largest prime number smaller than or equal to *N*.
1. The program must stop after it output the last prime number.

### Example Output
Your program shall generate output like the following examples except that 
your program may output different PIDs for the processes. 
```
$ make
gcc -o primes primes.c
$ ./primes 6
pid=70819 prime 2
pid=70820 prime 3
pid=70821 prime 5
$ ./primes 30
pid=70829 prime 2
pid=70830 prime 3
pid=70831 prime 5
pid=70832 prime 7
pid=70833 prime 11
pid=70834 prime 13
pid=70835 prime 17
pid=70836 prime 19
pid=70837 prime 23
pid=70838 prime 29
```

## Hints

+ Because the program could not know the length of 
the pipelines, the program may need some kind of **recursion** 
and must decide when a process should fork a new process 
to be its right neighbor. The body of the recursion has the following logic. You shall also build pipe and properly exit and wait. 

```
prime_sieve()

...
p = get a number from left neighbor
print p
loop:
    n = get a number from left neighbor
    if (p does not divide n)
       if ( not has_right_neighbor)
            create right neighbor process
            if (right neighbor process is newly created)
                prime_sieve()
       send n to right neighbor
```

+ Each process must determine when there wil be no more data from 
its left neighbor.
+ You can use the first process to generate all the integers 
from 2 through N and there is no need to 
store them in an array.
+ You can read and write an integer in binary format as follows:

<pre>
read(fd, &num, sizeof(num));
write(fd, &num, sizeof(num));
// OR when num is of type int 
read(fd, &num, sizeof(int));
write(fd, &num, sizeof(int));
</pre>

# Task B: Simple xv6 Shell (70 Points)
<code>Write a simple shell for XV6. The program shall run on the xv6-riscv OS in the QEMU environment    
in the School of Computing computer labs. </code>

## Overview

Your job for this task is to write a simple shell for xv6. A shell is a a command line interpreter (CLI) that accepts user commands 
and then executes each command in one or more separate processes. Your shell shall be able to:
 
1. run commands with arguments, 
1. handle input and output redirection,
1. set up a k-element pipelines, where k is within the range of 2 through 5,
1. run a list of commands in sequence, and 
1. read commands from a text file and execute the commands in the same way as they are read from the console.

## Specification

### Basic Shell
1. Your basic shell, called tsh (Tiger Shell), is basically an interactive loop: it repeatedly prints a
       prompt `tsh> ` (note the space after the greater-than sign), parses the input, executes
       the command specified on a line of input, and waits for the command to finish. This process is
       repeated until the user types the `exit` command.
1. You should put your shell source in user/tsh.c, and modify the Makefile to compile it.
1. An xv6 session with your shell might look like this:
    ``` 
    xv6 kernel is booting
    $ tsh
    tsh> grep Copyright < README
    tsh> exit
    $
    ```
 1. You should structure your shell such that it creates a new process for each 
 external command (an external command that is implemented as a separate program. 
 In contrast,  a built-in command must be executed in the same process as your 
 current shell session). Running a command in a separate process has several advantages.
 First, it protects the main shell process from any error that may occur in the user's 
 command. Second, it allows to run multiple commands concurrently.
1. Your can assume that given an external command, either there is a program in current 
 search path corresponding to the command or there is no such program. For the first case, 
 execute the command as the shell normally does. For the second case, prints an error message
 "An error has occurred" and then proceeds to next iteration. 
 Note that a normal shell like *Bash* will search the executable in a list of directories 
 specified in the path variable and execute the first executable found. We make this 
 simplification because we are using a primitive version of the xv6 and hasn't studied 
 file system yet.  For example,
    ```
    tsh> grep Copyrigh README
    Copyright 2006-2019 Frans Kaashoek, Robert Morris, and Russ Cox.
    tsh> unimplemented_command
    An error has occured
    tsh> 
    ```
### Built-in Commands
1. Whenever your shell accepts a command, it should check whether the command is a built-in
command or an external command. If the command is a built-in command, your shell shall 
invoke the function in your implementation corresponding to such built-in command. 
1. While most Unix shells have numerous built-in commands, in this project, you shall implement
**exit** and **cd**. The syntax and semantics of the 
three commands are described as follows.
1. **exit [n]** 
   * Cause the shell to exit with a status of n.
   * If n is omitted, the exit status is that of the last command executed.
1. **cd [dir]**
   * Change the current directory to dir.
   * If dir is not supplied, the value of the HOME shell variable is the default.
   * If dir does not exit, prints an error message "An error has occurred" and then 
        proceeds to next iteration.
<!--
1. **pwd**
   * Print the absolute pathname of the current working directory.

1. **set [var=value]**
   * Assign `value` to the shell variable `var`.
   * If the set command has no argument, print all shell variables at the time the command is executed.
   * To simplify the implementation, you can assume:
      1. no more than five variables will be defined; 
      1. both the name and value of a variable are string type; 
      1. a variable name has less than 16 characters; and 
      1. the value of a variable has less than 64 characters.      
-->
### Command
1. You can assume all commands will be provided in a single command line.
1. Each command line has less than 128 characters.
1. Each command line has less than 5 simple commands, separated by either '|' or ';' but not both.
1. A simple command is a command that consists of a word followed by a list of arguments and 
an optional IO redirection.
1. Each simple command has no more than 5 arguments.
1. If a command line contains multiple simple commands separated by '|', then only the first simple 
   command can redirect its standard input and 
   only the last simple command can redirect its standard output.

### Redirection
Many times, a shell user prefers to send the output of a program to a file rather than
to the screen or to read the input to a program from a file rather than from the console. 
This feature is called I/O redirection. You shell will implement a simplified version of I/O redirection.
1. syntax: **command [argument ...] < input**
     * Redirect standard input to file input
     * If the file input does not exist, prints an error message "An error has occurred" and then proceeds to next iteration.
1. syntax: **command [argument ...] > output**
    * Redirect standard output to file output, new file mode.
    * If the file output already exists, that file will be overwritten; otherwise it will be created.
1. syntax: **command [argument ...] >> output**
     * Redirect standard output to file output, appending mode.
     * If the file output already exists, the output from command will append to that file; otherwise it will be created.   
1. syntax: **command [argument ...] 2> error**
    * Redirect standard error to file error, new file mode.
    * If the file error already exists, that file will be overwritten; otherwise it will be created.
1. syntax: **command [argument ...] 2>> error**
     * Redirect standard error to file output, appending mode.
     * If the file error already exists, the error from command will append to that file; otherwise it will be created.
1. You don't need to implement I/O redirection for internal commands.

### List commands
1. A list command is a list of simple commands separated by ';'. For example,
    ```
    tsh> echo a > b; wc b; wc README
    1 1 2 b
    43 286 1982 README
    tsh> 
    ```
1. Given a list command, you shell should execute each simple command included in the list command in the order 
from left to right.

### Pipeline commands
1. A pipeline command is a list of commands connected by '|'. For example, 
    ```
    tsh> cat README | grep Copy | wc
    1 9 65
    tsh>
    ```
1. For a pipeline command, a command reads input from its left neighbor and writes output to its right neighbor when 
such neighbor exists. The leftmost process does not have a left neighbor and the rightmost neighbor does 
not have a right neighbor.
1.  Only the leftmost process can redirect its standard input from a file.
1.  Only the rightmost process can redirect its standard output to a file.

### Batch mode execution
1. Your shell should be able to read the commands from a batch script file and execute the commands in sequence. For example, 
   assume the file `batch.sh` contains a command line `cat README | grep Copy | wc`, you can run the script as follows.
    ```
    tsh> tsh batch.sh 
    1 9 65
    tsh>
    ```
1. A straightforward implementation of batch mode execution is to execute `tsh` in a new process and redirect the 
standard input of such process to the batch script file.  

## Test

### Test cases

You can use the following test cased during your development.
<pre>
    echo hello there
    echo something > file.txt
    ls | grep READ
    grep lion < data.txt | wc > count
    echo echo hello | nsh
    find . b | xargs grep hello
 </pre>
### Test program
The xv6 authors have provided an xv6 test program testsh, source in user/testsh.c. You can use the program 
to check if your shell can passes all the tests, like this:
 <pre>
    $ testsh tsh
    simple echo: PASS
    simple grep: PASS
    two commands: PASS
    output redirection: PASS
    input redirection: PASS
    both redirections: PASS
    simple pipe: PASS
    pipe and redirects: PASS
    lots of commands: PASS
    passed all tests
</pre>
