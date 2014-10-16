---
title: Hash functions in Cryptol 2
published: May 4, 2014
excerpt: Various hash functions implemented in the Cryptol 2 Language
tags: Cryptol, FP, hash
---

* toc

Hash Functions in Cryptol 2
===========================

Let us begin with some basic defintions:

<div class="callout">
**Defintion**: A **hash function** is any function that maps data of arbitrary
length to data of a fixed length.
</div>

More formally, we can express this with a **type-signature** in Cryptol's
polymorphic type system in the following way:

~~~ {.haskell}
 hashFunction   :  {n,a}  (n >= 0) => a -> n -> [n]
~~~

What this says is that, the function **hashFunction** takes an arbitrary input
of polymorphic type **a**, a length **n bits** and returns an stream of **fixed
length n bits** with the co-domain constraint that **n** must be greater than
or equal to zero.

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

Table of common CRC polynomials
-------------------------------

Name            Polynomial                                                                                                       Numerical Bit Representation
----            ----------                                                                                                       ----------------------------
CRC16-CCITT     $x^{16} + x^12 + x^5 + 1$                                                                                        0x1021
CRC-32          $x^{32} + x^{26} + x^{23} + x^{22} + x^{16} + x^{12} + x^{11} + x^{10} + x^8 + x^7 + x^5 + x^4 + x^2 + x + 1$    0x04C11DB7
CRC-64-ISO      $x^{64} + x^4 + x^3 + x + 1$                                                                                     0x000000000000001B

Polynomials in $GF(2^k)$ for some $k$
-------------------------------------

In Cryptol we have the following notation to representation a polynomial
algebraically:

~~~ {.haskell}
 crc8  = <| x^^8 + x^^2 + x + 1 |>
 crc32 = <| x^^32 + x^^26 + x^^23 + x^^22 + x^^16 + x^^12 + x^^11 + x^^10 + x^^8 + x^^7 + x^^5 + x^^4 + x^^2 + x + 1 |>
 crc64 = <| x^^64 + x^^4 +x^^3 + x + 1 |>
~~~

A Cryptol implementation by **vanila** is given:

~~~ {.haskell}
 module crc where

 crc1 : [2]
 crc1 = <| x + 1 |> // parity test

 crc8 : [9]
 crc8  = <| x^^8 + x^^2 + x + 1 |>

 crc32 : [33]
 crc32 = <| x^^32 + x^^26 + x^^23 + x^^22 + x^^16 + x^^12 + x^^11 + x^^10 + x^^8 + x^^7 + x^^5 + x^^4 + x^^2 + x + 1 |>

 crc poly text = pmod (join text) poly
~~~

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

Adler-32
--------

Adler-32 is a checksum algorithm which was invented by Mark Adler in 1995, and
is a modification of the Fletcher checksum.

Adler-32 is defined in the following way; Suppose some message **M** is of
length **n bytes**, then the Adler-32 hash is:

$$
adler32(M) = a(M) + b(M) \mod 65521
$$

where the functions $a$ and $b$ are defined as recursion relations:

$$
\begin{align}
 a_k &= a_{k-1} + m_k \\
 b_k &= b_{k-1} + a_k
\end{align}
$$

under the constraint $1\leq k \leq n$ and initial conditions $a_0 = 1 + m_0$
and $b_0 = a_0$, where each $m_i$ are the 8bit representation of each character
in a given message string **M**. Here is a tabulation of the behaviour:

Input   $a(m)$                $b(m)$
-----   ------                ------
$m_0$   $1 + m_0 = a_0$       $0 + a_0 = b_0$
$m_1$   $a_0 + m_1 = a_1$     $b_0 + a_1 = b_1$
$m_2$   $a_1 + m_2 = a_2$     $b_1 + a_2 = b_2$
.       .                     .
.       .                     .
.       .                     .
$m_n$   $a_{n-1} + m_n = a_n$     $b_{n-1} + a_n = b_n$

We can then directly implement the Cryptol implementation as two recursive list
comprehessions which are read modulo prime $p=65521$ as the computed as the
Adler-32 checksum as follows:

~~~ {.haskell}
module adler32 where

adler32 : {n} (fin n) => [n][32] -> [32]
adler32 message = (as!0)%p + (bs!0)%p * 65536
 where
  p = 65521
  as = [1] # [a + m | a <- as | m <- message]
  bs = [0] # [a + b | a <- tail as | b <- bs]

// Test:
//adler32 [ zero # c | c <- "Wikipedia" ]
//300286872
~~~

<div class="callout">
**Thanks:** I would just like to thank a **vanila** on the **#cryptol**
freenode.net IRC channel for help with this one.
</div>


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

Bernstein's Hash
----------------

The Bernstein's hash is defined by the following recursion relation:

$$
H_{k+1} = 33 * H_k + m_k : 0 \leq k < n
$$

for some message **M** with length **n** bytes, with initial condition $H_0=$
some numerical salt.

We can then directly implement the Cryptol implementation as a recursive list
comprehessions:

~~~ {.haskell}
module Bernstein where

// initial salt value = [0]
bernHash : {n} (fin n) => [n][32] -> [32]
bernHash message = (hs!0)
  where hs = [0] # [33*h + m | h <- hs | m <- message]

// Test:
//bernHash [ zero # c | c <- "Wikipedia" ]
//2800982103
~~~

Suppose we would like to know if the hash has collisions, that is to say a
non-injective function. A injection is defined as the condition, if $f(x)=f(y)
\Rightarrow x=y$. Hence we express the condition as so:

~~~ {.haskell}
prop_injective x y = if bernHash x == bernHash y then x == y else True
~~~

Searching the entire state-space by randomised testing is fairly useless in
Cryptography, as seen:

~~~ {.bash}
Bernstein> :check (prop_injective : [9][32] -> [9][32] -> Bit)
Using random testing.
passed 100 tests.
Coverage: 0.00% (100 of 2^^576 values)
~~~

Here we searched $2^{576}$ possible values and found no problems, however our
coverage is $0.00\%$! Rather than trying to search for values that make a
function work we should search for values that do not as to obtain a
contradition, if any. We may achieve this goal by using the **SAT solver** as
follows:

~~~ {.bash}
Bernstein> :prove (prop_injective : [2][32] -> [2][32] -> Bit)
(prop_injective : [2][32] -> [2][32] -> Bit) [3020467164, 2155207698]
                                              [1914480668, 4293023698] = False
~~~

and we have found a counter example!

Modified Bernstein's Hash
-------------------------

The modified Bernstein's hash replaces addition with XOR for the combining
step. Hence the modified Bernstein's hash is defined by the following recursion
relation:

$$
H_{k+1} = 33 * H_k \oplus m_k : 0 \leq k < n
$$

for some message **M** with length **n** bytes, with initial condition $H_0=$
some numerical salt.

We can then directly imedately go to the Cryptol implementation:

~~~ {.haskell}
module Bernstein where

// initial salt value = [0]
bernHash : {n} (fin n) => [n][32] -> [32]
bernHash message = (hs!0)
  where hs = [0] # [33*h ^ m | h <- hs | m <- message]
~~~


Cryptographic Hash Functions
============================

Summary
-------

Name            Length        Type
----            ------        ----
SHA-3 (Keccak)  arbitrary     Sponge function
Skin            arbitrary     Unique Block Iteration
