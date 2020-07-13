# Bit-Twiddling-Hacks
By Sean Eron Anderson
seander@cs.stanford.edu

Individually, the __code snippets here are in the public domain__ (unless otherwise noted) — feel free to use them however you please. The aggregate collection and descriptions are © 1997-2005 Sean Eron Anderson. The code and descriptions are distributed in the hope that they will be useful, but __WITHOUT ANY WARRANTY__ and without even the implied warranty of merchantability or fitness for a particular purpose. As of May 5, 2005, all the code has been tested thoroughly. Thousands of people have read it. Moreover, [Professor Randal Bryant](http://www-2.cs.cmu.edu/~bryant/), the Dean of Computer Science at Carnegie Mellon University, has personally tested almost everything with his [Uclid code verification system](http://www-2.cs.cmu.edu/~uclid/). What he hasn't tested, I have checked against all possible inputs on a 32-bit machine. __To the first person to inform me of a legitimate bug in the code, I'll pay a bounty of US$10 (by check or Paypal)__. If directed to a charity, I'll pay US$20.

## Contents

- About the operation counting methodology
- Compute the sign of an integer
- Detect if two integers have opposite signs
- Compute the integer absolute value (abs) without branching
- Compute the minimum (min) or maximum (max) of two integers without branching
- Determining if an integer is a power of 2
- Sign extending
    - Sign extending from a constant bit-width
    - Sign extending from a variable bit-width
    - Sign extending from a variable bit-width in 3 operations
- Conditionally set or clear bits without branching
- Conditionally negate a value without branching
- Merge bits from two values according to a mask
- Counting bits set
    - Counting bits set, naive way
    - Counting bits set by lookup table
    - Counting bits set, Brian Kernighan's way
    - Counting bits set in 14, 24, or 32-bit words using 64-bit instructions
    - Counting bits set, in parallel
    - Count bits set (rank) from the most-significant bit upto a given position
    - Select the bit position (from the most-significant bit) with the given count (rank)
- Computing parity (1 if an odd number of bits set, 0 otherwise)
    - Compute parity of a word the naive way
    - Compute parity by lookup table
    - Compute parity of a byte using 64-bit multiply and modulus division
    - Compute parity of word with a multiply
    - Compute parity in parallel
- Swapping Values
    - Swapping values with subtraction and addition
    - Swapping values with XOR
    - Swapping individual bits with XOR
- Reversing bit sequences
    - Reverse bits the obvious way
    - Reverse bits in word by lookup table
    - Reverse the bits in a byte with 3 operations (64-bit multiply and modulus division)
    - Reverse the bits in a byte with 4 operations (64-bit multiply, no division)
    - Reverse the bits in a byte with 7 operations (no 64-bit, only 32)
    - Reverse an N-bit quantity in parallel with 5 * lg(N) operations
- Modulus division (aka computing remainders)
    - Computing modulus division by 1 << s without a division operation (obvious)
    - Computing modulus division by (1 << s) - 1 without a division operation
    - Computing modulus division by (1 << s) - 1 in parallel without a division operation
- Finding integer log base 2 of an integer (aka the position of the highest bit set)
    - Find the log base 2 of an integer with the MSB N set in O(N) operations (the obvious way)
    - Find the integer log base 2 of an integer with an 64-bit IEEE float
    - Find the log base 2 of an integer with a lookup table
    - Find the log base 2 of an N-bit integer in O(lg(N)) operations
    - Find the log base 2 of an N-bit integer in O(lg(N)) operations with multiply and lookup
- Find integer log base 10 of an integer
- Find integer log base 10 of an integer the obvious way
- Find integer log base 2 of a 32-bit IEEE float
- Find integer log base 2 of the pow(2, r)-root of a 32-bit IEEE float (for unsigned integer r)
- Counting consecutive trailing zero bits (or finding bit indices)
    - Count the consecutive zero bits (trailing) on the right linearly
    - Count the consecutive zero bits (trailing) on the right in parallel
    - Count the consecutive zero bits (trailing) on the right by binary search
    - Count the consecutive zero bits (trailing) on the right by casting to a float
    - Count the consecutive zero bits (trailing) on the right with modulus division and lookup
    - Count the consecutive zero bits (trailing) on the right with multiply and lookup
- Round up to the next highest power of 2 by float casting
- Round up to the next highest power of 2
- Interleaving bits (aka computing Morton Numbers)
    - Interleave bits the obvious way
    - Interleave bits by table lookup
    - Interleave bits with 64-bit multiply
    - Interleave bits by Binary Magic Numbers
- Testing for ranges of bytes in a word (and counting occurances found)
    - Determine if a word has a zero byte
    - Determine if a word has a byte equal to n
    - Determine if a word has byte less than n
    - Determine if a word has a byte greater than n
    - Determine if a word has a byte between m and n
- Compute the lexicographically next bit permutation

## About the operation counting methodology

When totaling the number of operations for algorithms here, any C operator is counted as one operation. Intermediate assignments, which need not be written to RAM, are not counted. Of course, this operation counting approach only serves as an approximation of the actual number of machine instructions and CPU time. All operations are assumed to take the same amount of time, which is not true in reality, but CPUs have been heading increasingly in this direction over time. There are many nuances that determine how fast a system will run a given sample of code, such as cache sizes, memory bandwidths, instruction sets, etc. In the end, benchmarking is the best way to determine whether one method is really faster than another, so consider the techniques below as possibilities to test on your target architecture.

## Compute the sign of an integer

```c
int v;      // we want to find the sign of v
int sign;   // the result goes here 

// CHAR_BIT is the number of bits per byte (normally 8).
sign = -(v < 0);  // if v < 0 then -1, else 0. 
// or, to avoid branching on CPUs with flag registers (IA32):
sign = -(int)((unsigned int)((int)v) >> (sizeof(int) * CHAR_BIT - 1));
// or, for one less instruction (but not portable):
sign = v >> (sizeof(int) * CHAR_BIT - 1); 
```

The last expression above evaluates to sign = v >> 31 for 32-bit integers. This is one operation faster than the obvious way, sign = -(v < 0). This trick works because when signed integers are shifted right, the value of the far left bit is copied to the other bits. The far left bit is 1 when the value is negative and 0 otherwise; all 1 bits gives -1. Unfortunately, this behavior is architecture-specific.

Alternatively, if you prefer the result be either -1 or +1, then use:

```c
sign = +1 | (v >> (sizeof(int) * CHAR_BIT - 1));  // if v < 0 then -1, else +1
```

On the other hand, if you prefer the result be either -1, 0, or +1, then use:

```c
sign = (v != 0) | -(int)((unsigned int)((int)v) >> (sizeof(int) * CHAR_BIT - 1));
// Or, for more speed but less portability:
sign = (v != 0) | (v >> (sizeof(int) * CHAR_BIT - 1));  // -1, 0, or +1
// Or, for portability, brevity, and (perhaps) speed:
sign = (v > 0) - (v < 0); // -1, 0, or +1
```

If instead you want to know if something is non-negative, resulting in +1 or else 0, then use:

```c
sign = 1 ^ ((unsigned int)v >> (sizeof(int) * CHAR_BIT - 1)); // if v < 0 then 0, else 1
```

Caveat: On March 7, 2003, Angus Duggan pointed out that the 1989 ANSI C specification leaves the result of signed right-shift implementation-defined, so on some systems this hack might not work. For greater portability, Toby Speight suggested on September 28, 2005 that CHAR_BIT be used here and throughout rather than assuming bytes were 8 bits long. Angus recommended the more portable versions above, involving casting on March 4, 2006. [Rohit Garg](http://rpg-314.blogspot.com/) suggested the version for non-negative integers on September 12, 2009.

## Detect if two integers have opposite signs

```c
int x, y;               // input values to compare signs

bool f = ((x ^ y) < 0); // true iff x and y have opposite signs
```
Manfred Weis suggested I add this entry on November 26, 2009.

## Compute the integer absolute value (abs) without branching

```c
int v;           // we want to find the absolute value of v
unsigned int r;  // the result goes here 
int const mask = v >> sizeof(int) * CHAR_BIT - 1;

r = (v + mask) ^ mask;
```

Patented variation:

```c
r = (v ^ mask) - mask;
```

Some CPUs don't have an integer absolute value instruction (or the compiler fails to use them). On machines where branching is expensive, the above expression can be faster than the obvious approach, r = (v < 0) ? -(unsigned)v : v, even though the number of operations is the same.

On March 7, 2003, Angus Duggan pointed out that the 1989 ANSI C specification leaves the result of signed right-shift implementation-defined, so on some systems this hack might not work. I've read that ANSI C does not require values to be represented as two's complement, so it may not work for that reason as well (on a diminishingly small number of old machines that still use one's complement). On March 14, 2004, Keith H. Duggar sent me the patented variation above; it is superior to the one I initially came up with, `r=(+1|(v>>(sizeof(int)*CHAR_BIT-1)))*v`, because a multiply is not used. Unfortunately, this method has been [patented](http://patft.uspto.gov/netacgi/nph-Parser?Sect1=PTO2&Sect2=HITOFF&p=1&u=/netahtml/search-adv.htm&r=1&f=G&l=50&d=ptxt&S1=6073150&OS=6073150&RS=6073150) in the USA on June 6, 2000 by Vladimir Yu Volkonsky and assigned to [Sun Microsystems](http://www.sun.com/). On August 13, 2006, Yuriy Kaminskiy told me that the patent is likely invalid because the method was published well before the patent was even filed, such as in [How to Optimize for the Pentium Processor](http://www.goof.com/pcg/doc/pentopt.txt) by Agner Fog, dated November, 9, 1996. Yuriy also mentioned that this document was translated to Russian in 1997, which Vladimir could have read. Moreover, the Internet Archive also has an old [link](http://web.archive.org/web/19961201174141/www.x86.org/ftp/articles/pentopt/PENTOPT.TXT) to it. On January 30, 2007, Peter Kankowski shared with me an [abs version](http://smallcode.weblogs.us/2007/01/31/microsoft-probably-uses-the-abs-function-patented-by-sun/) he discovered that was inspired by Microsoft's Visual C++ compiler output. It is featured here as the primary solution. On December 6, 2007, Hai Jin complained that the result was signed, so when computing the abs of the most negative value, it was still negative. On April 15, 2008 Andrew Shapira pointed out that the obvious approach could overflow, as it lacked an (unsigned) cast then; for maximum portability he suggested `(v < 0) ? (1 + ((unsigned)(-1-v))) : (unsigned)v`. But citing the ISO C99 spec on July 9, 2008, Vincent Lefèvre convinced me to remove it becasue even on non-2s-complement machines `-(unsigned)v` will do the right thing. The evaluation of `-(unsigned)v` first converts the negative value of `v` to an unsigned by adding `2**N`, yielding a 2s complement representation of `v`'s value that I'll call `U`. Then, `U` is negated, giving the desired result, `-U = 0 - U = 2**N - U = 2**N - (v+2**N) = -v = abs(v)`.
