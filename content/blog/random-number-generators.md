---
title: Random Numbers
description: 
date: 2024-01-01
tags:
  - Engineering
---
At printer_scanner, I spend a lot of time playing around with Random Number Generators. From an artistic standpoint it adds a lot of interest to projects I'm working on because when you offer to choice to someone (or something else) in the creation of an art object, I become more than a author in that objects creation, but also an observer. 

But what are Random Number Generators anyways! How does Math.Random() actually work, and are "they" just lying to me when they say it's random?

Computers are linear. They are big, giant counters. But an essential part of computing is randomness. We use randomness in everything, from art to gambling, statistical sampling, computer simulation, to cryptography. printer_scanner utilizes random number generators for much of its work, claiming to be the most random website on the internet. This claim is disputed by most. But, how does a machine built to be linear create randomness? There are a few ways. We have to create algorithms that mimic randomness. These are called Pseudorandom Number Generators, or PRNG’s. These algorithms produce outputs that resemble uniformly distributed random values but are deterministic.

Early approaches to a PRNG suggested by John von Neumann, middle square method: take a number, square it, remove the middle digits of the resulting number as a "random number," and use it as a seed for the next iteration. The problem is that all sequences eventually repeat themselves, some very quickly, such as "0000."

This essay will break down the two random number generation algorithms most frequently in use on our computers. The first is xorshift128+, used by most browsers, and the second is variations on the Mersenne Twister, used by most programming languages. 

*xorshift128+* is the algorithm used by most browsers. When you run a random number generator in the browser, such as Math.random() in JavaScript, these languages elect to have the browser choose the algorithm. We call this implementation-defined. *xorshift128+* is the preferred algorithm of the browser because it is incredibly lightweight and the fastest algorithm within their class. Let's look at the whole *xorshift128+* in detail:

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

The algorithm starts by defining two seed values. The algorithm aims to take two numbers and manipulate them enough times through arithmetic that they become unrecognizable. To do this, we use bitwise operators.

Programming languages are written on top of each other to reduce complexity, so the programmer does not have to understand everything about how a computer works to create a program. Typically, we reduce the complexity of computer programs by using layers of abstraction. We represent bitwise operators in the code with the symbols *>>* and *<<.* However, these bitwise operators reach underneath the program and manipulate the code at the level of the bit, or the 1's and 0's.

The symbols *<<* and *>>* designate left-shifts and right-shifts. For example, if we had 8 << 3, the left shift operator shifts the first operand, 8, the designated number of bits to the left, in this case, 3. Excess bits shifted off to the left are discarded, and zero bits shifted from the right. This algorithm starts by left-shifting our seed value by 23 bits.

This is a very clever algorithmic approach. By performing shifts at the bit level, we can ensure that, in appearance, the new number does not relate to the one preceding it.

Then, we apply our second operator, the **exclusive or** operator, *XOR*. We write this programmatically as =^. This means one or the other, but not both. It compares the binary representations of the two numbers and outputs a 0 where the corresponding bits are the same and a one where they are different. In this case, we compare the binary representations of our seed_1 value and our left-shifted seed one. The algorithm goes through several more cycles of left and right-shifting and finally outputs our "random" number.

Two questions may arise from looking at this algorithm. First, why did they shift the bits by the numbers 23, 17, and 26? Researchers carefully selected these numbers because they create the largest **period** in this random number generator. A period is the number of times a program can run before it starts to repeat itself. *xorshift128+* has a period of length of 2^128 - 1 (hence the name).

The second question you may be asking is how a seed gets selected. *xorshift128+* is not cryptographically secure. (For any Math.random() users reading this, use window.crypto.getRandomValues for cryptographically secure numbers). Seeds often act as a backdoor preventing something from being truly random. It doesn't matter how many times you shake up a number. Anyone can reverse-engineer the value as long as a seed is deterministically selected. It would be like a hurricane proofing a house and then leaving the windows open. **If you know the generator’s internal state, you see the future.**

Brendan Eich developed JavaScript in 10 days in the early nineties. Until 1992, there was a ban on the export of cryptography in the United States. That law gradually became relaxed until 1992. But when JavaScript was first developed, creating cryptographically secure algorithms to be released by JavaScript would be the equivalent of exporting munitions to enemies of the state.

The second algorithm standardly implemented by programming languages such as Python, Ruby, R, IDL, and PHP is a Mersenne Twister algorithm, developed in 1997 by Makoto Matsumoto and Takuji Nishimura. The Mersenne Twister uses a unique approach to generating seed values. They get real-time clock values that return to the millisecond. The use of real-time numbers is a popular and somewhat secure way to generate random numbers. This is because a millisecond is impossibly small to detect with the human eye.

Here is a sample of the Mersenne Twister written in C. The Mersenne Twister passes all of the Diehard tests. These are tests written to assure the quality of random number generators. Because of this, this algorithm is much more complex, uses more memory and is slower than other algorithms.

```c
/* An implementation of the MT19937 Algorithm for the Mersenne Twister
 * by Evan Sultanik.  Based upon the pseudocode in: M. Matsumoto and
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

The Mersenne Twister's main advantage is an enormous period length. 2^19937−1. 2^19937 − 1, for reference, is way more than the number of atoms in the observable universe. While a long period is not guaranteed quality in a random number generator, short periods can be problematic. Sequences with too short a period can be observed, recorded, and reused by an attacker. Sequences with long periods force the adversary to select alternate attack methods.

Yet, all is not perfect in terms of non-predictability. The Twister's algorithm keeps track of its state in 624 32-bit values. If an attacker could gather 624 sequential values, then the entire sequence—forward and backward—could be reverse-engineered. This feature is not specific to the Mersenne Twister. Most PRNG’s have a state machine used to generate the next value in the sequence. Knowledge of the state effectively.

This algorithm provides enough randomness for specific tasks like video games but is unsuitable for requiring high-quality randomnesses, such as cryptography applications, statistics, or numerical analysis.

There are a few reasons for this. First, the seed isn't random enough. Though taking millisecond clock values are random enough to the human eye to be considered entirely random, they are still too deterministic for reverse engineering purposes.

If we wanted to create a secure random algorithm for use such as banking, we would need a level of unpredictability with our seed values. The great John von Neumann said, “Anyone who considers arithmetical methods of producing random digits is, of course, in a state of sin.” When producing seed values, unpredictably is impossible to achieve arithmetically.

If we wanted to create a truly random number generator, we would need to use Hardware random number generators or HRNG's. They take raw input data from the actual high entropy real-world situations such as thermal noise, white electrical noise, the decay of a radioactive particle, avalanche noise in semiconducting materials. Tracking mouse movements can also be considered random enough.

When we use many cryptographic algorithms, your computer stores random values generated through hardware input (such as mouse movements) and store the values for use as seed values.