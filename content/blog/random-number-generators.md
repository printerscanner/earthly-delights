---
title: Random Numbers
description: 
date: 2024-01-01
tags:
  - Engineering
---
At printer_scanner, I spend a lot of time playing with Random Number Generators. From an artistic standpoint, it adds a lot of interest to projects I'm working on because when you offer a choice to someone or something in the creation of an art object, I become more than an author in the creation of that object; I am also an observer. 

But what are Random Number Generators anyway? And is Math.Random() really as random as it seems?

Computers are linear. They are big, giant counters. However, an essential part of computing is randomness. We use randomness in everything, from art to gambling, statistical sampling, computer simulation, and cryptography. printer_scanner utilizes random number generators for much of its work, claiming to be the most random website on the internet. This claim is disputed by most. But how does a machine built to be linear create randomness? There are a few ways. We have to develop algorithms that mimic randomness. These are called Pseudorandom Number Generators, or PRNGs. These algorithms produce outputs that resemble uniformly distributed random values but are deterministic.

The earliest PRNG was developed by John von Neumann using the middle square method. This method takes a number, squares it, and removes the middle digits of the resulting number as a "random number." This random number is used as a seed for the next iteration. The problem is that all sequences eventually repeat themselves, some very quickly, such as "0000."

Today, two random number generation algorithms are most frequently used on computers. The first is xorshift128+, which most browsers use, and the second is variations on the Mersenne Twister, a more secure algorithm that most programming languages use. 

*xorshift128+* is the algorithm used by most browsers. When you run a random number generator in the browser, such as Math. random() in JavaScript, these languages elect the browser to choose an algorithm. We call this implementation-defined. *xorshift128+* is the preferred algorithm of browsers because it is incredibly lightweight and the fastest algorithm within its class. Let's look at the whole *xorshift128+* in detail:

```c
uint64_t seed_0 = 1;
uint64_t seed_1 = 2;

uint64_t xorshift128plus() {
  uint64_t s1 = seed_0;
  uint64_t s0 = seed_1;
  seed_0 = s0;
  s1 ^= s1 << 23;
  s1 ^= s1 >> 17;
  s1 ^= s0;
  s1 ^= s0 >> 26;
  seed_1 = s1;
  return seed_0 + seed_1;
}
```

*xorshift128+* starts by defining two seed values. It then manipulates them through arithmetic enough times that they become unrecognizable. To do this, bitwise operators are used.

Programming languages are written on top of each other to reduce complexity, so the programmer can understand only some things about how a computer works to create a program. Typically, we reduce the complexity of computer programs by using layers of abstraction. We represent bitwise operators in the code with the symbols *>>* and *<<.* However, these bitwise operators reach underneath the program and manipulate the code at the level of the bit.

The symbols *<<* and *>>* designate left-shifts and right-shifts. For example, if we had 8 << 3, the left shift operator shifts the first operand, 8, the specified number of bits to the left, in this case, 3. Excess bits shifted to the left are discarded, and zero bits shifted from the right. In line 7 of the code above, *xorshift128+* starts by left-shifting our seed value by 23 bits.

This is a very clever algorithmic approach. By performing shifts at the bit level, we can ensure that, in appearance, the new number does not relate to the one preceding it.

Then, we apply our second operator, the **exclusive operator**, *XOR*. We write this programmatically as *=^*. This means one or the other, but not both. It compares the binary representations of the two numbers and outputs a 0 where the corresponding bits are the same and one where they are different. In this case, we compare the binary representations of our *seed_1* value and our left-shifted seed one. The algorithm goes through several more cycles of left and right shifting and finally outputs our "random" number.

Two questions may arise from looking at this algorithm. First, why did they shift the bits by the numbers 23, 17, and 26? Researchers carefully selected these numbers because they create the largest **period** in this random number generator. A period is the number of times a program can run before it starts to repeat itself. *xorshift128+* has a period of length of 2^128 - 1 (hence the name).

The second question you may be asking is how a seed gets selected. *xorshift128+* is not cryptographically secure. For any Math. random() users reading this, use window.crypto.getRandomValues for cryptographically secure numbers. Seeds often act as a backdoor, preventing something from being truly random. It doesn't matter how many times you shake up a number. Anyone can reverse-engineer the value as long as a seed is deterministically selected. It would be like hurricane-proofing a house and then leaving the windows open. **If you know the generator’s internal state, you see the future.**

Brendan Eich developed JavaScript in 10 days in the early 1990s. Until 1992, the United States banned the export of cryptography. That law gradually relaxed in the 1990s. But when JavaScript was first developed, creating cryptographically secure algorithms to be released by JavaScript was viewed as equivalent to exporting munitions to enemies of the state.

The second algorithm standardly implemented by programming languages such as Python, Ruby, R, IDL, and PHP is the Mersenne Twister algorithm, developed in 1997 by Makoto Matsumoto and Takuji Nishimura. The Mersenne Twister uses a unique approach to generating seed values. The seeds are generated from real-time clock values that return to the millisecond. Real-time numbers are a popular and somewhat secure way to generate random numbers. This is because a millisecond is impossible to detect with the human eye.

Here is a sample of the Mersenne Twister written in C. The Mersenne Twister passes all of the Diehard tests. These are tests written to assure the quality of random number generators. Because of this, this algorithm is much more complex, uses more memory, and is slower than other algorithms.

