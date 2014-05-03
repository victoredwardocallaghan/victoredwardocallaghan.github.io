---
title: Hash functions in Cryptol 2
published: May 4, 2014
excerpt: Various hash functions implemented in the Cryptol 2 Language
tags: Cryptol, FP, hash
---

* toc

Hash Functions in Cryptol 2
===========================

..

---


Cyclic Redundancy Checks
========================

Summary
-------

Name    Length    Type
----    ------    ----
crc16   16 bits   CRC
crc32   32 bits   CRC
crc64   64 bits   CRC

The selection of generator polynomial is the most important part of
implementing the CRC algorithm. The polynomial must be chosen to maximize the
error-detecting capabilities while minimizing overall collision probabilities.

Suppose the generator polynomial is $g(x) = p(x)(1 + x)$, where $p(x)$ is a
primtive polynomial of degree $r-1$, then the maximum total block length is
$2^{r-1} - 1$. The code is then able to detect single, double, triple and any
odd number of errors.

Table of polynomial representations of common CRC's
---------------------------------------------------

Name            Polynomial                                                                                                       Numerical Bit Representation
----            ----------                                                                                                       ----------------------------
CRC16-CCITT     $x^{16} + x^12 + x^5 + 1$                                                                                        0x1021
CRC-32          $x^{32} + x^{26} + x^{23} + x^{22} + x^{16} + x^{12} + x^{11} + x^{10} + x^8 + x^7 + x^5 + x^4 + x^2 + x + 1$    0x04C11DB7
CRC-64-ISO      $x^{64} + x^4 + x^3 + x + 1$                                                                                     0x000000000000001B


Checksums
=========

Summary
-------

Name          Length    Type
----          ------    ----
xor-8         8  bits   sum
fletcher-4    4  bits   sum
fletcher-8    8  bits   sum
fletcher-16   16 bits   sum
fletcher-32   32 bits   sum
Adler-32      32 bits   sum


Non-Cryptographic Hash Functions
================================

Summary
-------

Name                    Length        Type
----                    ------        ----
Pearson hashing         8 bits        xor/table
Zobrist hashing         variable      xor
Bernstein hash          32 bits       -
Jenkins hash function   32/64 bits    xor/addition


Cryptographic Hash Functions
============================

Summary
-------

Name            Length        Type
----            ------        ----
SHA-3 (Keccak)  arbitrary     Sponge function
Skin            arbitrary     Unique Block Iteration


Modified Bernstein's Hash
-------------------------

Bernstein's hash replaces addition with XOR for the combining step.

~~~ {.haskell}
 (==)   :  {a}   (Cmp a) => a -> a -> Bit
 (!=)   :  {a}   (Cmp a) => a -> a -> Bit
 (===)  :  {a,b} (Cmp b) => (a -> b) -> (a -> b) -> a -> Bit
 (!==)  :  {a,b} (Cmp b) => (a -> b) -> (a -> b) -> a -> Bit

 (<)    :  {a} (Cmp a) => a -> a -> Bit
 (>)    :  {a} (Cmp a) => a -> a -> Bit
 (<=)   :  {a} (Cmp a) => a -> a -> Bit
 (>=)   :  {a} (Cmp a) => a -> a -> Bit

 min    :  {a} (Cmp a) => a -> a -> a
 max    :  {a} (Cmp a) => a -> a -> a

 instance Cmp Bit
 // No instance for functions.
 instance (Cmp a, fin n) => Cmp [n] a
 instance (Cmp a, Cmp b) => Cmp (a,b)
 instance (Cmp a, Cmp b) => Cmp { x : a, y : b }
~~~
