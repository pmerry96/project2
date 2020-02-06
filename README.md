# CPSC/ECE 3200: Introduction to Operating System - Project #2

In this project, you will work on two programs that help you (1) to be familiar with the concepts of process management and system calls; (2) to understand the functionalities and internals of the shell; and (3) to gain hand-on experience of writing system programs.

# Task A: Concurrent Prime Sieve (30 Points)

<code>Write a concurrent version of prime sieve using pipes. The program shall run on a Linux machine 
in the School of Computing computer labs. </code>

## Overview

Your goal for this task is to write a concurrent version of prime sieve program using pipes. In mathematics, the Sieve of Eratosthenes is an ancient algorithm for finding all prime numbers up to any given limit. A classical Sieve program finds all prime numbers up to a given integer *N*
by repeatedly dropping the multiples of newly founded prime numbers. If you are unfamiliar with the algorithm, you can read its wikipage at https://en.wikipedia.org/wiki/Sieve_of_Eratosthenes.

The idea of conncurent prime sieve is due to Doug McIlroy, inventor of Unix pipes. The following figure and text borrowed from the page (https://swtch.com/~rsc/thread/) created by Russ Cox, one of the XV6 authors.

```
p = get a number from left neighbor
print p
loop:
    n = get a number from left neighbor
    if (p does not divide n)
        send n to right neighbor
```

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
the pipelines, the program may need some kind of recursion 
and must decide when a process should fork a new process 
to be its right neighbor.
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