```c
/* An implementation of the MT19937 Algorithm for the Mersenne Twister
 * by Evan Sultanik.  Based upon the pseudocode in M. Matsumoto and
 * T. Nishimura, "Mersenne Twister: A 623-dimensionally
 * equidistributed uniform pseudorandom number generator," ACM
 * Transactions on Modeling and Computer Simulation Vol. 8, No. 1,
 * January pp.3-30 1998.
 *
 * http://www.sultanik.com/Mersenne_twister
 */

#define UPPER_MASK		0x80000000
#define LOWER_MASK		0x7fffffff
#define TEMPERING_MASK_B	0x9d2c5680
#define TEMPERING_MASK_C	0xefc60000

#include "mtwister.h"

inline static void m_seedRand(MTRand* rand, unsigned long seed) {
  /* set initial seeds to mt[STATE_VECTOR_LENGTH] using the generator
   * from Line 25 of Table 1 in: Donald Knuth, "The Art of Computer
   * Programming," Vol. 2 (2nd Ed.) pp.102.
   */
  rand->mt[0] = seed & 0xffffffff;
  for(rand->index=1; rand->index<STATE_VECTOR_LENGTH; rand->index++) {
    rand->mt[rand->index] = (6069 * rand->mt[rand->index-1]) & 0xffffffff;
  }
}

/**
* Creates a new random number generator from a given seed.
*/
MTRand seedRand(unsigned long seed) {
  MTRand rand;
  m_seedRand(&rand, seed);
  return rand;
}

/**
 * Generates a pseudo-randomly generated long.
 */
unsigned long genRandLong(MTRand* rand) {

  unsigned long y;
  static unsigned long mag[2] = {0x0, 0x9908b0df}; /* mag[x] = x * 0x9908b0df for x = 0,1 */
  if(rand->index >= STATE_VECTOR_LENGTH || rand->index < 0) {
    /* generate STATE_VECTOR_LENGTH words at a time */
    int kk;
    if(rand->index >= STATE_VECTOR_LENGTH+1 || rand->index < 0) {
      m_seedRand(rand, 4357);
    }
    for(kk=0; kk<STATE_VECTOR_LENGTH-STATE_VECTOR_M; kk++) {
      y = (rand->mt[kk] & UPPER_MASK) | (rand->mt[kk+1] & LOWER_MASK);
      rand->mt[kk] = rand->mt[kk+STATE_VECTOR_M] ^ (y >> 1) ^ mag[y & 0x1];
    }
    for(; kk<STATE_VECTOR_LENGTH-1; kk++) {
      y = (rand->mt[kk] & UPPER_MASK) | (rand->mt[kk+1] & LOWER_MASK);
      rand->mt[kk] = rand->mt[kk+(STATE_VECTOR_M-STATE_VECTOR_LENGTH)] ^ (y >> 1) ^ mag[y & 0x1];
    }
    y = (rand->mt[STATE_VECTOR_LENGTH-1] & UPPER_MASK) | (rand->mt[0] & LOWER_MASK);
    rand->mt[STATE_VECTOR_LENGTH-1] = rand->mt[STATE_VECTOR_M-1] ^ (y >> 1) ^ mag[y & 0x1];
    rand->index = 0;
  }
  y = rand->mt[rand->index++];
  y ^= (y >> 11);
  y ^= (y << 7) & TEMPERING_MASK_B;
  y ^= (y << 15) & TEMPERING_MASK_C;
  y ^= (y >> 18);
  return y;
}

/**
 * Generates a pseudo-randomly generated double in the range [0..1].
 */
double genRand(MTRand* rand) {
  return((double)genRandLong(rand) / (unsigned long)0xffffffff);
}
```

The Mersenne Twister's main advantage is an enormous period length. 2^19937−1. 2^19937 − 1, for reference, is way more than the number of atoms in the observable universe. While a long period is not guaranteed quality in a random number generator, short periods can be problematic. An attacker can observe, record, and reuse sequences within too short a period. Sequences with long periods force the adversary to select alternate attack methods.

Yet, all could be better in terms of non-predictability. The Twister's algorithm keeps track of its state in 624 32-bit values. If an attacker could gather 624 sequential values, the entire forward and backward sequence could be reverse-engineered. This feature is not specific to the Mersenne Twister. Most PRGs have a state machine used to generate the next value in the sequence. Knowledge of the state effectively.

This algorithm provides enough randomness for specific tasks like video games but is unsuitable for requiring high-quality randomness, such as cryptography applications, statistics, or numerical analysis.

There are a few reasons for this. First, the seed needs to be more random. Though millisecond clock values are random enough to the human eye to be considered entirely random, they are still too deterministic for reverse engineering purposes.

If we want to create a secure random algorithm for banking, we need unpredictability with our seed values. It is impossible to achieve unpredictability with seed values when generating them arithmetically. We must use Hardware random number generators (HRNGs) to create a truly random number generator. These generators take raw input data from high-entropy real-world situations such as thermal noise, white electrical noise, radioactive particle decay, and avalanche noise in semiconducting materials. Tracking mouse movements can also be considered random enough.

If you ever find yourself building a random number generator to secure data, don't forget the words of the great John von Neumann, “Anyone who considers arithmetical methods of producing random digits is, of course, in a state of sin.”