# How random is Math.random()

Computers are linear. They are big, giant counters. But an essential part of computing is randomness. We use randomness in everything, from art to gambling, statistical sampling, computer simulation, to cryptography. printer_scanner utilizes random number generators for much of its work, claiming to be the most random website on the internet. This claim is disputed by most. But, how does a machine built to be linear create randomness? There are a few ways. We have to create algorithms that mimic randomness. These are called Pseudorandom Number Generators, or PRNG’s. These algorithms produce outputs that resemble uniformly distributed random values but are deterministic.

Early approaches to a PRNG suggested by John von Neumann, middle square method: take a number, square it, remove the middle digits of the resulting number as a "random number," and use it as a seed for the next iteration. The problem is that all sequences eventually repeat themselves, some very quickly, such as "0000."

This essay will break down the two random number generation algorithms most frequently in use on our computers. The first is xorshift128+, used by most browsers, and the second is variations on the Mersenne Twister, used by most programming languages. We will break down how these two algorithms work.

xorshift128+ is the algorithm used by most browsers. When you run a random number generator in the browser, such as Math.random() in JavaScript, these languages elect to have the browser choose the algorithm. We call this implementation-defined. xorshift128+ is the preferred algorithm of the browser because it is incredibly lightweight and the fastest algorithm within their class. Let's look at xorshift128+ in detail:

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

Programming languages are written on top of each other to reduce complexity, so the programmer does not have to understand everything about how a computer works to create a program. Typically, we reduce the complexity of computer programs by using layers of abstraction. We represent bitwise operators in the code with the symbols >> and <<. However, these bitwise operators reach underneath the program and manipulate the code at the level of the bit, or the 1's and 0's.

The symbols << and >> designate left-shifts and right-shifts. For example, if we had 8 << 3, the left shift operator shifts the first operand, 8, the designated number of bits to the left, in this case, 3. Excess bits shifted off to the left are discarded, and zero bits shifted from the right. This algorithm starts by left-shifting our seed value by 23 bits.

This is a very clever algorithmic approach. By performing shifts at the bit level, we can ensure that, in appearance, the new number does not relate to the one preceding it.

Then, we apply our second operator, the **exclusive or** operator, XOR. We write this programmatically as =^. This means one or the other, but not both. It compares the binary representations of the two numbers and outputs a 0 where the corresponding bits are the same and a one where they are different. In this case, we compare the binary representations of our seed_1 value and our left-shifted seed one. The algorithm goes through several more cycles of left and right-shifting and finally outputs our "random" number.

Two questions may arise from looking at this algorithm. First, why did they shift the bits by the numbers 23, 17, and 26? Researchers carefully selected these numbers because they create the largest **period** in this random number generator. A period is the number of times a program can run before it starts to repeat itself. xorshift128+ has a period of length of 2^128 - 1 (hence the name).

The second question you may be asking is how a seed gets selected. xorshift128+ is not cryptographically secure. (For any Math.random() users reading this, use window.crypto.getRandomValues for cryptographically secure numbers). Seeds often act as a backdoor preventing something from being truly random. It doesn't matter how many times you shake up a number. Anyone can reverse-engineer the value as long as a seed is deterministically selected. It would be like a hurricane proofing a house and then leaving the windows open. **If you know the generator’s internal state, you see the future.**

Brendan Eich developed JavaScript in 10 days in the early nineties. Until 1992, there was a ban on the export of cryptography in the United States. That law gradually became relaxed until 1992. By developing chronologically, secure algorithms to be released by JavaScript would be the equivalent of exporting munitions to enemies of the state.

The second algorithm standarly implemented by programming languages such as Python, Ruby, R, IDL, and PHP random number generators use a Mersenne Twister algorithm, developed in 1997 by Makoto Matsumoto and Takuji Nishimura. The Mersenne Twister uses real-time clock values that return to the millisecond as a seed. The use of real-time numbers is a popular and somewhat secure way to generate random numbers. This is because a millisecond is impossibly small to detect with the human eye.

Here is a sample of the Mersenne Twister written in JavaScript. The Mersenne Twister passes all of the Diehard tests, tests written to assure the quality of random number generators. Because of this, this algorithm is much more complex, uses more memory and is slower than other algorithms.

