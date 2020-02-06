# CPSC/ECE 3200: Introduction to Operating System - Project #2

In this project, you will work on two programs that help you (1) to be familiar with the concepts of process management and system calls; (2) to understand the functionalities and internals of the shell; and (3) to gain hand-on experience of writing system programs.

# Task A: Concurrent Prime Sieve

<code>Write a concurrent version of prime sieve using pipes.</code>

## Overview

Your goal is to write a concurrent version of prime sieve program using pipes. In mathematics, the Sieve of Eratosthenes is an ancient algorithm for finding all prime numbers up to any given limit. A classical Sieve program finds all prime numbers up to a given integer *N*
by repeatdely dropping the multiples of newly founded prime numbers. If you are unfamiliar with the algorithm, you can read its wikipage at https://en.wikipedia.org/wiki/Sieve_of_Eratosthenes.

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




to use pipe and fork to set up the pipeline. The first process feeds the numbers 2 through 35 into the pipeline. For each prime number, you will arrange to create one process that reads from its left neighbor over a pipe and writes to its right neighbor over another pipe. Since xv6 has limited number of file descriptors and processes, the first process can stop at 35.

## Specification