```jsx
var MersenneTwister = function(seed) {
  if (seed == undefined) {
    seed = new Date().getTime();
  } 
  /* Period parameters */  
  this.N = 624;
  this.M = 397;
  this.MATRIX_A = 0x9908b0df;   /* constant vector a */
  this.UPPER_MASK = 0x80000000; /* most significant w-r bits */
  this.LOWER_MASK = 0x7fffffff; /* least significant r bits */
 
  this.mt = new Array(this.N); /* the array for the state vector */
  this.mti=this.N+1; /* mti==N+1 means mt[N] is not initialized */

  this.init_genrand(seed);
}  
 
/* initializes mt[N] with a seed */
MersenneTwister.prototype.init_genrand = function(s) {
  this.mt[0] = s >>> 0;
  for (this.mti=1; this.mti<this.N; this.mti++) {
      var s = this.mt[this.mti-1] ^ (this.mt[this.mti-1] >>> 30);
   this.mt[this.mti] = (((((s & 0xffff0000) >>> 16) * 1812433253) << 16) + (s & 0x0000ffff) * 1812433253)
  + this.mti;
      /* See Knuth TAOCP Vol2. 3rd Ed. P.106 for multiplier. */
      /* In the previous versions, MSBs of the seed affect   */
      /* only MSBs of the array mt[].                        */
      /* 2002/01/09 modified by Makoto Matsumoto             */
      this.mt[this.mti] >>>= 0;
      /* for >32 bit machines */
  }
}
 
/* initialize by an array with array-length */
/* init_key is the array for initializing keys */
/* key_length is its length */
/* slight change for C++, 2004/2/26 */
MersenneTwister.prototype.init_by_array = function(init_key, key_length) {
  var i, j, k;
  this.init_genrand(19650218);
  i=1; j=0;
  k = (this.N>key_length ? this.N : key_length);
  for (; k; k--) {
    var s = this.mt[i-1] ^ (this.mt[i-1] >>> 30)
    this.mt[i] = (this.mt[i] ^ (((((s & 0xffff0000) >>> 16) * 1664525) << 16) + ((s & 0x0000ffff) * 1664525)))
      + init_key[j] + j; /* non linear */
    this.mt[i] >>>= 0; /* for WORDSIZE > 32 machines */
    i++; j++;
    if (i>=this.N) { this.mt[0] = this.mt[this.N-1]; i=1; }
    if (j>=key_length) j=0;
  }
  for (k=this.N-1; k; k--) {
    var s = this.mt[i-1] ^ (this.mt[i-1] >>> 30);
    this.mt[i] = (this.mt[i] ^ (((((s & 0xffff0000) >>> 16) * 1566083941) << 16) + (s & 0x0000ffff) * 1566083941))
      - i; /* non linear */
    this.mt[i] >>>= 0; /* for WORDSIZE > 32 machines */
    i++;
    if (i>=this.N) { this.mt[0] = this.mt[this.N-1]; i=1; }
  }

  this.mt[0] = 0x80000000; /* MSB is 1; assuring non-zero initial array */ 
}
 
/* 
   Sean McCullough (banksean@gmail.com)
	 A C-program for MT19937, with initialization improved 2002/1/26.
   Coded by Takuji Nishimura and Makoto Matsumoto.
 
   Copyright (C) 1997 - 2002, Makoto Matsumoto and Takuji Nishimura,
   All rights reserved.
*/
```

The Mersenne Twister's main advantage is an enormous period length. 2^19937−1. 2^19937 − 1, for reference, is way more significant than the number of atoms in the observable universe. While a long period is not guaranteed quality in a random number generator, short periods, such as the 232 standards in many older software packages, can be problematic. Sequences with too short a period can be observed, recorded, and reused by an attacker. Sequences with long periods force the adversary to select alternate attack methods.

Yet all is not perfect in terms of non-predictability. The MT19937 algorithm keeps track of its state in 624 32-bit values. If an attacker could gather 624 sequential values, then the entire sequence—forward and backward—could be reverse-engineered. This feature is not specific to the Mersenne Twister. Most PRNG’s have a state machine used to generate the next value in the sequence. Knowledge of the state effectively.

This algorithm provides enough randomness for specific tasks like video games but is unsuitable for requiring high-quality randomnesses, such as cryptography applications, statistics, or numerical analysis.

There are a few reasons for this. First, the seed isn't random enough. Though taking millisecond clock values are random enough to the human eye to be considered entirely random, they are still too deterministic for reverse engineering purposes.

If we wanted to create a secure random algorithm for use such as banking, we would need a level of unpredictability with our seed values. The great John von Neumann said, “Anyone who considers arithmetical methods of producing random digits is, of course, in a state of sin.” When producing seed values, unpredictably is impossible to achieve arithmetically.

If we wanted to create a truly random number generator, we would need to use Hardware random number generators or HRNG's. They take raw input data from the actual high entropy real-world situations such as thermal noise, white electrical noise, the decay of a radioactive particle, avalanche noise in semiconducting materials. Tracking mouse movements can also be considered random enough.

When we use many cryptographic algorithms, your computer stores random values generated through hardware input (such as mouse movements) and store the values for use as seed values.